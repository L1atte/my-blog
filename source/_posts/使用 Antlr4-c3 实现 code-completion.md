---
title: 使用 Antlr4-c3 实现 code-completion
date: 2023/3/27
updated: 2023/3/27
tags: 
- IDE
- Antlr4
- code-completion
- antlr4-c3
categories: IDE
description: "一个好的 IDE 不可缺少的就是 code-completion 功能，下面我将会使用  `antlr4-c3` 来实现这一功能。"
---

# 使用 Antlr4-c3 实现 code-completion

## 简介

code-completion 不是一个简单的主题，我将会根据 Kotlin 的语法，编写一个简单的 demo 来展示 antlr4-c3 的基本运行情况。

然后，我们将会通过一下方式逐步完善我们的例子

- 确定光标的位置
- 根据标识符的上下文提供 suggestions
- 完成部分匹配
- 集成模糊搜索
- 提高准确性
- 改善性能

## 前置准备

我们需要安装一个 `antlr4-c3` 和 `antlr4ts`，像下面这样

```shell
npm i antlr4-c3
npm i antlr4ts
npm i -D antlr4ts-cli
```

其中 `antlr4-c3` 是 code-completion 的核心部分，而`antlr4ts`可以为`antlr4`输出 TypeScript 类型的语法文件，`antlr4ts-cli`是配套的命令行工具

## antlr4-c3 工作原理

Antlr4-c3 的工作原理是计算`parse tree`中某一点可用的`token`的集合

例如，让我们考虑下面的语句：

```kotlin
fun test() {
    try {
        doSomething()
    } // flag
}
```

如果我们把光标放在第一个闭合的大括号之后（flag 的位置），我们可以期待引擎建议使用`catch`和`finally`标记。因为根据语法，这些标记可以放在这个位置，以形成有效的程序

为了计算候选标记（candidate tokens）的集合，`antlr4-c3`使用了和`ANTLR4`生成的解析器相同的数据结构和算法来完成工作。实际上，`antlr4-c3`模拟了解析器的状态及。因为，antlr4-c3 的行为会和语言的语法保持一致

我们应该注意到，`antlr4-c3`是以标记（tokens）的形式工作的，我们**并不会得到可以随时展示给用户的字符串**。把这些标记翻译成有意义的字符串是接下来我们要做的事情

## Suggesting Keywords

让我们来看看一个基础的语法解析过程，我们将会看到如何进行语法分析、初始化`antlr4-c3`和向其索要 tokens

第一步，创建解析器并解析所需的代码

```js
let input = CharStreams.fromString(code);
let lexer = new KotlinLexer(input);
let parser = new KotlinParser(new CommonTokenStream(lexer));
parser.kotlinFile();
```

需要注意的是，创建解析器（parser）并不需要成功（比如存在不合法的代码）。因为我们可以处理错误的代码——`ANTLR4`可以从错误的代码中恢复，如果我们使用默认的错误处理策略，它将在每一种情况下建立一个解析树

从错误输入构建的 parse tree 将会包含错误节点，但仍然可以为我们的目的使用它--尽管我们可能不得不考虑到一些节点缺少我们通常期望的信息这一事实。

无论如何，引擎本身并不使用解析树来计算其建议。原则上，运行解析器只是为了填充标记流。然而，我们实际上会利用解析树来对引擎进行微调。

一旦我们填充了标记流，我们就可以要求引擎提供一个完成候选列表：

```js
let index = /* we’ll show how to compute this later */;
let core = new CodeCompletionCore(parser);
let candidates = core.collectCandidates(index);
```

## 确定光标的位置

我们传递给 `CodeCompletionCore` 的数字，在前面的例子称为 `index`，代表我们提供给解析器的数据流中一个 token 的位置。近似的说，我们可以说它指向光标下的标记。

我们将使用几个坐标来表示光标在文档中的位置：行和列

考虑到 ANTLR 解析器的工作方式，使用一个单一的数字，即字符在输入流中的位置会更简单

然而，使用行和列可以更容易地与语言服务器（LSP：language server protocol）和一般的编辑器整合

所以，我们创建一个函数，比如：

```js
export type CaretPosition = { line: number, column: number };
export function computeTokenIndex(parseTree: ParseTree, caretPosition: CaretPosition): number {
	if (parseTree instanceof TerminalNode) {
		return computeTokenIndexOfTerminalNode(parseTree, caretPosition);
	} else {
		return computeTokenIndexOfChildNode(parseTree, caretPosition);
	}
}
```

两个函数将执行实际工作，一个用于 terminal nodes，一个用于 intermediate nodes。

```js
function computeTokenIndexOfTerminalNode(parseTree: TerminalNode, caretPosition: CaretPosition) {
	let start = parseTree.symbol.charPositionInLine;
	let stop = parseTree.symbol.charPositionInLine + parseTree.text.length;
	if (parseTree.symbol.line == caretPosition.line && start <= caretPosition.column && stop >= caretPosition.column) {
		return parseTree.symbol.tokenIndex;
	} else {
		return undefined;
	}
}

function computeTokenIndexOfChildNode(parseTree: ParseTree, caretPosition: CaretPosition) {
	for (let i = 0; i < parseTree.childCount; i++) {
		let index = computeTokenIndex(parseTree.getChild(i), caretPosition);
		if (index !== undefined) {
			return index;
		}
	}
	return undefined;
}
```

需要注意的是，在上面的代码中。假设我们语言中的标记——至少是我们可能想要完成的 token 不会跨越多行（只有一行）

这是合理的，因为在典型的语言中，可以跨越多行的标记只包括注释和可能的字符串，而我们并不期望补全在注释和字符串中工作。

如果我们遇到任何不寻常的语言，标识符可以跨越多行，我们就必须修改上述代码来处理这些标记。

## 计算代码建议（code suggestions）

现在我们知道如何计算一个有意义的 token 索引，让我们回到最初的例子，其中包括这一行：

```js
let candidates = core.collectCandidates(index);
```

`collectCandidates`函数返回以`CandidatesCollection`为元素的数组（即 `Array<CandidatesCollection>`），是给定位置（ index ）的候选建议列表。

其中，`CandidatesCollection`的定义为：

```typescript
type TokenList = number[];
type RuleList = number[];
type CandidateRule = {
	startTokenIndex: number;
	ruleList: RuleList;
};

class CandidatesCollection {
	tokens: Map<number, TokenList>;
	rules: Map<number, CandidateRule>;
}
```

现在让我们关注 `tokens`，这是一个 `map` 对象，每一个 key 都对应一个 TokenList（定义在解析器的词汇表里）。我们将其提取出来：

```js
let tokenName = parser.vocabulary.getSymbolicName(tokenType).toLowerCase();
```

大多数情况下，我们可以直接提供该字符串作为建议：

```js
let completions = [];
candidates.tokens.forEach((_, k) => completions.push(parser.vocabulary.getSymbolicName(k).toLowerCase()));
return completions;
```

然而，这只适用于规则名称与解析后的文本相匹配的 token，常见于关键字的匹配。比如

```g4
PACKAGE: 'package' ;
```

这种情况下，对于 token 类型 PACKAGE，我们会建议使用字符串 'package' 。然而，这只是一种特殊情况，并不是所有的关键字都遵循这种模式。我们可以在必要时处理规则的少数例外情况。例如：Kotlin 词典定义了以下规则

```
NOT_IN: '!in' (WS | NL)+;
```

我们可以像这样处理

```js
if (k == KotlinParser.NOT_IN) {
	completions.push("!in");
} else {
	completions.push(parser.vocabulary.getSymbolicName(k).toLowerCase());
}
```

可以发现，如何编写语法文件对 code-completion 有影响。在后面这点更加明显

## 过滤不需要的 token

在任何情况下，只要 antlr4-c3 建议的 token 不是关键字，例如 LPAREN（左大括号）或 Identifier （标识符，可以理解为变量名），那么我们的方法就会失效

在 LPAREN token 或 SEMICOLON （分号）等情况下，我们可能根本不想要什么建议，因为这些只是句法标记

当候选建议是一个 Identifier token 时，我们不会希望直接建议一个字符串 'indentifier' 。相反，我们会希望建议一个有意义的“符号”。比如，这可以是当前上下文的一个变量的名称，或者是当前命名空间的一个类的名称。

通常情况下，代码补全的大部分价值在于建议 'identifier' （变量）

现在，我们只需要配置一下 `ignoreToken` 就可以完成过滤

```js
core.ignoredTokens = new Set([KotlinParser.LPAREN, KotlinParser.SEMICOLON]);
```

然后， `collectCandidates` 将既不会返回 LPAREN ，也不会返回 SEMICOLORN

接下来，我们会完成建议 `identifier`
