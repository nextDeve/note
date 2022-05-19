#### **模拟call**

- 第一个参数为`null`或者`undefined`时，`this`指向全局对象`window`，值为原始值的指向该原始值的自动包装对象，如 `String`、`Number`、`Boolean`
- 为了避免函数名与上下文(`context`)的属性发生冲突，使用`Symbol`类型作为唯一值
- 将函数作为传入的上下文(`context`)属性执行
- 函数执行完成后删除该属性
- 返回执行结果

```javascript
Function.prototype.myCall = function (context, ...args) {
    context = (context ?? window) || new Object(context)
    const key = Symbol()
    context[key] = this
    const result = context[key](...args)
    delete context[key]
    return result
}
function myfunc1() {
    this.name = 'Lee';
    this.myTxt = function (txt) {
        console.log('i am', txt);
    }
}
function myfunc2() {
    myfunc1.myCall(this);
}
var myfunc2Instance = new myfunc2();
console.log(myfunc2Instance);
myfunc2Instance.myTxt('Geing'); // i am Geing
console.log(myfunc2Instance.name);	// Lee
```

注：ES2020新特性,`Null`判断符 `??`

#### **模拟apply**

- 前部分与`call`一样
- 第二个参数可以不传，但类型必须为数组或者类数组

```javascript
Function.prototype.myApply = function (context) {
    context = (context ?? window) || new Object(context);
    const key = Symbol();
    const args = arguments[1];
    context[key] = this;
    let result;
    if (args) {
        result = context[key](...args);
    } else {
        result = context[key]();
    }
    return result;
}
function myfunc1() {
    this.name = 'Lee';
    this.myTxt = function (txt) {
        console.log('i am', txt);
    }
}
function myfunc2() {
    myfunc1.myApply(this);
}
var myfunc2Instance = new myfunc2();
console.log(myfunc2Instance);
myfunc2Instance.myTxt('Geing'); // i am Geing
console.log(myfunc2Instance.name);	// Lee
```

注：代码实现存在缺陷，当第二个参数为类数组时，未作判断

#### 沙盒模型

##### 1.代理沙盒

```javascript
window = global
function proxySanbox() {
    const originalWindow = window;
    this.fackWindow = {}
    this.proxy = new Proxy(this.fackWindow, {
        set(target, p, value) {
            target[p] = value;
            return true;
        },
        get(target, p) {
            return target[p] || originalWindow[p];
        }
    })
}
const sanbox1 = new proxySanbox();
const sanbox2 = new proxySanbox();
window.a = 'a in window';
console.log(window.a);//a in window
((window) => {
    window.a = 'a in sanbox1';
    console.log(window.a);//a in sanbox1
})(sanbox1.proxy);
((window) => {
    window.a = 'a in sanbox2';
    console.log(window.a);//a in sanbox2
})(sanbox2.proxy);
console.log(window.a);//a in window
```

##### 2.快照沙盒

```javascript
window = global
class snapshotSanbox {
    constructor() {
        this.snapshot = {}
        this.modifyProps = {};
    }
    active() {
        for (const prop in window) {
            if (window.hasOwnProperty(prop)) {
                this.snapshot[prop] = window[prop];
            }
        }
        Object.keys(this.modifyProps).forEach((prop) => {
            window[prop] = this.modifyProps[prop]
        })
    }
    inActive() {
        for (const prop in window) {
            if (window.hasOwnProperty(prop)) {
                if (window[prop] !== this.snapshot[prop]) {
                    this.modifyProps[prop] = window[prop];
                    window[prop] = this.snapshot[prop];
                }
            }
        }
    }
}
const sanbox1 = new snapshotSanbox();
const sanbox2 = new snapshotSanbox();
window.a = 'a in window';
console.log(window.a); //a in window
sanbox1.active()
window.a = 'a in sanbox1';
console.log(window.a);//a in sanbox1
sanbox1.inActive()
console.log(window.a);//a in window
sanbox2.active()
window.a = 'a in sanbox2';
console.log(window.a);//a in sanbox2
sanbox2.inActive()
console.log(window.a);//a in window
sanbox1.active()
console.log(window.a);//a in sanbox1
sanbox2.active()
console.log(window.a);//a in sanbox2
sanbox2.inActive()
console.log(window.a);//a in sanbox1
sanbox1.inActive()
console.log(window.a);//a in window
```

