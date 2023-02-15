
# TinyWebServer_Study
学习开源项目 "TinyWebServer" 的仓库

**项目功能点**
* 使用 线程池 + 非阻塞socket + epoll(ET和LT均实现) + 事件处理(Reactor和模拟Proactor均实现) 的并发模型
* 使用状态机解析HTTP请求报文，支持解析GET和POST请求
* 访问服务器数据库实现web端用户注册、登录功能，可以请求服务器图片和视频文件
* 实现同步/异步日志系统，记录服务器运行状态
* 使用自定义定时器来关闭短连接的非活跃客户端
* 经Webbench压力测试可以实现上万的并发连接数据交换

## 项目框架图
* 框架图
![框架图](https://github.com/Laity8/picture_resource_resort/blob/master/TinyWebServer_repository/webserver_frame.jpg "框架图")
## 项目模块分析
* **数据库连接池模块**

  数据库连接池模块主要分为两部分，一部分是连接池的设计，另一部分是连接池的RAII设计，即“资源获取即初始化"。
  * 连接池设计
      
     数据库连接池在初始化时会生成固定数量的MYSQL连接存储在链表（队列也可）中，外部可以随时通过线程安全的方法来拿走空闲的数据库连接，用完之后再次添加到链表中。线程安全方面使用信号量加以控制，对于这种初始资源固定的用信号量控制很方便。
  * RAII设计
 
     防止数据库连接资源泄露或者不能及时归还到池中而设计的类，在构造和析构中分别调用数据库连接的生成和释放。在使用是利用局部变量的生存期即可及时释放资源。
* **半同步半反应堆线程池 发编程模式**
  
  半同步半反应堆是半同步半异步并发模式的变种，在本项目中写了两种事件处理模式，分别是reactor模式和proactor模式（用同步IO模拟，因为异步IO不成熟）
  * reactor事件处理模式
    
    主线程监听事件是否就绪；工作线程进行接受新连接，读取数据和逻辑处理，使用同步IO。
  * proactor事件处理模式

    主线程监听事件是否完成，主线程和内核进行接受新连接，读取数据等IO操作；工作线程负责逻辑处理，使用异步IO。
  * 工作流程

    epoll_wait监听到就绪事件，对于reactor模式，主线程判断后将任务放入任务队列中；工作线程通过竞争后拿到任务进行IO操作和逻辑处理。
    对于proactor模式，主线程进行连接操作和数据读取（同步IO），工作线程进行逻辑处理。
* **HTTP状态机**
 
    http状态机分为主，从状态机。主状态机分别用于解析请求报文的请求行，请求头，空行和请求数据。从状态机主要对请求报文的每一行进行完整的拿取。同时配合主状态机进行消息的提取。最后生成对应的响应报文（状态行，响应头，空行，响应正文）。主要处理GET和POST请求，中间进行CGI校验，然后发送html文本和文件（图片和视频）。
* **同步线程池封装模块**
    对原始的线程同步机制功能进行RAII封装。有锁，条件变量，信号量。与变量的生命周期绑定便于资源的管理。
* **定时器设计模块**

* **异步日志系统模块**
 
