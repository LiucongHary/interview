## 输入URL地址后，浏览器发生了什么

 ### URL解析  
 - 首先会开启一个线程，浏览器接收到传入的URL到开启一个网络线程是在浏览器内部发生的  
 -  URL编码   
  1 对整个URL编码: 处理空格/中文 （encodeURI/decodeURI）  
  url = 'www.baidu.com/?name=小红&from=http://www.xiaohongshu.com'  
  2 主要对传递的参数信息编码 （encodeURIComponent/decodeURIComponent）   
  url = `www.baidu.com/?name=${encodeURIComponent(小红)}&from=${encodeURIComponent(http://www.xiaohongshu.com)}`  
  3 escape 不是所有语言支持 unescape解码的 不支持用  

  #### 备注url地址说明：   
  protocol：协议头 http/https  
  host: 主机域名/ip www.baidu.com  
  port:端口号 8080  
  path:目录路径 index.html  
  query:查询参数  
  fragment:片段(次级资源信息：锚点)  
 ### 浏览器缓存检查（浏览器会新建一个网络请求线程下载所需的资源）  
 （备注：进程和线程的关系  
 一个进程有n个线程，只有有一个线程崩溃，整个进程就会崩溃）  
 #### 缓存分为强缓存和协商缓存  
 先检查是否有强缓存  
  - 有，且未失效，走强缓存
  - 没有或者已失效，检测是否有协商缓存  
    1 有，走协商缓存
    2 没有，获取最新数据

 ####  缓存的设置
（http长连接  
 	 http1.0短连接：每进行一次http通信就要断开一次TCP链接  
http1.1   
版本最好的变化是引入的持久链接，及TCP连接默认不关闭，可以被多个请求复用，不用声明  Connection:keep-alive  
http1.1版还引入管道机制（必发发送），即在同一个TCP连接中，客户端可以同时发送多个请求。这样发送请求不用等待响应即可直接发送下一个请求  
客户端请求2个资源，1.1之前，在同一个TCP连接中，先发送A请求，然后等服务器做出回应，收到后再发出B请求。管道机制则是允许浏览器同时发出A请求和B请求，但是服务器还是按照顺序，先回应A请求，再回应B请求(对头堵塞)  
解决方案：  
1 减少请求（文件的合并）  
2 同时多开持久连接（多开TCP连接，使用多个域名）  
http2  
在一个连接中，客户端和浏览器都可以同时发送多个请求或回应，而且不用按照顺序一一对应）  

http缓存：优先使用强缓存  
强制缓存  
   浏览器对于强缓存的处理：根据第一次请求资源时返回的响应头来确定，无须与服务器进行任何通信  
1 Expires:服务器写死一个时间 
Expires: new Date('2023-9-9 00:00:00').toUTCString()  
缺点：客户端时间和服务器时间可能不同步 (因为服务器写的是固定时间)  
2  http1.1 以后 添加了cache-control属性
Cache-control:"max-age=5" 时间是s ，第一次拿到资源后5s后重新发起请求
（解决了客户端和服务器时间不同步问题）。   
协商缓存  
    浏览器携带缓存标识向服务器发起GET请求，由服务器根据缓存标识决定是否使用缓存的过程
  	告诉客户端该资源使用协商缓存
客户端使用缓存数据之前问一下服务器缓存有效吗？  
服务端：  
有效：返回304，客户端使用本地缓存资源  
无效：直接返回新的资源数据，客户端直接使用  
    1 res.setHeader('Cache-Control':'no-cache'); // 告诉客户端该资源使用协商缓存
   资源的修改时间  
使用 last-modified：文件最后的更新时间  
缺陷：对资源进行了编辑，但是内容并没有发生任何变化，但是后重新请求资源  
 		res.setHeader('last-modified',mtime.toUTCString())  
2 http1.1 解决了这个问题 ETag的协商缓存  
基于文件内容生成一个唯一的密码戳    
etagContent = etag(data);    
ifNoneMatch = req.headers['if-none-match']; // 下次在请求的时候，把上次客户端返回的etagContent传给服务器，服务器判断两个值是否相同，判断是否重新请求资源  
```
if(ifNoneMatch===etagContent){
res.statusCode = 304;
res.end();
return
}
res.setHeader('Cache-Control':'no-cache'); // 告诉客户端该资源使用协商缓存
res.setHeader('etag':etagContent); // 把改资源的密码戳告诉客户端
```
 #### 缓存位置
 内存缓存 Memory Cache （加载js  ECStack  栈缓存 ）浏览器关闭，缓存的内容消失  
 硬盘缓存  Disk Cache  (C盘 D盘 。。。)  
 打开网页：查找 disk cache 中是否有匹配，如果有则使用，如没有则发送网络请求  
 普通刷新(F5)：因TAB没关闭，因此Memory Cache是可用的，会被优先使用，其次才是Disk Cache  
 强制刷新：浏览器不使用缓存，因此发送的请求头部均带有 Cache-control:no-cache,服务直接返回200和最新内容

注意： 
 1 HTML页面一般不做强缓存  
 每一次html的请求都是正常的HTTP请求  
 1）服务器更新后，让资源名称和之前不一样，这样会导入全新的资源  
   ```
    index.xxxxx.js  
    index.yyyyy.js  
   ```
  所以webpage会设置hash  
 2) 当文件更新后，在HTML倒入的时候，设置一个后缀（时间戳）
   ```
   <script src="index.js?111">  
   <script src="index.js?222">
   ```

 ### DNS解析（域名解析）

 客户端->DNS解析（拿到外网ip）->服务器端（向服务器发送请求）  
 DNS也是有缓存的 
 如果之前解析过，可能会在本地有缓存（有周期）  
 - 递归查询  
 浏览器缓存--本地host文件--本地DNS解析器缓存 -- 本地DNS服务器缓存  
 - 迭代查询  
 本地服务器   
 &nbsp; &nbsp; &nbsp;  &nbsp; &nbsp; &nbsp; &nbsp;-- 根域名服务器   
 &nbsp; &nbsp; &nbsp;  &nbsp; &nbsp; &nbsp; &nbsp;-- 顶级域名服务器   
 &nbsp; &nbsp; &nbsp;  &nbsp; &nbsp; &nbsp; &nbsp;-- 权威域名服务器  
#### 性能 DNS解析 

1 减少DNS查找（一个页面中尽可能少用不同的域名：资源都放在相同的服务器上）  
    1.1 延长缓存时间   
    1.2 减少域名，一般不超过4个域名 2-4个    
  项目中会把不同的资源放在不同的服务器上    
  虽然增加了DNS解析时间，但是web服务器、数据服务器、图片承受压力不一样   
  -  资源的合理利用
  -  抗压能力增强
  -  提高HTTP并发
  - 。。。 

2 DNS预解析
```
<link rel="dns-prefetch" href="a.com"> // DNS预解析 a.com文件 异步
<script src="a.com"></script> 
```
 ### TCP三次握手，建立和服务的链接通道
  seq序号，用来标识从TCP源端向目的端发送的字节流，发起方发送数据时对此进行标记    
  ack确认序号，只有ACK标志为1时，确认序号字段才有效，ack=seq+1     
 标记位    
 ACK：确认序号有效    
 RST：重置连接    
 SYN：发起一个新连接    
 FIN：释放一个连接    
 TCP作为一种可靠控制协议，其核心思想：既要保证数据可靠传输，又要提高传输的效率
 UDP非可靠的  
 ### 数据传输
 HTTP报文：  
 请求报文  
 响应报文  
 响应状态码   

 ### TCP四次挥手
为什么连接的时候是三次握手，关闭的时候确实4次挥手？  
 服务器端收到客户端的SYN请求报文后，可以直接发送SYN+ACK报文  
 但关闭连接时，当服务器端收到FIN报文时，很可能并不会立即关闭链接，所以只能先回复一个ACK报文，告诉客户端："你发的FIN报文我收到了"，只能等到服务器端所有报文都发送完了，我才能发送FIN报文，因此不能一起发送，所以需要四次握手  
Connection:keep-alive 保证TCP通道建立完成后，可以不关闭 （1.0需要手动设置）   
 ### 页面渲染 

 ## 性能优化 
  1 利用缓存   
   &nbsp; 1.1 对静态资源文件实现强缓存和协商缓存（文件更新，如何保证及时更新）  
   &nbsp; 1.2 对于不经常更新的接口数据采用本地存储做数据缓存（cookie/localStorage/vuex）  
2 DNS优化  
  &nbsp; 2.1 分服务器部署，增加http并发性（导致DNS解析变慢）  
  &nbsp; 2.2 DNS预加载 
3 TCP的三次握手和四次挥手  
   &nbsp; Connection:keep-alive 
4 数据传输   
  &nbsp; 4.1 减少数据传输大小   
   &nbsp; &nbsp; 4.1.1 内容或者数据压缩（webpack等）  
   &nbsp; &nbsp; 4.1.2 服务器一定要开启GZIP压缩（一般能压缩60%左右）  
   &nbsp; &nbsp; 4.1.3 大批量数据分批请求（例如：下拉刷新或者分页，保证首次加载请求数据少）  
  &nbsp; 4.2 减少HTTP请求次数    
   &nbsp; &nbsp; 4.1.1 资源文件合并处理  
   &nbsp; &nbsp; 4.1.2 字体图标、雪碧图、图片base64  
5 CDN服务器“地域分布式”  
6 采用http2.0  

==========  
网络优化是前端性能优化内容的重点，因为大部分的消耗都发生在网络层，尤其是第一次页面加载，如何减少等待时间很重要“减少白屏的效果和时间”  
- LOADDING 人性化体验  
- 骨架屏：客户端骨屏 + 服务器骨架屏  
- 图片延迟加载  


  


  