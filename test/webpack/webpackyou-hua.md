# webpack优化

## 开发工具心得：如何 10 倍提高你的 Webpack 构建效率

![](../../.gitbook/assets/image.png)

webpack 是个好东西，和 NPM 搭配起来使用管理模块实在非常方便。而 Babel 更是神一般的存在，让我们在这个浏览器尚未全面普及 ES6 语法的时代可以先一步体验到新的语法带来的便利和效率上的提升。在 React 项目架构中这两个东西基本成为了标配，但 commonjs 的模块必须在使用前经过 webpack 的构建\(后文称为 build\)才能在浏览器端使用，而每次修改也都需要重新构建（后文称为 rebuild）才能生效，如何提高 webpack 的构建效率成为了提高开发效率的关键之一。

### 1. Webpack 的构建流程

在开始正式的优化之前，让我们先回顾一下 Webpack 的构建流程，有哪些关键步骤，只有了解了这些，我们才能分析出哪些地方有优化的可能性。

![](../../.gitbook/assets/image %283%29.png)

> 图2：webpack is a module bundler.

首先，我们来看看官方对于 Webpack 的理念阐释，webapck 把所有的静态资源都看做是一个 module，通过 webpack，将这些 module 组成到一个 bundle 中去，从而实现在页面上引入一个 bundle.js，来实现所有静态资源的加载。所以详细一点看，webpack 应该是这样的：

![](../../.gitbook/assets/image %281%29.png)

> 图3：Every static asset should be able to be a module --webpack

通过 loader，webpack 可以把各种非原生 js 的静态资源转换成 JavaScript，所以理论上任何一种静态资源都可以成为一个 module。  
当然 webpack 还有很多其他好玩的特性，但不是本文的重点因此不铺开进行说明了。了解了上述的过程，我们就可以根据这些过程的前后处理进行对应的优化，接下来我们会针对 build 和 rebuild 的过程给与相应的意见。

### 2. RESOLVE

我们先从解析模块路径和分析依赖讲起，有人可能觉得这无所谓，但当项目应用依赖的模块越来越多，越来越重时，项目越来越大，文件和文件夹越来越多时，这个过程就变得越来越关乎性能。

#### 2.1 减小 Webpack 覆盖的范围

> build +, rebuild +

webpack 默认会去寻找所有 resolve.root 下的模块，但是有些目录我们是可以明确告知 webpack 不要管这里，从而减轻 webpack 的工作量。这时会用到 `module.noParse` 参数。

#### 2.2 Resolove.root VS Resolove.moduledirectories

> build +, rebuild +

`root` 和 `moduledirectories` 如果只从用法上来看，似乎是可以互相替代的。但因为 `moduledirectories` 从设计上是取相对路径，所以比起 `root` ，所以会多 parse 很多路径。

```javascript
resolve: {
    root: path.resolve('src/node_modules'),
    extensions: ['', '.js', '.jsx']
},
resolve: {
    modulesDirectories: ['node_modules', './src'],
    extensions: ['', '.js', '.jsx']
},
```

上面的配置，只会解析

```text
./src/node_modules/a
```

==== 此处有修改 2016/09/10 感谢 [@lili\_21](https://segmentfault.com/u/lili_21) ====

而下面的配置会解析

```javascript
/some/folder/structure/node_modules/a
/some/folder/structure/src/a
/some/folder/node_modules/a
/some/folder/src/a
/some/node_modules/a
/some/src/a
/node_modules/a
/src/a
```

大部分的情况下使用 `root` 即可，只有在有很复杂的路径下，才考虑使用 `moduledirectories`，这可以[明显提高 webpack 的构建性能](https://github.com/webpack/webpack/issues/1574#issuecomment-157520561)。这个 [issue](https://github.com/webpack/webpack/issues/472#issuecomment-55706013) 也很详细地讨论了这个问题。

### 3. LOADERS

webpack 官方和社区为我们提供了各种各样 loader 来处理各种类型的文件，这些 loader 的配置也直接影响了构建的性能。

#### 3.1 Babel-loader: 能者少劳

> build ++, rebuild ++

以 babel-loader 为例，我们在开发 React 项目时很可能会使用到了 ES6 或者 jsx 的语法，因此使用到 babel-loader 的情况很多，最简单的情况下我们可以这样配置，让所有的 js/jsx 通过 babel-loader：

```javascript
module: {
    loaders: [
      {
          test: /\.js(x)*$/,
          loader: 'babel-loader',
          query: {
              presets: ['react', 'es2015-ie', 'stage-1']
          }
      }
    ]
}
```

上面这样的做法当然是 ok 的，但是对于很多的 npm 包来说，他们完全没有经过 babel 的必要（成熟的 npm 包会在发布前将自己 es5，甚至 es3 化），让这些包通过 babel 会带来巨大的性能负担，毕竟 babel6 要经过几十个插件的处理，虽然 babel-loader 强大，但能者多劳的这种保守的想法却使得 babel-loader 成为了整个构建的性能瓶颈。所以我们可以使用 `exclude`，大胆地屏蔽掉 npm 里的包，从而使整包的构建效率飞速提高。

```javascript
module: {
    loaders: [
      {
          test: /\.js(x)*$/,
          loader: 'babel-loader',
          exclude: function(path) {
              // 路径中含有 node_modules 的就不去解析。
              var isNpmModule = !!path.match(/node_modules/);
              return isNpmModule;
          },
          query: {
              presets: ['react', 'es2015-ie', 'stage-1']
          }
      }
    ]
}
```

甚至，在我们十分确信的情况下，使用 include 来限定 babel 的使用范围，进一步提高效率。

```javascript
var path = require('path');
module.exports = {
    module: {
        loaders: [
          {
              test: /\.js(x)*$/,
              loader: 'babel-loader',
              include: [
                // 只去解析运行目录下的 src 和 demo 文件夹
                path.join(process.cwd(), './src'),
                path.join(process.cwd(), './demo')
              ],
              query: {
                  presets: ['react', 'es2015-ie', 'stage-1']
              }
          }
        ]
    }
}
```

### 4. PLUGINS

webpack 官方和社区为我们提供了很多方便的插件，有些插件为我们开发和生产带来了很多的便利，但是不合适地使用插件也会拖慢 webpack 的构建效率，而有些插件虽然不会为我们的开发上直接提供便利，但使用他们却可以帮助我们提高 webpack 的构建效率，这也是本文会提到的。

#### 4.1 SourceMaps

> build +

SourceMaps 是一个非常实用的功能，可以让我们在 chrome debug 时可以不用直接看已经 bundle 过的 js，而是直接在源代码上进行查看和调试，但完美的 SourceMaps 是很慢的，webpack 官方提供了七种 sourceMap 模式共大家选择，性能对比如下：

| devtool | build speed | rebuild speed | production supported | quality |
| :--- | :--- | :--- | :--- | :--- |
| eval | +++ | +++ | no | generated code |
| cheap-eval-source-map | + | ++ | no | transformed code \(lines only\) |
| cheap-source-map | + | o | yes | transformed code \(lines only\) |
| cheap-module-eval-source-map | o | ++ | no | original source \(lines only\) |
| cheap-module-source-map | o | - | yes | original source \(lines only\) |
| eval-source-map | -- | + | no | original source |
| source-map | -- | -- | yes | original source |

具体各自的区别请参考 [https://github.com/webpack/do...](https://github.com/webpack/docs/wiki/configuration#devtool) ，我们这里推荐使用 cheap-source-map，也就是去掉了column mapping 和 loader-sourceMap（例如 jsx to js） 的 sourceMap，虽然带上 `eval` 参数的可以快更多，但是这种 sourceMap 只能看，不能调试，得不偿失。

#### 4.2 OPTIMIZATION

> build ++，rebuild ++

webpack 提供了一些可以优化浏览器端性能的优化插件，如UglifyJsPlugin，OccurrenceOrderPlugin 和 DedupePlugin，都很实用，也都在消耗构建性能（UglifyJsPlugin 非常耗性能），如果你是在开发环境下，这些插件最好都不要使用，毕竟脚本大一些，跑的慢一些这些比起每次构建要耗费更多时间来说，显然还是后者更会消磨开发者的耐心，因此，只在正产环境中使用 OPTIMIZATION。

#### 4.3 CommonsChunk

> rebuild +

当你的 webpack 构建任务中有多个入口文件，而这些文件都 require 了相同的模块，如果你不做任何事情，webpack 会为每个入口文件引入一份相同的模块，显然这样做，会使得相同模块变化时，所有引入的 entry 都需要一次 rebuild，造成了性能的浪费，CommonsChunkPlugin 可以将相同的模块提取出来单独打包，进而减小 rebuild 时的性能消耗。这里有一篇很通俗易懂的使用方法：[http://webpack.toobug.net/zh-...](http://webpack.toobug.net/zh-cn/chapter3/common-chunks-plugin.html) ，感兴趣的朋友不妨一试。

#### 4.4 DLL & DllReference

> build +++, rebuild +++

除了正在开发的源代码之外，通常还会引入很多第三方 NPM 包，这些包我们不会进行修改，但是仍然需要在每次 build 的过程中消耗构建性能，那有没有什么办法可以减少这些消耗呢？DLLPlugin 就是一个解决方案，他通过前置这些依赖包的构建，来提高真正的 build 和 rebuild 的构建效率。  
鉴于现有的资料对于这两个插件的解释都不是很清楚，笔者这里翻译了一篇[日本同学的文章](http://qiita.com/pirosikick/items/c77db84dbed4c447a6fe#dllバンドルとは)，通过一个简单的例子来说明一下这两个插件的用法。我们举例，把 react 和 react-dom 打包成为 dll bundle。  
首先，我们来写一个 [DLLPlugin](https://github.com/webpack/docs/wiki/list-of-plugins#dllplugin) 的 config 文件。

> webpack.dll.config.js

```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    vendor: ['react', 'react-dom']
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].dll.js',
    /**
     * output.library
     * 将会定义为 window.${output.library}
     * 在这次的例子中，将会定义为`window.vendor_library`
     */
    library: '[name]_library'
  },
  plugins: [
    new webpack.DllPlugin({
      /**
       * path
       * 定义 manifest 文件生成的位置
       * [name]的部分由entry的名字替换
       */
      path: path.join(__dirname, 'dist', '[name]-manifest.json'),
      /**
       * name
       * dll bundle 输出到那个全局变量上
       * 和 output.library 一样即可。 
       */
      name: '[name]_library'
    })
  ]
};
```

执行 webpack 后，就会在 dist 目录下生成 dll bundle 和对应的 manifest 文件

```bash
$ ./node_modules/.bin/webpack --config webpack.dll.config.js
Hash: 36187493b1d9a06b228d
Version: webpack 1.13.1
Time: 860ms
        Asset    Size  Chunks             Chunk Names
vendor.dll.js  699 kB       0  [emitted]  vendor
   [0] dll vendor 12 bytes {0} [built]
    + 167 hidden modules

$ ls dist
./                    vendor-manifest.json
../                   vendor.dll.js
```

manifest 文件的格式大致如下，由包含的 module 和对应的 id 的键值对构成。

```javascript
cat dist/vendor-manifest.json
{
  "name": "vendor_library",
  "content": {
    "./node_modules/react/react.js": 1,
    "./node_modules/react/lib/React.js": 2,
    "./node_modules/process/browser.js": 3,
    "./node_modules/object-assign/index.js": 4,
    "./node_modules/react/lib/ReactChildren.js": 5,
    "./node_modules/react/lib/PooledClass.js": 6,
    "./node_modules/fbjs/lib/invariant.js": 7,
...
```

好，接下来我们通过 [DLLReferencePlugin](https://github.com/webpack/docs/wiki/list-of-plugins#dllreferenceplugin) 来使用刚才生成的 DLL Bundle。

首先我们写一个只去 `require` react，并通过 `console.log` 吐出的 `index.js`。

```javascript
var React = require('react');
var ReactDOM = require('react-dom');
console.log("dll's React:", React);
console.log("dll's ReactDOM:", ReactDOM);
```

再写一个不参考 Dll Bundle 的普通 webpack config 文件。

> webpack.conf.js

```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    'dll-user': ['./index.js']
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].bundle.js'
  }
};
```

执行 webpack，会在 dist 下生成 dll-user.bundle.js，约 700K，耗时 801ms。

```bash
$ ./node_modules/.bin/webpack
Hash: d8cab39e58c13b9713a6
Version: webpack 1.13.1
Time: 801ms
             Asset    Size  Chunks             Chunk Names
dll-user.bundle.js  700 kB       0  [emitted]  dll-user
   [0] multi dll-user 28 bytes {0} [built]
   [1] ./index.js 145 bytes {0} [built]
    + 167 hidden modules
```

接下来，我们加入 [DLLReferencePlugin](https://github.com/webpack/docs/wiki/list-of-plugins#dllreferenceplugin)

> webpack.conf.js

```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    'dll-user': ['./index.js']
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].bundle.js'
  },
  // ----在这里追加----
  plugins: [
    new webpack.DllReferencePlugin({
      context: __dirname,
      /**
       * 在这里引入 manifest 文件
       */
      manifest: require('./dist/vendor-manifest.json')
    })
  ]
  // ----在这里追加----
};
```

```bash
./node_modules/.bin/webpack
Hash: 3bc7bf760779b4ca8523
Version: webpack 1.13.1
Time: 70ms
             Asset     Size  Chunks             Chunk Names
dll-user.bundle.js  2.01 kB       0  [emitted]  dll-user
   [0] multi dll-user 28 bytes {0} [built]
   [1] ./index.js 145 bytes {0} [built]
    + 3 hidden modules
```

结果是非常惊人的，只有2.01K，耗时 70 ms，无疑大大提高了 build 和 rebuild 的效率。实际放到页面上看下是否可行。

```markup
<body>
  <script src="dist/vendor.dll.js"></script>
  <script src="dist/dll-user.bundle.js"></script>
</body>
```

![](https://segmentfault.com/img/remote/1460000005770049)

因为 Dll bundle 在依赖安装完毕后就可以进行了，我们可以在第一次执行 dev server 前执行一次 dll bundle 的 webapck 任务。

**4.4.1 和 external 的比较**

有人会说，这个和 用 `webpack` 的 `externals` 配置把 require 的 module 指向全局变量有点像啊。

```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    'ex': ['./index.js']
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].bundle.js'
  },
  externals: {
    // require('react')はwindow.Reactを使う
    'react': 'React',
    // require('react-dom')はwindow.ReactDOMを使う
    'react-dom': 'ReactDOM'
  }
};
```

```markup
<body>
  <script src="dist/react.min.js"></script>
  <script src="dist/react-dom.min.js"></script>
  <script src="dist/ex.bundle.js"></script>
</body>
```

这里有两个主要的区别：

1. 像是 `react` 这种已经打好了生产包的使用 `externals` 很方便，但是也有很多 npm 包是没有提供的，这种情况下 `DLLBundle` 仍可以使用。
2. 如果只是引入 npm 包一部分的功能，比如 `require('react/lib/React')` 或者 `require('lodash/fp/extend')` ，这种情况下 `DLLBundle` 仍可以使用。
3. 当然如果只是引用了 `react` 这类的话，`externals` 因为配置简单所以也推荐使用。

#### 4.5 [HappyPack](https://github.com/amireh/happypack)

> build +, rebuild +

webpack 的长时间构建搞的大家都很 unhappy。于是 @amireh 想到了一个点子，既然 loader 默认都是一个进程在跑，那是否可以让 loader 多进程去处理文件呢？

![](https://segmentfault.com/img/remote/1460000005770054)

happyPack 的文档写的很易懂，这里就不再赘述，happyPack 不仅利用了多进程，同时还利用缓存来使得 rebuild 更快。下面是插件作者给出的性能数据：

> For the main repository I tested on, which had around 3067 modules, the build time went down from 39 seconds to a whopping ~10 seconds when there was yet no

1. Successive builds now take between 6 and 7 seconds.

> Here's a rundown of the various states the build was performed in:

| Elapsed \(ms\) | Happy? | Cache enabled? | Cache present? | Using DLLs? |
| :--- | :--- | :--- | :--- | :--- |
| 39851 | NO | N/A | N/A | NO |
| 37393 | NO | N/A | N/A | YES |
| 14605 | YES | NO | N/A | NO |
| 13925 | YES | YES | NO | NO |
| 11877 | YES | YES | YES | NO |
| 9228 | YES | NO | N/A | YES |
| 9597 | YES | YES | NO | YES |
| 6975 | YES | YES | YES | YES |

> The builds above were run on Linux over a machine with 12 cores.

### 5. 其他

上面我们针对 webpack 的 resolve、loader 和 plugin 的过程给出了相应的优化意见，除了这些哪些优化点呢？其实有些优化贯穿在这个流程中，比如缓存和文件 IO。

#### 5.1 Cache

无论在何种性能优化中，缓存总是必不可少的一部分，毕竟每次变动都只影响很小的一部分，如果能够缓存住那些没有变动的部分，直接拿来使用，自然会事半功倍，在 webpack 的整个构建过程中，有多个地方提供了缓存的机会，如果我们打开了这些缓存，会大大加速我们的构建，尤其是 rebuild 的效率。

**5.1.1 webpack.cache**

> rebuild +

webpack 自身就有 cache 的配置，并且在 watch 模式下自动开启，虽然效果不是最明显的，但却对所有的 module 都有效。

**5.1.2 babel-loader.cacheDirectory**

> rebuild ++

babel-loader 可以利用系统的临时文件夹缓存经过 babel 处理好的模块，对于 rebuild js 有着非常大的性能提升。

**5.1.3 HappyPack.cache**

> build +, rebuild +

上面提到的 happyPack 插件也同样提供了 cache 功能，默认是以 `.happypack/cache--[id].json` 的路径进行缓存。因为是缓存在当前目录下，所以他也可以辅助下次 build 时的效率。

#### 5.2 FileSystem

默认的情况下，构建好的目录一定要输出到某个目录下面才能使用，但 webpack 提供了一种很棒的读写机制，使得我们可以直接在内存中进行读写，从而极大地提高 IO 的效率，开启的方法也很简单。

```javascript
var MemoryFS = require("memory-fs");
var webpack = require("webpack");

var fs = new MemoryFS();
var compiler = webpack({ ... });
compiler.outputFileSystem = fs;
compiler.run(function(err, stats) {
  // ...
  var fileContent = fs.readFileSync("...");
});
```

当然，我们还可以通过 webpackDevMiddleware 更加无缝地就接入到 dev server 中，例如我们以 express 作为静态 server 的例子。

```javascript
var compiler = webpack(webpackCfg);

var webpackDevMiddlewareInstance = webpackDevMiddleware(compiler, {
   // webpackDevMiddleware 默认使用了 memory-fs
   publicPath: '/dist',
   aggregateTimeout: 300, // wait so long for more changes
   poll: true, // use polling instead of native watchers
   stats: {
       chunks: false
   }
});

var app = express();
app.use(webpackDevMiddlewareInstance);
app.listen(xxxx, function(err) {
   console.log(colors.info("dev server start: listening at " + xxxx));
   if (err) {
     console.error(err);
   }
}
```



