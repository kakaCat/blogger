

win10环境

配置系统环境变量配置

3.1 启动NAMESERVER

start mqnamesrv.cmd

3.2 启动BROKER

start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true



启动控制台

1、访问 <https://github.com/apache/rocketmq-externals/> ， 
使用git将代码clone下来

2、修改项目配置信息 
这是一个用springboot编写的工程，进入到 `rocketmq-externals\rocketmq-console\src\main\resources`目录下，编辑 application.properties 文件， 修改mq的连接地址信息：

rocketmq.config.namesrvAddr=localhost:9876

rocketmq-externals\rocketmq-console目录下，执行：

mvn spring-boot:run

