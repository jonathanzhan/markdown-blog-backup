---
title: Webpack配置介绍
date: 2019-03-05 19:22:43
tags:
- node
categories:
- 前端设计
---
webpack.config.js为webpack的默认配置,我们可以为开发环境和生产环境分别做不同的配置.下面一一介绍每个配置的作用
<!-- more -->
### webpack配置
```js
var path = require('path');
var webpack = require('webpack')
var HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
  entry: {
    app: './src/main.js'
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: '',
    filename: '[name].js'
  },
  resolve: {
    extensions: ['', '.js', '.jsx', '.json'],
    alias: {
      'src': path.resolve(__dirname, './src'),
      'assets': path.resolve(__dirname, './src/assets'),
      'components': path.resolve(__dirname, './src/components')
    }
  },
  module: {
    loaders: [
      {
        test: /\.js|jsx?$/,
        exclude: /node_modules/,
        loader: 'babel',
        query: {
          presets: ['es2015', 'react']
        }
      },
      {
        test: /\.css$/,
        loader: 'style!css',
        exclude: /node_modules/
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url',
        query: {
          limit: 10000,
          name: path.join('static', 'img/[name].[hash:7].[ext]')
        }
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
  ]
}
```

### entry
`entry`是入口配置项,可以说`string`,`Array`或者一个`Object`
```js
entry: {
  app: './src/main.js'
}
entry: './src/main.js'
//如果页面有多个入口可以这样写:
entry: ['./src/home.js', '.src/profile.js']
//或
entry: {
  home: './src/home.js',
  profile: './src/profile.js'
}
```

### output
`output`是输出配置

```js
output: {
  path: path.resolve(__dirname, './dist'),
  publicPath: '/',
  filename: '[name].js',
  chunkFilename: '[id].[hash].js'
}
```
path是文件输出到本地的路径，publicPath是文件的引用路径，可用来被一些Webpack插件用来处理CSS，HTML文件中的URL，一般用于生产模式，filename是打包后的入口文件名，chunkFilename是每个模块编译后的文件名，其中[hash]是用来唯一标识文件，主要用来防止缓存。
#### path
path仅仅用来告诉webpack在哪里存放结果文件。上面的例子中,最终的打包文件会放到与当前脚本文件同级目录的dist目录下
```
hello-webpack
  +dist
  -webpack.config.js
```

#### publicPath
文件的引用路径，可用来被一些Webpack插件用来处理CSS，HTML文件中的URL，在开发模式下建议使用相对路径，在生产模式中，如果你的资源文件放在别的服务器上，可以使用服务器的地址。当然你也可以不用配置`publicPath`，。
在项目中我使用了`url-loader`加载图片

```js
{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    loader: 'url',
    query: {
      limit: 10000,
      name: path.join('static', 'img/[name].[hash:7].[ext]') # 图片最终的输出路径
    }
}
```
在`main.js`中使用了图片
```js
import Girl from 'assets/img/girl.jpg'
```

那么最终浏览器访问图片的路径就是
`static/img/girl.7672e53.jpg `

所以可以根据开发环境和生产环境配置不同的`publicPath`。
在生产环境中，由于我的资源文件放在项目目录下，所以可以这样配置`output`:
```js
output: {
  path: path.resolve(__dirname, './dist'),
  publicPath: './',
  filename: 'js/[name].[chunkhash].js',
  chunkFilename: `js/[id].[chunkhash].js`
}
```

#### filename
入口文件打包后的名称，`[name]`对应着入口文件的key值,例如`app.js`，这对多入口文件是很有用的，因为入口文件可以有多个,但是`filename`只能有一个

#### chunkFilename
非入口文件打包后的名称,未被列在entry中,却又需要被打包出来的文件的命名配置规则。

### loader
```js
module: {
    loaders: [
      {
        test: /\.js|jsx?$/,
        exclude: /node_modules/,
        loader: 'babel',
        query: {
          presets: ['es2015', 'react']
        }
      },
      {
        test: /\.css$/,
        loader: 'style!css',
        exclude: /node_modules/
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url',
        query: {
          limit: 10000,
          name: path.join('static', 'img/[name].[hash:7].[ext]')
        }
      }
    ]
  }
```
由于webpack本身只能处理javascript模块,如果要处理其他类型的文件,就需要使用loader进行转换。Loader可以理解为是模块和资源的转换器,他本身是一个函数,接受源文件作为参数,返回转换的结果。不同的loader可以将各种类型的文件转换为浏览器能够接受的格式如JS,Stylesheets等。
* `test` 参数用来指示当前配置项针对哪些资源,当参数匹配时，就是使用对应的loader.
* `exclude` 参数用来剔除需要忽略的资源。
* `include` 参数用来表示本loader配置仅针对哪些目录文件，从名称上就可以认为跟`exclude`作用相反。
* `loader/loaders`参数,用来指示用哪个/哪些loader来处理目标资源,这俩表达的其实是一个意思，只是写法不同，在webpack2中loaders被替换成了rules,其实就是loaders规则
### plugin
插件可以完成更多loader不能完成的功能。
插件的使用一般是在webpack的配置信息plugins选项中指定。
loader是在打包前或打包过程中作用于单个文件。plugin通常在打包过程结束后，作用于包或者chunk级别。
以下是一些常用的插件

#### extract-text-webpack-plugin
ExtractTextPlugin的作用是把各个chunk加载的css代码合并成一个css文件并在页面加载的时候以`<link>`的形式进行加载。

```js
var ExtractTextPlugin = require('extract-text-webpack-plugin')

module: {
  loaders: [
    {
      test: /\.css$/, 
      loader:ExtractTextPlugin.extract("style-loader","css-loader") }
  ]
},
plugins: [
  new ExtractTextPlugin(path.join('static', 'css/[name].[contenthash].css'))
]
```
> 注意：如果想要把CSS放到HTML的style标签中，可以不使用extract-text-webpack-plugin，只要用css-loader和style-loader就可以了。
#### html-webpack-plugin
html-webpack-plugin，是用来生产html的，其中filename是生产的文件路径和名称，template是使用的模板，inject是指将js放在body还是head里。为true会将js放到body里

```js
new HtmlWebpackPlugin({
    filename: 'index.html',
    template: 'index.html',
    inject: true
  }),
```

#### uglifyJSPlugin
uglifyJSPlugin是将代码进行压缩的。
```js
new webpack.optimize.UglifyJsPlugin({
  compress: {
    warnings: false
  }
})
```

#### CommonsChunkPlugin
CommonsChunkPlugin是将多个入口文件之间共享的块打包成一个独立的js文件。至此，你只需要在每个页面都加载这个公共代码的js文件，就可以既保持代码的完整性，又不会重复下载公共代码了。
```js
new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    filename: '[name].[chunkhash].js',
    minChunks: 4
  })
```

* `name`，给这个包含公共代码的chunk命个名（唯一标识）。
* `filename`，如何命名打包后生产的js文件。
* `minChunks`，公共代码的判断标准：某个js模块被多少个chunk加载了才算是公共代码。
* `chunks`，表示需要在哪些chunk（也可以理解为webpack配置中entry的每一项）里寻找公共代码进行打包。不设置此参数则默认提取范围为所有的chunk。

### resolve
```js
resolve: {
    extensions: ['', '.js', '.jsx', '.json'],
    alias: {
      'src': path.resolve(__dirname, './src'),
      'assets': path.resolve(__dirname, './src/assets'),
      'components': path.resolve(__dirname, './src/components')
    }
  }
```
`esolve.extensions`是对模块后缀名的简写，配置后，原本是`require('./components/app.jsx')` 可以简写为`require('./components/app')`

`resolve.alias`是别名，配置后，比如原本是`require('./src/components/nav.jsx')`可以简写为`require('components/nav.jsx')`。

### Webpack中的hash与chunkhash
#### hash与chunkhash
按照官方的定义`hash`就是webpack的每一次编译(compilation)所产生的hash值，`chunkhash`从字面上理解就是每一个chunk的hash值。那么什么时候会产生编译以及chunk又是什么东西？

#### compilation
`compilation`对象代表某个版本的资源对应的编译进程。当使用Webpack的development中间件时，每次检测到项目文件有改动就会创建一个compilation，进而能够针对改动生产全新的编译文件。以及在每次执行`webpack`命令时都会创建一个compilation。也就是说当创建了一个compilation，我们所有需要打包的文件(js,css,img,font等)都会产生相同的hash。
如果在项目中我们使用hash作为编译输出文件的hash的话，像这样：
```js
entry: {
    home: './src/home.js',
    profile: './src/profile.js'
},
output: {
    path: './dist',
    filename: 'js/[name].[hash].js'
}
```
那么在编译后所有的文件名都会使用相同的hash值，这样带来的问题是，上面两个js文件任何一个改动都会影响另外文件的最终文件名。上线后，另外文件的浏览器缓存也全部失效。这肯定不是我们想要的结果。

那么如何避免这样的问题呢？

答案就是使用`chunkhash`。
按照上面所说，`chunkhash`是每一个`chunk`的`hash`值，`chunk`就是模块(webpack中一切皆模块)，`chunkhash`也就是根据模块内容计算出的`hash`值。所以某个文件的改动只会影响它本身的`hash`值，不会影响其他文件。
所以可以将上面的filename改为：
```js
filename: 'js/[name].[chunkhash].js'
```
这样的话每个文件的hash值都不相同，上线后无改动的文件不会失去缓存。
不过使用`chunkhash`也不能解决所有问题，比如打包css文件。

#### js与css共用相同chunkhash的解决方案
前文提到了webpack的编译理念，webpack将style视为js的一部分，所以在计算`chunkhash`时，会把所有的js代码和style代码混合在一起计算。所以，不论是修改了js代码还是css代码，整个chunk的内容都改变了，计算所得的chunkhash自然就一样了。

那么如何解决这种问题呢？
#### contenthash
webpack默认将js/style文件统统编译到一个js文件中，可以借助`extract-text-webpack-plugin`将style文件单独编译输出。所以我们可以这样配置:

```js
new ExtractTextPlugin('./dist/css/[name].[contenthash].css')
```
`contenthash`代表的是文本文件内容的hash值，也就是只有style文件的hash值。这样编译输出的js和css文件将会有其独立的hash值。










