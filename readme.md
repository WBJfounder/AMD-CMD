##AMD 与 RequireJS
###AMD
 Asynchronous Module Definition，用白话文讲就是 异步模块定义，异步是再也熟悉不过的词了，所有的模块将被异步加载，模块加载不影响后面语句运行。所有依赖某些模块的语句均放置在回调函数中。

 AMD规范定义了一个自由变量或者说是全局变量 define 的函数。

 define( id?, dependencies?, factory ); 

 第一个参数 id 为字符串类型，表示了模块标识，为可选参数。若不存在则模块标识应该默认定义为在加载器中被请求脚本的标识。如果存在，那么模块标识必须为顶层的或者一个绝对的标识。

 第二个参数，dependencies ，是一个当前模块依赖的，已被模块定义的模块标识的数组字面量。

 第三个参数，factory，是一个需要进行实例化的函数或者一个对象。

 创建模块标识为 alpha 的模块，依赖于 require， export，和标识为 beta 的模块
<script>
define("alpha", [ "require", "exports", "beta" ], function( require, exports, beta ){
    export.verb = function(){
        return beta.verb();
        // or:
        return require("beta").verb();
    }
});
</script>
一个返回对象字面量的异步模块
<script>
define(["alpha"], function( alpha ){
    return {
        verb : function(){
            return alpha.verb() + 1 ;
        }
    }
});
</script>
无依赖模块可以直接使用对象字面量来定义
<script>
define( function( require, exports, module){
    var a = require('a'),
          b = require('b');

    exports.action = function(){};
} );
</script>
