# Webpack中文文档

## 什么是webpack

在考虑使用任何工具前，你需要明确一个重要的问题就是该工具能解决你的问题。webpack是一个模块打包器，这意味着它的目的是合并一组模块（包括模块间的依赖）。输出可能是一个一个或多个文件。当然除了捆绑模块之外，webpack也可以对你的文件执行各种操作：例如，将scss转换为css，将typescript转换为javascript。甚至可以压缩所有图像文件!但是为什么要打包模块呢？

### 打包模块的目的

除了使用许多`<script>`标签之外，我们没有办法将为浏览器提供的javascript代码拆分为多个文件。我们需要编写想要用于HTML代码的每个文件的源代码并不是很方便。社区提出了一些解决方法：CommonJS(在Node.js)中实现和AMD。在ES6之后，我们可以使用ES6的语法去做到。 

## ES6模块

使用ES6，定义了一个内置于JavaScript语言规范中的标准格式的模块。但是，并不意味着浏览器中能很好的实现。即使浏览器支持ES6，我们也可能希望将模块打包到更少的文件中。为了开始使用webpack，我们需要对ES6模块的语法有一定的了解。

### export

**export**语句用于创建JavaScript模块。可以使用它来导出对象(包括函数)和原始值。需要注意，导出的模块处于严格模式。导出有两种类型：**命名**和**默认**。

#### 命名export 

每个模块可以有多个命名导出。

lib.js

```js
export function sum(a, b) {
  return a + b;
}

export function sub(a, b) {
  return a - b;
}

function divide(a, b) {
  return a / b;
}

export { divide };
```

如果想在声明后导出某些内容，则需要将其包装在大括号中(如：divide)。

#### 默认导出

每个模块只能有一个默认导出。

dog.js

```js
export default class Dog {
  bark() {
    console.log('bark!');
  }
}
```

### import

**import**用于从导入其他模块。

#### 导入整个模块的内容

index.js

```js
import * as lib from './lib.js';

console.log(lib.sum(2,1));
console.log(lib.sub(3,1));
console.log(lib.divide(6,3));
```

可以为导入的模块设置任何所需的名称。如果要从具有默认导出的模块中导入整个内用，则将其置于**default**属性中。

index.js

```js
import * as Dog from './dog.js';

const dog = new Dog.default();
dog.bark();
```

#### 导入一个或多个命名导出

index.js

```js
import { sum, sub, divide } from './lib';

console.log(sum(1,2));
console.log(sub(3,1));
console.log(divide(6,3));
```

需要注意导入值的名称必须和导出的相匹配。

#### 导入默认导出

index.js

```js
import Dog from './dog';

const dog = new Dog();
dog.bark();
```

注意，可以使用任何名称导入默认导出。因此可以执行以下操作：

index.js

```js
import Cat from './dog.js';

const dog = new Cat();
dog.bark();
```

其他更多(导出)[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/export]和(导入)[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/import]示例，可参看MDN Web文档。

## webpack基本概念

从版本4开始，webpack不需要任何配置。它有一组默认值。如果要创建配置文件，则需要将其命名为`webpack.config.js`。

### webpack.config.js

由于是在Node.js中编写webpack配置文件，因此它使用CommonJS类型的模块。

`webpack.config.js`导出一个对象。如果你通过控制台运行webpack，它将查找该文件并使用它。

**Entry**

Webpack需要一个入口，它指示webpack从哪里开始打包模块。默认值如下：

webpack.config.js

```js
module.exports = {
  entry: './src/index.js'
}
```

这意味着webpack将转到`./src/index.js`文件，并开始打包。如果在index中导入其他任何js文件，webpack也会处理它们。

可以拥有多个入口点，但对于单页应用程序，通常只有一个入口点。

**Output**

**output**是webpack输出打包的配置。默认为`./dist/main.js`文件。

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'main.js'
  }
}
```

## 运行webpack

首先需要安装webpack。

```bash
npm init -y
npm install webpack
```

这将创建一个带有webpack目录的`node_modules`，以及`package.json`和`package-lock.json`。

打开package.json文件并修改脚本:

```js
"scripts": {
  "build": "webpack"
}
```

通过这个，运行`npm run build`将使用`node_modules`目录中的webpack。

### 多个入口及输出

前面的功能并不需要任何配置文件。如果想做其他操作，就需要创建`webpack.config.js`。

**entry**

配置详中的entry不必是字符串。如果需要多个入口，可以使用对象。

webpack.config.js

```js
module.exports = {
  entry: {
    first: './src/one.js',
    second: './src/two.js'
  }
}
```

使用该代码，我们就创建了两个入口点。这在开发多页面时会很有用。

**output**

这里存在一个问题：默认情况下，只会生成一个输出文件。这个问题很容易解决。

webpack.config.js

```js
const path = require('path');

module.exports = {
  entry: {
    first: './src/one.js',
    second: './src/two.js'
  },
  output: {
    filename '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

使用上面的代码，我们指明可能会有多个文件作为输出。所有的文件现在将有一个不同的名称，这里是`first.bundle.js`和`second.bundle.js`，和我们的entry一样。

如果以之前的方式运行webpack，它将找到`webpack.config.js`文件并应用配置。

## 添加loaders

使用loaders最好的方式是在`webpack.config.js`文件中通过`module.rules`属性指定它们。

### css-loader

**css-loader**用于解析导入的css文件

```bash
npm install css-loader
```

思考以下配置:

webpack.config.js

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use:'css-loader'
      }
    ]
  }
}
```

**rules**

所有loaders数组，这些规则将被应用到每个导入的文件，test是一个正则表达式，对于验证通过的会使用use指定的loader

**use**

指明对于test匹配通过的文件应用的loader

## 链式loaders

通过上面的配置，我们可以通过JavaScript导入css文件。但这不足以让css真正产生作用，因此需要使用一种方式，让我们的样式在浏览器中正常工作，**style-loader**可以作为一种有效的方式。

```bash
npm install style-loader
```

这就意味着需要使用两个loader，可以通过链式loader做到这一点。

webpack.config.js

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  }
};
```

这里**use**使用的是一个数组。需要注意的是**loader的执行顺序与数组顺序相反**。

style.css

```css
body {
  background-color: black;
}
```

index.js

```js
import './style.css';
```

上面的配置将按以下的顺序执行:

1. Webpack 将尝试解析style.css文件.
2. 文件名将匹配`/\.css$/`正则表达式对应的rule
3. 该文件将由**css-loader**解析
4. **css-loader**将解析的结果传递给**style-loader**
5. **style-loader**返回包含该样式的JavaScript代码

默认情况下，webpack编译后将生成`./dist/main.js` 。此文件中，将包含将所有样式添加到`<style>`标记的代码。

如果在html中引入main.js文件

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Webpack App</title>
</head>
<body>
  <script type="text/javascript" src="main.js"></script>
</body>
</html>
```

得到的结果如下：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Webpack App</title>
  <style type="text/css">body {
    background-color:black;
  }</style></head>
  <body>
    <script type="text/javascript" src="main.js"></script>
  </body>
</html>
```

### sass-loader

根据前面的内容，可以轻松的为项目添加sass/scss支持，这里使用**sass-loader**。

```bash
npm install sass-loader
```

将sass-loader加入链式loader中:

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader']
      }
    ]
  }
}
```

现在，就可以在js中导入scss文件了！在被**css-loader**解析前，**sass-loader**会将文件由scss转换成css。

## loader传参

loader可以接受额外的参数，这里通过**url-loader**来解释它。

```bash
npm install url-loader file-loader
```

webpack.config.js

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader']
      },
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 5000
            }
          }
        ]
      }
    ]
  }
}
```

如果需要传递参数给loader，则需要使用对象的方式来代替字符串，该对象有两个属性: **loader**\(loader的名称\)和**options**\(传递的参数\)。

**url-loader**用于将图片转换为base64地址，对于很小的图片，直接放在代码中有助于减少请求数。对于很大的图片，作为单独文件比较好，这样浏览器可以并行获取这些图片。

**url-loader**中的**limit**属性用于对图片的大小做出限制，超过该大小的将不会被转换为base64，而是采用**url-loader**直接复制文件。

```css
body {
  background-image: url('./big-background.png');
}

.icon {
  background-image: url('./icon.png');
}
```

以上配置将输出：

```html
<style type="text/css">body {
  background-image: url(ca3ebe0891c7823ff1e137d8eb5b4609.png); }

.icon {
  background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABIAAAASCAYAAABWzo5XAAAALElEQVR4AWMYIWAU1FPLoP9AXEFI0QEi8H+YYdQyqIEaXuumRhh1DZdUMwoATlYWfwh9eYkAAAAASUVORK5CYII=); }
</style>
```

由于`big-background.png`的大小超过limit的限制，它将被复制到`dist`目录下并生成一个随机文件名，而`icon.png`将会被转换成base64地址.

## 使用babel转换JavaScript

### babel-loader

babel-loader允许我们使用babel来转换JavaScript，可以让我们使用高版本的JavaScript或者一些当前浏览器没有实现的功能，而不用担心在老的浏览器上的兼容性问题。

```bash
npm install babel-loader babel-core babel-preset-env
```

webpack.config.js

```js
module.exports = {
  module: {
    rules: {
      test: /\.js$/,
      exclude: /(node_modules)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    }
  }
}
```

这里使用了**exclude**属性，值为一个正则表达式，如果一个文件的path匹配这个正则，这个文件将不会被转换。

## 如何使用plugins

最基本使用loaders的方式是将它放到我们配置文件中的plugins属性中，需要使用**new**创建一个实例去调用它。

### html-webpack-plugin

手动添加JavaScript文件到HTML中是很蠢的方法，一个有效的方式是使用**html-webpack-plugin**。

```bash
npm install html-webpack-plugin
```

使用方式如下:

webpack.config.js

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin()
  ]
}
```

这将创建一个index.html文件并把它放置在dist目录下，我们输出的javascript代码将被链接到结束&lt;body&gt;前.

index.html

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Webpack App</title>
</head>
<body>
<script type="text/javascript" src="main.js"></script>
</body>
</html>
```

当我们文件数量增加时，我们不用手动添加文件到我们的html文件中，另外，为了解决浏览器缓存，文件名使用了hash值，文件名可能会发生变化，使得**HtmlWebpackPlugin**更有效，因为我们不用手动去处理它。

### plugins传参

我们可以传递额外的参数到plugins。一个简单的例子是传递template到**HtmlWebpackPlugin**。

webpack.config.js

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
}
```

通过以上配置，将使用我们指定的template来代替默认创建的template。

### 重复使用同一个plugin

为什么我们使用plugin时需要使用**new**关键字。因为这样我们可以多次使用同一个plugin。

创建多页面应用程序时，我们可能希望输出多个html文件。这里可以通过多次使用**HtmlWebpackPlugin**来完成。

webpack.config.js

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: {
    one: './src/one.js',
    two: './src/two.js'
  },
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  plugins: [
    new HtmlWebpackPlugin({
        filename: 'one.html',
        template: './src/one.html',
        chunks: ['one']
    }),
    new HtmlWebpackPlugin({
        filename: 'two.html',
        template: './src/two.html',
        chunks: ['two']
    })
  ]
}
```

插件将根据chunks数组来匹配对应的入口文件，运行以上配置将生成: `one.html`,`one.bundle.js`,`two.html`,`two.bundle.js`.

### plugins和loaders同时使用

前面我们使用style-loader和css-loader注入我们的css到&lt;style&gt;标签中。如果要提供实际的css文件，需要使用**mini-css-extract-plugin。**

> 在webpack4之前，使用的是ExtractTextWebpackPlugin来完成。

```bash
npm install mini-css-extract-plugin
```

webpack.config.js

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin(),
    new MiniCssExtractPlugin()
  ]
}
```

index.js

```js
import './style.css';
```

通过HtmlWebpackPlugin，生成的css文件将自动链接到我们的HTML。输出的结果如下:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Webpack App</title>
  <link rel="stylesheet" href="main.css">
</head>
<body>
  <script type="text/javascript" src="main.js"></script>
</body>
</html>
```

使用该配置运行webpack将为每个导入到JavaScript文件中的css创建一个CSS文件。更改此行为需要使用到**SplitChunksPlugin**。

## 代码拆分

webpack4中发生了一些变化。在快速捆绑方面，它引入了**SplitChunksPlugin**来代替**CommonsChunksPlugin**。

### 什么是代码拆分？

代码拆分允许我们将代码拆分为多个文件。如果使用得当，能提高应用的性能。主要原因是，浏览器会缓存我们的代码，每次变更时，浏览网站的人都需要重新去下载变更后的文件。但是，对于一些外部依赖项，不会经常发生变更。所以将这些依赖拆分为单独的文件，访问者就不必重新下载这些依赖。

使用webpack会产生一个或多个bundles，其中包含源代码的最终版本，由chunks构成。

#### entry

**entry**是一个定义了我们应用从哪里开始执行，webpack从哪里开始打包的文件。可以定义一个\(单页应用\)或者多个\(多页应用\)入口文件。

定义一个**entry**将创建一个chunk，如果只定义了一个string的**entry**，chunk将被命名为main。如果使用对象定义了多个**entry**，**entry**的key将作为chunk的名称。以下两者是等价的:

```js
extry: './src/index.js'
```

```js
entry: {
  main: './src/index.js'
}
```

#### output

**output**决定了Webpack在何处以什么样的方式输出我们的`bundle`及`assets`。虽然可以指定多个**entry**,但只能指定一个**output**配置。这就是我们入口中名称的作用。虽然可以为bundle指定一个具体的文件名，但由于要拆分我们的代码,所以不应该这么做。更好的方式是使用`[name]`为输出文件的文件名创建模板:

```js
output: {
  filename: '[name].[chunkhash].bundle.js',
  path: path.resolve(__dirname, 'dist')
}
```

这里需要注意的是`[chunkhash]`:它是一个基于chunk的hash,根据文件内容生成。当文件内容发生变更时，他才会更改。这样浏览器就可以使用特定的方式去缓存它。如果文件名发生变更，浏览器就会去重新加载变更后的文件。一个chunkhash的例子：`0c553ebfd158e16da428`

我们main chunk将被打包到名为`main.[chunkhash].bundle.js` 中。

### SplitChunksPlugin

由于**SplitChunksPlugin**，可以将应用程序的某些部分移动到单独的文件中。如果模块在多个chunk中使用，则可以在他们之间轻松共享。

utilities/users.js

```js
export default [
  { firstName:'Adam', age: 28 },
  { firstName:'Jane', age: 24 },
  { firstName:'Ben',  age: 21 },
  { firstName:'Lucy', age: 40 }
]
```

a.js

```js
import _ from 'lodash';
import users from './utilities/users';

const adam = _.find(users, { firstName: 'Adam' });
```

b.js

```js
import _ from 'lodash';
import users from './utilities/users';

const lucy = _.find(users, { firstName: 'Lucy' });
```

webpack.config.js

```js
module.exports = {
  entry: {
    a: './src/a.js',
    b: './src/b.js'
  },
  output: {
    filename: '[name].[chunkhash].bundle.js',
    path: __dirname + '/dist'
  }
}
```

运行上面的配置，webpack将会创建两个文件: `a.[chunkhash].bundle.js` 和 `b.[chunkhash].bundle.js`。这两个文件中都会包含lodash库。前面说过，为共享库创建单独的文件时webpack默认的行为，但这涉及到异步chunk，意味着我们异步导入文件。要涉及到所有chunk，需要稍微修改下webpack配置:

webpack.config.js

```js
module.exports = {
  entry: {
    a: './src/a.js',
    b: './src/b.js'
  },
  output: {
    filename: '[name].[chunkhash].bundle.js',
    path: __dirname + '/dist'
  },
  optimization: {
    splitChunks: {
      chunks: 'all'
    }
  }
}
```

现在运行配置，可以看到多了一个 `vendors~a~b.[chunkhash].bundle.js`并且里面包含了lodash库，这是由于有一些开箱即用的**cacheGroups**默认配置

```js
splitChunks: {
  chunks: 'all',
  cacheGroups: {
    vendors: {
      test: /[\\/]node_modules[\\/]/,
      priority: -10
    },
    default: {
      minChunks: 2,
      priority: -20,
      reuseExistingChunk: true
    }
  }
}
```

首先,**vendors**中包含`node_modules`中的文件。其次，默认的**cacheGroups**会被应用到其他的共享模块。

仔细观察`a.[chunkhash].bundle.js`和`b.[chunkhash].bundle.js`，会发现这两个文件中都包含了users.js中的内容。原因在于，默认情况下，**SplitChunksPlugin**仅对大于30kb的文件进行拆分。可以通过配置去改变：

webpack.config.js

```js
module.exports = {
  entry: {
    a: './src/a.js',
    b: './src/b.js'
  },
  output: {
    filename: '[name].[chunkhash].bundle.js',
    path: __dirname + '/dist'
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      minSize: 0
    }
  }
}
```

这将导致创建一个`a~b.[chunkhash].bundle.js`，这里改变的是默认**cacheGroups**的配置。由于users.js的大小小于30kb，因此在不更改**minSize**的情况下，它不会被打包到单独的文件。在真实的项目中，这样可以防止浏览器对于很小的文件产生额外的请求，有助于提升性能。

这里也可以只针对utilities目录去处理:

webpack.config.js

```js
module.exports = {
  entry: {
    a: './src/a.js',
    b: './src/b.js'
  },
  output: {
    filename: '[name].[chunkhash].bundle.js',
    path: __dirname + '/dist'
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        utilities: {
          test: /[\\/src[\\/]utilities[\\/]/,
          minSize: 0
        }
      }
    }
  }
}
```

现在打包的结果将包含4个文件: `a.[chunkhash].bundle.js`,`b.[chunkhash].bundle.js`,`vendors~a~b.[chunkhash].bundle.js`和`utilities~a~b.[chunkhash].bundle.js`。尽管我们现在设置**splitChunks.minSize**为**0**，默认的cache group也不会被创建。这是由于这些文件已经被之前创建的**utilites**组所覆盖到了，它拥有默认的优先级**0**。仔细观察的话，你会发现默认的cache group设置的默认优先级是**-20**。对于相同优先级的cache group，后声明的会被舍弃掉。

**SplitChunksPlugin**也设置了其他的默认参数，可以参靠[SplitChunksPlugin文档](https://webpack.js.org/plugins/split-chunks-plugin).

## webpack内置的生产优化

### 为什么要优化代码?

在开发中，我们希望我们的代码易于阅读，因此添加了大量空格（制表符,空格,换行符）和注释。虽然它改进了我们的代码，但是它会是我们的文件更大。另一方面为了提升用户体验而牺牲可读性是不可取的。所以我们需要一定的解决方案来处理这一问题。

### mode: production

Webpack4中引入**mode**参数，在这之后我们需要指定这个参数，如果不指定这个参数，webpack会使用默认的值production，并且会产生一条警告信息。使用`mode: "production"`，Webpack将为我们设置一些配置，这样，我们输出的代码就更适合生产环境。下面是它具体产生的作用。

#### UglifyJsPlugin

将**mode**设置为production会将UglifyJsPlugin添加到我们的配置中。它可以通过压缩和缩小来使我们的代码更短，更快。执行从缩短变量名称，删除空格到删除冗余代码这样简单的任务。默认情况下，它解析每个**.js**文件。虽然Webpabk4根据所选**mode**来进行优化，但仍可以通过**optimization**属性对其进行配置。

webpack.config.js

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
  mode: 'production',
  // 使用mode: 'production'将激活下面的配置
  optimization: {
      minimize: true,
      minimizer: [
        new UglifyJsPlugin()
      ]
  }
}
```

当然，也可以传递uglifyOptions给**UglifyJsPlugin**。比如**compress**属性。

```js
new UglifyJsPlugin({
  uglifyOptions: {
    compress :{
      /* (...) */
    }
  }
})
```

这些属性可以让我们自己去配置**UglifyJsPlugin**，从而是我们的代码更短更轻量。其他的属性及默认值可以参考[官方文档](https://github.com/mishoo/UglifyJS2/tree/harmony#compress-options)。

#### DefinePlugin

该插件允许我们创建在编译时解析的全局常量。如果使用`mode: 'production'`，webpack将设置`"process.env.NODE_ENV": JSON.stringify("production")`，默认情况下:

webpack.config.js

```js
module.exports = {
  mode: 'production',
  plugins: [
    new webpack.DefinePlugin({
      "process.env.NODE_ENV": JSON.stringify("production")
    })
  ]
}
```

这里需要注意，由于是直接文本替换，所以给予属性的值必须包含实际引号。可以用过`JSON.stringify("production")`或者`'"production"'`来完成。

在编译时，意味着你如果你在代码中使用_process.env.NODE\_ENV_，它将被给定的值所替换。

```js
console.log(process.env.NODE_ENV);
if(process.env.NODE_ENV === 'production') {
  console.log('this is production!');
}
```

编译后的代码中不会保留_process.env.NODE\_ENV_。使用webpack运行上面的代码将生成:

```js
console.log("production");
if(true) {
  console.log("this is production!");
}
```

在**UglifyJsPlugin**完成缩小后，它可以简化为:

```js
console.log("production");
console.log("this is production!");
```

#### NoEmitOnErrorsPlugin

该插件将帮助我们在编译期间处理错误。例如，可能存在你**import**的文件webpack无法解析，在这种情况下，webpack会创建一个包含该错误的新版本应用程序。通过该插件，webpack就不会去创建新的版本了。

webpack.config.js

```js
const webpack = require('webpack');

module.exports = {
  mode: 'production',
  // 使用 mode: 'production' 将激活下面的配置
  plugins: [
    new webpack.NoEmitOnErrorsPlugin()
  ]
}
```

#### ModuleConcatenationPlugin

默认情况下，Webpack将bundle中每个模块包装在单独的闭包函数中。这些闭包函数会使JavaScript执行起来慢一些。以下是一个示例：

one.js

```js
const dog = 'Fluffy';
export const one = 1;
```

two.js

```js
const dog = 'Fluffy';
export const two = 2;
```

index.js

```js
import { one } from './one';
import { two } from './two';
const dog = 'Fluffy';

console.log(one, two);
```

如果不使用ModuleConcatenationPlugin，输出包如下:

```js
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _one__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);
/* harmony import */ var _two__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(2);



const dog = 'Fluffy';

console.log(_one__WEBPACK_IMPORTED_MODULE_0__["one"], _two__WEBPACK_IMPORTED_MODULE_1__["two"]);

/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "one", function() { return one; });
const dog = 'Fluffy';
const one = 1;

/***/ }),
/* 2 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "two", function() { return two; });
const dog = 'Fluffy';
const two = 2;

/***/ })
```

当**mode**设置为production时，插件将开始执行所有的工作。由于这个原因，输出包现在在一个范围内。更少的函数意味着更小的开销，以下是输出的示例，为了更清楚的看到输出，这里将配置中的`optimazation.minimize`设置为**false**。实际上minimizer现在知道模块间的依赖关系，可以让我们的代码更加优化。

main.js

```js
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);

// CONCATENATED MODULE: ./src/one.js
const dog = 'Fluffy';
const one = 1;
// CONCATENATED MODULE: ./src/two.js
const two_dog = 'Fluffy';
const two = 2;
// CONCATENATED MODULE: ./src/index.js



const src_dog = 'Fluffy';

console.log(one, two);

/***/ })
```

## development server

更好的开发体验是在`watch`模式下运行webpack。尝试运行

```bash
webpack -watch
```

现在每次进行一些更改时，webpack都会重新build项目的代码。**webpack-dev-server**就是如此。它不是将文件写入目标目录，而是在内存中处理它们。构建后，输出到本地服务器。

### webpack-dev-server

首先，安装它。

```bash
npm install webpack-dev-server
```

然后，在**package.json**的`scripts`中添加它:

```js
"scripts": {
    "build": "webpack",
    "start": "webpack-dev-server"
}
```

输入`npm start`运行它。可以看到如下消息：

```js
[wds]: Project is running at http://localhost:8080/
```

在浏览器中访问[http://localhost:8080/](http://localhost:8080/)。每次变更代码后，网站都会更新，刷新页面后,就可以看到变更了。

### 热模块替换

为了更好的提升开发体验，可以通过热模块更新来跳过刷新浏览器。比如，当我们改变某些样式时，不用重新加载整个应用就可以看到变化。

> 目前，热模块加载暂时不支持**MiniCssExtractPlugin**。所以在开发环境中，我们还是得用**style-loader**。

当运行**webpack-dev-server**时，它使用与常规构建相同的配置文件。可以添加**devServer**到**webpack.config.js**，来启动热模块替换。

```js
devServer: {
    hot: true
}
```

之后将`webpack.HotModuleReplacementPlugin`添加到plugins中。

webpack.config.js

```js
const webpack = require('webpack');

module.exports = {
  devServer: {
      hot: true
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
}
```

> 注意: 运行`webpack-dev-server -hot`也将添加**HotModuleReplacementPlugin**到plugins中。这样会带来一定的问题。

这样使用css就像一个小挂件一样，但是使用JavaScript，它还需要一个额外的步骤。

index.js

```js
import { divide } from './divide';

console.log(`6 / 2 = ${divide(6, 2)}`);

if(module.hot) {
  module.hot.accept();
}
```

divide.js

```js
export function divide(a, b) {
  return a/b;
}
```

运行`module.hot.accept()`将使模块可替换。这也适用于它导入的所有其他模块。使用上面的代码意味着在index.js中运行`accept()`也可以使 divide 模块可替换。

### webpack-serve

虽然webpack-dev-server使用了很长时间，但它现在只是维护模式，一些新的功能不会很快更新。代替的是**webpack-serve**，它使用的是WebSockets。是webpack-dev-server相对较新的版本。

```bash
npm install webpack-serve
```

package.json

```js
"scripts": {
  "build": "webpack",
  "start": "webpack-serve"
}
```

由于**webpack-serve**默认使用**HotModuleReplacementPlugin**，因此添加`webpack.HotModuleReplacementPlugin`会导致错误。

### mode:“development”

#### DefinePlugin

```js
module.exports = {
  mode: "development",
  // 使用 mode: "development" 将激活下面的配置
  plugins: [
    new webpack.DefinePlugin({
        "process.env.NODE_ENV": JSON.stringify("development")
    })
  ]
}
```

#### NamedModulesPlugin

通过该插件，我们可以看到被替换模块的相对路径。

```js
[WDS] App updated. Recompiling...
[WDS] App hot update...
[HMR] Checking for updates on the server...
[HMR] Updated modules:
[HMR] - ./src/style.css
[HMR] App is up to date
```

否则，我们就只能看到一个id而不是`./src/style.css`。

#### NamedChunksPlugin

它与**NamedModulesPlugin**用途类似。通过该插件，不仅`modules`将具有名称，`chunks`也将具有名称。当应用程序运行时，可以通过`window.webpackJsonp`属性查看它们。

**NamedModulesPlugin**和**NamedChunksPlugin**的另一个优点是，不再使用可以通过添加和删除依赖项而更改的ID。由于在实际输出文件中使用了id或名称，因此更改它们将更改文件的hash值，即使模块内容没有改变。使用**NamedModulesPlugin**和**NamedChunksPlugin**将帮我们处理浏览器的缓存。以下是一个示例:

没有 **NamedModulesPlugin**和 **NamedChunksPlugin**:

```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([[2],{
  /***/ 6:
  (...) // divide.js module output code

  /***/ 7:
  (...) // substract.js module output code
]);
```

使用 **NamedModulesPlugin**和 **NamedChunksPlugin**:

```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([["utilities~main"],{
  /***/ "./src/utilities/divide.js":
  (...) // divide.js module output code

  /***/ "./src/utilities/substract.js":
  (...) // substract.js module output code
]);
```

### Devtool

设置`mode:"development"`，还会将 **devtool** 的值设置为eval，来启用源映射。

```js
module.exports = {
  mode: "development",
  // 使用 mode: "development"将激活下面的配置
  devtool: "eval"
}
```

对代码进行打包，丑化，压缩可以使代码更小并且性能更好，可以提升用户体验。但是调试这样的代码却比较复杂，通过sourceMap，可以将输出的代码和源文件进行映射。由于这一点，我们可以更加方便的使用调试器并在查看实际的代码时设置断点。

### Tree Shaking

我们经常在具有大量其他导出的文件上使用命名导入。他可能会创建一个我们不会导入所有内容的情况，但无论如何webpack都会包含一个完整的模块。通过**tree shaking**，可以帮助我们消除这些不用的代码。从而优化包的体积。

为了使用**tree shaking**，需要一些满足一些要求。首先，必须使用ES6模块，而不是类似**CommonJS**这种。如果使用Babel，会带来一些问题，因为它默认配置是将任何模块类型转换为CommonJS。可以使用`modules: false`，通过`.babelrc`或`webpack.config.js`文件去修复这个问题。

.babelrc

```js
{
    "presets": [
        [
            "env",
            {
                "modules": false
            }
        ]
    ]
}
```

webpack.config.js

```js
module: {
  rules: [
    {
        test: /\.js$/,
        exclude: /(node_modules)/,
        use: {
            loader: 'babel-loader',
            options: {
                presets: ['env', { modules: false }]
            }
        }
    }
  ]
}
```

通常我们会使用**UglifyJsPlugin**。默认情况下，`mode: "production"`会添加它。当然其他情况下，你也可以手动添加它。

还有一件事是需要开启**optimization.usedExports**。`mode: "production"`也会添加这个。它用于告诉webpack确定每个模块的已使用导出。通过它，webpack将在打包的文件中添加额外的注释，如`/* unused harmony export */`，这样UglifyJsPlugin可以在后面解析它。

下面是一个示例:

utilities.js

```js
export function add(a, b) {
  return a + b;
}

export function sub(a, b) {
  return a - b;
}
```

index.js

```js
import {add} from './utilities';

console.log(add(1,2));
console.log(add(3,4));
```

在没有正确配置的情况下输出：

```js
/*(...)*/

/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "add", function() { return add; });
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "substract", function() { return substract; });
function add(a, b) {
    return a + b;
}
function subtract(a, b) {
    return a - b;
}

/***/ })
/******/ ]);
```

这里，可以看到webpack没有**tree sharking**我们的package，**add** 和 **sub** 方法都存在。调整webpack的配置:

webpack.config.js

```js
const webpack = require('webpack');
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
const UglifyJs = require('uglify-es');

const DefaultUglifyJsOptions = UglifyJs.default_options();
const compress = DefaultUglifyJsOptions.compress;
for(let compressOption in compress) {
    compress[compressOption] = false;
}
compress.unused = true;

module.exports = {
    mode: 'none',
    optimization: {
        minimize: true,
        minimizer: [
            new UglifyJsPlugin({
                uglifyJsOptions: {
                    compress,
                    mangle: false,
                    output: {
                        beautify: true
                    }
                }
            })
        ],
     	usedExports: true
    }
}
```

这里关闭了**UglifyJsPlugin**选项，这样可以更清楚的看到代码发生了什么。运行上面的配置:

```js
/* (...) */

/* 0 */
/***/ function() {
    "use strict";
    // CONCATENATED MODULE: ./src/utilities.js
        function add(a, b) {
        return a + b;
    }
    // CONCATENATED MODULE: ./src/index.js
    console.log(add(1, 2));
    console.log(add(3, 4));
    /***/}
/******/ ]);
```

由于使用了 **optimization.usedExports** 和 **UglifyJsPlugin** 的 **unused** 选项，因此删除了不必要的代码。注意，这是UglifyJsPlugin中的默认行为，因此使用默认的选项也会移除不用的代码（除了运用了许多其他压缩过程）。

### 对libary的Tree Shaking

如果打算对libary使用**Tree shaking**，首先需要使用ES6模块。这对于libary来说并不是很明确。比如**lodash**，查看它提供的代码，可以看到它并没有使用ES6模块。

假如要使用 **lodash** 提供的 `debounce` 功能。

index.js

```js
import _ from 'lodash';

console.log(_.debounce);
```

现在我们的输出中会包含整个 **lodash** 库。`import _ from 'lodash'`时无法避免这种情况。有人就想了一个方法并创建了一个名为`lodash-es`的package，它提供了作为ES6模块导出的 **lodash** 库。

```js
import {debounce} from 'lodash-es';

console.log(debounce);
```

但是，webpack仍然无法使用**tree shaking**。根据ECMAScript规范，所有子模块都需要进行评估，因为它们可能包含副作用。如果包作者想告知他的库中没有副作用，可以在 **package.json** 中告知。查看 **lodash** 的 **package.json**，会发现他有`"sideEffects": false`。

**webpack默认忽略sideEffects标记**。并且更改我们设置**optimization.sideEffects**为true的行为。我们可以手动处理它或者通过 `mode: "production"`。

webpack.config.js

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    mode: 'none',
    optimization: {
        minimize: true,
        minimizer: [
            new UglifyJsPlugin()
        ],
        usedExports: true,
        sideEffects: true
    },
    plugins: [
        new HtmlWebpackPlugin()
    ]
}
```

现在**lodash**库就可以被webpack **tree shaking** 了

## Dynamic Import

前面的代码中，ECMAScript模块都是静态的。在运行代码之前，必须明确指定要导入和导出的内容。通过Dynamic Import，可以已一种额外的方式来动态导入模块。比如根据用户的操作，来动态加载需求的模块。例如，在用户打开单页应用程序的子页面时，才加载子页面的代码。这样就能节省用户首次打开应用程序的时间。

### 使用Dynamic Import

Dynamic import是一个函数，接收一个字符串参数并返回一个promise。当模块加载完成，promise的状态将变为resolved。

```js
document.addEventListener('DOMContentLoaded', () => {
	const button = document.querySelector('#divideButton');
	button.addEventListener('click', () => {
		import('./utilities/divide')
			.then(divideModule => {
				console.log(divideModule.divide(6, 3)); // 2
			})
	});
});
```

打开开发者工具中的network选项卡，当单击按钮时，可以看到模块会被下载，而不是立即下载。当再次单击按钮时，不会再次下载该模块。

对webpack使用动态导入，该块将创建一个新的异步模块。

这种模块将被打包到一个单独的文件中。使用表达式指定文件路径时需要注意。考虑以下例子:

```js
let fileName = '';

document.addEventListener('DOMContentLoaded', () => {
	const button = document.querySelector('#divideButton');
	button.addEventListener('click', () => {
		import(`./utilities/${fileName}`)
			.then(divideModule => {
				console.log(divideModule.divide(6, 3)); // 2
			})
	});
});
```

使用该代码构建项目后，你将会发先webpack为utilities目录中的每个模块创建了不同的异步chunk，这是因为webpack在编译时无法知道将导入哪些模块。

同时，对于全部动态语句(如`import(pathToFile)`)对于webpack来说将不会起作用，因为webpack至少需要知道一些文件的路径信息。由于pathToFile可能是项目中的任何文件路径，webpack将会为每个目标路径创建异步模块。你可以通过下面的方式，自定义它的行为。

### 在webpack中使用特殊注释

**import** 规范中不允许你使用出文件名之外的任何其他参数。但是可以借助webpack的特殊注释来做到这一点。

#### webpackInclude和webpackExclude

前面提到webpack将为我们指向的目录中每个模块创建异步块。虽然这是默认行为，但是可以更改。

**webpackExclude**

通过webpackExclude正则表达式，将与可导入的潜在文件进行匹配，任何匹配的模块不会被打包在一起。

```js
import(
	`./utilities/${fileName}`,
	/* webpackExclude: /subtract.js$/ */
)
```
使用上面的代码，即使subtract.js位于utilities目录，它也不会被打包。

**wepackInclude** 与 **webpackExclude** 的作用相反。使用它，则只打包指定正则匹配的模块。

#### webpackMode

该属性定义了解析动态导入的模式。以下是其支持的值：

**lazy**

默认的mode，它将为动态导入的每个模块生成异步模块。

**lazy-once**

为导入调用匹配的所有模块生成单个异步chunk。

```js
import(
	`./utilities/${fileName}`,
	/* webpackMode: "lazy-once" */
)
.then(divideModule => {
	console.log(divideModule.divide(6, 3)); // 2
})
```

使用此代码，webpack将为utilities目录中所有模块创建一个异步chunk。

**eager**

阻止webpack创建额外的chunk，所有导入的chunk都将包含在当前chunk中，因此不会产生额外的请求。promise仍让会有返回，但是它将自动resolved。动态导入时使用**eager**与静态导入的区别时直到import()调用时，模块中的代码才会被执行。

**weak**

阻止产生的额外请求，只有模块被其他方式加载时，Promise的结果才会是resolved，其他情况下，将为rejected。

#### webpackChunkName

用来指定创建的chunk的名称。可以同`[index]`和`[request]` 变量结合使用。

`[index]`动态导入的当前文件的索引，`[request]` 导入文件路径的动态部分。

```js
const fileName = 'divide';

import(
	`./utilities/${fileName}`,
	/* webpackChunkName: "utilities-[index]-[request]" */
)

```

以上代码生成的文件名是 utilities-0-divide.js。

在确定只生成一个异步chunk的情况下(如,不使用动态路径或者lazy-once模式)，不能使用`[index]`和`[request]`。

### 通过prefetch和preload提高性能

webpack4.6.0中添加了**prefetch**和**preload**的支持。使用这些方式将改变浏览器处理异步chunk的加载方式。

**prefetch**

通过prefetch，可以指示将来可能需要该模块。浏览器将在空闲时间下载它，并且在父块完成后下载。

```js
import(
	`./utilities/divide`,
	/* webpackPrefetch: true */
	/* webpackChunkName: "utilities" */
)
```

这将导致`<link rel="prefetch" as="script" href="utilities.js">`被添加到页面头部，浏览器将在空闲时间去加载它。


**preload**

通过preload标记资源，可以指示现在需要它。此异步块将和父块并行加载。如果首先加载父块，在等待异步块加载的过程中仍可以显示该页面。这可能提升一定的性能。

```js
import(
	`./utilities/divide`,
	/* webpackPreload: true */
	/* webpackChunkName: "utilities" */
)
```

上面的代码将会将`<link rel="preload" as="script" href="utilities.js">`插入到页面头部。错误的使用**webpackPreload**将会损害一定的性能。
