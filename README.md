# chatserver
基于负载均衡的tcp集群聊天服务器和客户端，基于muduo实现，tcp，redis，mysql。
cd build
rm -rf*
cmake ..
make

cd bin/
./ChatServer
./ChatClient
127.0.0.1 6000

CREATE TABLE Users (
    id INT AUTO_INCREMENT PRIMARY KEY COMMENT '用户id',
    name VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名',
    password VARCHAR(50) NOT NULL COMMENT '用户密码',
    state ENUM('online', 'offline') DEFAULT 'offline' COMMENT '当前登录状态'
);
CREATE TABLE Friends (
    userid INT NOT NULL COMMENT '用户id',
    friendid INT NOT NULL COMMENT '好友id',
    CONSTRAINT 联合主键名称被定义为pk_friends PRIMARY KEY (userid, friendid)
) COMMENT='用户好友关系表';

#联合主键名称被定义为pk_friends

CREATE TABLE AllGroup (
    id INT AUTO_INCREMENT PRIMARY KEY COMMENT '组id',
    groupname VARCHAR(50) NOT NULL UNIQUE COMMENT '组名称',
    groupdesc VARCHAR(200) DEFAULT '' COMMENT '组功能描述'
);

CREATE TABLE GroupUser (
    groupid INT NOT NULL COMMENT '组id',
    userid INT NOT NULL COMMENT '组员id',
    grouprole ENUM('creator', 'normal') DEFAULT 'normal' COMMENT '组内角色',
    PRIMARY KEY (groupid, userid)
);

CREATE TABLE OfflineMessage (
    userid INT NOT NULL COMMENT '用户id',
    message VARCHAR(500) NOT NULL COMMENT '离线消息（存储Json字符串）'
);


数据库中的的表多少：一万行左右

系统流程：



负载均衡

：nginx
![image-20240629123938194](C:\Users\陈大煜\AppData\Roaming\Typora\typora-user-images\image-20240629123938194.png)

客户端不需要知道服务器有几台



每个服务器上都会有__userConnMap_，用户1在服务器1上登录，用户2 在服务器2上登录，那么用户1的_userConnMap中就没有用户2，这样用户1给用户2发送消息的时候就默认离线消息，但这时用户2其实是已经登录的。

解决方法，如果在_userConnMap中没有找到用户2，则去看数据库中用户2状态 是否在线，不在线则发送离线消息，在线则说明用户2登录在了其他的服务器上。

服务器1是怎么样把消息传到服务器2中的呢？

一开始想的是在各个服务器之间建立tcp连接进行通信，相当于在服务器之间进行广播，但这样服务器耦合度太高，不适合扩展，且占用太多socket资源。



引入中间件消息队列：

**这里要是面试官问熟悉redis吗？可以说为了解决以上的问题，引入了中间件消息队列，即redis，**

![image-20240629130150572](C:\Users\陈大煜\AppData\Roaming\Typora\typora-user-images\image-20240629130150572.png)

​	处理流程：建立服务器后，每个服务器都会和消息队列进行连接，订阅消息，如果用户1在_userConnMap中没有发现用户2，但在数据库中发现了，则会publish chat json给client2，因为用户2的服务器订阅了消息，那么有消息的时候就会通知client2对应的服务器

[ubuntu环境安装配置nginx流程_ubuntu安装nginx-CSDN博客](https://blog.csdn.net/nandao158/article/details/135393877?ops_request_misc=&request_id=&biz_id=102&utm_term=nginx ubuntu&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-135393877.142^v100^pc_search_result_base4&spm=1018.2226.3001.4187)

/usr/sbin/nginx：主程序
/etc/nginx：存放配置文件
/usr/share/nginx：存放静态文件
/var/log/nginx：存放日志

service nginx start  # 启动nginx
service nginx reload  # 重新加载nginx配置文件

nginx -s reopen            # 重启 Nginx
nginx -s stop              # 停止 Nginx

nginx -v



观察者模式



git步骤



1在github上创建仓库 注意增加一个readme

2复制仓库地址

3 git clone git@github.com:chendayuxixi/chatserver.git  这只是相当于连接 此时本地会出现一个文件夹 也就是创建仓库的名字

4  ssh -T git@github.com 链接github

5 将需要上传的文件移到仓库名字的文件夹中 注意空文件夹不会上传上去 如果需要这个空文件夹上传的话 可以加一个readme文件到空文件夹中

6 git add .    //暂存文件

7git commit -m "描述" //上传文件

8 git push origin master //这里的master是一个创建的分支

如果遇上

error: src refspec master does not match any.
error: 无法推送一些引用到 'git@github.com:chendayuxixi/chatserver.git'

则说明没有这个分支

可以用git branch查看一下

git checkout -b "master"切换到“master”分支，这个命令的意思是有则切换 没有则创建

9 git merge main

10 git push origin master
