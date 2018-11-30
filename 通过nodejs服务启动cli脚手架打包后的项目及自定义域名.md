# 通过nodejs服务启动cli脚手架打包后的项目及自定义域名

>之前都是使用tomcat做本地服务的搭建，今天闲来无事研究下如何使用node快速搭建本地服务，总结下来，就是分分钟的事情，简直不要太爽～

## 操作步骤

1. 创建一个本地文件夹
2. 打开终端，切换到创建的文件夹目录，
在命令行运行(mac)
```js
// 安装express
npm install express
// 创建server.js文件（>用于创建文件）
>server.js
// 创建一个文件夹（eg:webapp）
mkdir webapp
// 切换到webapp文件夹
cd webapp
// 复制项目文件到webapp里，如，vue-cli打包后的index.html和static文件夹
cd .. 

```
3. 打开server.js文件，复制以下代码

```js
let express = require('express')
let app = express()

app.use(express.static(__dirname + '/webapp'))

app.listen('3000')
console.log('working on 3000')
```

4. 命令行运行：

```js
 node servser.js
```

如果webapp下的项目不包含服务端请求，在浏览器里打开localhost:3000就可以正常访问了

>Macos中非root用户不能监听<1024的端口，这个是内核代码里写着的，如果要操作1024以下的端口可以运行

```js
sudo node servser.js
```

生产环境最好不要这样操作，风险很大

## 如果包含服务端请求怎么办？
>一般本地开发都会配置代理，这里也一样，只要配置好代理就可以了，安装[express-http-proxy](https://www.npmjs.com/package/express-http-proxy)包即可

```js
npm install express-http-proxy --save
```
将server.js改为：

```js
let express = require('express')
let proxy = require('express-http-proxy')
let app = express()

app.use(express.static(__dirname + '/webapp'))
app.use('/', proxy('api.demo.com'))

app.listen('3000')
console.log('working on 3000')
```
重新运行

```js
node server
```
刷新浏览器，搞定。

## 如何自定义域名？
只要修改hosts文件即可，如

```js
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost
::1             www.demo.com
::1             sub.demo.com
```
打开www.demo.com:3000即可访问

## 不想加上端口号？
配置为默认端口号80即可，这样访问就不需要访问加端口号了
********
END
