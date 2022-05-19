## Node

- Node 可以理解成 ECMAScript + 内置的模块组成的，引用第三方模块， npm系统 node package manager。

- node是基于js的，前端来写服务端通过node是最方便的，node的性能问题 内部采用的多线程。
- node中主线程还是单线程的 （文件读写，内部还是基于多线程的）， 非阻塞 异步I/O.基于事件驱动

- node中没有锁的概念
- node的应用场景 处理 i/o (文件读写)密集型，不适合处理cpu密集型 (压缩加密 和计算相关的)
- node中处理多个异步操作同一个文件件的场景 （靠队列实现）
- 可以解析js语法
  - 服务端渲染vue，react  
  - 中间层 （跨域） 
  - 前端会用node来实现很多工具 
  - 可以做后端  mongo mysql

传统的语言是什么样子的？ 多线程同步

- 默认多个线程访问同一个资源 需要加锁 
- 在高并发场景下 ，多线程需要不停的切换时间片   “并发” 、“并行”
- 并行的概念 多线程  在不同的cpu下执行操作。 采用多进程的方式，让系统更加稳定，而且可以实现高并发

## Process | Global |Commander | Debug

​	服务端全局变量原则是global， 但是node在执行的时候为了实现模块化，会在执行代码时，外部包装一个函数，这个函数在执行的时候会改变this指向。可以直接访问这些变量。后面的五个属性都可以在文件中直接访问，但是不能通过global来获取

- **process**
  - platform  或如平台 windows -> win32 ,mac => darwin
  - cwd  current working directory (你在哪执行的这个文件)
  - env 环境变量，默认我们代码在执行的时候回去读取电脑中的环境
  - argv  执行命令时所带的参数 1.代表的是可执行node.exe 2.执行的是哪个文件
  
- **commander ** 命令行管家，第三方模块用的时候需要下载
 ```javascript
  const { program } = require('commander');
  program.version('0.0.1');
  program
      .option('-d, --debug', 'output extra debugging')
      .option('-s, --small', 'small pizza size')
      .option('-p, --pizza-type <type>', 'flavour of pizza')
      .command('run').action(() => {
          console.log('run')
      });
  
  program.parse(process.argv);
  const options = program.opts();
  console.log(options)
 ```

- **__filename**

- **__dirname**

- **exports**

- **module**

- **require**

## Node常用API

- **fs**
- **path**
- **vm**
- **Buffer**
- **event**
- **http**



## node中的模块 

- 1.内置模块 、核心模块  2.第三方模块 commander/co  3.文件模块(有相对路径 、 绝对路径)。模块遵循 commonjs 规范  es6Module规范。
- es6Module “静态”导入, 在编译的时候就可以知道使用了哪些变量, 可以实现tree-shaking  (都是发送http请求)
- commonjs 模块 “动态导入”, commonjs 模块是不支持tree-shaking  (读文件 同步操作)

- commonjs 模块定义了自己的规范，按照规范来使用就可以

  - 如果想使用哪个模块 就require谁 (后缀可以省略 默认会查找.js 文件， 没有js找.json)

  - 如果这个模块需要被别人使用，需要导出具体的内容  module.exports 更改这个变量导出的结果

  - 在node中每个文件js，json文件都是一个模块

  - 一个包中含有多个模块 （每个包都必须配置一个package.json 文件）
