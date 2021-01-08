---
title: node中的path模块常用介绍
date: 2019-03-05 18:56:27
tags:
- node
categories:
- 前端设计
---
对于前端开发的同学相信大家都会用过path这个模块去解析路径，比较遗憾的是，以往都只是百度其某个方法的用法，没有完整的去看过接口文档，导致现在阅读一些代码的时候碰到path的其他方法一脸懵逼，所以趁着项目不忙去看了一下[官方文档](https://nodejs.org/docs/latest/api/path.html#path_path_basename_path_ext)并做个笔记。
<!-- more -->
### path.basename(path)
参数:
* arg1: 字符串类型路径
* arg2: 可选参数,文件拓展名

返回值:arg1的最后一部分

由于此方法在不同的系统解析windows路径出现不一样的效果，可使用path.win32.basename方法代替此方法。
```js
// mac os环境
 
var path = require('path');
 
console.log(path.basename('C:/windows/'));
console.log(path.basename('C:\\windows\\test.js'));
console.log(path.basename('C:/windows/test.js'));
console.log(path.basename('C:/windows/test.js', '.js'));
console.log(path.win32.basename('C:/windows/'));
console.log(path.win32.basename('C:\\windows\\test.js'));
console.log(path.win32.basename('C:/windows/test.js'));
console.log(path.win32.basename('C:/windows/test.js', '.js'));
 
// 结果
/*
windows
C:\windows\test.js
test.js
test
windows
test.js
test.js
test
*/
 
// windows 10 环境
const path = require('path');
 
console.log(path.basename('C:\\windows\\test.js'));
console.log(path.basename('C:/windows/test.js'));
console.log(path.win32.basename('C:\\windows\\test.js'));
console.log(path.win32.basename('C:/windows/test.js'));
 
// 结果
/*
test.js
test.js
test.js
test.js
*/
```

### path.delimiter
参数:多个路径合并的字符串

返回值:系统所用的多个路径合并的分隔符,(POSIX标准系统返回值为':'，Windows系统返回值为：';')

```js
console.log(process.env.PATH);
console.log(process.env.PATH.split(path.delimiter));
 
// mac os
/*
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
[ '/usr/local/bin', '/usr/bin', '/bin', '/usr/sbin', '/sbin' ]
*/
```

### path.dirname(path)
返回路径中代表文件夹路径的字符串

### path,format(object)

```js
console.log(path.format({
    dir: 'home/user/dir',
    root: '/test',
    base: 'file.txt',
    name: 'anotherFile',
    ext: 'md'
}));
 
// home/user/dir/file.txt

{
    dir: string,   // 路径
    root: string,  // 根路径，dir存在时会忽略此参数
    base: string,  // 文件全名，为`${name}.${ext}`
    name: string,  // 文件名，base存在时会忽略此参数
    ext: string    // 文件拓展名，base存在时会忽略此参数
}

```

### path.join([...paths])
参数:字符串数组(如果字符串为空,会被忽略,'.'表示当前路径,'..'表示父级路径)

```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
// Returns: '/foo/bar/baz/asdf'
```

### path.normalize(path)
返回参数路径path的标准路径字符串,简单的解释就是规范化路径

```js
//on POSIX:
path.normalize('/foo/bar//baz/asdf/quux/..');
// Returns: '/foo/bar/baz/asdf'

//On Windows:
path.normalize('C:\\temp\\\\foo\\bar\\..\\');
// Returns: 'C:\\temp\\foo\\'
path.win32.normalize('C:////temp\\\\/\\/\\/foo/bar');
// Returns: 'C:\\temp\\foo\\bar'

```

### path.relative(from, to)
返回from路径到to路径的相对路径

### path.resolve([...paths])
参数:路径字符串数组

返回值:有参数路径组成的一个绝对路径,组成规则如下:
1. 从右往左拼接字符串，如果此过程中拼接结果出现绝对路径，则停止解析，返回此绝对路径；
2. 若从右往左拼接字符串都没有出现绝对路径，则以当前工作目录路径作为前缀返回拼接结果

```js
path.resolve('/foo/bar', './baz');
// Returns: '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/');
// Returns: '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif');
// If the current working directory is /home/myself/node,
// this returns '/home/myself/node/wwwroot/static_files/gif/image.gif'

```

### path.sep
返回操作系统的默认路径分隔符
* `\` on Windows
* `/` on POSIX

### path.isAbsolute(path)
判断参数path是否是绝对路径

### 综合实例
```js
var path = require("path");

// 格式化路径
console.log('normalization : ' + path.normalize('/test/test1//2slashes/1slash/tab/..'));
//normalization : /test/test1/2slashes/1slash
// 连接路径
console.log('joint path : ' + path.join('/test', 'test1', '2slashes/1slash', 'tab', '..'));
//joint path : /test/test1/2slashes/1slash
// 转换为绝对路径
console.log('resolve : ' + path.resolve('main.js'));
//resolve : /web/com/1427176256_27423/main.js
// 路径中文件的后缀名
console.log('ext name : ' + path.extname('main.js'));
//ext name : .js

```