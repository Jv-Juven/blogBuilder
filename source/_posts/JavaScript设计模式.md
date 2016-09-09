---
title: JavaScript设计模式
date: 2016-09-08 11:54:41
tags: JavaScript
---
# 设计模式
----
## 工厂模式
> 简而言之，就是会有一个函数只做生成实例对象的事情，根据传入的参数决定生成的实例。

```javascript
// 工厂方法（代码有问题）
var factory = function () {
    var obj = {};
    var Constructor = Array.prototype.shift.call( arguments );
    obj.__proto__ = Constructor.prototype;
    // obj.__proto__ = typeof Constructor .prototype === "number" ? Object.prototype : Constructor.prototype;
    var ret = Constructor.call( obj, arguments );
    return ret;
    // return typeof ret === "object" ? ret : obj;
}
// 调用
var instance = factory( String, "factory" );
```

## 观察者模式
> 又叫订阅——发布模式

```js
// 定义一个观察者模式的函数
var caller = function () {
    var listen,
        trigger;
    var obj = {};
    var _this = this;
    // 定义监听
    listen = function ( key, fn ) {
        var ret = obj[key] != null ? obj[key] : obj[key] = [];
        return Array.prototype.push.call( obj[key], fn );
    }
    // 定义触发
    trigger = function () {
        var _len, fn, key, ret;
        key = Array.prototype.shift.call( arguments );
        ret = obj[key] != null ? obj[key] : obj[key] = [];
        for ( var i = 0, _len = obj[key].length; i < _len; i ++ ) {
            fn = ret[i];
            if ( fn.apply( _this, arguments ) === false ) {
                return false;
            }
        }
    }
    return {
        listen: listen,
        trigger: trigger
    }
}
var eat = caller();
eat.listen("money", function( price ) {
    if ( parseInt(price) >= 0 ) {
        console.log("我有钱了，");
    }
});
eat.listen("money", function( price ) {
    if ( parseInt(price) >= 0 ) {
        console.log("有饭吃！");
    }
});
eat.listen("fan", function ( food ) {
    console.log("正在吃" + food);
});
eat.trigger( "money", 100 );
```
这里附上整理一个例子的代码：
```js
// 修改 AlloyTeam 的例子
Events = function() {
    var listen, log, obj, one, remove, trigger, __this;
    obj = {}; // 事件列表对象，每个属性都是一个执行的函数数组
    __this = this;
    listen = function( key, eventfn ) {  // key是事件句柄，eventfn是事件出发的处理函数数组的元素
        var stack, _ref;  // stack是盒子
        stack = ( _ref = obj[key] ) != null ? _ref : obj[ key ] = [];
        return stack.push( eventfn );
    };
    one = function( key, eventfn ) {
        remove( key );
        return listen( key, eventfn );
    };
    remove = function( key ) {
        var _ref;
        return ( _ref = obj[key] ) != null ? _ref.length = 0 : void 0;
    };
    // 触发事件所对应的所有函数
    trigger = function() {  //面试官打电话通知面试者
        var fn, stack, _i, _len, _ref, key;
        key = Array.prototype.shift.call( arguments );
        stack = ( _ref = obj[ key ] ) != null ? _ref : obj[ key ] = [];
        for ( _i = 0, _len = stack.length; _i < _len; _i++ ) {
            fn = stack[ _i ];
            if ( fn.apply( __this,  arguments ) === false) {
                return false;
            }
        }
    }
    return {
        listen: listen,
        one: one,
        remove: remove,
        trigger: trigger
    }
}
```

## 策略模式
> ##### 所有人以相同方式，做不同的事
> 将相同的方式抽象成一个类，将不同的事分别定义为不同的对象

```js
// 执行验证，此处为演示
var Validator = function () {
    var __this = this;
    // __this.data = Array.prototype.shift.call(arguments);
    // __this.rules = Array.prototype.shift.call(arguments);
    for (attr in __this.data) {
        var value = __this.data[attr];
        var rule = __this.dataValidate[attr].rule;
        if (!__this.rules[rule](value)) {
            console.log(__this.dataValidate[attr].message);
        } else {
            console.info(attr + "通过验证");
        }
    }
}
// 需要验证的数据（可以写成一个实例的对象）
Validator.prototype.data = {
    name: "李四",
    age: 29,
    idCard: 331021198609289103,
    comments: "刚入住的酒店，感觉不错，环境干净整洁。但是，FUCK，服务员态度太差了。",
}
// 验证规则（可以写成一个实例的对象）
Validator.prototype.rules = {
    // 字符串不为空
    noNull: function (value) {
        if (!!value) {
            return value.length === 0 ? false : true;
        } else {
            return false;
        }
    },
    isAge: function (value) {
        value = parseInt(value);
        if (isNaN(value)) {
            return false;
        } else {
            return true
        }
    },
    idCard: function (value) {
        // 待完善...
        return false;
    },
}
// 字段验证信息（可以写成一个实例的对象）
Validator.prototype.dataValidate = {
    name: {
        rule: "noNull",
        message: "姓名不能为空",
    },
    age: {
        rule: "isAge",
        message: "请输入年龄",
    },
    idCard: {
        rule: "idCard",
        message: "请输入正确的身份证号码",
    },
    comments: {
        rule: "noNull",
        message: "评论不能为空",
    },
}
// 实例化验证类
new Validator();
```
或者我们可以这样改写：
```js
var validate; // 对象变量
var params = {}
// 执行验证，此处为演示
var Validator = function ( params ) {
    var __this = this;
    // 将不同的部分赋值
    __this.data = params.data;
    __this.rules = params.rules;
    __this.dataValidate = params.dataValidate;
    for (attr in __this.data) {
        var value = __this.data[attr];
        var rule = __this.dataValidate[attr].rule;
        if (!__this.rules[rule](value)) {
            console.log(__this.dataValidate[attr].message);
        } else {
            console.info(attr + "通过验证");
        }
    }
}
// 需要验证的数据（可以写成一个实例的对象）
params.data = {
    name: "李四",
    age: 29,
    idCard: 331021198609289103,
    comments: "刚入住的酒店，感觉不错，环境干净整洁。但是，FUCK，服务员态度太差了。",
}
// 验证规则（可以写成一个实例的对象）
params.rules = {
    // 字符串不为空
    noNull: function (value) {
        if (!!value) {
            return value.length === 0 ? false : true;
        } else {
            return false;
        }
    },
    isAge: function (value) {
        value = parseInt(value);
        if (isNaN(value)) {
            return false;
        } else {
            return true
        }
    },
    idCard: function (value) {
        // 待完善...
        return false;
    },
}
// 字段验证信息（可以写成一个实例的对象）
params.dataValidate = {
    name: {
        rule: "noNull",
        message: "姓名不能为空",
    },
    age: {
        rule: "isAge",
        message: "请输入年龄",
    },
    idCard: {
        rule: "idCard",
        message: "请输入正确的身份证号码",
    },
    comments: {
        rule: "noNull",
        message: "评论不能为空",
    },
}
// 实例化验证类
validate = new Validator(params);
```

## 适配器模式
> 理解同 _适配器_ ，就是将写法不同功能大体一致的两份js代码通过 _适配器_ 连接起来

下面展示一个例子，jQuery与其他写法的适配：

```js
var $id = function (id) {
    return jQuery( "#" + id )[0];
}
```

## 模板方法模式
> 主要是关于父类与子类之间的继承关系，通常使用原型实现父类与子类之间的继承。

> __注意__ `模板方法模式` 主要是类的复制问题
