---
title: 使用 Antlr4-c3 实现 code-completion
date: 2023/3/27
updated: 2023/3/30
tags: 
- IDE
- Antlr4
- code-completion
- antlr4-c3
categories: IDE
description: 一个好的 IDE 不可缺少的就是 code-completion 功能，下面我将会使用  `antlr4-c3` 来实现这一功能。
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

## 建议 `identifier`

如果我们收集两个信息，就可以确定哪些标识符可以放在光标下面

- 哪一种类型的标识符可以插入当前位置
- 哪些标识符是当前上下文中可见

让我们来看看如何计算每个集合

## 哪一种类型的标识符（变量）可以插入当前位置？

一般来说，在源代码中的任何位置，只有某些种类的标识符才是有效的。

比如，在编写 1 + ... 这样的表达式时，我们可以插入一个变量、一个函数或者一个方法的名称，但不能插入一个类型或者命名空间的名称

我们怎么样才能计算出哪种标识符是有效的？如果我们正在集成一个现有的解释器、编译器或者转译器，我们可能已经又了在解析源代码后处理和分析的工具，而且我们可以利用这些工具来达到我们的目的。请记住，这些工具比如对包含错误节点的解析树有很好的适应性，这些错误节点是由畸形的输入造成的

然而，如果我们不能轻易地使用我们已经拥有的或知道如何计算的信息，或者如果我们只有一个解析器（如我们的例子），我们仍然可以使用语法，结合antlr4-c3，为我们带来好处。

事实上，`antlr4-c3`返回的候选建议（candidate suggestions）既包括了 token 也包括了 规则（rules）。到目前为止，我们只是考虑了 token，现在是时候来关注 rule 了

从技术上来讲，只要我们想做，我们可以引用某个类的静态成员。我们简化这一过程

## Rules!（规则）

对于 antlr4-c3，如果你什么都不做， `antlr4-c3`就不会生成 rules map。但是，我们也可以指示具体规则，当我们从 `simpleIdentifier`开始

```js
core.preferredRules = new Set([ KotlinParser.RULE_simpleIdentifier ]);
```

`simpleIdentifier`是Kotlin解析器语法中的一条规则。它包装了标识符解析器规则，允许人们使用一些 "软关键字 "作为标识符。软关键字是一些词，比如 "抽象 "或 "注释"，它们在某些语句中使用时才是关键字。除此以外，我们可以把它们作为名称使用，例如，变量的名称。

现在，当我们收到完成度候选人时，我们可以检查我们是否可以建议一个标识符：

```js
if(candidates.rules.has(KotlinParser.RULE_simpleIdentifier)) {
    completions.push(...suggestIdentifiers());
}
```

我们不要忘记，我们仍然不想建议使用 "标识符 "这个字面关键词：

```js
candidates.tokens.forEach((_, k) => {
    if(k == KotlinParser.Identifier) {
        //Skip, we’ve already handled it above
    } else if(...) { ... } else {
        completions.push(parser.vocabulary.getSymbolicName(k).toLowerCase());
    }
});
```

还要注意的是，我们不能通过告诉引擎忽略`Identifier`标记来避免上面的 if 语句。如果我们这样做，它也会停止返回simpleIdentifier规则，因为antlr4-c3的工作方式。

## 符号表

到目前为止，我们还有一个问题：我们如果知道哪些变量在光标下的上下文是可见的？

编译器或者其他翻译器通常采用一种叫做符号表的数据结构来保存关于符号（标识符）的信息，如名称、范围、类型和其他特征。我们可以建立并使用这样的结构来跟踪哪些变量是可见的，从而给出合理的建议

幸运的是，antlr4-c3 提供了开箱即用的符号表实现，它是相当通用的，可以适用于很多语言。特别是，它已经有了代表几种符号的类（变量、函数和命名空间等）和一个我们可以拓展的类层次结构。我们可以用它来模拟嵌套作用域，并将每一个符号与解析树中的节点联系起来

## 通过 Visitor 建立符号表

我们可以通过使用 parse tree 的 visitor 来填充一个符号表。在这里，我们不会展示详细的实现，但会提供一个简单的 visitor，它只处理基本的上下文，以供参考

visitor 可以利用 antlr4 提供和生成的类和接口

```js
export class SymbolTableVisitor extends AbstractParseTreeVisitor<SymbolTable> implements KotlinParserVisitor<SymbolTable>
```

visitor 返回一个符号表，我们给这个对象比 visitor 更长的生命周期，使得 visitor 不能重复使用

```js
protected defaultResult(): SymbolTable {
    return this.symbolTable;
}
```

这只是设计选择之一，而不是为 visitor 的每一次运行都创建一个新的符号表

我们将在构造函数中初始化符号表，并且使用一个字段`scope`来跟踪当前的词法上下文

```js
constructor(
    protected readonly symbolTable: SymbolTable = new SymbolTable("", {}),
    protected scope = symbolTable.addNewSymbolOfType(ScopedSymbol, undefined)) {
    super();
}
```

我们将在下一节讨论上下文的问题

现在，关键的部分来了：我们必须在发现变量声明的时候记录它们。我们通过提供一个 `visitVariableDeclaration` 的回调实现

```JS
visitVariableDeclaration = (ctx: VariableDeclarationContext) => {
   this.symbolTable.addNewSymbolOfType(VariableSymbol, this.scope, ctx.simpleIdentifier().text);
   return this.visitChildren(ctx);
};
```

我们现在可以使用这个简单的符号表来提供变量名建议

```js
if(candidates.rules.has(KotlinParser.RULE_variableRead)) {
    let symbolTable = new SymbolTableVisitor().visit(parseTree);
    completions.push(...suggestVariables(symbolTable));
}
```

下面是 `suggestVariables` 函数的实现

```js
function suggestVariables(symbolTable: SymbolTable) {
    return symbolTable.getNestedSymbolsOfType(VariableSymbol).map(s => s.name);
}
```

## 处理上下文中的变量

然而，上面的例子根本没有处理范围问题。我们建议程序中的所有变量名在变量名可能出现的任何一点上都可以使用，不管语言的范围规则如何。

与符号表一样，如果我们有一个解释器或编译器，并且可以重复使用其中的部分内容，我们可能已经有代码来计算标识符的范围。但是，如果我们没有，本节将帮助我们建立它。

所以，现在让我们看看我们可能如何处理范围问题。在这里，我们将只为每个函数声明引入一个新的作用域，以演示这个想法：

```js
visitFunctionDeclaration = (ctx: FunctionDeclarationContext) => {
    return this.withScope(ctx, RoutineSymbol, [ctx.identifier().text], () => this.visitChildren(ctx));
};
```

antlr4-c3 的符号表中的符号也可以作为作用域。事实上，它们形成了一个层次结构，所以很容易创建一个从最具体到最不具体的作用域链，并在我们遍历树的过程中对它们进行跟踪：

```js
protected withScope<T>(tree: ParseTree, type: new (...args: any[]) => ScopedSymbol, args: any[], action: () => T): T {
    const scope = this.symbolTable.addNewSymbolOfType(type, this.scope, ...args);
    scope.context = tree;
    this.scope = scope;
    try {
        return action();
    } finally {
        this.scope = scope.parent as ScopedSymbol;
    }
}
```

我们从addNewSymbolOfType方法中复制了类型参数的复杂签名，我们调用该方法将新符号（范围）添加到当前符号（范围）中。

注意，我们还跟踪了范围的上下文。在我们的例子中，这是解析树中范围生效的部分：函数定义。我们将在后面使用它，以提供范围化的建议。

## 将两者结合

现在我们跟踪了标识符的范围，我们可以用它来提供适当的建议。我们将重写我们的 suggestVariables 方法，以利用我们在上一步中计算的范围：

```js
function suggestVariables(symbolTable: SymbolTable, context: ParseTree) {
    const scope = getScope(context, symbolTable);
    let symbols: Symbol[];
    if(scope instanceof ScopedSymbol) { //Local scope
        symbols = getAllSymbolsOfType(scope, VariableSymbol);
    } else { //Global scope
        symbols = symbolTable.getSymbolsOfType(VariableSymbol);
    }
    return symbols.map(s => s.name);
}
```

我们将很快展示getAllSymbolsOfType的实现，但首先要注意，现在 suggestVariables 需要搜索符号的上下文。这个上下文基本上就是位于光标下的那部分解析树，我们可以从中确定适用的作用域。

计算上下文的最佳位置是我们已经计算过的在托词下的标记的索引。因此，我们将把 computeTokenIndex...方法改为 computeTokenPosition...，并让它们同时返回索引和上下文：

```js
export type TokenPosition = { index: number, context: ParseTree };
export function computeTokenPosition(parseTree: ParseTree, caretPosition: CaretPosition): TokenPosition {
    //...
}
```

而在computeTokenPositionOfTerminalNode中，我们现在将返回一个TokenPosition：

```js
return { index: parseTree.symbol.tokenIndex, context: parseTree };
```

现在我们都准备好了，我们可以看看如何确定适用的范围和那里可见的所有变量名。

首先，为了确定作用域，我们利用antlr4-c3的符号表对作用域的支持，从我们的上下文开始，沿着解析树向上走，直到找到一个匹配的作用域：

```js
function getScope(context: ParseTree, symbolTable: SymbolTable) {
    if(!context) {
        return undefined;
    }
    const scope = symbolTable.symbolWithContext(context);
    if(scope) {
        return scope;
    } else {
        return getScope(context.parent, symbolTable);
    }
}
```

然后，我们将通过从最接近的范围开始，向上计算可见变量的集合：

```js
function getAllSymbolsOfType<T extends Symbol>(scope: ScopedSymbol, type: new (...args: any[]) => T): T[] {
    let symbols = scope.getSymbolsOfType(type);
    let parent = scope.parent;
    while(parent && !(parent instanceof ScopedSymbol)) {
        parent = parent.parent;
    }
    if(parent) {
        symbols.push(...getAllSymbolsOfType(parent as ScopedSymbol, type));
    }
    return symbols;
}
```

## 关于上下文的解释

我们的上下文实现只是一个起步，我们还没有处理几个现实的问题，包括：

来自其他文件的符号：在我们的语言中，我们可能会参考来自我们导入的库的名称，或者来自超类的名称，等等。在这里，我们一次只处理一个文件。

索引：我们已经建立了形成线性嵌套层次的作用域模型。然而，在一个有许多交叉引用的源文件的项目中，作用域很容易分支成一棵大树，而线性搜索会变得效率太低。真实世界的IDE为我们的源代码、库、平台代码等建立索引。

局部变量：许多语言允许在一个较窄的作用域中重新声明一个变量，从而在一些封闭的作用域中对同名的变量进行阴影。在这种情况下，我们的代码会返回相同的名字两次。

排序：我们没有以任何方式明确地对结果进行排序--它们被隐式地从最具体到最不具体地进行排序。这可能是好的，但是我们可能想按字母顺序来展示它们，例如。

如何处理这些和其他的改进，就留给读者吧。

## 部分匹配

通常情况下，当我们实现 code-completion 时，我们已经输入了关键词或者标识符的一部分，我们会希望编辑器能为我们完成它

例如，考虑下面的 kotlin 语句

```kotlin
fun test() {
    try {
        doSomething()
    } ca｜ // 光标
}
```

在这里，我们使用 | 表示光标所处的位置，当输入了字母 'ca' 后，我们希望引擎能够建议 'catch' 这个关键词，而不是其他不以 'ca' 开头的关键词，例如 'finally'。

然而，到目前为止，我们所写的代码并没有考虑到用户可能已经输入的字符，现在让我们看看如何利用这些信息来改进

首先，连同光标下的标记位置，我们返回需要匹配的文本：

```js
export type TokenPosition = { index: number, context: ParseTree, text: string };
```

然后，在 computeTokenPositionTerminalNode 中：

```js
return {
    index: parseTree.symbol.tokenIndex,
    context: parseTree,
    text: parseTree.text.substring(0, caretPosition.column - start)
};
```

可以看到，要匹配的文本只是 token 的一部分。或者，我们可以永远选择 token 的全部文本。最好的策略是取决于用户的期望和编辑器的约定。总之，当我们希望代码补全的时候，大部分情况关心点是在 token 的末尾，所以区别并不大

## 过滤候选建议

现在，让我们来看看计算代码建议的时候如何利用这些新信息。我们希望当前 token 不是被排除的类型的时候，才执行部分匹配，因为我们不希望与封闭的括号或者分号匹配：

```js
const isIgnoredToken =
    position.context instanceof TerminalNode &&
    ignored.indexOf(position.context.symbol.type) >= 0;
const textToMatch = isIgnoredToken ? '' : position.text;
```

考虑到这一点，让我们来过滤我们建议的关键词：

```js
candidates.tokens.forEach((_, k) => {
    let candidate;
    if(k == KotlinParser.Identifier) {
        //Skip, we’ve already handled it above
    } /* else if(...) { //Tokens with names that don’t match with a keyword
        candidate = ...;
    }*/ else {
        candidate = parser.vocabulary.getSymbolicName(k).toLowerCase();
    }
    if(candidate) {
        maybeSuggest(candidate, textToMatch, completions);
    }
});
```

我们定一个新的函数 `maybeSuggest` 如下所示

```js
function maybeSuggest(completion: string, text: string, completions: any[]) {
    if (tokenMatches(completion, text)) {
        completions.push(completion);
    }
}
```

其中，它会调用 `tokenMatches` 函数

```js
function tokenMatches(completion: string, text: string): boolean {
    return text.trim().length == 0 ||
           completion.toLowerCase().startsWith(text.toLowerCase());
}
```

可以看到，如果只过滤那些有实际文本的标记。如果一个标记没有文本或者是空字符串，那我们认为它是匹配的

同样的，建议变量时，我们只在 `variableRead` 表达式中执行部分匹配

```js
let variable = position.context;
while(!(variable instanceof VariableReadContext) && variable.parent) {
    variable = variable.parent;
}
const textToMatch = variable ? position.text : '';
return symbols.map(s => s.name).filter(n => tokenMatches(n, textToMatch));
```

现在我们完成了关键词和变量的部分匹配
