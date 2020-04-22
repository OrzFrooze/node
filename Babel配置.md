### Babel入门

Babel是一个广泛使用的转码器，可以将ES6代码转换为ES5代码，从而在现有的环境执行。

#### 1. 配置文件

​		Babel配置文件是.babelrc，存放在项目根目录下，用来设置转码规则和插件。基本格式如下：

```json
{
    "presets":[],//设定转码规则
    "plugins":[]//配置插件
}
```

官方提供的规则

```bash
# ES2015转码规则
$ npm install --save-dev babel-preset-es2015

# react转码规则
$ npm install --save-dev babel-preset-react

# ES7不同阶段语法提案的转码规则（共有4个阶段），选装一个
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3
```

.babelrc规则配置实例

```json
{
    "presets":[
        "es2015",
        "react",
        "stage-2"
    ],
    "plugins":[]
}
```

#### 2. 命令行转码工具 babel-cli

​		官方提供babel-cli工具,用于命令行转码。

```bash
##安装方式
npm i babel-cli -g //全局安装时,项目开发依赖全局环境,也无法支持不同的项目使用不同的版本Babel
npm i babel-cli -D //项目中安装
##基本使用方法
# 转码结果输出到标准输出
$ babel example.js

# 转码结果写入一个文件
# --out-file 或 -o 参数指定输出文件
$ babel example.js --out-file compiled.js
# 或者
$ babel example.js -o compiled.js

# 整个目录转码
# --out-dir 或 -d 参数指定输出目录
$ babel src --out-dir lib
# 或者
$ babel src -d lib

# -s 参数生成source map文件
$ babel src -d lib -s
```

#### 3. babel-node

​		babel-cli工具自带一个babel-node命令,提供一个支持ES6的REPL环境。它支持Node的REPL环境的所有功能，而且可以直接运行ES6代码。

```bash
#执行babel-cli进入PEPL环境
$ babel-node
> 

#直接运行es6脚本
$ babel-node es6.js

#项目中使用
{
	scripts:{
		"dev":"babel-node es.js"
	}
}
```

####  4. babel-register

​		`babel-register`模块改写`require`命令，为它加上一个钩子。此后，每当使用`require`加载`.js`、`.jsx`、`.es`和`.es6`后缀名的文件，就会先用Babel进行转码。

> ```bash
> $ npm install --save-dev babel-register
> ```

使用时，必须首先加载`babel-register`。

> ```bash
> require("babel-register");
> require("./index.js");
> ```

然后，就不需要手动对`index.js`转码了。

需要注意的是，`babel-register`只会对`require`命令加载的文件转码，而不会对当前文件转码。另外，由于它是实时转码，所以只适合在开发环境使用。

#### 5. babel-core

​		如果某些代码需要调用Babel的API进行转码，就要使用`babel-core`模块。

安装命令如下。

> ```bash
> $ npm install babel-core --save
> ```

然后，在项目中就可以调用`babel-core`。

> ```javascript
> var babel = require('babel-core');
> 
> // 字符串转码
> babel.transform('code();', options);
> // => { code, map, ast }
> 
> // 文件转码（异步）
> babel.transformFile('filename.js', options, function(err, result) {
>   result; // => { code, map, ast }
> });
> 
> // 文件转码（同步）
> babel.transformFileSync('filename.js', options);
> // => { code, map, ast }
> 
> // Babel AST转码
> babel.transformFromAst(ast, code, options);
> // => { code, map, ast }
> ```

配置对象`options`，可以参看官方文档http://babeljs.io/docs/usage/options/。

下面是一个例子。

> ```javascript
> var es6Code = 'let x = n => n + 1';
> var es5Code = require('babel-core')
>   .transform(es6Code, {
>     presets: ['es2015']
>   })
>   .code;
> // '"use strict";\n\nvar x = function x(n) {\n  return n + 1;\n};'
> ```

上面代码中，`transform`方法的第一个参数是一个字符串，表示需要转换的ES6代码，第二个参数是转换的配置对象。

#### 6. babel-polyfill

​		Babel默认只转换新的JavaScript句法（syntax），而不转换新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法（比如`Object.assign`）都不会转码。

举例来说，ES6在`Array`对象上新增了`Array.from`方法。Babel就不会转码这个方法。如果想让这个方法运行，必须使用`babel-polyfill`，为当前环境提供一个垫片。

安装命令如下。

> ```bash
> $ npm install --save babel-polyfill
> ```

然后，在脚本头部，加入如下一行代码。

> ```javascript
> import 'babel-polyfill';
> // 或者
> require('babel-polyfill');
> ```

Babel默认不转码的API非常多，详细清单可以查看`babel-plugin-transform-runtime`模块的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)文件。

#### 7. 浏览器环境中使用方式



```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.4.4/babel.min.js"></script>
<script type="text/babel"> //注明类型
// Your ES6 code
</script>
```

注意，网页中实时将ES6代码转为ES5，对性能会有影响。生产环境需要加载已经转码完成的脚本。

下面是如何将代码打包成浏览器可以使用的脚本，以`Babel`配合`Browserify`为例。首先，安装`babelify`模块。

> ```bash
> $ npm install --save-dev babelify babel-preset-es2015
> ```

然后，再用命令行转换ES6脚本。

> ```bash
> $  browserify script.js -o bundle.js \
>   -t [ babelify --presets [ es2015 react ] ]
> ```

上面代码将ES6脚本`script.js`，转为`bundle.js`，浏览器直接加载后者就可以了。

在`package.json`设置下面的代码，就不用每次命令行都输入参数了。

> ```javascript
> {
>   "browserify": {
>     "transform": [["babelify", { "presets": ["es2015"] }]]
>   }
> }
> ```