## 安装

新建一个工程目录，从命令行进入该目录，并执行npm的初始化命令

```bash
npm init
```

生成一个package.json文件，相当于npm项目的说明书，里面记录了项目名称，版本，仓库地址等信息。

```bash
npm install webpack webpack-cli --save-dev
```

这里我们同时安装了webpack以及web pack-cli。webpack是核心模块，web pack-cli则是命令行工具。由于我们将Webpack安装在了本地，因此无法直接在命令行内使用"webpack"指令。工程内部只能使用`npx Webpack <command>` 形式，在未简化指令前。

## 打包第一个应用

1. 首先在刚刚的工程目录下新建几个文件

   > Index.js
   >
   > ```javascript
   > import addContent from './add-content.js'
   > document.write('my first webpack app');
   > addContent()
   > ```
   >
   > add-content.js
   >
   > ```javascript
   > export default function() {
   >   document.write('hello world')
   > }
   > ```
   >
   > index.html
   >
   > ```html
   > <!DOCTYPE html>
   > <html lang="en">
   > <head>
   >     <meta charset="UTF-8">
   >     <meta name="viewport" content="width=device-width, initial-scale=1.0">
   >     <meta http-equiv="X-UA-Compatible" content="ie=edge">
   >     <title>Document</title>
   > </head>
   > <body>
   >     <script src="./dist/bundle.js"></script>
   > </body>
   > </html>
   > ```
   >
   > 

   

2. 然后在终端输入打包命令

   ```bash
   npx webpack --entry=./index.js --output-filename=bundle.js --mode=development
   ```

   > 1. entry参数是资源打包的入口。Webpack从这里开始进行模块依赖查找，得到index.js和add-content.js两个模块。
   > 2. Output-filename是输出资源名，打包完工程会出现一个dist目录，包含的bundle.js就是打包结果。
   > 3. mode参数是打包模式

   

3. 用浏览器打开index.html看效果