#### **关于H5的pushState、replaceState**

##### 1. pushState

###### 说明

浏览器不会向服务端请求数据，直接改变url地址，可以类似的理解为变相版的hash；但不像hash一样，浏览器会记录pushState的历史记录，可以使用浏览器的前进、后退功能作用

###### 使用方法

pushState(state, title, url)

###### 参数说明

state: 可以通过history.state读取
 title: 可选参数，暂时没有用，建议传个短标题
 url: 改变过后的url地址、

##### 2. replaceState

###### 说明

不同于pushState，replaceState仅仅是修改了对应的历史记录

##### 3. popstate

###### 说明

当用户在浏览器点击进行后退、前进，或者在js中调用`histroy.back()`，`history.go()`，`history.forward()`等，会触发popstate事件；但pushState、replaceState不会触发这个事件。

