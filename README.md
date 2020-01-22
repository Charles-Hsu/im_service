
# im service
1. 支持点对点消息, 群组消息, 聊天室消息
2. 支持集群部署
3. 单机支持50w用户在线
4. 单机处理消息5000条/s
5. 支持超大群组(3000人)

*服务器硬件指标：32g 16核*

## 编译运行

1. 安装go编译环境

   参考链接:https://golang.org/doc/install

2. 下载im_service代码

       cd $GOPATH/src/github.com/GoBelieveIO
       git clone https://github.com/GoBelieveIO/im_service.git

3. 安装依赖

       cd im_service
       dep ensure

4. 编译

       cd im_service   
       mkdir bin
       make install
    
   可执行程序在bin目录下
   
- 有錯誤訊息
         
      command-line-arguments
      ./sio.go:94:28: undefined: engineio.BinaryMessage
      ./sio.go:118:10: undefined: engineio.MessageText
      
需要去修改 im/sio.go 把 engineio.BinaryMessage --> engineio.BINARY, engineio.MessageText --> engineio.TEXT

5. 安装mysql数据库, redis, 并导入db.sql

       mysql> source db.sql;

- redis 的 port 為 6379, 如果沒有啟動會有 connection refused 的 error
    
      % redis-cli
      Could not connect to Redis at 127.0.0.1:6379: Connection refused
      not connected>
      
. 執行下列指令

    $ redis-server /usr/local/etc/redis.conf &
    % redis-cli
    127.0.0.1:6379> ping
    PONG
    127.0.0.1:6379>

6. 配置程序
   配置项的说明参考ims.cfg.sample, imr.cfg.sample, im.cfg.sample
   
       cp ims.cfg.sample ims.cfg
       cp imr.cfg.sample imr.cfg
       cp im.cfg.sample im.cfg

7. 启动程序

  * 创建配置文件中配置的im&ims消息存放路径

        mkdir /tmp/im
        mkdir /tmp/pending  <--- 參考 im.cfg 裡定義的
        mkdir /tmp/impending   <--- 原來的 README.md 的

  * 创建日志文件路径
    
        mkdir ./data/logs/ims
        mkdir ./data/logs/imr
        mkdir ./data/logs/im

  * 启动im服务

        pushd `dirname $0` > /dev/null
        pwd
        /Users/chia/go/src/github.com/GoBelieveIO/im_service
    
        BASEDIR=`pwd`
        nohup $BASEDIR/bin/ims -log_dir=$BASEDIR/data/logs/ims ims.cfg >$BASEDIR/data/logs/ims/ims.log 2>&1 &
        nohup $BASEDIR/bin/imr -log_dir=$BASEDIR/data/logs/imr imr.cfg >$BASEDIR/data/logs/imr/imr.log 2>&1 &
        nohup $BASEDIR/bin/im  -log_dir=$BASEDIR/data/logs/im  im.cfg  >$BASEDIR/data/logs/im/im.log 2>&1 &
    
        $ ps -ef | grep /bin/im
        501 44394 38935   0 12:29PM ttys020    0:00.40 /Users/chia/go/src/github.com/GoBelieveIO/im_service/bin/ims -log_dir=/Users/chia/go/src/github.com/GoBelieveIO/im_service/data/logs/ims ims.cfg
        501 44399 38935   0 12:30PM ttys020    0:00.05 /Users/chia/go/src/github.com/GoBelieveIO/im_service/bin/imr -log_dir=/Users/chia/go/src/github.com/GoBelieveIO/im_service/data/logs/imr imr.cfg
        501 44419 38935   0 12:30PM ttys020    0:01.02 /Users/chia/go/src/github.com/GoBelieveIO/im_service/bin/im -log_dir=/Users/chia/go/src/github.com/GoBelieveIO/im_service/data/logs/im im.cfg
        
        $ sudo lsof -iTCP -sTCP:LISTEN -n -P
        COMMAND   PID   USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
        ims     44394   chia    8u  IPv6 0xebb5d44c048cbc9b      0t0  TCP *:13333 (LISTEN)
        ims     44394   chia    9u  IPv6 0xebb5d44c048cb67b      0t0  TCP *:3334 (LISTEN)
        imr     44399   chia    4u  IPv6 0xebb5d44c048c97db      0t0  TCP *:4444 (LISTEN)
        
    
        //nohup $BASEDIR/ims -log_dir=/data/logs/ims ims.cfg >/data/logs/ims/ims.log 2>&1 &
        nohup $BASEDIR/bin/ims -log_dir=$BASEDIR/data/logs/ims ims.cfg >$BASEDIR/data/logs/ims/ims.log 2>&1 &
        //nohup bin/ims ims.cfg -log_dir=./data/logs/ims > ./data/logs/ims/ims.log 2>&1 &

        //nohup $BASEDIR/imr -log_dir=/data/logs/imr imr.cfg >/data/logs/imr/imr.log 2>&1 &
        nohup $BASEDIR/bin/imr -log_dir=$BASEDIR/data/logs/imr imr.cfg >$BASEDIR/data/logs/imr/imr.log 2>&1 &
        //nohup bin/imr -log_dir=./data/logs/imr imr.cfg >./data/logs/imr/imr.log 2>&1 &

        //nohup $BASEDIR/im -log_dir=/data/logs/im im.cfg >/data/logs/im/im.log 2>&1 &
        nohup $BASEDIR/bin/im -log_dir=$BASEDIR/data/logs/im im.cfg >$BASEDIR/data/logs/im/im.log 2>&1 &
    

## token的格式

    连接im服务器token存储在redis的hash对象中,脱离API服务器测试时，可以手工生成。
    $token就是客户端需要获得的, 用来连接im服务器的认证信息。
    key:access_token_$token
    field:
        user_id:用户id
        app_id:应用id


## 官方QQ群
1. 450359487(一群)，已满。
2. 416969931(二群)，加群请附加说明信息。

## 官方网站
   https://developer.gobelieve.io/

## 相关产品
   https://goubuli.mobi/
