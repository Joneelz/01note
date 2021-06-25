## 1、启动函数中

```js
__webpack_require__.m : 挂载所有的modules；
__webpack_require__.c : 挂载已缓存的modules；
__webpack_require__.d : 定义exports的getter；
__webpack_require__.r : 将module设置为es6模块；
__webpack_require__.t : 根据不同的场景返回对应处理后的模块或值；
__webpack_require__.n : 返回getter，内部区分是否为es6模块；
__webpack_require__.o : Object.prototype.hasOwnProperty功能封装；
__webpack_require__.p : output配置项中的publicPath属性；
```

# node

各种库总结：https://github.com/huaize2020/awesome-nodejs/blob/main/README-zh-CN.md

## 0、Node部分模块

- url 模块-----解析 url

  ```js
  const url = require("url"); //直接引入
  let obj = url.parse("https://locathost:3000?a=1&b=2", true); //true可将query分隔成对象
  console.log(obj);
  ```

- querystring 模块----解析 url 中参数

  ```js
  const querystring = require("querystring"); //直接引入
  let str = "wd=abcdd&rsv_spt=1";
  let obj = querystring.parse(str); //将参数解析成对象
  console.log(obj);
  ```

- uuid 模块----生成 uuid 号

  ```js
  const uuid = require("uuid/v4"); //需安装后引入
  console.log(uuid().replace(/\-/g, ""));
  ```

- dns 模块

  ```js
  const dns = require("dns");

  dns.lookup("www.baidu.com", (err, data) => {
    //根据域名查ip
    if (err) {
      console.log("错了");
    } else {
      console.log(data);
    }
  });

  let ip = "111.206.223.206"; //根据ip查域名
  dns.lookupService(ip, 80, (err, data) => {
    if (err) {
      console.log("错了", err);
    } else {
      console.log(data);
    }
  });
  ```

- assert 断言

  ```js
  const assert = require("assert");
  function calc(a, b) {
    //对参数校验，不符合要求抛出错误
    assert(
      typeof a == "number" && typeof b == "number",
      "除法中两个东西，都得是数字"
    );
    assert(b != 0, "分母不能是0");
    return a / b;
  }
  console.log(div("a c", 0));
  ```

## 1、process --当前进程

```js
process.cwd(); //current working directory 当前工作目录
process.chdir('..'); //切换到上级目录
process.execPath //可执行文件的绝对路径
process.platform //运行平台
process.argv //属性值为数组
process.env  //操作系统环境信息
process.title //窗口标题
process.arch  //处理器架构
process.kill(pid,[signall])
process.uptime() //当前程序的运行时间
/**
 V8引擎最大使用内存量是1.7个G
  rss: 20512768, 常驻内存
  heapTotal: 8388608, 堆内存的总申请量
  heapUsed: 4338520,//已经使用的量
  external: 8224 }  //外部内存的使用量，buffer的内存是单独分配的，属于external
 */
process.memoryUsage() //内存的使用信息
process.on('beforeExit', function () { //进程退出事件
    console.log('beforeExit');
    http.createServer((req, res) => {
        res, end('hello world');
    }).listen(8080);
});
```

### 1.1 spawn

`child_process.spawn(command[, args][, options])` ，是一个异步方法

- `command` 要运行的命令。
- `args` 字符串参数的列表。
- `detached` 使子进程独立于其父进程运行。 具体行为取决于平台

```js
//子进程文件1.test.js
for (let i = 0; i < process.argv.length; i++) {
    process.stdout.write('hello ' + process.argv[i] + '\r\n');
}
process.stdin.on('data', function (data) {
    process.stdout.write('子'+data);
});

//主进程文件
let { spawn } = require('child_process');
let path = require('path');
//pipe 在父进程和子进程之间建立一个管道
//如果放的是一个流，则意味着父进程和子进程共享 一个流
let p1 = spawn('node', ['test1.js', 'zfpx'], {
    cwd: path.join(__dirname, 'test1'), //子进程的工作目录。
    stdio: [process.stdin, process.stdout, 'pipe'] //子进程的stdio配置
    //若使用ipc通信，则可通过message接收子进程发送的消息
    // stdio: ['ipc', process.stdout, 'ignore']  
    });
let p2 = spawn('node', ['test2.js'], {
    cwd: path.join(__dirname, 'test1'),
    stdio: ['pipe', 'pipe', 'pipe']
});

p1.on('message', function (msg) { //使用ipc通信可通过该方法接收子进程消息
    console.log(msg);
});
//process.on('message', function (msg) { //使用ipc通信子进程可通过该方法接收父进程消息和发送
//     console.log(msg);
//     process.send('test3:' + msg);
// });

//指定了pipe,则意味着可以在父进程里得到 p1.stdout得到的子进程的标准输出
p1.stdout.on('data', function (data) {
    console.log(data.toString());
    p2.stdin.write(data);
});
//每个进程 都会有标准输入流标准输出流错误输出流当这些流关闭的时候会触发close事件
p1.on('close', function () {
    console.log('子进程1关闭');
});
//当这个进程退出的时候会触发exit事件
p1.on('exit', function () {
    console.log('子进程1退出');
});
p1.on('error', function (err) {
    console.log('子进程1开启失败' + err);
});
```

### 1.2 fork

`fork exec execFile 它们其实都是基于spawn的语法糖(改进方法)`

- 普通操作

  ```js
  //主进程文件
  let { spawn } = require('child_process');
  let child = fork('fork.js', ['joiner'], { //开启子进程执行fork文件
      cwd: __dirname,
      silent: true // 安静的，不共享子进程的输入输出流等
  })
  child.on('message', function (data) {
      console.log(data);
  });
  child.send({ name: 'joiner' });
  
  //子进程文件
  process.stdout.write('world' + process.argv[2]);
  process.on('message', function (msg) {
      process.send('子进程:' + JSON.stringify(msg));
  });
  ```

- 让子进程开一个server服务

```js
//主进程文件
//send方法其实可以有两个参数 ，第一个参数是任意类型 第二个参数只能是http server net server socket
let c = require('child_process');
let http = require('http');
let os = require('os');
let server = http.createServer(function (req, res) {
    res.setHeader('Content-Type', 'text/html;charset=utf8');
    res.end('请求在父进程被处理');
});

server.listen(8080);

for (let i = 0; i < os.cpus.length; i++) {
    let p1 = c.fork('server.js', [], {
        cwd: __dirname
    });
    p1.send('server', server);
}

//子进程文件
let http = require('http');
process.on('message', function (msg, server) {
    if (msg == 'server') {
        http.createServer(function (req, res) {
            res.setHeader('Content-Type', 'text/html;charset=utf8');
            res.end('请求子进程中被处理');
        }).listen(server);
    }
});
```

### 1.3 exec

可以启用一个用于运行某个命令的子进程并缓存子进程的输出结果，是一个同步方法

```js
//exec同步执行一个shell命令
let { exec } = require('child_process');
let path = require('path');
//用于使用 shell执行命令
let p1 = exec('node test1.js a b c ', { maxbuffer: 1024 * 1024, encoding: 'utf8', cwd: path.join(__dirname, 'test2') }, function (err, stdout, stderr) {
    console.log(err);
    console.log(stdout);
});
//其实会向子进程发射一个信号 SIGTERM,杀死子进程 p1
setTimeout(function () {
    p1.kill();
}, 3000);

//子进程文件中------------------------------
process.on('SIGTERM', function () {
    console.log(arguments);
});
```



### 1.4 execFile

可以使用execFile开启一个专门用于运行某个可执行文件的子进程，类似exec，但直接衍生命令，无需先衍生一个shell

```js
//exec同步执行一个shell命令
let { execFile } = require('child_process');
let path = require('path');
let p1 = execFile('node', ['test2.js', 'a', 'b', 'c'], { cwd: path.join(__dirname, 'test2') },function (err, stdout, stderr) {
        console.log(err);
        console.log(stdout);
    });

//子进程文件中-----------------------
for (let i = 0; i < process.argv.length; i++) {
    console.log(process.argv[i]);
}
```





## 2、setImmediate和process.nextTick

```js
/**
 * nextTick setImmediate 区别和联系
 * nextTick 把回调函数放在当前执行栈的底部
 * setImmediate 把回调函数放在事件队列的尾部
 */
function read(){
 setImmediate(function(){
   console.log(1);
   setImmediate(function(){
     console.log(2);
     process.nextTick(function(){
       console.log(0);
     })
     setImmediate(function(){
       console.log(3);
     })
   })
 })
}
read(); // 1 2 0 3
```

## 3、events模块

```js
let EventEmitter = require('events');
let util = require('util');
//这是一个类
function Bell() {
  EventEmitter.call(this);//继承私有属性
}
//进行原型继承 继承公用
//Object.setPrototypeOf(ctor.prototype, superCtor.prototype);
//ctor.prototype.__proto__ = superCtor.prototype;
util.inherits(Bell,EventEmitter);
let bell = new Bell();
//学生要进教室
function studentInClassroom(roomNumber,things){
  console.log(`学生带着${things}进${roomNumber}教室`);
}
function teacherInClassroom(roomNumber,things){
  console.log(`讲师带着${things}进${roomNumber}教室`);
}
function masterInClassroom(roomNumber,things){
  console.log(`老师带着${things}进${roomNumber}教室`);
}
bell.setMaxListeners(5);
bell.addListener('响',studentInClassroom);
bell.on('响',teacherInClassroom);
bell.once('响',masterInClassroom);
bell.emit('响','301','书');
bell.emit('响','301','书');
```

## 4 、util模块，内置工具类

```js
let { promisify, inspect,inherits } = require('util');
console.log(inspect(obj,{depth:2})); //将对象转成字符串
console.log(isArray([]));
inherits(Student,Person) // 使用util的方法来实现继承
const stat = promisify(fs.stat); //将stat方法封装成一个promise
const readdir = promisify(fs.readdir); 
```

## 5、module.exports 和 exports

```js
(funciton(exports, require, module, __filename, __dirname) { // 包装头
  console.log('hello world!') // 原始文件
}); // 包装尾
```

- require方法

  ```js
  /**
   * 在node.js里通过require方法加载其它模块
   * 这个加载是同步的,为了模块实现缓存，当第一次加载一个模块之后，会缓存这个模块的exports对象。以后如果再次加载这个模块的话，则直接从缓存中取，不需要再次
   * 加载了
   * 1. 找到这个文件
   * 2. 读取此文件模块的内容
   * 3. 把它封装在一个函数里立刻执行
   * 4. 执行后把模块的module.exports对象赋给school
   */
  
  console.log(require);
  /**
   * resolve
   * main
   * extensions
   * cache
   */
  require.main中
  /*
   id: '.', //模块ID 入口模块的ID永远为.
   exports: {},//导出对象，默认是一个空对象
   parent: null,//父模块 此模块是谁哪个模块来加载的
   filename: 'D:\\vipcode\\201801\\5.module\\useschool.js', //当前模块的绝对路径
   loaded: false,  //是否加载完成
   children: [],//此模块加载了哪些模块
   paths: 第三方模块的查找路径
   */
  //当你想知道一个模块的绝对路径的时候，但又不想真正加载它的时候，可以用require.resolve
  ```


## 6、Buffer

- 定义Buffer的方式

  ```js
  //表示分配 一个长度为6个字节的Buffer,会把所有的字节设置为0,可以提供默认值
  const buf1 = Buffer.alloc(6,2);
  const buf2 = Buffer.allocUnsafe(6); //分配一块没有初始化的内存
  const buf3 = Buffer.from('Node');
  ```

- 填充Buffer的方式

  ```js
  let buf4 = Buffer.alloc(6);
  buf4.fill(3,1,3);//1-填充的值  2-填充的开始索引  3-结束索引
  buf4.write("前",0,3,'utf8');  //1写的字符串 2填充的开始索引 3填充的字节长度 4编码
  buf4.write('端',3,3,'utf8');
  ```

- Buffer相关方法

  ```js
  let buf5 = Buffer.alloc(6);
  buf5.writeInt8(0,0); //向指定的索引写入一个8位的整数,也就是说占用一个字节的整数  //[0,0,0]
  buf5.writeInt8(16,1); //[0,10,0]
  buf5.writeInt8(32,2); //[0,10,20]
  
  let buf6 = Buffer.alloc(4);
  buf6.writeInt16BE(256,0); // Big Endian 大头在前,也就是高位在前
  console.log(buf6); // [01,00,00,00]
  let s = buf6.readInt16BE(0);
  console.log(0x10,0o10,10,0b10);//Buffer永远输出十进制
  
  buf6.writeInt16LE(256,2);//[,,00,01]  //LT就是低位在前
  let s2 = buf6.readInt16LE(2);//0100
  console.log(buf6);//[01,00,00,01]
  
  console.log(buf6.toString()); //把buffer转成字符串
  //长度为6，每个都是1
  let buf7 = Buffer.alloc(6,1);
  let buf8 = buf7.slice(2,6); // 截取一段Buffer,浅拷贝
  console.log(buf8);
  buf8.fill(4);
  console.log(buf7);
  ```

- 实现在Buffer的split方法

  > Buffer没有split方法，但有indexOf方法	

  ```js
  Buffer.prototype.split = function (sep) {
      let pos = 0;//记录当前是从哪个索引开始查找分隔符
      let len = sep.length;//分隔符的字节长度 2
      let index = -1;//查找到的分隔子串所在的索引
      let parts = [];
      while (-1 != (index = this.indexOf(sep, pos))) {
          parts.push(this.slice(pos, index));
          pos = index + len;
      }
      parts.push(this.slice(pos))
      return parts;
  }
  let buf = Buffer.from('12**34**56');
  console.log(buf)
  console.log(buf.split('**'));//["12","34","56"]
  ```

  

- string_decoder使用

  ```js
  let buf9 = Buffer.from('前端开发');
  let buf10 = buf9.slice(0,5);//5
  let buf11 = buf9.slice(5,7);//7
  let buf12 = buf9.slice(7);
  let {StringDecoder} = require('string_decoder');
  let sd = new StringDecoder();
  //write就是读取buffer的内容，返回一个字符串
  //write的时候会判断是不是一个字符，如果是的话就输出不是的话则缓存在对象内部，等下次write的时候会把前面缓存的字符加到第二次write的buffer上再进行判断
  console.log(sd.write(buf10));//前
  console.log(sd.write(buf11));//端
  console.log(sd.write(buf12));//开发
  ```

- node读取的文件均为Buffer，但中国使用的GBK编码不在node支持范围，一般借助iconv-lite这个第三方包来转换编码

  ```js
  var iconv = require('iconv-lite')
  function readGBKText(pathname){
      var bin = fs.readFileSync(pathname)
      return iconv.decode(bin,'gbk')
  }
  ```

  

## 7、fs模块

第三方库，mkdirp：可递归创建文件夹，fs-extra：fs的扩展模块，方便操作文件系统， glob：匹配文件或文件夹

```js
var glob = require("glob")
// options is optional
glob("**/*.js", options, function (er, files) {
  // files is an array of filenames.
  // If the `nonull` option is set, and nothing
  // was found, then files is ["**/*.js"]
  // er is an error object or null.
})
glob.sync('./src/pages/**/*.vue').forEach
```

- fs.readFile(path[, options], callback) 

  ```js
  fs.readFile('/etc/passwd',{encoding:'utf8',flag:'a'}, (err, data) => {
    if (err) throw err;
    console.log(data);
  }); // encoding表示编码方式，flag表示读取方式，有多参数可选
  ```

- fs.writeFile(file, data[, options], callback)

  ```js
  fs.writeFile('/etc/passwd','node',{encoding:'utf8',flag:'a',mode:0o666}, (err, data) => {
    if (err) throw err;
    console.log(data);
  }); // encoding表示编码方式，flag表示读取方式，有多参数可选,mode表示权限相关，具体参照linux
  ```

- fs.appendFile(path, data[, options], callback) 参数与上述方法类似

- fs.open(path[, flags[, mode]], callback)      

- fs.read(fd, buffer, offset, length, position, callback)     fs.readdir  读取目录，若不加参数，会读取整个缓存区数据

  ```js
  let buffer = Buffer.alloc(3);
  // fd file descriptor 文件描述符 他代表对当前文件的描述
  // process.stdout.write(); // 标准输出  1
  // process.stderr.write();// 错误输出 2
  fs.open(path.join(__dirname,'1.txt'),'r',0o666,function (err,fd) { 
      // offset表示的是 buffer从那个开始存储
      // length就是一次想读几个
      // postion 代表的是文件的读取位置，默认可以写null 当前位置从0开始
      // length不能大于buffer的长度
      fs.read(fd,buffer,0,2,0,function(err,bytesRead){ // bytesRead 读取到个数
          console.log(err,bytesRead);
      })
  });
  ```

- fs.write(fd, buffer[, offset[, length[, position]]], callback)  调用write方法写入文件时，并不会直接写入物理文件，而是会先写入缓存区，再批量写入物理文件

  ```js
  // 使用open，read,write方法来实现一个copy方法
  function copy(source,target){
      let size = 3; // 每次来三个
      let buffer = Buffer.alloc(3);
      fs.open(path.join(__dirname,source),'r',function(err,rfd){
          fs.open(path.join(__dirname,target),'w',function(err,wfd){
              function next(){
                  fs.read(rfd,buffer,0,size,null,function(err,bytesRead){
                      if(bytesRead>0){ // 读取完毕了 没读到东西就停止了
                          fs.write(wfd,buffer,0,bytesRead,null,function(err,byteWritten){
                              next();
                          })
                      }else{
                          //迫使操作系统立刻把缓存区的内容写入物理文件，确保内容写入到文件中 
                          fs.fsync(wfd,function(){ 
                              fs.close(wfd,function(){ // 写入的
                                  console.log('关闭','拷贝成功')
                              })
                          })
                      }
                  })
              }
              next();
          })
      });
  }
  copy('1.txt','2.txt');
  ```

- fs.mkdir(path[, options], callback) 创建文件夹

  ```js
  fs.mkdir('a/b/c',function(err){  // 目录创建必须保证父级存在才能创建
      console.log(err);
  });
  
  //可使用一个第三方库来创建目录(父组不存在时会按给定的路径创建)
  let mkdirp = require('mkdirp');
  mkdirp.sync('a/b/c');
  ```

- fs.readdir(path[, options], callback) 读取目录

- fs.rmdir(path[, options], callback)  删除文件夹，空目录才能删除

- fs.unlink(path, callback) 删除文件

- fs.access(path[, mode], callback) 判断资源是否有权限访问

- fs.rename(oldPath, newPath, callback) 重命名

- fs.truncate(path[, len], callback)  截断文件，只保留文件的len长度

- fs.watchFile(filename[, options], listener)  监控文件

  ```js
  fs.watchFile('./1.txt',function(current,prev){
      if(Date.parse(current.ctime)==0){ 
          console.log('删除')
      }else if(Date.parse(prev.ctime) === 0){
          console.log('创建')
      }else{
          console.log('修改');
      }
  });
  ```

- fs.stat(path[, options], callback) 获取文件的信息

  ```js
  fs.stat("./1.txt", function (err, stats) {
      stats.size; //获取文件的大小；
      stats.atime.toLocaleString(); //获取文件最后一次访问的时间；
      stats.birthtime.toLocaleString(); //文件创建的时间；
      stats.mtime.toLocaleString(); //文件最后一次修改时间；
      stats.ctime.toLocaleString() //状态发生变化的时间；
      stats.isFile() //判断是否是文件；是返回true；不是返回false；
      stats.isDirectory() //判断是否是目录；是返回true、不是返回false；
  })
  ```

- 使用广度优先来遍历一个文件夹

  ```js
  function wide(dir) { // wide('./a')
    let arr = [dir]
    while (arr.length > 0) {
      const current = arr.shift()
      console.log(current);
      const stat = fs.statSync(current)
      if (stat.isDirectory()) {
        const files = fs.readdirSync(current)
        files.forEach(item => {
          arr.push(path.join(current, item))
        })
      }
    }
  }
  ```

## 8、path模块

- path.join([...paths]) 连接几个路径

  ```js
  path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
  // Returns: '/foo/bar/baz/asdf'
  ```

- path.resolve([...paths]) 从当前目录出发，解析出一个决定路径

  ```js
  path.resolve('/foo/bar', './baz'); // Returns: '/foo/bar/baz'
  path.resolve('/foo/bar', '/tmp/file/'); // Returns: '/tmp/file'
  ```

- path.delimiter 环境变量分隔符

- path.sep  文件路径分割符

- path.relative(from, to) 获取两个路径之间的相对路径

- path.basename(path[, ext])  获取文件名

  ```js
  path.basename('/foo/bar/baz/asdf/quux.html');  // Returns: 'quux.html'
  path.basename('/foo/bar/baz/asdf/quux.html', '.html');  // Returns: 'quux'
  ```

## 9、stream流

  ```js
 // 读取流，写入流，读写流(压缩，加密)
  const fs = require("fs");
  let rs = fs.createReadStream("www/1.html"); //创建一个读取流
  let ws = fs.createWriteStream("www/2.html"); //创建一个写入流

  rs.pipe(ws); //通过管道符pipe将读取流传入写入流

  rs.on("error", (err) => {
    //读取错误触发
    console.log("读取失败");
  });
  ws.on("end", () => {
    console.log("读取完成");
  });
  ws.on("error", (err) => {
    console.log("写入失败");
  });
  ws.on("finish", () => {
    console.log("写入完成");
  });

  // 通过流对读取文件压缩
  const fs = require("fs");
  const zlib = require("zlib");

  let gz = zlib.createGzip();
  let rs = fs.createReadStream("www/2.png");
  let ws = fs.createWriteStream("www/2.png.gz");
  rs.pipe(gz).pipe(ws);
  ```

### 1 可读流 

可读流有两种工作模式：flowing（流动模式）和paused（暂停模式），两个模式可相互切换

1.  flowing模式下，可读流自动从系统底层读取数据，并通过EventEmitter接口的事件迟快将数据提供给应用，可通过三种途径切换到flowing模式：
    - 监听‘data'事件
    - 调用stream.resume()方法
    - 调用stream.pipe()方法将数据发送到writable

  2.  paused模式下，必须显式调用stream.read()方法来从流中读取数据片段,可通过以下途径切换到该模式
      - 若不存在管道目标，可通过调用stream.pause()方法实现
      - 若存在管道目标，可以通过取消‘data‘事件监听，并调用stream.uppipe()方法移除所有管道目标来实现

- fs.createReadStream(path[, options])   

  ```js
  //flowing模式
  let fs = require('fs');//456789
  let rs = new ReadStream(path.join(__dirname,'1.txt'),{  //自定义的可写流
      flags:'r', // 文件的操作是读取操作
      encoding:'utf8', // 默认是null null代表的是buffer
      autoClose:true, // 读取完毕后自动关闭
      highWaterMark:3,// 默认是64k  64*1024b
      start:0, // 123 456 789  
      end:3 // 包前又包后
  });
  rs.setEncoding('utf8');
  rs.on('data', function (data) { //flowing模式
      console.log(data)
      rs.pause() // 暂停读取和发射data事件
      setTimeout(function () {
          rs.resume() //恢复读取并触发data事件
      },2000)
  }) // 创建可读流后便能接收到数据
  rs.on('end', function () { console.log('end') })
  rs.on('error', function (err) { console.log(err) })
  rs.on('open', function () { console.log('open') })
  rs.on('close', function () { console.log('close') })
  ```

  ```js
  // 使用paused模式,监听readable事件，会进入暂停模式，可读流会去底层读取文件，将读到的文件放到缓存区里
  let fs = require('fs');//456789
  let rs = fs.createReadStream('./13.stream/2.txt', {
      highWaterMark: 3 // 缓冲区大小,字节，默认是64k
  });
  
  // 会立即从文件中读取highWaterMark个数据,读完后填充缓存区，然后触发readable事件
  rs.on('readable', function () { 
      let ch = rs.read(1) // 从缓存区读取1个字节
      console.log(ch);
      setTimeout(()=>{ 
          // 当读了一个字节后，发现缓存区只有2个字节，会再次读取highWaterMark个字节并填充到缓存区
          console.log(rs._readableState.length); // 缓存区的个数
      },200)
  })
  ```

- 自定义可读流

  ```js
  //自定义可读流
  let { Readable } = require('stream');
  class Counter extends Readable {
    constructor() {
      super();
      this.index = 3
    }
    _read() {
      if (this.index-- > 0) {
        this.push(this.index + '');
      } else {
        this.push(null);
      }
    }
  }
  let counter = new Counter();
  counter.on('data', function (data) {
    console.log(data.toString())
  });
  ```

  

### 2 可写流

```js
const fs = require('fs');
const ws = fs.createWriteStream('./1.txt',{  // 就两个方法 write end
    flags:'w',
    mode:0o666,
    autoClose:true,
    highWaterMark:3, // 默认写是16k
    encoding:'utf8',
    start:0
});
// 写入的数据必须是字符串或者buffer
// flag代表是否能继续写,但是返回false 也不会丢失，就是会把内容放到内存中
let flag = ws.write(1+'','utf8',()=>{}); // 异步的方法
console.log(flag);
flag = ws.write(1+'','utf8',()=>{}); // 异步的方法
console.log(flag);

ws.end('ok'); // 当写完后 就不能再继续写了
ws.write('123'); //write after end
// 抽干方法 当都写入完后会触发drain事件
// 必须缓存区满了 满了后被清空了才会出发drain
ws.on('drain',function(){
    console.log('drain')
});
fs.read('文件描述符吗,读取到哪个buffer,内容从buffer的哪个位置上开始,读取文中内容的长度,读取文件的位置')
```

- 自定义可写流

  ```js
  let { Writable } = require('stream');
  let arr = [];
  let ws = Writable({
    write(chunk, encoding, callback) {
      arr.push(chunk);
      callback();//进行下一次写入
    }
  });
  for (let i = 0; i < 5; i++) {
    ws.write('' + i, 'utf8', () => { });
  }
  ws.end();
  setTimeout(function () {
    console.log(arr.toString());
  }, 500);
  ```

### 3 pipe

```js
let { Writable, Readable } = require('stream');
let i = 0;
let rs = Readable({
    highWaterMark: 2,
    read() {
        if (i < 10) {  this.push('' + i++)} else {
            this.push(null);
        }
    }
});
let ws = Writable({
    highWaterMark: 2,
    write(chunk, encoding, callback) {
        console.log(chunk.toString());
        //callback();
    }
});
rs.pipe(ws);
setTimeout(function () {
    console.log(rs._readableState.buffer);//2
    console.log(ws._writableState);//2
}, 500);
```

### 4 Duplex 双工流

```js
let { Duplex } = require('stream');
let index = 0;
let s = Duplex({
   read() {
      if (index++ < 3)
         this.push('a');
      else
         this.push(null);
   },
   write(chunk, encoding, cb) {
      console.log(chunk.toString().toUpperCase());
      cb();
   }
});
//process.stdin 标准输入流
//process.stdout标准输出流
process.stdin.pipe(s).pipe(process.stdout);
```

### 5 Transform 转换流

```js
let {Transform}  = require('stream');
//转换流是实现数据转换的
let t = Transform({
    transform(chunk,encoding,cb){
        this.push(chunk.toString().toUpperCase());
        cb();
    }
});
process.stdin.pipe(t).pipe(process.stdout);
```

### 6 对象流，使用Transform

```js
let { Transform } = require('stream');
let fs = require('fs');
let rs = fs.createReadStream('./user.json');
//普通流里的放的是Buffer,对象流里放的对象
let toJSON = Transform({
  readableObjectMode: true,//就可以向可读流里放对象
  transform(chunk, encoding, cb) {
    //向可读流里的缓存区里放
    this.push(JSON.parse(chunk.toString()));
  }
});
let outJSON = Transform({
  writableObjectMode: true,//就可以向可读流里放对象
  transform(chunk, encoding, cb) {
    console.log(chunk);
    cb();
  }
});
rs.pipe(toJSON).pipe(outJSON);
```

## 10、tcp

- 创建一个tcp服务，监听server和socket的相关事件，并调用相关方法设置或获取服务信息

  ```js
  let net = require('net');
  //当客户端连接上来的时候会执行对应的回调函数
  //socket其实是一个可读可写流，是一个双工流
  let server = net.createServer({});
  server.on('connection', function (socket) {
      //表示客户端连接的总数量是1个。
      server.maxConnections = 1;
      //获取当前有多少个客户端正在连接服务器
      server.getConnections((err, count) => {
          console.log(`欢迎光临，现在连接的客户端总数量是${count}个,客户端连接的总数量是${server.maxConnections}`);
      });
      console.log(socket.address());
      socket.setEncoding('utf8');
      //如何获取可读流里的数据?
      socket.on('data', function (data) {
          console.log('服务器接收客户端发过来的数据:', data);
          socket.write('服务器回应:' + data);
      });
      //服务器收到客户端发出的关闭连接请求时，会触发end事件
      //在这个地方客户端没有真正关闭，只是开始关闭。当真正关闭的时候还会触发一个close事件
      socket.on('end', function () {
          console.log('客户端已关闭');
          //close 服务器端有一个方法叫close，close的意思是如果执行了此方法，那么此客户端将不再接收新的连接，
          //但是也不会关闭现有服务器
          //一旦调用此方法，则当所有的客户端关闭跟本服务器的连接后，将关闭服务器
          server.unref();
      });
      // setTimeout(function () {
      //     //在5秒之后会关闭掉此服务器。不再接收新的客户端了
      //     server.close();
      // }, 5000);
      //hasError如果为true表示异常关闭，否则表示正常关闭
      socket.on('close', function (hasError) {
          console.log('客户端真正关闭', hasError);
      });
      socket.on('error', function (err) {
          console.log(err);
      });
  });
  server.on('close', function () {
      console.log('服务器端已关闭');
  });
  server.on('error', function (err) {
      console.log(err);
  });
  server.listen(8080, function () {
      console.log(server.address());
      console.log('服务器端已经启动');
  });
  ```

- 创建tcp服务，并将客户端的输入的信息使用可写流写入到文件中

  ```js
  let net = require('net');
  let path = require('path');
  let ws = require('fs').createWriteStream(path.join(__dirname, 'msg.txt'));
  let server = net.createServer(function (socket) { //socket代表跟客户端的连接
      socket.pause();
      //设置客户端的超时时间 如果客户端一直不输入超过一定的时间就认为超时了
      socket.setTimeout(2 * 1000);
      //write after end 在文件关闭掉之后再次写入
      socket.on('timeout', function () {
          console.log('timeout');
          //默认情况下，当可读流读到末尾的时候会关闭可写流
          socket.pipe(ws, { end: false });
      });
  });
  server.listen(8080);
  ```

- 创建tcp服务，并使用可写流向客户端发送文件

  ```js
  //当客户端访问服务器的时候，服务器会发送给客户端一个文件
  let net = require('net');
  let path = require('path');
  let rs = require('fs').createReadStream(path.join(__dirname, '1.test'));
  net.createServer(function (socket) {
      rs.on('data', function (data) {
          let flag = socket.write(data);//可写流缓存区是否满了
          console.log('flag=', flag);
          console.log('缓存的字节数=', socket.bufferSize);
      });
      socket.on('drain', function () {
          console.log('TCP缓存区中的数据已经发送');
      });
  }).listen(8080);
  ```

## 11、http

### 1 创建http服务，获取请求体

http服务器是继承自tcp服务器 http协议是应用层协议，是基于TCP,并对请求和响应进行了包装

```js
let http = require('http');
let url = require('url');
let server = http.createServer();
//当客户端连接上服务器之后执行回调
server.on('connection', function (socket) { //客户端连接上来之后先触发connection事件，
    console.log('客户端连接 ');
});
//req代表客户端的连接，server服务器把客户端的请求信息进行解析，然后放在req上面
//请求都会触发request事件。
server.on('request', function (req, res) {  //req是可读流  res是一个可写流 write
    console.log(req.method);//获取请求方法名
    let { pathname, query } = url.parse(req.url, true);
    console.log(req.url);//获取请求路径 
    console.log(req.headers);//请求头对象
    let result = [];
    req.on('data', function (data) { 
        result.push(data);
    });
    req.on('end', function () {
        let r = Buffer.concat(result);//请求体
        console.log(r.toString());
        
        //如果进行响应
        res.end(r);
    })
});
server.on('close', function (req, res) {
    console.log('服务器关闭 ');
});
server.on('error', function (err) {
    console.log('服务器错误 ');
});
server.listen(8080, function () {
    console.log('server at http://localhost:8080');
});
```



### 2 创建http服务，设置响应体

```js
let http = require('http');
let mime = require('mime');
let server = http.createServer(function (req, res) {  //req和res都是从socket来的
    console.log('headersSent1', res.headersSent);//响应头是否已经发送过了
    res.writeHead(200, {  //writeHead一旦调用会立刻向客户端发送，setHeader不会
        "Content-Type": "text/html;charset=utf8"
    })
    //可以根据不同的文件内容类型返回不同的Content-Type
    res.setHeader('Content-Type', mime.getType(pathname));
    //当调用writeHead 或者调用write方法的时候才会向客户端发响应头
    console.log('headersSent2', res.headersSent);//响应头是否已经发送过了
    res.end('byebye');
    // res.statusCode = 404;//设置响应码 
    // res.sendDate = false;//Date响应头默认会设置，如果真的不想要，可以设置为false 
    // res.setHeader('Content-Type', 'text/html;charset=utf8');//设置响应头
    // console.log('getHeader1', res.getHeader('Content-Type'));//获取响应头
    // res.removeHeader('Content-Type');//删除响应头
    // res.write('hello');
    // res.end();
});
server.on('connection', function (socket) {
    console.log('connection');
    socket.on('end', function () {
        console.log('连接end');
    });
    socket.on('close', function () {
        console.log('连接close');
    });
});
server.listen(8080);
```

### 3 使用http模板写一个客户端

```js
let http = require('http');
let options = {  //头分四种 通用头 请求头 响应头 实体头
    host: 'localhost',
    port: 8080,
    method: 'POST',
    headers: {
        //"Content-Type": 'application/x-www-form-urlencoded',
        "Content-Type": 'application/json'
    }
}
let req = http.request(options); //请求并没有真正发出 req也是一个流对象，它是一个可写流
req.on('response', function (res) {  //当服务器端把请求体发回来的时候，或者说客户端接收到服务器端响应的时候触发
    console.log(res.statusCode);
    console.log(res.headers);
    let result = [];
    res.on('data', function (data) {
        result.push(data);
    });
    res.on('end', function (data) {
        let str = Buffer.concat(result);
        console.log(str.toString());
    });
});
//write是向请求体里写数据
req.write('{"name":"nodejs"}');
//req.write('name=nodejs&age=9');
//是结束写入请求体，只有在调用end的时候才会真正向服务器发请求
req.end();
```





## 12、zlib压缩和解压模块

- zlib 压缩模块（需设置响应头）

  ```js
    const http = require("http");
    const fs = require("fs");
    const url = require("url");
    const zlib = require("zlib"); //引入压缩模块
  
    http
      .createServer((req, res) => {
        let { pathname, query } = url.parse(req.url, true);
        res.setHeader("Content-Encoding", "gzip"); //需设置响应头，否则浏览器无法正常解析
  
        let rs = fs.createReadStream(`www${pathname}`); //创建读取流
        let gz = zlib.createGzip(); //创建压缩
  
        rs.pipe(gz).pipe(res); //通过管道将读取流压缩，并返回响应
  
        rs.on("error", function () {
          res.writeHeader(404);
          res.write("not found");
          res.end();
        });
      })
      .listen(8080);
  ```

- 可读流的压缩方法

  ```js
  let fs = require('fs');
  let path = require('path');
  let zlib = require('zlib');
  console.log(process.cwd());//current working directory 当前工作目录
  //用于实现压缩 transform转换流，继承自duplex双工流，
  
  function gzip(src) { //压缩方法
      fs.createReadStream(src)
          .pipe(zlib.createGzip())
          .pipe(fs.createWriteStream(src + '.gz'));
  }
  gzip(path.join(__dirname, 'msg.txt'));
  
  function gunzip(src) {  //解压方法
      fs.createReadStream(src)
          .pipe(zlib.createGunzip())
      	//basename 从一个路径中得到文件名，包括扩展名的,可以传一个扩展名字符参数，去掉扩展名,extname 是获取扩展名
          .pipe(fs.createWriteStream(path.join(__dirname, path.basename(src, '.gz'))))
  }
  gunzip(path.join(__dirname, 'msg.txt.gz'));
  ```

- 非流的压缩方法

  ```js
  let zlib = require('zlib');
  let str = 'hello';
  zlib.gzip(str, (err, buffer) => {
      console.log(buffer.length); // buffer是压缩后的结果
      zlib.unzip(buffer, (err, data) => {
          console.log(data.toString());
      });
  });
  ```

## 13、crypto加密和解密

```js
  // crypto 加密模块
  const crypto = require("crypto");
  let hash = crypto.createHash("md5"); //md5  sha1  sha256  sha512  ripemd160
  //hash.update()方法将字符串相加，然后在hash.digest()将字符串加密返回
  //hash.update('abcdef'); 一次和分开写是一样效果
  hash.update("abc");
  hash.update("def");
  console.log(hash.digest("hex")); //以16进度输出
```

node中使用OpenSSL类库作为内部实现加密解密的手段，windows需要下载OpenSSL

- 散列（哈希）算法

  crypto.getHashes()    获取可用的加密算法，如md5,sha1,sha256等

  crypto.createHash('sha1')   创建加密方式

  ```js
  /**
   * 1.可以用来检验要下载的文件是否被改动过
   * 2.对密码进行加密 123456 => md5值
  **/
  let crypto = require('crypto');
  let str = 'hello';
  //console.log(crypto.getHashes());  // 
  let md5 = crypto.createHash('sha1'); // 创建
  md5.update('hello');//指定要加密的值
  md5.update('world');//再次添加要加密的值,两次的效果等同于md5.update('helloworld')
  console.log(md5.digest('hex'));//输出md5值，指定输出的格式 hex 十六进制
  ```

- HMAC算法，将散列算法与一个密钥结全在一起，以阻止对签名完整性的破坏

   crypto.createHmac('sha1', key);   调用方法，传入加密方式和key

  ```js
  let crypto = require('crypto');
  let path = require('path');
  let fs = require('fs');
  let key = fs.readFileSync(path.join(__dirname, 'rsa_private.key')); //从文件中读取key
  let hmac = crypto.createHmac('sha1', key); // 又称为加盐算法
  hmac.update('123');
  let result = hmac.digest('hex');
  console.log(result);
  ```

- 对称加密，blowfish算法是一种对称（加密和解密用的是同一个密钥）的加密算法

  ```js
  //对称加密 加密和上面的摘要
  let crypto = require('crypto');
  let path = require('path');
  let fs = require('fs');
  let str = 'asdbasdfsdf';
  let pk = fs.readFileSync(path.join(__dirname, 'rsa_private.key'));
  let cipher = crypto.createCipher('blowfish', pk);
  let result = cipher.update(str, 'utf8', 'hex');
  result = cipher.final('hex');//输出加密后的结果
  console.log(result);
  // 解密
  let decipher = crypto.createDecipher('blowfish', pk);
  let r = decipher.update(result, 'hex', 'utf8');
  r = decipher.final('utf8');
  console.log(r);
  ```

## 14、yargs的使用（需安装）

- bat的简单使用

  ```js
  //hello.bat文件
  node 2.hello.js %1 %2  //该可执行文件会执行该文件，并将命令行传入的参数传入到文件中
  ```

- process.argv 获取命令行工具传入的参数

  ```js
  console.log('hello' + process.argv);
  ```

- yargs解析命令行参数

  ```js
  let yargs = require('yargs'); //解析命令行参数，把参数数组变成对象的形式
  let argv = yargs.options('n', {  //命令行执行node .\2.hello.js -n aaaaaaaaa
      alias: 'name',//别名
      demand: true,//必填
      default: 'zfpx',
      description: '请输入你的姓名'
  }).usage('hello [options]')
    .help()//指定帮助信息
    .example('hello -name zfpx', '执行hello命令，然后传入name参数为zfpx')
    .alias('h', 'help')
    .argv;
  console.log('hello ' + argv.name); //可获取命令行输入的name参数
  ```

## 15、cache相关的方法

- Last-Modified

  1.  第一次访问服务器的时候，服务器返回资源和缓存的标识('Last-Modified')，客户端则会把此资源缓存在本地的缓存数据库中。
  2. 第二次客户端需要此数据的时候，要取得缓存的标识(if-modified-since)，然后去问一下服务器我的资源是否是最新的。
  3. 如果是最新的则直接使用缓存数据，如果不是最新的则服务器返回新的资源和缓存规则，客户端根据缓存规则缓存新的数据。

- ETag

  1.  第一次服务器返回的时候，会把文件的内容算出来一个标识('ETag')，发给客户端

  2.  客户端看到etag之后，也会把此标识符('if-none-match')保存在客户端，下次再访问服务器的时候，发给服务器

- Cache-Control和Expires，强制缓存，指定缓存的过期时间

  1.  把资源缓存在客户端，如果客户端再次需要此资源的时候，先获取到缓存中的数据，看是否过期，如果过期了，再请求服务器

  2.  如果没过期，则根本不需要向服务器确认，直接使用本地缓存即可

  ```js
  //综合应用
  let http = require('http');
  let url = require('url');
  let path = require('path');
  let fs = require('fs');
  let { inspect } = require('util')
  let mime = require('mime'); // 根据扩展名获取响应类型
  http.createServer(function (req, res) {
      let { pathname } = url.parse(req.url, true);
      let filepath = path.join(__dirname, pathname);
      fs.stat(filepath, (err, stat) => {
          if (err) {
              return sendError(req, res);
          } else {
           // -----------------Last-Modified的处理方式-----------------
              let ifModifiedSince = req.headers['if-modified-since'];
              let LastModified = stat.ctime.toGMTString();
              if (ifModifiedSince == LastModified) {
                  res.writeHead(304);
                  res.end('');
              } else {
                  return send(req, res, filepath, stat);
              }
  		// -----------------ETag的处理方式----------------------------
              let ifNoneMatch = req.headers['if-none-match'];
              let out = fs.createReadStream(filepath);
              let md5 = crypto.createHash('md5');
              out.on('data', function (data) {
                  md5.update(data);
              });
              out.on('end', function () {
                  //1.相同的输入相同的输出 2 不同的输入不同的输入 3 不能从输出反推出输入 
                  let etag = md5.digest('hex');
                  let etag = `${stat.size}`;
                  if (ifNoneMatch == etag) {
                      res.writeHead(304);
                      res.end('');
                  } else {
                      return send(req, res, filepath, etag);
                  }
            });
          }
      });
  }).listen(8080, function () {
      console.log('app is running');
  });
  function sendError(req, res) {
      res.end('Not Found');
  }
  function send(req, res, filepath, stat) {
      res.setHeader('Content-Type', mime.getType(filepath));
      //发给客户端之后，客户端会把此时间保存起来，下次再获取此资源的时候会把这个时间再发回服务器
      res.setHeader('Last-Modified', stat.ctime.toGMTString());
      fs.createReadStream(filepath).pipe(res);
      //-----------------ETag--------------------------
      res.setHeader('ETag', etag);
      fs.createReadStream(filepath).pipe(res);
      //-----------------------------------------------
      res.setHeader('Expires', new Date(Date.now() + 30 * 1000).toUTCString());
      res.setHeader('Cache-Control', 'max-age=30');
      fs.createReadStream(filepath).pipe(res);
  }
  ```
  

## 16、多语言功能实现

根据请求头的`Accept-Language:zh-CN,zh;q=0.9`来返回某种语言

```js
let http = require('http');
let server = http.createServer(request);
server.listen(8080);
const lanPack = {
    en: {
        title: 'welcome'
    },
    zh: {
        title: '欢迎光临'
    },
    default: 'en'
}
function request(req, res) {
    //实现服务器和客户端的协议，选择客户端最想要的，并且服务器刚好有的,请求头的格式
    // Accept-Language:zh-CN,zh;q=0.9,en;q=0.8,jp;q=0.7
    let acceptLanguage = req.headers['accept-language'];
    if (acceptLanguage) {
        const lans = acceptLanguage.split(',').map(function (item) {
            let values = item.split(';');
            let name = values[0];
            let q = values[1] ? parseFloat(values[1].split('=')[1]) : 1;
            return {
                name, q
            }
        }).sort((a, b) => b.q - a.q);  // [{lan:'zh-CN',q:1},{lan:'zh',q:1}]
        let lan = lanPack.default;//默认的语言
        for (let i = 0; i < lans.length; i++) {
            //如果说此语言在语言包里有，那么就使用此语言
            if (lanPack[lans[i].name]) {
                lan = lans[i].name;
                break;
            }
        }
        res.end(lanPack[lan].title.toString());
    }
}
```

## 17、图片防盗链

根据请求头的中的`referer`或`refer`来返回或阻止对应的图片

```js
let http = require('http');
let fs = require('fs');
let url = require('url');
let path = require('path');
const whiteList = [
    '192.168.1.101',
    'localhost'
]
let server = http.createServer(function (req, res) { // 模拟图片服务器
    let refer = req.headers['referer'] || req.headers['refer'];
    //如果说有refer的话，则表示是从HTML页面中引用过来的
    if (refer) {
        //http://192.168.1.101:8000/refer.html
        //http://imgsrc.baidu.com/mm.jpg
        let referHostname = url.parse(refer, true).hostname;//192.168.1.101
        let currentHostName = url.parse(req.url, true).hostname;//imgsrc.baidu.com
        if (referHostname != currentHostName && whiteList.indexOf(referHostname) == -1) {
            res.setHeader('Content-Type', 'image/jpg');
            fs.createReadStream(path.join(__dirname, 'forbidden.jpg')).pipe(res);
            return;
        }
    }
    console.log('normal');
    res.setHeader('Content-Type', 'image/png');
    fs.createReadStream(path.join(__dirname, 'a.png')).pipe(res);
});
server.listen(3000);
```

## 18、代理服务器

又称网络代理，是一种特殊的网络服务，允计一个网络终端（一般是客户端）通过这个服务与另一个网络终端（一般是服务端）进行非直接的连接，有利于保障网络终端网络的隐私或安全，防止攻击，使用一个第三方模块

- 正向代理 帮助或代理局域网内的用户访问外网
- 反向代理 用来代理局域网内的服务器的

```js
npm install http-proxy -S
```

```js
let proxy = require('http-proxy');
let http = require('http');
let proxyServer = proxy.createProxyServer();
let server = http.createServer(function (req, res) {
    proxyServer.web(req, res, {
        target: 'http://localhost:9000'
    });
}).listen(8000);
//http-proxy的简单实现
function web(req, res, options) {
    let { host, port, pathname } = url.parse(req.url);
    let opts = {
        host,
        port,
        method: req.method,
        path: pathname,
        header: req.headers
    }
    opt.host = options.target;
    http.request(opts, function (response) {
        response.pipe(res);
    });
}
```

## 19、虚拟主机

请求的默认端口是`80`，根据请求的域名将请求转发到不同的服务器上

```js
/**
 * 弹性计算云服务器 ECS 一个完整的服务器
 * 虚拟主机 服务器上的一个目录
 */
const http = require('http');
const proxyServer = require('http-proxy');
const ps = proxyServer.createProxyServer();
const config = {
    "aaa.com": "http://localhost:8000",
    "bbb.com": "http://localhost:9000"
}
//nginx 核心功能，反向代理
let server = http.createServer(function (req, res) {
    let host = req.headers['host'];
    let target = config[host];
    if (target) {
        ps.web(req, res, {
            target
        });
    } else {
        res.end(host);
    }
}).listen(80);
```

## 20、获取用户设备信息

根据请求头中的`user-agent`信息获取后使用第三方库`user-agent-parser`来解析

```js
let http = require('http');
let parse = require('user-agent-parser')
let server = http.createServer(function (req, res) {
    let userAgent = req.headers['user-agent'];
    console.log(userAgent);
    let userAgentObj = parse(userAgent);
    res.end(JSON.stringify(userAgentObj));
});
server.listen(8080);
```

## 21 处理 post 请求方法

```js
let server = http.createServer((req, res) => {
  //GET数据获取方法
  let { pathname, query } = url.parse(req.url, true);
  //POST数据获取方法
  let aBuffer = []; //使用数组保存post提交的数据
  req.on("data", (data) => {
    aBuffer.push(data);
  });
  req.on("end", () => {
    let data = Buffer.concat(aBuffer); //提交完成后将数据保存在Buffer中
    //post请求的请求头以'multipart/form-data'开头
    if (req.headers["content-type"].startsWith("multipart/form-data")) {
      let post = {};
      let files = {};
      //提取分隔符
      const boundary =
        "--" + req.headers["content-type"].split("; ")[1].split("=")[1];
      //第一步、用分隔符切分
      let arr = data.split(boundary);
      //第二步、扔掉头尾(<>、<--\r\n>)
      arr.shift();
      arr.pop();
      //第三步、每一项的头尾扔掉(\r\n....\r\n)
      arr = arr.map((item) => item.slice(2, item.length - 2));
      //第四步、找第一个"\r\n\r\n"，一切两半——前一半:信息，后一半:数据
      arr.forEach((item) => {
        let n = item.indexOf("\r\n\r\n");
        let info = item.slice(0, n);
        let data = item.slice(n + 4);
        info = info.toString();
        let total = 0;
        let complete = 0;
        if (info.indexOf("\r\n") == -1) {
          //只有一行——普通数据
          let key = common.parseInfo(info).name;
          let val = data.toString();
          post[key] = val;
        } else {
          //两行——文件数据
          total++;
          let json = common.parseInfo(info);
          let key = json.name;
          let filename = json.filename;
          let type = json["Content-Type"];
          let filepath = `upload/${uuid().replace(/\-/g, "")}${path.extname(
            filename
          )}`;
          files[key] = { filename, type, filepath };
          fs.writeFile(filepath, data, (err) => {
            if (err) {
              console.log("文件写入失败");
            } else {
              console.log("写入完成");
              complete++;
              console.log(post, files);
            }
          });
        }
      });
    } else {
      //urlencoded
      let post = querystring.parse(data.toString());
      console.log(post);
    }
  });
});
server.listen(8080);
```

## 22 基于 events 的 router

1. 原理

   ```js
   const Event = require("events").EventEmitter; //EventEmitter——事件队列
   let ev = new Event();
   //ev监听
   ev.on("blue", (a, b, c, d) => {
     console.log("接收到了1个事件：", a, b, c, d);
   });
   ev.on("blue", (a, b, c, d) => {
     //相同事件名可再次触发
     console.log("接收到了2个事件：", a, b, c, d);
   });

   //ev触发
   let res = ev.emit("blue", 12, 5, 8, 99); //返回是否触发成功
   console.log("emit", res);
   ```

2. router 文件

   ```js
   //router.js
   const Event = require("events").EventEmitter;
   module.exports = new Event();
   ```

3. 主文件（定义 send 方法，获取入参并触发相关方法）

   ```js
   const http = require("http");
   const fs = require("fs");
   const url = require("url");
   const router = require("./libs/router"); //导入自定义的router
   const zlib = require("zlib");

   http
     .createServer((req, res) => {
       let { pathname, query } = url.parse(req.url, true); //获取入参
       req.query = query; //将获取入参挂在req上

       res.send = function (data) {
         //给res增加一个send方法
         if (!(data instanceof Buffer) && typeof data != "string") {
           data = JSON.stringify(data);
         }
         res.write(data);
       };

       //不是一个接口处理
       if (false == router.emit(pathname, req, res)) {
         //pathname没有触发成功
         let rs = fs.createReadStream(`www${pathname}`); //2.读取文件
         let gz = zlib.createGzip();
         res.setHeader("Content-Encoding", "gzip");
         rs.pipe(gz).pipe(res);
         rs.on("error", (err) => {
           //3.读取失败
           res.writeHeader(404);
           res.write("not found");
           res.end();
         });
       } else {
         //是接口，处理方法相同
       }
     })
     .listen(8080);
   ```

4. 处理路由文件

   ```js
   const router=require('./router'); //导入上述router文件
   router.on('/login', (req, res)=>{
     let {user, pass}=req.query; //获取挂在req上的入参
     res.send({code: 0, msg: '登陆成功'}); //使用res上定义的send方法
     res.end();
   });
   router.on('/reg', (req, res)=>{
     let {user, pass}=req.query;
       res.send({code: 0, msg: '注册成功'});
       res.end();
     }
   });
   ```

## 23 ejs 使用

<%= 输出文字——转义输出(html 标签->html 实体)
<%- 输出 html——非转义输出(html 标签->原样) -%>

```js
const express = require("express");
const consolidate = require("consolidate"); //安装并引入模版引擎管理模块
let server = express();
server.listen(8080);

//1.选择一种模板引擎
server.engine("html", consolidate.ejs); //该模块指定要使用的模版引擎
//2.指定模板文件的扩展名
server.set("view engine", "ejs");
//3.指定模板文件的路径
server.set("views", "./template");

server.get("/aaa", (req, res) => {
  res.render("2", { arr: [12, 5, 88, 99] });
});
```

## 24 使用 multer 上传文件

```js
const multer = require("multer"); //安装并引入

//文件POST数据 配置
let multerObj = multer({ dest: "./upload/" });
server.use(multerObj.any());

//以post方式提交文件时会自动将文件保存指定目录中，并可在req中获取保存后相关的文件信息
for (let i = 0; i < req.files.length; i++) {
  //上传多个文件
  req.files;
  req.files[i].fieldname;
  req.files[i].filename;
  req.files[i].path;
}
```

# express

## 1、路由

根据不同的方法和不同的路径返回不同的内容

> ~~~js
> const express = require('./express');
> const app = express();
> ~~~

- 基础路由

  ~~~js
  app.get('/hello', function (req, res) {
      res.end('get');
  });
  app.post('/hello', function (req, res) {
      res.end('post');
      res.send('post');
      res.json('post');
  });
  
  //匹配所有路径，不管是get或put所有方法都能处理
  app.all('*', function (req, res) { //*表示匹配所有的路径
      res.end('404');
  });
  ~~~

- 有next参数

  ```js
  app.get('/', function (req, res, next) {
      console.log(1);
      //如果出错，会把错误交给next,然后跳过后面所有的正常处理函数，交给错误处理中间件来进行处理
      next('Wrong');
  }, function (req, res, next) {
      console.log(11);
      next();
  })
  app.get('/', function (req, res, next) {
      console.log(3);
      res.end('ok');
  });
  ```

- 使用router

  匹配到`/user`路径后，不同的请求方法使用不同的处理函数

  ```js
  app.route('/user').get(function (req, res) {
      res.end('get');
  }).post(function (req, res) {
      res.end('post');
  }).put(function (req, res) {
      res.end('put');
  }).delete(function (req, res) {
      res.end('delete');
  })
  ```

- 带路由参数并使用`app.param`

  匹配到带参数的路由后，根据参数作相应操作，并将参数传入param的回调函数中

  ~~~js
  app.param('id', function (req, res, next, id) { // 参数有多个时，可为数组
     User.find(id, function (err, user) { //根据id相关操作
      if (err) {
        next(err)
      } else if (user) {
        req.user = user //没有错误则将user信息挂到req上
        next()
      } else {
        next(new Error('failed to load user'))
      }
    })
  })
  
  app.get('/user/:id', function (req, res) { 
    console.log(req.user) //此处可获取到挂载的user信息,可将user信息整合到req的params中
    res.end()
  })
  ~~~

> ~~~js
> app.listen(8080, function () {  //启动一个端口号为8080的服务器
>     console.log('server started at 8080');
> });
> ~~~

## 2、中间件

> 中间件使用use来定义，首先加载的中间件功能也将首先执行。next是一个函数，调用它则意味着当前的中间件执行完毕，可以继续向下执行别的中间件
>
> 调用next的时候如果传一个任意参数就表示此函数发生了错误，然后express就会跳过后面所有的中间件和路由交给错误处理中间件来处理

- 开发中间件

  一般作用是增加日志等功能，不做其它应用操作

  ```js
  //没有路径的中间件,所有请求都会进入
  app.use(function (req, res, next) {
    res.setHeader('Content-Type', 'text/html;charset=utf8');
    console.log('LOGGED')
    next()
  })
  
  app.get('/', function (req, res) {
    res.send('Hello World!')
  })
  ```

- 使用中间件

  ```js
  app.use('/hello', function (req, res, next) { //匹配到hello后先经过中间件做相关处理
      console.log('hello');
      next();
  });
  app.get('/hello', function (req, res) { //再对get方法的请求做相关处理
      res.end('hello');
  });
  ```

- 错误处理中间件（有四个参数）

  ```js
  app.use('/hello', function (err, req, res, next) { //hello路径的处理出错时捕获错误
      res.end('hello ' + err);
  });
  
  app.use(function (err, req, res, next) {  //捕获所有的错误
      console.log('error');
  });
  ```

- 使用新的路由容器

  ```js
  const user = express.Router();// 
  //子路径里的路径是相对于父路径 
  user.use(function (req, res, next) { //当请求路径为/user时进入
      console.log('Ware', Date.now());
      next();
  });
  user.use('/name',function (req, res, next) { //当请求路径为/user/name时进入
      console.log('Ware', Date.now());
      next();
  });
  app.use('/user', user); //user第二个参数是处理函数 (req,res,next)
  ```

## 3、获取请求参数

```js
app.get('/user', function (req, res) {
    console.log(req.query); // {name:'zfpx',age:8}
    console.log(req.path); // /user
    console.log(req.hostname);//主机机
    res.end('ok');
});
```

## 4、使用模板引擎

```js
//views是用来设置模板存放根目录
app.set('views', path.resolve('views'));
//设置模板引擎 ,如果render的没有指定模板后台名，会以这个作为后缀名
app.set('view engine', 'html');
//用来设置模板引擎，遇到html结尾的模板用ejs来进行渲染
app.engine('html', require('ejs').__express);

//匹配到根路径时人使用indexr的模板
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});
```

## 5、常用中间件

### 5.1 静态资源

>  `express.static(root, [options])`

| 属性         | 描述                                            | 类型     | 默认值     |
| ------------ | ----------------------------------------------- | -------- | ---------- |
| dotfiles     | 控制点文件服务,可选值为 allow,deny,ignore       | String   | “ignore”   |
| etag         | 控制etag生成工                                  | Boolean  | true       |
| extensions   | 设置文件后缀名补充                              | Boolean  | false      |
| index        | 设置目录访问的返回,设置为 false可以禁止目录访问 | Mixed    | index.html |
| lastModified | 根据文件修改时间设置Last-Modified报头           | Boolean  | true       |
| maxAge       | 设置 Cache-Control报头的缓存控制时间,单位为毫秒 | Number   | 0          |
| redirect     | 当路径名是目录时,重定向到包含结尾/的目录        | Boolean  | true       |
| setheaders   | http函数用于为文件设置HTTP头                    | Function |            |

简单用法

```js
app.use(express.static(path.join(__dirname, 'public')));
```

带参数

```js
app.use(express.static(path.join(__dirname, 'public')), {
    extensions: ['html', 'htm'],
    setHeaders(req, res, callback) {
        res.setHeader('time', Date.now());
    }
});
```

### 5.1 body-parser(express已自带此功能)

> 对post请求的请求体请进解析

```js
app.use(text()) //解析纯文本
app.use(bodyParser.json()) //处理json的请求体
//extended为true时，使用qs模块来解析请求体，false时，使用querystring来解析
app.use(bodyParser.urlencoded({extended:true})) //处理表单格式，urlencoded格式的请求体
```

客户端使用GBK编码且使用gzip压缩请求内容，服务端解压并解析案例

- 客户端

  ```js
  let http = require('http');
  let zlib = require('zlib');
  let iconv = require('iconv-lite'); //node不支持中国特有编码的GBK，故需引库
  let options = {
      host: 'localhost',
      port: 8080,
      method: 'POST',
      path: '/user',
      headers: {
          'Content-Type': "text/plain; charset=gbk", //请求头中须有编码
          "Content-Encoding": "gzip"// 代表此请求体是经过压缩过的
      }
  }
  let req = http.request(options, function (response) {
      response.pipe(process.stdout);
  });
  let body = iconv.encode('前端开发', 'gbk'); //body使用gbk编码
  zlib.gzip(body, function (err, data) { //body使用gzip压缩
      req.end(data);
  });
  ```

- 服务端

  ```js
  app.use(text());   //text可以body-parser自带中间件方法，下为实现方法
  function text() {
      return function (req, res, next) {
          let contentType = req.headers['content-type'];
          let typeObj = type.parse(contentType); //type为'content-type'库提供的方法
          let charset = typeObj.parameters.charset;
          let ct = typeObj.type;
          if (ct == 'text/plain') {
              let buffers = [];
              req.on('data', function (data) {
                  buffers.push(data);
              });
              req.on('end', function () {
                  let r = Buffer.concat(buffers);
                  let encoding = req.headers['content-encoding'];
                  if (encoding == 'gzip') {
                      r = zlib.gunzipSync(r); //解压请求体
                  }
                  if (charset == 'gbk') {
                      req.body = iconv.decode(r, 'gbk'); //解码请求体
                  } else {
                      req.body = r.toString();//name=前端开发
                  }
                  next();
              });
          } else {
              next();
          }
      }
  }
  ```

## 6、cookie

### 6.1 原生基础用法

```js
let http = require('http');
let server = http.createServer(function (req, res) {
    if (req.url == '/write') {   //向客户端写入cookie
        res.setHeader('Set-Cookie', "name=joiner");
        res.end('write ok');
    } else if (req.url == '/read') {
        //客户端第二次请求的时候会向服务器发送   Cookie
        let cookie = req.headers['cookie'];
        res.end(cookie);
    } else {
        res.end('Not Found');
    }
}).listen(8080);
```

### 6.2 使用express

express将设置的方法封装了，但获取的并没有封装，需要借助第三方库`cookie-parser`来解析

```js
const express = require('express');
const cookieParser = require('cookie-parser')
const app = express();
app.use(cookieParser());
app.get('/write', function (req, res) {
  //express封装的功能，signed表示对cookie加签名（不是加密），作用是防浏览器端更改
  res.cookie('name', 'joiner',{signed:true}); 
  res.end('write ok');
});
app.get('/read', function (req, res) {
  //let cookie = req.headers['cookie'];//原生获取name=joiner
  let cookie = req.cookies //cookieParser中间件提供的功能，只能读取到未签名的cookie
  let cookie = req.signedCookies //能读取到签名后的cookie,若cookie被修改，则读取值为false
  res.send(cookie);
});
app.listen(8080);
```

### 6.3 cookie的相关参数

cookie最后是以下面的格式，通过` res.setHeader('Set-Cookie', cookie)`方法来设置

> Set-Cookie:name=joiner; Domain=localhost; Path=/read; Max-Age=10000; Expires=Wed, 07 M

```js
// express封装的方法
res.cookie('name', 'jonee', {
    httpOnly: true, //不允许客户端通过浏览的cookie访问
    secure: true, //只允许https的协议使用
    maxAge: 10 * 1000, //10秒后会失效
    path: '/read', //路径以/read为开头都可以访问
    domain: 'localhost', //域名设置
    expires: new Date(Date.now() + 10 * 1000) //绝对过期时间
});
```

## 7、session

- expres中session设置一般使用第三方库`express-session`

  ```js
  const session = require('express-session');
  let FileStore = require('./store')(session);
  app.use(session({
      name: 'sid', //cookie中的name，默认是connect.sid
      resave: true,
      saveUninitialized: true,
      rolling: true,
      store: new FileStore({  //自定义存放session的位置
          root: path.join(__dirname, 'sessions'),
          maxAge: 10 * 1000
      }),
      //genid() {
      //    return uuid.v4();
      // },
      secret: 'joiner',
      cookie: {
          expires: new Date(Date.now() + 10000)
      }
  }));
  ```
  
  该库提供的是一个中间件方法，有以下参数：
  
  | 参数              | 描述                                                         |
  | :---------------- | :----------------------------------------------------------- |
  | name              | 设置 cookie中,保存 session的字段名称,默认为 connect.sid      |
  | store             | session的存储方式,默认存放在内存中,也可以使用 redis, mongodb等 |
  | secret            | 通过设置的 secret字符串,来计算hash值并放在 cookie中,使产生的 signedCookie防改 |
  | cookie            | 设置存放 session id的相关选项，默认为(default:{path:'/',httponly:true,secure:false,maxAge:null}) |
  | genid             | 产生一个新的 session时,所使用的函数,默认使用uid2这个npm包    |
  | rolling           | 每个请求都重新设置一个 cookie,默认为 false                   |
  | saveUninitialized | 是指无论有没有 session cookie,每次请求都设置个 session cookie,默认给个标示为 connect.sid |
  | resave            | 是指每次请求都重新设置 session coiecookie假设你的是10分钟过期,每次请求都会再设置10分钟 |
  
    注：store在未设置时默认保存在内存中，若需保存在其它地方，需使用第三方或自定义库，该库需提供set和get等方法，可自定义一个store，将session保存在文件中
```js
//   ./store文件
let util = require('util');
let fs = require('fs');
let path = require('path');
let mkdirp = require('mkdirp');
module.exports = function (session) {
    let Store = session.Store;
    util.inherits(FileStore, Store);
    function FileStore(options) {
        //Store.call(this);
        let { root } = options;//root就是存放session文件的根目录 
        this._maxAge = options.maxAge || 0;;//最长存活时间
        this.root = root;
        mkdirp.sync(root); //根据路径创建对应文件或文件夹
    }

    FileStore.prototype.resolve = function (sid) {
        return path.join(this.root, sid + '.json');
    }
    FileStore.prototype.set = function (sid, session, callback) {
        fs.writeFile(this.resolve(sid), JSON.stringify(session), callback)
    }
    FileStore.prototype.get = function (sid, callback) {
        fs.readFile(this.resolve(sid), 'utf8', function (err, data) {
            if (err) callback(err);
            data = JSON.parse(data);
            callback(null, data);
        });
    }
    FileStore.prototype.destroy = function (sid, callback) {
        fs.unlink(this.resolve(sid), callback);
    }
    return FileStore;
}
```

- 当使用了session中间件后，

  - **设置**session的方法是：` req.session.count = count;`
  - **获取**session的方法是：session = req.session.count

  ```js
  app.get('/', function (req, res) {
      let count = req.session.count; //获取
      console.log(count);
      if (count) {
          count = count + 1;
      } else {
          count = 1;
      }
      req.session.count = count; //设置
      res.send(`欢迎你的第${count}次访问`);
  });
  ```

# koa

## 1、基本使用

```js
const Koa = require('koa');
const app = new Koa();
//koa推荐使用async
//ctx(context)是koa提供一个对象，包括一些常见的方法和属性
app.use(async function (ctx, next) {
    console.log(1);
    await next();
    console.log(2);
});
app.use(async function (ctx, next) {
    console.log('a');
});
// 1 a 2
app.listen(8080);
```

## 2、获取请求参数

```js
const Koa = require('koa');
const app = new Koa();
//ctx上有koa封装的request和response，还有原生的req和res
app.use(function (ctx, next) {
    console.log(ctx.method);
    console.log(ctx.url);
    console.log(ctx.headers);
    console.log(ctx.type); //请求体的内容类型
    console.log(ctx.querystring);//原始的查询字符串
    console.log(ctx.query); //查询字符串转成对象
    //koa中使用ctx.res.end或ctx.res.write方法无效
    ctx.body = ctx.headers; //设置请求响应体，请求体可以是字符串，Buffer，对象或流
    //ctx.response.body = ctx.headers; //与上述方法相同
});
app.listen(8080);
```

## 3、解析请求体

> 可使用第三方库`koa-bodyparser`,但该库无法解析文件，`koa-better-body`可以解析文件，两者使用方法相同

```js
const Koa = require('koa');
const path = require('path');
const convert = require('koa-convert'); // 将默认的generator中间件转成koa2的中间件
//const bodyParser = require('koa-bodyparser'); //不能解析文件
const bodyParser = require('koa-better-body');
const app = new Koa();
app.use(convert(bodyParser({
    uploadDir: path.join(__dirname, 'uploads') //指定上传文件保存的目录 
})));
app.listen(3000);

//如表单需上传文件，就需要给表单增加一个enctype="multipart/form-data"
app.use(async function (ctx, next) {
    if (ctx.url == '/user' && ctx.method == 'POST') {
        //当使用了bodyparser中间件后，会将请求体解析并赋给ctx.request.body
        ctx.body = ctx.request.fields; //获取请求体中的字段
    } else {
        await next();
    }
});
```

## 4、cookie

- `ctx.cookies.get(name,[optins])` //读取上下文请求中的 cookie

- `ctx.cookies.set(name,value,[options]` //在上下文中写入 cookie

  - domain: 写入 cookie所在的域名

  - path:写入 cookie所在的路径

  - maxAge: Cookie最大有效时长

  - expires: cookie失效时间

  - httpOnly:是否只用http请求中获得

  - overwirte: 是否允许重写
  
  ```js
  app.use(async (ctx, next) => {
    console.log(ctx.ur1);
    if (ctx.url == '/write') {
      ctx.cookies.set('name', 'joiner');
      ctx.body = 'write'
    } else {
      next()
    }
  })
  app.use(async (ctx) => {
    if (ctx.url == '/read') {
      ctx.body = ctx.cookies.get('name')
    }
  })
  ```
  
  

## 5、session

> npm install koa-session

```js
const Koa = require('koa')
const session = require('koa-session');
const app = new Koa();
app.keys = ['joiner'];
app.use(session({}, app));
app.use(async (ctx) => {
  let visit = ctx.session.visit;
  if (visit) {
    visit = visit + 1;
  } else {
    visit = 1;
  }
  ctx.session.visit = visit;
  ctx.body = `这是你的第${visit}次访问`
});
app.listen(3000);
```

## 6、模板引擎

> npm i koa-view ejs -S

```js
const Koa = require('koa');
const views = require('koa-views');
const path = require('path')
const app = new Koa();
app.use(views(path.join(_dirname, './views'), {
  extension: 'ejs'
}));

app.use(async ctx => {
  await ctx.render('index', { name: '前端开发' })
});

app.listen(3000, () => {
  console.log('server is starting at port 3000');
});
```

## 7、静态资源中间件

> npm i koa-static -S

```js
const static = require('koa-static')
const app = new Koa()
app.use(static(path.join(__dirname, 'public')))
app.use(async (ctx) => {
  ctx.body = 'Not Found'
})
```

## 8、koa简单实现

```js
class Koa {
    constructor() {
        this.middleware = [];
    }
    use(fn) {
        this.middleware.push(fn);
    }
    listen(port) {
        let middleware = this.middleware;
        require('http').createServer((req, res) => {
            let ctx = { req, res };
            // koa2.0 3.0原理
            dispatch(0);
            function dispatch(idx) {
                middleware[idx](ctx, () => next(idx + 1));
            }
            //koa1.0原理
            // (middleware.reduceRight((val, item) => {  
            //     return item.bind(null, ctx, val);
            // }, (function () { })))()
        }).listen(port);
    }
}
module.exports = Koa;
```

## 9、常用中间件

| 中间件         | 作用                                                |
| :------------- | :-------------------------------------------------- |
| koa-body       | 可以实现文件上传，同时也可以让koa能获取post请求参数 |
| koa-json-error | 错误处理中间件 可配置环境是否返回stack              |
| koa-parameter  | 校验请求参                                          |
| koa-router     | 路由中间件                                          |
| koa-static     | 处理静态资源                                        |
| koa-jwt        | 权限控制中间件                                      |
| koa-log4       | 日志处理模块                                        |
| jsonwebtoken   | 用于鉴权与认证                                      |
| mongoose       | 链接mongodb数据库                                   |
| nodemon        | 检测本地文件改动自动重启服务                        |
| cross-env      | 运行跨平台设置和使用环境变量的脚本。                |
| xml2js         | 将xml转换为js对象                                   |
| sha1           | sha1加密算法                                        |

# mongoDB

## 1、介绍

-  MongoDB是一个基于分布式文件存储的开源数据库系统
-  MongoDB将数据存储为一个文档,数据结构由键值(key=>value)对组成。 MongoDB文档类似于JSON对象。字段值可以包含其他文档,数组及文档数组。

## 2、安装

> window下直接官网安装，可安装可视化工具Robomongo或mongoVUE
>
> bin目录中，mongo是客户端，mongod是服务端，

将mongoDB安装目录的bin目录配置在环境变量中

## 3、启动和连接

1. 创建一个数据库所在目录，如： D:/mongodb/data ，在命令行输入`mongod --dbpath=刚创建的目录`

2. 通过配置项启动

   | 参数      | 含义                            |
   | --------- | ------------------------------- |
   | --dbpath  | 指定数据库文件的目录            |
   | --port    | 端口 默认是27017 28017          |
   | --fork    | 以后台守护的方式进行启动        |
   | --logpath | 指定日志文件输出路径            |
   | --config  | 指定一个配置文件                |
   | --auth    | 以安全方式启动数据库,默认不验证 |

   ```js
   //mongo.conf文件
   dapath=D:\mongo\data   //data为文件夹，须先创建
   logpath=D:\mongo
   port=5000
   ```

   **`mongod --config mongo.conf`来启动**   

   **`mongo --port 5000`来连接**

## 4、配置服务（可选）

- 在系统服务面板可配置一个mongodb的服务，方便的启动或关闭

1. 删除服务 `sc delete mongodb`

2. 添加服务

   ```js
   //port:指定端口号，默认270017  dbpath:数据存放目录  logpath:日志目录  logappend:日志是追加模式
   //directoryperd:针对 每个数据库创建单独文件夹  serviceName:服务名称
   mongod.exe --port 8080 --dbpath E:\mdata --logpath E:\mlog --logappend --directoryperdb --serviceName mongodb --install
   
   net start mongodb
   net stop mongodb
   ```

- 使用命令行启动服务

  ```js
   mongod --dbpath=./data  // 指定数据库所在目录
   mongod --dbpath=./data --auth    //安全方式启动数据库,连接时需用户名和密码
  ```

- 使用命令行连接服务并简单操作

  ```js
  mongo //启动连接数据库
  exit //退出连接
  ```

## 5、基本概念

- 数据库：MongoDB的单个实例可以容纳多个独立的数据库,比如一个学生管理系统就可以对应一个数据库实例

- 集合：数据库是由集合组成的,一个集合用来表示一个实体如学生集合

- 文档：集合是由文档组成的,一个文档表示一条记录比如一位同学张三就是一个文档

## 6、数据库操作

```js
show dbs  //查看显示所有的数据库  
db  //或db.getName()查看当前操作的数据库
use school    //切换到指定的(school)数据库(若没有会新建一个)
db.createCollection('students')  //在school中创建一个students的数据库
db.dropDatabase()  //删除数据库

db.shutdownServer()  //关闭数据库
```

> 插入的数据会自动生成一个ObjectId，使用 MySQL等关系型数据库时,主键都是设置成自增的。但在分布式环境下,这种方法就不可行了,会产生冲突。为此MongoDBObjectld采用了一个称之为的类型来做主键 Objectld是一个2字节的BSON类型字符串。按照字节顺序,依次代表：
>
> - 4字节:UNX时间戳
>
> - 3字节:表示运行 MongoDB的机器
>
> - 2字节:表示生成此id的进程
>
> - 3字节:由一个随机数开始的计数器生成的值

## 7、集合操作

### 7.1 查看查询插入数据

| 语法                                     | 含义                               |
| ---------------------------------------- | ---------------------------------- |
| show collections                         | 查看当前数据库下的所有集合         |
| db.students.help()                       | 查看帮助                           |
| db.students.count()                      | 查看某一集合数据长度               |
| db.students.insert({name:'node',age:10}) | 向数据库中插入一条数据，可以为数组 |
| db.students.save({_id:2,name:'joiner'})  | 相当于insert or update             |
| db.students.find()                       | 查询students的数据                 |

### 7.2更新文档

```js
db.collection.update(
	<query>,  //query查询条件,指定要更新符合哪些条件的文档，也有相关操作符
    //$set  {$set:{age:19}} 直接指定更新后的值,$unset删除该属性
    //$inc  {$inc:{age:1}} 在原基础上累加
    <updateOjb>, //更新后的对象，也可指定一些更新的操作符
    {
    	upsert:<boolean>, //可选,如果不存在符合条件的记录时是否插入updateObj，默认是 false不插入
	    multi:<boolean> //mongodb默认只更新找到的第一条记录,如果这个参数为true就更新所有符合条件的记录
    }
)
```

- 普通数据使用案例

  ```js
  db.students.update(
      {name:'joiner'},
      //{age:18}, 将第一条name为'joiner'的数据改为{age:32},覆盖除id外的所有数据
      //{$set:{age:19}}, 将第一条name为'joiner'的数据的age改为44,只修改age,若没有age,则添加该属性
      //{$unset:{age:19}}, 将第一条name为'joiner'的数据的age属性删除掉
      //{$inc:{age:1}}, 将第一条name为'joiner'的数据的age加1后保存，只修改age
      {
          upsert:true，//若没找到查询条件的记录，则增加一条
          multi:true  //根据查询条件查出所有的进行操作
      }
  )
  ```

- 带数组的使用案例

  ```js
  db.students.update(
      {name:'joiner'},
      //$push {$push:{hobby:'smoking'}} 向hobby的数组push一条数据
      //$pop {$pop:{hobby:1}} 删除hobby的数组的最后一条数据
      //$addToSet {$addToSet:{hobby:'play'}} 向数组中添加一项，若数组中已有该项则不添加
      //$each 循环向数组中插入一些数据
      {
          $set:{'hobby.1':'play'}, //从查询到的数据中，将hobby数组索引为1的值更改为'play'
          $push:{hobby:'smoking'}, //向hobby的数组push一条数据
          $pop:{hobby:1}, //删除hobby的数组的最后一条数据
  	    $addToSet:{hobby:'play'}, //数组hobby中添加一项，若数组中已有该项则不添加
          $addToSet:{hobby:{$each:['fish','travel']}} //将'fish'和'travel'遍历后添加到hobby中
      }
  )
  ```


### 7.3 使用命令 runCommand

`db.runCommand()` 可在连接数据库的命令行操作，也可执行文件

1. 先定义一个命令文件为index.js

   ```js
   var command = {
     findAndModify: 'students', //要操作的集合
     query: { name: 'joiner' },// 查询条件, 指定操作的集合的范围
     update: { $set: { age: 1010 } }, //指定如何更新,就是把年龄改为100岁
     fields: { age: true, name: true, _id: false }, // 指定返回的字段,false为不返回
     sort: { age: 1 }, //按age进行升序排列
     new: true // 是否返回更新后的文档，true为是
   }
   var db = connect('school') //连接数据库
   var result = db.runCommand(command) //执行命令返回结果
 printjson(result)
   ```

   批量向数据库中插入1000条数据

   ```js
   var db = connect('school') //连接数据库
   var stu = []
   for (var i = 0; i < 1000; i++) {
     stu.push({ name: 'joiner' + i, age: i })
   }
 db.students.insert(stu)
   ```

2. 在命令行执行`mongo ./index.js`

3. 若已使用mongo连接了数据库，则直接使用`load(./index)`即可

### 7.4删除文档

```js
db.collection.remove(
	<query>, //可选，删除文档条件
    {
	    justOne:<boolean> //可选，设为true或1，则只删除匹配到的多个文档中的第一个
    }
)

db.students.remove({name:'joiner'})

db.runCommand({drop:'students'}) //用命令的方法删除
```

### 7.5查询文档

- 查询所有文档

  ```js
  db.students.find()
  db.students.find({name:'joiner'}) //查询所有name为joiner的
  //正则查询
  db.students.find({name:/j.*r/}) //查询name符合正则的
  ```

- 查询指定列

  ```js
  //字段值为1表示除本字段外其它字段不返回，值为0表示除本字段外其它字段都返回
  db.students.find({queryWhere},{name:1,age:1}) 
  ```

- 查询一条 **findOne**

  ```js
  db.students.findOne({name:'joiner'})  //查询name为joiner的
  ```

- 查找包含条件的数据 **$in**

  ```js
  db.students.find({age:{$in:[30,100]}},{name:1,age:1})
  ```

- 查找不包含条件的数据 **$nin**

  ```js
  db.students.find({age:{$nin:[30,100]}},{name:1,age:1})
  ```

- 取反 $not，将条件使用$not包起来

  ```js
  db.students.find({age:{$not:{$in:[30,100]}}}) //查询不包含30和100的数据，效果与$nin相同
  db.students.find({age:{$not:{$gt:20,$lt:70}}}) //查询不大于30和不小于100的数据
  ```

- 大于小于，大于等于，小于等于

  ```js
  db.students.find({age:{$gt:50,$lt:70}}) //查询age大于50小于50的值
  db.students.find({age:{$gte:50,$lte:70}}) //查询age大于等于50，小于等于70的值
  ```

- 数组操作

  ```js
  //按所有元素匹配 hobby有且只包含"smoking"和"read"
  let result = db.student.find({hobby:["smoking", "read"] });
  //匹配一条
  let result = db.student.find({hobby:"read" });
  //$al1 同时具备两个条件，hobby包含smoking和read
  let result = db.student.find({hobby:{$all: ['smoking', "read"] } });
  //$in  hobby包含smoking或read
  let result = db.student.find({hobby:{$in: ['smoking', "read"] } })
  //$size  hobby数组长度为4
  let result = db.student.find({hobby:{$size: 4 } });
  //$slice  查询hobby长度为5的值，并返回hobby数组的截取项
  let result = db.student.find({hobby:{$size:5}}, {hobby:{$slice:1}, _id: false});
  let result = db.student.find({hobby:{$size:5}}, {hobby:{$slice:-1}, _id: false})
  ```

- $where 万能查询（效率不高）

  ```js
  //$where后跟一个js语句，this代表当前文档，语句为true的内容返回
  db.students.find({$where:"this.age>30&&this.age<100"})
  db.students.find({$where:"this.age%7==0"}) //age与7取余为零的返回
  ```

- cursor 游标

  ```js
  var db = connect('school')
  //返回的是一个游标,指向结果集的一个指针
  var cursor = db.students.find()
  // while (cursor.hasNext()) {  //可以使用while遍历，也可使用forEach遍历
  //   var record = cursor.next()
  //   printjson(record);
  // }
  cursor.forEach(function (item) {
    printjson(item)
  })
  ```

- $or 或查询

  ```js
  db.students.find({$or:[{name:'joiner'},{age:'100'}]}) //查询name为joiner或age为100的
  //查询name为joiner,age为18或100的
  db.students.find({name:'joiner',$or:[{age:'18'},{age:'100'}]})
  ```

## 8、分页查询

```js
//skip跳过几条，limit查询几条，sort排序,1为正序，-1为倒序
db.students.find().skip(3).limit(3) //跳过3条再查询3条
db.students.find().sort({age:1}).skip(3).limit(3) //跳过3条再查询3条并将结果按age的正序排列
```

## 9、导入导出数据

| 参数             | 含义               |
| :--------------- | :----------------- |
| -h[--host]       | 连接的数据库       |
| --port           | 端口号             |
| -u               | 用户名             |
| -p               | 密码               |
| -d[--db]         | 导出的数据库       |
| -c[--collection] | 指定导出的集合     |
| -o               | 导出的文件存储路径 |
| -q               | 进行过滤           |

- mongoimport：导入数据

  ```js
  mongoimport -d school -c students stu.bak	//将stu.bak导入到school中的students中
  ```

- mongoexport：导出数据 ，会将集合导出一个json文档，若二进制文件会出错

  ```js
  mongoexport -d school -c students stu.bak  //将school中的students导出到stu.bak中
  ```

- mongodump 将数据库备份，二进制文件也按二进制备份

  ```js
  mongodump -o mdmp  //将整个数据库备份
  mongodump -d school -o mdmp //将school备份至mdmp文件夹中
  ```

- mongorestore 恢复数据库

  ~~~js
  mongorestore -o mdmp   //将整个数据库恢复
  mongorestore -d school mdmp/school //将mdmp/school恢复到school数据库中
  ~~~

## 10、锁定和解锁数据库

```js
db.runCommand({fsync:1,lock:1}) //清空缓存区，将锁定数据库，此时读取和写入将暂停
db.fsyncUnlock() //解锁数据库
```

## 11、安全措施

### 11.1 查看角色 

```js
show roles
```

内置角色

- 数据库用户角色:read、 readWrite

- 数据库管理角色: dbAdmin、 dbowner、 userAdmin

- 集群管理角色: clusterAdmin、 clusterManager、 clusterMonitor 、hostManage

- 备份恢复角色: backup、 restore; 

- 所有数据库角色: readAnyDatabase、 readWriteAnyDatabase、 userAdminAnyDatabase、 dbAdminAnyDatabase 

- 超级用户角色:root 

- 内部角色: __system

### 11.2 创建用户

```js
use school //先切换到数据库，
db.createUser({
    user:'joiner', 
    pwd:'123456',
    roles:[{   //设置权限
        db:'school', //创建该数据库的用户
        role:'read'	 //该用户对该数据库的权限
    },
     'read', //如不设置具体db，则其它数据库均有可读权限
  ]
})
```

## 12、高级命令

### 12.1查找不重复的值

```js
db.runCommand({distinct:'students',key:'home'}) //查询数据库students中key为home的数据
```

### 12.2 分组group

```js
db.runCommand({
    group:{
        ns:'集合名称',
        key:'分组的键',
        initial:'初始值',
        $reduce:'分解器',
        condition:'条件',
        finalize:'完成时的处理器'
    }
})
//案例 
db.runCommand({
  group: {
    ns: 'students',
    key: { home: true },
    initial: { total: 0 },
    $reduce: function (doc, result) {
      result.total += doc.age;
    },
    condition: { age: { $gt: 1 } },
    finalize: function (result) {
      result.desc = '本城市的总年龄为' + result.total;
    }
  }
});
```

### 12.3 runCommand常用命令

```js
db.runCommand({buildInfo:1}) //查看信息
db.runCommand({getLastError:'students'}) //查看students数据库最后一次的错误
```

## 13、固定集合

集合的文档数和文档大小是固定的，超过固定大小后，后插入的数据会覆盖之前的数据，通过 createCollection来创建一个固定集合,且 capped选项设置为true

```js
db.logs.isCapped() //判断logs集合是否为固定集合

//size是整个集合空间大小,单位为【KB】,max是集合文档个数上线,单位是【个】, capped封顶，给true
db.createCollection('logs', {size: 50, max: 5, capped: true});

db.runCommand({convertToCapped: "logs",size: 5)); //非固定集合转为固定集合
```

## 14、文件系统gridfs

gridfs是mongodb自带的文件系统,使用二进制存储文件，可以以BSON格式保存二进制对象，BSON对象的体积不能超过4M。所以 mongodb提供了 mongofiles。它可以把一个大文件透明地分割成小文件(256K),从而保存大体积的数据。

- GridFS用于存储和恢复那些超过16M(BSON文件限制)的文件(如:图片、音频、视频等)。

- GridFS用两个集合来存储一个文件:fs. files与fs. chunks

- 每个文件的实际内容被存在 chunks(二进制数据)中,和文件有关的meta数据(filename,content_type,还有用户自定义的属性)将会被存在 files集合中。

```js
//将test.txt上传到myfiles中
mongofiles -d myfiles put test.txt  //-d数据库名称  -l源文件位置   -put指定文件名

mongofiles -d myfiles get test.txt //获取并下载文件

mongofiles -d myfiles list //查看所有文件

mongofiles -d myfiles delete test.txt //删除文件
```

- eval服务器端脚本,可以执行js语句，定义js全局变量，定义函数

  ```js
  db.eval("1+1");
  db.eval("return hello'");
  db.system.js.insert({ _id: "x", value: 1 });
  db.eval("return x")
  db.system.js.insert({ id: "say", value: function () { return 'hello' } });
  db.eval("say()");
  ```

## 15、索引

## 16、mongoose

## 17、插件，聚合，虚拟属性等





# myslq

## 1、安装和配置 

官网下载安装，配置文件：安装目录的`my.ini`文件

- port端口号
- basedir安装目录
- datadir数据存放访目录
- charcter-set-server字符集
- default-storage-engine 存储引擎
- sqlmode 语法模式
- max-connections 最大连接数

## 2、基本操作

```js
net start MySQL  //启动数据库，实际启动的是安装目录下的mysqld可执行文件
net stop MySQL  //停止数据库

mysql -h 127.0.0.1 -p3306 -uroot -p123456   //连接数据库
exit //断开连接

show databases; //查看已有数据库
use test; //切换数据库

show tables //显示有哪些表
show tables from mysql  //查看mysql中有哪些表

desc student  //查看当前数据库中的表结构
select database(); //查看并返回当前数据库
```

## 3、数据完整性

日期时间型：year   timstamp   time  date   datetime

字符串型：set	enum	blob	text	varchar	char

数值型：整数 tinyint	smallint	mediumint	int	bigint	

​			   小数--浮点：float	Double

​			   小数--定点：decimal

## 4、SQL

### 1.查询语句

```sql
-- ASC：升序   DESC：降序
SELECT <列名> FROM <表名> [WHERE <查询条件表达式>] [ORDER BY <排序的列名>[ASC或DESC]]
-- CONCAT是一个内置函数，可以实现字符串的连接
SELECT CONCAT(firstName,lastName) FROM users;
--操作符 AND OR NOT BETWEEN
-- age大于10并且小于20
SELECT * FROM users WHERE age>10 AND age<20  
-- age不大于10并且不小于20
SELECT * FROM users WHERE NOT(age>10 AND age<20) 
-- age在20至50中间
SELECT * FROM users WHERE age BETWEEN 20 AND 50 
-- 查询年龄等于12或18
SELECT * FROM users WHERE age IN (12,18)
-- 查询年龄不等于12或18
SELECT * FROM users WHERE age NOT IN (12,18)
-- 查询home为‘北京’，按年龄进行降序排列
SELECT * FROM users WHERE home='北京' ORDER BY age DESC
--查询今天过生日的人
SELECT * FROM users WHERE DAY(NOW()) = DAY(birthday) AND MONTH(NOW()) = MONTH(birthday)
--根据生日查询年龄
SELECT name,YEAR(NOW())-YEAR(birthday) FROM users
--模糊搜索
SELECT * FROM Persons WHERE City LIKE 'N%'   //搜索城市名为N开头
SELECT * FROM Persons WHERE City LIKE '%g'   //搜索城市名为g结尾
SELECT * FROM Persons WHERE City LIKE '%lon%'  //搜索城市名包含lon
SELECT * FROM Persons WHERE City NOT LIKE '%lon%'  //搜索城市名不包含lon
```

### 2. 插入语句

```sql
INSERT [INTO] 表名 [(列名)] VALUES (值列表)
-- 向表中插入一条数据，可以省略列名，但需输入完整的字段（包含列名）
INSERT INTO users(firstName,lastName,age,birthday) VALUES('赵','六','100','1984-2-12')
```

### 3.更新语句

```SQL
UPDATE 表名 列名=更新值 [WHERE 更新条件]
--更新语句
UPDATE users SET age=16,birthday='1993-09-03' WHERE id=3
-- 将email为null的列更新
UPDATE users SET email = '7@qq.com' WHERE email IS NULL 
-- 将email不为null的列更新
UPDATE users SET email = '7@qq.com' WHERE email IS NOT NULL 
```

### 4.删除语句

如果在删除主表数据，则要先删除子表记录

```sql
DELETE [FROM] 表名 [WHERE <删除条件>]
--删除id为7的记录
DELETE FROM users WHERE id=7
--删除整张表(主键会保留,能恢复)
DELETE FROM users
--删除整张表(全部删除，无法恢复)
TRUNCATE TABLE users
```

### 5.函数

#### 5.1字符函数

| 函数名称  | 描述                           |语句|
| --------- | ------------------------------ | --------- |
| CONCAT    | 字符串连接                     |SELECT CONCAT(last_name,'_',first_name) FROM users|
| CONCAT_WS | 使用指定分隔符进行字符连接     ||
| FORMAT    | 数字格式化                     |SELECT FORMAT(10000.178,2)|
| LOWER     | 转小写字母                     |SELECT LOWER('ABC') FROM users|
| UPPER     | 转大写字母                     |SELECT UPPER('abc') FROM users|
| LEFT      | 返回字符串s开始的最左边n个字符 |SELECT LEFT('abc123',3)|
| RIGHT     | 返回字符串s开始的最左边n个字符 |SELECT RIGHT('abc123',3)|
| SUBSTR | 截取字符串 |SELECT SUBSTR('zfpx',2)<br />SELECT SUBSTR('zfpx',2,3)|

#### 5.2数字函数

| 函数名称 | 描述       | 语句                                           |
| -------- | ---------- | ---------------------------------------------- |
| CEIL     | 向上取整   | SELECT CEIL(1.2)                               |
| FLOOR    | 向下取整数 |                                                |
| DIV      | 取整数     |                                                |
| MOD      | 取余       |                                                |
| POWER    | 幂运算     |                                                |
| ROUND    | 四舍五入   | SELECT ROUND(2.5)  <br />SELECT ROUND(2.555,2) |
| TRUNCATE | 数字截取   | SELECT TRUNCATE(12.12312,3)                    |

#### 5.3日期函数

| 函数名称     | 描述                 | 语句                                                         |
| ------------ | -------------------- | ------------------------------------------------------------ |
| NOW          | 当前日期和时间       | SELECT NOW()                                                 |
| CURDATE      | 当前日期             |                                                              |
| CURTIME      | 当前时间             |                                                              |
| DATE_ADD     | 日期增加             | SELECT DATE_ADD(NOW(),INTERVAL 3 MONTH)<br />//当前日期增加3个月 |
| DATEDIFF     | 计算两个时间的时间差 | SELECT DATEDIFF(DATE_ADD(NOW(),INTERVAL 3 DAY),NOW())        |
| DATE_FORMATE | 日期格式化           | SELECT DATE_FORMAT(NOW(),'%Y年%m月%d日  %H时%i分%s秒')       |

#### 5.4其它函数

| 名称           | 描述                     | 语句                                                         |
| -------------- | ------------------------ | ------------------------------------------------------------ |
| CONNECTION     | 当前连接数据的ID号       | SELECT CONNECTION_ID()                                       |
| DATABASE       | 查看当前数据库           | SELECT DATABASE()                                            |
| VERSION        | 查看数据库版本号         | SELECT VERSION()                                             |
| LAST_INSERT_ID | 查看最后一条插入数据的id | SELECT LAST_INSERT_ID()                                      |
| user           | 查看当前登录的用户名     | SELECT USER()                                                |
| MD5            | 获取一个值的摘要算法     | SELECT MD5('123456')                                         |
| PASSWORD       | 修改当前用户的密码       | SELECT PASSWORD('123456')<br />SELECT User,Password from mysql.user |

#### 5.5流程控制函数

| 名称 | 描述 | 语句                                                         |
| ---- | ---- | ------------------------------------------------------------ |
| IF   | 判断 | SELECT IF(gender=1,'男','女') FROM users                     |
| CASE | 判断 | SELECT <br />students,CASE grade <br />WHEN grade>90 then '优秀'<br />WHEN grade>70 then '良好'<br />ELSE '不及格'<br />FROM users |

#### 5.6模糊查询

| 通配符 | 解释                             | 示例                      | 符合条件的值                 |
| ------ | -------------------------------- | ------------------------- | ---------------------------- |
| _      | 一个字符                         | LINK 'a_'                 | as，ad等                     |
| %      | 任意长度的字符串                 | LIKE 'c%'<br />LIKE '%c%' | code，cat等<br />所有包含c的 |
| []     | 括号中所指定范围内的一个字符     | LIKE '1[35]5'             | 135或155                     |
| [^]    | 不在括号中所指定范围内的一个字符 | `LIKE '1[^1-2]5'`         | 135或185等                   |

### 6.聚合查询

对一组值进行计算，并返回计算后的值，一般用来统计数据

```sql
-- 在score表中查询grade的和
SELECT SUM(grade) FROM score
-- 查询grade的最大值
SELECT MAX(grade) FROM score
-- 查询grade的最小值
SELECT MIN(grade) FROM score
-- 查询grade的最平均值，并四舍5入
SELECT ROUND(AVG(grade),0) FROM score
-- 查询数据的条数(不包含null)
SELECT COUNT(grade) FROM score
-- 查询数据的条数(只要一行有数据就计作一条)
SELECT COUNT(*) FROM score
```

### 7.分组

按某的值进行分组

```sql
-- 注：SELECT列表中只能包含被分组的列
SELECT 列名，表询表达式 FROM <表名> WHERE <条件> GROUP BY <分组字段> 
HAVING 分组后的过渡条件 ORDER BY 列名 [ASC,DESC] LIMIT 偏移量，条数

-- 从score表中将结果按student_id分组，并返回stduent_id和成绩（grade）的平均值
SELECT student_id,AVG(grade) FROM score GROUP BY student_id;
```

### 8.子查询

查询语句中用小括号嵌套语句

```sql
-- 查询年龄大于平均年龄的学生
SELECT * FROM sudent WHERE age > (SELECT ROUND(AVG(age),2) FROM student)
```

### 9.表连接

```sql
-- 从score,student,course三张表中查询 score.student_id = studend.id AND score.course_id = course.id的数据
SELECT student.name,course.name,score.grade FROM score,student,course 
WHERE score.student_id = studend.id AND score.course_id = course.id

-- 等同于
SELECT student.name,course.name,score.grade FROM score 
INNER JOIN student on score.student_id = student.id 
INNER JOIN course on score.course_id = course.id
```







# TypeScript

## 1、安装

```js
npm install typescript -g //全局安装
tsc demo.ts  //安装后编译

npm install -g ts-node  // 全局安装支持ts的node 可直接运行ts文件

tsc -init //生成配置文件
tsc // 对当前文件夹下的ts文件使用tsconfig.json进行编译
tsc -w // 监视文件变化自动打包
```

## 2、静态类型

- 定义静态类型

  ```js
  const count: number = 1
  const str:string = 'hello'
  ```

- 自定义静态类型

  ```ts
  interface Person {
    name: string;
    age: number;
  }
  const xiaomeng: Person = { // 自定义对象
    uname: "小明",
    age: 18,
  };
  //数组类型
  const persons: String[] = ["谢", "刘", "小"]  // 自定义数组
  const persons: Array<string> = ["谢", "刘", "小"] //效果等同
  //类类型
  class Person {}
  const dajiao: Person = new Person();  // dajiao必须是一个Person类对应的对象
  // 函数类型
  const XiaoJian: () => string = () => {  
    return "大脚";
  };
  
  const da : Date = new Date() //日期类型
  const reg : RegExp = new RegExp('/d','gi')
  ```

## 3、类型注释和类型推断

- 类型注解，显示的告诉代码变量的类型
- 类型推断，根据赋值返回值信息等推断变量类型

## 4、函数参数和返回类型定义

- 普通函数参数和返回类型

  ```ts
  function getTotal(one: number, two: number): number {
    return one + two;
  }
  const total = getTotal(1, 2);
  
  let mySum: (x: number, y: number) => number = function (x: number, y: number): number {
      return x + y;
  };
  ```

- 参数是对象（使用解构）的注解方式

  ```js
  function add({ one, two }: { one: number, two: number }): number {
    return one + two;
  }
  const total1 = add({ one: 1, two: 2 })
  ```

## 5、数组类型的定义

- 一般数组类型定义

  ```ts
  const numberArr: number[] = [1, 2, 3, 4]
  const stringArr: string[] = ["a", "b", "c"];
  const undefinedArr: undefined[] = [undefined, undefined];
  const arr: (number | string)[] = [1, "string", 2];
  ```

- 数组中对象类型的定义

  ```ts
  const persons: { name: string, age: Number }[] = [ //原始，较复杂
    { name: "刘", age: 18 },
    { name: "谢", age: 28 },
  ]
  // 定义一个type
  type Person = { name: string, age: Number };
  const persons: Person[] = [ 
    { name: "刘", age: 18 },
    { name: "谢", age: 28 },
  ]
  //也可以使用类
  class Person {
    name: string;
    age: number;
  }
  const persons: Person[] = [ 
    { name: "刘", age: 18 },
    { name: "谢", age: 28 },
  ]
  ```

## 6、元组的使用和类型约束

- 元组和数组类似，但是类型注解时会不一样。

```ts
//数组中的每个元素类型的位置给固定住了，这就叫做元组
const person: [string, string, number] = ["dajiao", "teacher", 28];
```

## 7、interface 接口

- 接口代表对象，与type有一定的区别

  ```ts
  interface Girl {
    name: string;
    age: number;
    height: number;
    weight?: number;  //可选
    [propName: string]: any; //允许加入任意值
    say(): string;  //方法
  }
  const girl:Gril = {
    name: '大脚',
    age: 18,
    height: 160,
    weight:55,
    sex:'女',
    say(){return 'hello'}
  }
  ```

- 接口和类的约束

  ```ts
  class Xiaohong implements Girl {
    name = "小红";
    age = 18;
    height = 160;
    say() {
      return "hello";
    }
  }
  ```

- 接口间的继承

  ```ts
  interface Teacher extends Girl {
    teach(): string;
  }
  const girl = {
    name: "小红",
    age: 18,
    height: 160,
    weight:55,
    sex: "女",
    say() {
      return "hello";
    },
    teach() {
      return "word";
    },
  };
  ```

## 8、类的概念和使用

- 类的概念和 ES6 中原生类的概念大部分相同

  ```ts
  class Lady {  //定义类
    content = 'hi'
    sayHello() {
      return this.content
    }
  }
  class Xiaohong extends Lady {  // 继承
    sayLove() {
      return 'I love you'
    }
    sayHello() { // 类方法重写
      return super.sayHello() + ',你好！'  // super关键字代表父类中的方法
    }
  }
  const goddess = new Xiaohong()
  console.log(goddess.sayHello());
  console.log(goddess.sayLove());
  ```

- public 公共的,允许在类的内部和外部被调用.

- private 只允许再类的内部被调用，外部不允许调用

- protected 允许在类内及继承的子类中使用

  ```js
  class Person {
    public name: string = 'joiner'
    private sex: string = '男'
    protected weight: number = 65
    public sayName() {
      console.log(this.name + 'sayHello');
    }
    public saySex() {
      console.log(this.sex);
    }
  }
  const person = new Person()
  person.name = 'jonee' //公共的
  person.sayName() 
  console.log(person.name);
  
  class Teacher extends Person {
    public sayWeight() {
      console.log(this.weight)  //允许在类内及继承的子类中使用
    }
  }
  const teacher = new Teacher()
  teacher.sayWeight()
  ```

- 类的构造函数，与es6基本一致

  ```js
  class Person {
    constructor(public name: string) {
    }
  }
  const person = new Person('joiner')
  console.log(person.name); //18
  
  class Teacher extends Person {
    constructor(public age: number) {
      super('joiner')
    }
  }
  const teacher = new Teacher(18)
  console.log(teacher.age); //18
  console.log(teacher.name);//joiner
  ```

- Getter、Setter 和 static及readonly 使用

  ```js
  class Person {
    public readonly _name: string // 定义只读属性_name,该属性不可修改
    constructor(private _age: number, name: string) {
      this._name = name
    }
    get age() {
      return this._age - 10
    }
    set age(age: number) {
      this._age = age + 3
    }
  }
  
  const joiner = new Person(28)
  joiner.age = 25
  console.log(joiner.age)
  
  class Girl {
    static sayLove() {
      return "I Love you";
    }
  }
  console.log(Girl.sayLove()); // 可直接使用类中定义的静态方法
  ```

- 抽象类的使用

```js
abstract class Person { // 抽象类的关键词是abstract,抽象方法也是abstract开头
  abstract skill()  //没有具体的方法，可以不写括号
}

class A extends Person { // 继承自抽象类，须有skill方法
  skill() {
    console.log('A ')
  }
}

class B extends Person { // 继承自抽象类，须有skill方法
  skill() {
    console.log('B')
  }
}

class C extends Person { // 继承自抽象类，须有skill方法
  skill() {
    console.log('C')
  }
}
```

## 9、tsconfig.json

- 生成配ts配置文件

  ```js
  tsc -init //生成配置文件
  tsc // 对当前文件夹下的ts文件使用tsconfig.json进行编译
  ```

- 常用配置

  ```js
  {
    "include": ["13.demo.ts"],
    "exclude": ["12.demo.ts"],
    "compilerOptions": {
      "target": "es6",
      "module": "commonjs",
      "sourceMap": true,         /* Generates corresponding '.map' file. */
      "outDir": "./build",       /* 输出目录*/
      "rootDir": "./",           /* 输入目录 */
      "removeComments": true,    /* 不输出注释 */
  
      "strict": true,            /* 开启严格模式 */
      "noImplicitAny": true,     /* 值为any（任意值），也要进行类型注释。 */
      "strictNullChecks": true,  /* null的校验 */
  
      "noUnusedLocals": true,    /* 报告未使用的局部变量的错误 */
      "noUnusedParameters": true,/* 报告未使用参数的错误。 */
    }
  }
  ```

## 10、联合类型和类型保护

一个变量可能有两种或两种以上的类型，称为联合类型，需要根据具体的类型来做相关操作，需要类型保护

- 类型断言--通过断言的方式确定传递过来的准确值

  ```js
  interface Waiter {
    jump: boolean;
    say: () => {};
  }
  interface Teacher {
    jump: boolean;
    skill: () => {};
  }
  function judgeWho(animal: Waiter | Teacher) {
    if (animal.jump) {  // 根据传入的jump类型来断言具体类型
      (animal as Teacher).skill();
    }else{
      (animal as Waiter).say();
    }
  }
  ```

- 使用`in`语法---用`in`来判断`animal`里有没有`skill()`方法

  ```js
  function judgeWhoTwo(animal: Waiter | Teacher) {
    if ("skill" in animal) { // 判断skill是否在入参中
      animal.skill();
    } else {
      animal.say();
    }
  }
  ```

- 使用typeof语法

  ```js
  function add(first: string | number, second: string | number) { 
    if (typeof first === "string" || typeof second === "string") {  //使用typrof
      return `${first}${second}`;
    }
    return first + second;
  }
  ```

- 使用instanceof来判断

  ```js
  function addObj(first: object | NumberObj, second: object | NumberObj) {
    if (first instanceof NumberObj && second instanceof NumberObj) {
      return first.count + second.count;
    }
    return 0;
  }
  ```

## 11、Enum 枚举类型

​	枚举(`enum`)类型会让程序有更好的可读性

```js
enum Status {
  AA=1, //默认值是0，后续依次叠加，可以给初始值
  BB,
  CC,
}
function getServe(status: any) {
  if (status === Status.AA) {
    return "a";
  } else if (status === Status.BB) {
    return "b";
  } else if (status === Status.CC) {
    return "c";
  }
}
const result = getServe(Status.BB);
console.log(`我要去${result}`);
```

## 12、泛型

泛型就是泛指的类型，使用`<>`（尖角号）定义

#### 12.1函数泛型

1. 简单类型并定义多个泛型

   ```js
   function join<T, P>(first: T, second: P) {
     return `${first}${second}`
   }
   console.log(join<number, string>(123, '312312')); 
   ```

2. 数组

   ```js
   function myFun<T>(params: T[]) {
     return params
   }
   console.log(myFun<string>(['123', '456']));
   
   function myFun1<T>(params: Array<T>) { //效果等同于上述使用方法
     return params
   }
   console.log(myFun1<string>(['123', '456']));
   ```

3. 类型推断（不建议使用）

   ```js
   function join<T, P>(first: T, second: P) {
     return `${first}${second}`;
   }
   join(1, "2");
   ```

#### 12.2 类的泛型

1. 初始类

   ```js
   class SelectGirl<T> {
     constructor(private girl: T[]) { }
     getGirl(index: number): T {
       return this.girl[index]
     }
   }
   const selectGirl = new SelectGirl<number>([1, 2, 3, 4, 5])
   console.log(selectGirl.getGirl(1));
   ```

2. 泛型的继承

   ```js
   interface Girl {
     name: string
   }
   class SelectGirl<T extends Girl>{  //传递过来的参数必须是一个对象，里边还要有name属性
     constructor(private girl: T[]) { }
     getGirl(index: number): string {
       return this.girl[index].name
     }
   }
   const selectGirl = new SelectGirl([{ name: '大' }, { name: '刘' }])
   console.log(selectGirl.getGirl(1));
   ```

3. 泛型约束

   ```js
   class SelectGirl<T extends number | string> { //要求T是number与string类型
     constructor(private girls: T[]) { }
     getGirl(index: number): T {
       return this.girls[index];
     }
   }
   const selectGirl = new SelectGirl(["大", "刘", "晓"]);
   console.log(selectGirl.getGirl(1));
   ```

## 13.命名空间-Namespace

1. 用命名空间实现组件化

   ```ts
   namespace Components {
     class Header { //定义一个类，局部，防止成为全局
       constructor() {
         const elem = document.createElement("div");
         elem.innerText = "This is Header";
         document.body.appendChild(elem);
       }
     }
       
     export class Page {  //导出一个类
       constructor() {
         new Header();
       }
     }
   }
       
   //使用时，引入打包的文件，使用导出的类名即可
    new Home.Page();
   ```

2. 多文件编译成一个文件

   先配置打包出的文件路径和文件名为：`"outFile": "./build/page.js"`，再将配置文件中的`module`更改为`amd `，多个文件将打包成一个

   ```ts
   //components.ts文件
   namespace Components {
     export class Header {
       constructor() {
         const elem = document.createElement("div");
         elem.innerText = "This is Header";
         document.body.appendChild(elem);
       }
     }
   }
     
   //home.ts文件
   namespace Home {
     export class Page {
       constructor() {
         new Components.Header();
       }
     }
   }
   //使用以上配置后将打包出一个文件，使用时
   new Home.Page()
   ```

   