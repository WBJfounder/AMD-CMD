## AMD 与 RequireJS
### AMD 
 Asynchronous Module Definition，用白话文讲就是 异步模块定义，异步是再也熟悉不过的词了，所有的模块将被异步加载，模块加载不影响后面语句运行。所有依赖某些模块的语句均放置在回调函数中。

 AMD规范定义了一个自由变量或者说是全局变量 define 的函数。

 define( id?, dependencies?, factory ); 

 第一个参数 id 为字符串类型，表示了模块标识，为可选参数。若不存在则模块标识应该默认定义为在加载器中被请求脚本的标识。如果存在，那么模块标识必须为顶层的或者一个绝对的标识。

 第二个参数，dependencies ，是一个当前模块依赖的，已被模块定义的模块标识的数组字面量。

 第三个参数，factory，是一个需要进行实例化的函数或者一个对象。

 创建模块标识为 alpha 的模块，依赖于 require， export，和标识为 beta 的模块
<pre>
define("alpha", [ "require", "exports", "beta" ], function( require, exports, beta ){
    export.verb = function(){
        return beta.verb();
        // or:
        return require("beta").verb();
    }
});
</pre>
一个返回对象字面量的异步模块
<pre>
define(["alpha"], function( alpha ){
    return {
        verb : function(){
            return alpha.verb() + 1 ;
        }
    }
});
</pre>
无依赖模块可以直接使用对象字面量来定义
<pre>
define( function( require, exports, module){
    var a = require('a'),
          b = require('b');
    exports.action = function(){};
} );
</pre>
类似于CommonJS方式定义
<pre>
define( function( require, exports, module){
    var a = require('a'),
          b = require('b');

    exports.action = function(){};
} );
</pre>
 在 AMD 规范中的 require 函数与一般的 CommonJS中的 require 不同。由于动态检测依赖关系使加载异步，对于基于回调的 require 需求强烈。
##### 局部与全局的require
局部的require需要在AMD模式中的define工厂函数传入require
<pre>
 define(['require'],function(require){
     //...

 })
 or;
 define(function(require,export,module){
     //..
 })
</pre>
局部的require需要其他特定的API来实现。<br>
全局的require函数是惟一全局作用域下的变量，像define一样。全局require并不是规范要求的，但是如果实现全局的require函数，那么其需要具有与局部require函数一样的以下限定：

1.模块标识视为绝对的，而不是相对的对应另一个模块标识。

2.只有在异步情况下，require的回调方式才被用来作为交互操作使用。因为他不可能在同步的情况下通过require（string）从顶层加载模块。

依赖相关的API会开始模块加载。如果需要有互操作的多个加载器，那么全局的require应该被加载顶层模块来代替。
<pre>
require(String)
define( function( require ){
    var a = require('a'); // 加载模块a
} );

require(Array, Function)
define( function( require ){
require( ['a', 'b'], function( a,b ){ // 加载模块a b 使用
        // 依赖 a b 模块的运行代码
    } ); 
} );

require.toUrl( Url )
define( function( require ){
    var temp = require.toUrl('./temp/a.html'); // 加载页面
} );
</pre>
### RequireJS
官网 http://www.requirejs.org/<br>
API http://www.requirejs.org/docs/api.html<br>
 RequireJS 是一个前端的模块化管理的工具库，遵循AMD规范，它的作者就是AMD规范的创始人 James Burke。所以说RequireJS是对AMD规范的阐述一点也不为过。<br>
 RequireJS 的基本思想为：通过一个函数来将所有所需要的或者说所依赖的模块实现装载进来，然后返回一个新的函数（模块），我们所有的关于新模块的业务代码都在这个函数内部操作，其内部也可无限制的使用已经加载进来的以来的模块。
   <pre>
        <script data-main='script/main' src='script/require.js'></script>
   </pre> 
 那么scripts下的main.js则是指定的主代码脚本文件，所有的依赖模块代码文件都将从该文件开始异步加载进入执行。

defined用于定义模块，RequireJS要求每个模块均放在独立的文件之中。按照是否有依赖其他模块的情况分为独立模块和非独立模块。

1.独立模块，不依赖其他模块。直接定义：
<pre>
    define({
        method:function(){},
        method:function(){}
    })
</pre>  
也等价于
<pre>
    define(function(){
        return{
            method:function(){},
            method:function(){}
        }
    })
</pre> 
2.非独立模块，对其他模块有依赖。
<pre>
    define([ 'module1', 'module2' ], function(m1, m2){
    ...
    });
</pre>
或者：
<pre>
    define( function( require ){
    var m1 = require( 'module1' ),
        m2 = require( 'module2' );
    ...
    });
</pre>
  简单看了一下RequireJS的实现方式，其 require 实现只不过是将 function 字符串然后提取 require 之后的模块名，将其放入依赖关系之中。

  在require方法调用模块时，其参数与define类似
<pre>
    require( ['foo', 'bar'], function( foo, bar ){
    foo.func();
    bar.func();
    } );
</pre>
在加载 foo 与 bar 两个模块之后执行回调函数实现具体过程。

当然还可以如之前的例子中的，在define定义模块内部进行require调用模块

<pre>
    define( function( require ){
    var m1 = require( 'module1' ),
          m2 = require( 'module2' );
    ...
    });
</pre>
 define 和 require 这两个定义模块，调用模块的方法合称为AMD模式，定义模块清晰，不会污染全局变量，清楚的显示依赖关系。AMD模式可以用于浏览器环境并且允许非同步加载模块，也可以按需动态加载模块。

## CMD与seaJS
### CMD 
在CMD中，一个模块就是一个文件，格式为：

define(factory);

全局函数define，用来定义模块。

参数 factory  可以是一个函数，也可以为对象或者字符串。

当 factory 为对象、字符串时，表示模块的接口就是该对象、字符串。

定义JSON数据模块：
<pre>
    define({ "foo": "bar" });
</pre>
通过字符串定义模版模块：
<pre>
    define('this is {{data}}')
</pre>
 factory 为函数的时候，表示模块的构造方法，执行构造方法便可以得到模块向外提供的接口。
<pre>
    define( function(require, exports, module) { 
    // 模块代码
    });
</pre>
define( id?, deps?, factory );<br>
 define也可以接受两个以上的参数，字符串id为模块标识，数组deps为模块依赖：
<pre>
    define( 'module', ['module1', 'module2'], function( require, exports, module ){
    // 模块代码
} );
</pre>
其与AMD规范用法不同。
require是factory的第一个参数。<br>
require(id)<br>
接收模块标识作为唯一的参数，用来获取其他模块提供的接口：
<pre>
    define(function( require, exports ){
    var a = require('./a');
    a.doSomething();
    });
</pre>
require.async(id,callback?);<br>
require是同步往下执行的，需要的异步加载模块可以使用 require.async 来进行加载：
<pre>
    define( function(require, exports, module) { 
    require.async('.a', function(a){
        a.doSomething();
       });
    });
</pre>
require.resolve( id )<br>
可以使用模块内部的路径机制来返回模块路径，不会加载模块。<br>
export是factory的第二个参数，用来向外提供模块接口<br>
<pre>
    define(function( require, exports ){
    exports.foo = 'bar'; // 向外提供的属性
    exports.do = function(){}; // 向外提供的方法
    });
</pre>
当然也可以使用return直接向外提供接口<br>
<pre>
    define(function( require, exports ){
    return{
        foo : 'bar', // 向外提供的属性
        do : function(){} // 向外提供的方法
        }
    });
</pre>
也可以简化为直接对象字面量的形式:
<pre>
    define({
    foo : 'bar', // 向外提供的属性
    do : function(){} // 向外提供的方法
    });
</pre>
与nodeJS中一样需要注意的是，一下方式是错误的：
<pre>
    define(function( require, exports ){
    exports = {
        foo : 'bar', // 向外提供的属性
        do : function(){} // 向外提供的方法
    }
});
</pre>
需要这么做
<pre>
    define(function( require, exports, module ){
    module.exports = {
        foo : 'bar', // 向外提供的属性
        do : function(){} // 向外提供的方法
    }
});
</pre>
传入的对象引用可以添加属性，一旦赋值一个新的对象，那么值钱传递进来的对象引用就会失效了。开始之初，exports 是作为 module.exports 的一个引用存在，一切行为只有在这个引用上 factory 才得以正常运行，赋值新的对象后就会断开引用，exports就只是一个新的对象引用，对于factory来说毫无意义，就会出错。<br>
module 是factory的第三个参数，为一个对象，上面存储了一些与当前模块相关联的属性与方法。<br>
module.id 为模块的唯一标识。

module.uri 根据模块系统的路径解析规则得到模块的绝对路径。

module.dependencies 表示模块的依赖。

module.exports 当前模块对外提供的接口。

### seaJS
官网 http://seajs.org/docs/<br>
API快速参考 https://github.com/seajs/seajs/issues/266
<pre>
    sea.js 核心特征：
        1. 遵循CMD规范，与NodeJS般的书写模块代码。
        2. 依赖自动加载，配置清晰简洁。
    兼容 Chrome 3+，Firefox 2+，Safari 3.2+，Opera 10+，IE 5.5+。
</pre>
seajs.use

用来在页面中加载一个或者多个模块
<pre>
    // 加载一个模块 
    seajs.use('./a');
    // 加载模块，加载完成时执行回调
    seajs.use('./a'，function(a){
    a.doSomething();
});
    // 加载多个模块执行回调
    seajs.use(['./a','./b']，function(a , b){
    a.doSomething();
    b.doSomething();
});
</pre>
其define 与 require 使用方式基本就是CMD规范中的示例。
## AMD 与 CMD 区别到底在哪里？
看了以上 AMD，requireJS 与 CMD， seaJS的简单介绍会有点感觉模糊，总感觉较为相似。因为像 requireJS 其并不是只是纯粹的AMD固有思想，其也是有CMD规范的思想，只不过是推荐 AMD规范方式而已， seaJS也是一样。

    AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。

    CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。

    类似的还有 CommonJS Modules/2.0 规范，是 BravoJS 在推广过程中对模块定义的规范化产出还有不少??

    这些规范的目的都是为了 JavaScript 的模块化开发，特别是在浏览器端的。

    目前这些规范的实现都能达成浏览器端模块化开发的目的。

区别：
1. 对于依赖的模块，AMD 是提前执行，CMD 是延迟执行。不过 RequireJS 从 2.0 开始，也改成可以延迟执行（根据写法不同，处理方式不同）。CMD 推崇 as lazy as possible.

2. CMD 推崇依赖就近，AMD 推崇依赖前置。看代码：
<pre>
    // CMD
    define(function(require, exports, module) {
    var a = require('./a')
    a.doSomething()
    // 此处略去 100 行
    var b = require('./b') // 依赖可以就近书写
    b.doSomething()
    // ...
})

    // AMD 默认推荐的是
    define(['./a', './b'], function(a, b) { // 依赖必须一开始就写好
    a.doSomething()
    // 此处略去 100 行
    b.doSomething()
    // ...
})
</pre>    
虽然 AMD 也支持 CMD 的写法，同时还支持将 require 作为依赖项传递，但 RequireJS 的作者默认是最喜欢上面的写法，也是官方文档里默认的模块定义写法。

3. AMD 的 API 默认是一个当多个用，CMD 的 API 严格区分，推崇职责单一。比如 AMD 里，require 分全局 require 和局部 require，都叫 require。CMD 里，没有全局 require，而是根据模块系统的完备性，提供 seajs.use 来实现模块系统的加载启动。CMD 里，每个 API 都简单纯粹。

4. 还有一些细节差异，具体看这个规范的定义就好，就不多说了。

## 本文知识简单写了写requirejs和seajs需要详细研究的加我qq929364695（一些扩展资料链接我放到txt文档中可以看下）
## 点赞star啊











  
