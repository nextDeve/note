#### **模拟call**

- 第一个参数为`null`或者`undefined`时，`this`指向全局对象`window`，值为原始值的指向该原始值的自动包装对象，如 `String`、`Number`、`Boolean`
- 为了避免函数名与上下文(`context`)的属性发生冲突，使用`Symbol`类型作为唯一值
- 将函数作为传入的上下文(`context`)属性执行
- 函数执行完成后删除该属性
- 返回执行结果

```javascript
Function.prototype.myCall = function(context, ...args) {
    context =  (context ?? window) || new Object(context)
    const key = Symbol()
    context[key] = this
    const result = context[key](...args)
    delete context[key]
    return result
}
function myfunc1(){
    this.name = 'Lee';
    this.myTxt = function(txt) {
        console.log( 'i am',txt );
    }
} 
function myfunc2(){
    myfunc1.myCall(this);
}
var myfunc3 = new myfunc2();
myfunc3.myTxt('Geing'); // i am Geing
console.log (myfunc3.name);	// Lee
```

注：ES2020新特性,`Null`判断符 `??`