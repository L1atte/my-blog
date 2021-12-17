# my-blog

## 需提前安装hexo-cli 和 gulp

```
npm install -g hexo-cli
npm install gulp -g
```

## node环境

node： v14.17.3

npm: 6.14.13

## 安装依赖

```
npm install
```

或者

```
cnpm install
```

更推荐使用淘宝镜像(cnpm)的方式，速度更快

使用cnpm可能会遇到

```
Install fail! Error: [hexo-githubcalendar@^1.2.3] Can't find package hexo-githubcalendar@^1.2.3
```

这样的报错，此时直接去npm官网下载`hexo-githubcalendar@^1.2.3`的包即可，然后继续运行`cnpm i`

## 开发环境下运行项目

```
npm run dev
```

或者

```
hexo clean && hexo generate && gulp && hexo server
```

## 部署项目

```
npm run build
```

或者

```
hexo clean && hexo generate && gulp && hexo d
```

