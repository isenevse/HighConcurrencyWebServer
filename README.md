Linux下C++高并发Web服务器，用于实践网络编程.

* 使用**线程池 + epoll(ET和LT均实现) + Proactor模式**的并发模型
* 使用**状态机**解析HTTP请求报文，支持解析**GET和POST**请求
* 通过访问服务器数据库实现web端用户**注册、登录**功能，可以请求服务器**图片和视频文件**
* 实现**同步/异步日志系统**，记录服务器运行状态
* 经Webbench压力测试可以实现**上万的并发连接**数据交换

基础测试
------------
* 服务器测试环境
	* Ubuntu版本22.04
	* MySQL版本8.0.35
* 浏览器测试环境
	* Windows、Linux均可
	* Chrome
	* Edge
	* 其他浏览器暂无测试

* 测试前确认已安装MySQL数据库

    ```C++
    // 建立yourdb库
    create database yourdb;

    // 创建user表
    USE yourdb;
    CREATE TABLE user(
        username char(50) NULL,
        passwd char(50) NULL
    )ENGINE=InnoDB;

    // 添加数据
    INSERT INTO user(username, passwd) VALUES('name', 'passwd');
    ```

* 修改main.c中的数据库初始化信息

    ```C++
    // root root修改为服务器数据库的登录名和密码
	// qgydb修改为上述创建的yourdb库名
    connPool->init("localhost", "root", "root", "yourdb", 3306, 8);
    ```

* 修改http_conn.cpp中的root路径

    ```C++
	// 修改为root文件夹所在路径
    const char* doc_root="/home/qgy/TinyWebServer/root";
    ```

* 生成server

    ```C++
    make server
    ```

* 启动server

    ```C++
    ./server port
    ```

* 浏览器端

    ```C++
    ip:port
    ```

个性化测试
------

> * I/O复用方式，listenfd和connfd可以使用不同的触发模式，代码中使用LT + LT模式，可以自由修改与搭配.

- [x] LT + LT模式
	* listenfd触发模式，关闭main.c中listenfdET，打开listenfdLT
	    
	    ```C++
	    //#define listenfdET       //边缘触发非阻塞
	    #define listenfdLT         //水平触发阻塞
	    ```
	* listenfd触发模式，关闭http_conn.cpp中listenfdET，打开listenfdLT
	    
	    ```C++
	    //#define listenfdET       //边缘触发非阻塞
	    #define listenfdLT         //水平触发阻塞
	    ```

	* connfd触发模式，关闭http_conn.cpp中connfdET，打开connfdLT
	    
	    ```C++
	    //#define connfdET       //边缘触发非阻塞
	    #define connfdLT         //水平触发阻塞
	    ```

- [ ] LT + ET模式
	* listenfd触发模式，关闭main.c中listenfdET，打开listenfdLT
	    
	    ```C++
	    //#define listenfdET       //边缘触发非阻塞
	    #define listenfdLT         //水平触发阻塞
	    ```
	
	* listenfd触发模式，关闭http_conn.cpp中listenfdET，打开listenfdLT
	    
	    ```C++
	    //#define listenfdET       //边缘触发非阻塞
	    #define listenfdLT         //水平触发阻塞
	    ```

	* connfd触发模式，打开http_conn.cpp中connfdET，关闭connfdLT
	    
	    ```C++
	    #define connfdET       //边缘触发非阻塞
	    //#define connfdLT         //水平触发阻塞
	    ```

> * 日志写入方式，代码中使用同步日志，可以修改为异步写入.

- [x] 同步写入日志
	* 关闭main.c中ASYNLOG，打开同步写入SYNLOG
	    
	    ```C++
	    #define SYNLOG //同步写日志
	    //#define ASYNLOG   /异步写日志
	    ```

- [ ] 异步写入日志
	* 关闭main.c中SYNLOG，打开异步写入ASYNLOG
	    
	    ```C++
	    //#define SYNLOG //同步写日志
	    #define ASYNLOG   /异步写日志
	    ```
* 选择I/O复用方式或日志写入方式后，按照前述生成server，启动server，即可进行测试.