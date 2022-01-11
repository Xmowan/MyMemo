# Zabbix

### 安装Zabbix

**Zabbix server**

官网：https://www.zabbix.com/cn/download

1）选择版本

![1641861054259](https://gitee.com/Xmowan/ims/raw/master/ims/1641861054259.png)

2）根据下面的步骤一步一部安装



```shell
# 准备yum源
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all
# 打开zabbix存储库
vin /etc/yum.repos.d/zabbix.repo
[zabbix-frontend]
...
enabled=1
...
# 安装zabbix server 和 zabbix-agent 如果监控自己 zabbix-agent可以不安装
yum install zabbix-server-mysql zabbix-agent
# 安装zabbix前端 也就是web界面
yum install centos-release-scl
# 安装zabbix前端 的各种包
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl

# 创建数据库 也可以使用一句创建
# mysql -uroot -p
password	
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password'; # 自定义密码
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
# 导库
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

# 给zabbix配置链接数据库的密码
vim /etc/zabbix/zabbix_server.conf
DBPassword=password

# 设置时区
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
; php_value[date.timezone] = Europe/Riga
php_value[date.timezone] = Asia/Shanghai

# 设置时间	
yum -y install ntpdata
ntpdate ntp.aliyun.com

# 启动Zabbix server和agent进程 并设置开机自启
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm

# 查看端口是否启动
netstat -tanlp
```

**Zabbix agent**

```shell
# 准备yum源
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all

# 下载安装 agent
 yum install -y zabbix-agent
 
# 修改配置文件
vim /etc/zabbix/zabbix_agentd.conf

Server=10.10.10.11          #被动模式服务器IP, 用于定义允许谁来采集数据
# StartAgents=3             #被动模式情况下, 预生成的子进程数, 默认为3. 如果设置为0, 将关闭被动模式
ServerActive=10.10.10.11    #主动模式服务器IP, 主动向谁上传数据
Hostname=web-server         #配置自己的主机名, 在后续配置监控项时要与此名称一致

# 设置时间	
yum -y install ntpdata
ntpdate ntp.aliyun.com

# 启动
systemctl start zabbix-agent
```

**测试是否可以连通**

```shell
# 服务端执行
# 下载 zabbix_get
yum -y insatll zabbix_get

# 测试
zabbix_get -s 10.10.10.13 -k system.hostname
web-server # 返回agent端的主机名 代表成功
```

---

### 基本配置流程

**创建主机组（host group)**

- 主机的逻辑组；可能包含主机和模板。一个主机组里的主机和模板之间并没有任何直接的关联。通常在给不同用户组的主机分配权限时候使用主机组。

![1641805946293](https://gitee.com/Xmowan/ims/raw/master/ims/1641805946293.png)

![1641805702682](https://gitee.com/Xmowan/ims/raw/master/ims/1641805702682.png)

![1641805804253](https://gitee.com/Xmowan/ims/raw/master/ims/1641805804253.png)

**创建主机（host）**

- 你想要监控的联网设备，有IP/DNS。

![1641805904421](https://gitee.com/Xmowan/ims/raw/master/ims/1641805904421.png)

![1641806310226](https://gitee.com/Xmowan/ims/raw/master/ims/1641806310226.png)

![1641806352168](https://gitee.com/Xmowan/ims/raw/master/ims/1641806352168.png)

**添加监控项（item）**

- 你想要接收的主机的特定数据，一个度量/指标数据。

![1641806549913](https://gitee.com/Xmowan/ims/raw/master/ims/1641806549913.png)

![1641806939316](https://gitee.com/Xmowan/ims/raw/master/ims/1641806939316.png)

![1641807022446](https://gitee.com/Xmowan/ims/raw/master/ims/1641807022446.png)

**触发器（trigger）**

- 一个被用于定义问题阈值和“评估”监控项接收到的数据的逻辑表达式

就好比我们设立一个目标（目标就是一个触发器），当达到目标时就干什么（干什么就是一个动作）

![1641812684052](https://gitee.com/Xmowan/ims/raw/master/ims/1641812684052.png)

![1641812595606](https://gitee.com/Xmowan/ims/raw/master/ims/1641812595606.png)

**图形**

- 通过统计图的方式显示被监控的数据

![1641812884316](https://gitee.com/Xmowan/ims/raw/master/ims/1641812884316.png)

![1641813017798](https://gitee.com/Xmowan/ims/raw/master/ims/1641813017798.png)

**创建媒介（media）**

- 发送告警通知的方式；传送途径

  当触发器触发时，通过什么 媒介的方式 告知管理员

  ![1641813583971](https://gitee.com/Xmowan/ims/raw/master/ims/1641813583971.png)

![1641860613587](https://gitee.com/Xmowan/ims/raw/master/ims/1641860613587.png)

测试然后再收件箱子查看

![1641816798353](https://gitee.com/Xmowan/ims/raw/master/ims/1641816798353.png)

**创建用户**

- 由谁来做什么动作

![1641860299870](https://gitee.com/Xmowan/ims/raw/master/ims/1641860299870.png)

![1641860395948](https://gitee.com/Xmowan/ims/raw/master/ims/1641860395948.png)

**动作（action）**

- 预先定义的应对事件的操作

一个动作由操作(例如发出通知)和条件(*什么时间*进行操作)组成

![1641813158677](https://gitee.com/Xmowan/ims/raw/master/ims/1641813158677.png)

![1641813337618](https://gitee.com/Xmowan/ims/raw/master/ims/1641813337618.png)

![1641813388411](https://gitee.com/Xmowan/ims/raw/master/ims/1641813388411.png)



### 模板详解

**创建模板**

![image-20220111103148246](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111103148246.png)

![image-20220111103425851](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111103425851.png)

**创建完成后 接下来就和之前单个添加的步骤一样**

![image-20220111103502715](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111103502715.png)

**使用模板**

![image-20220111104024573](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111104024573.png)

**模板的导入导出**

注意事项：在导入模板时需要将agent端的配置文件也做一个同步，不然会因为没有键值而导致失败！！

**选中要导出的模板然后导出**

会生成一个xml格式的模板文件

![image-20220111104756681](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111104756681.png)

![image-20220111104823370](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111104823370.png)

**导入模板**

![image-20220111105137574](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111105137574.png)

![image-20220111105113453](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111105113453.png)

**将主机与模板关联**

![image-20220111110316819](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111110316819.png)



### 自定义监控项

在 zabbix-agent 中 配置 UserParameters	# 自定义键值

方式一：通过一条命令实现

```shell
# 修改zabbix-agent配置文件 这里使用引用的方式
vim /etc/zabbix/zabbix_agentd.d/data.num.conf
UserParameter=file.num, ls /data | wc -l

# 上面的已经在前面使用过了 就是监控/data目录的文件个数
# 新需求：实现一个键值 监控指定目录的 文件个数
# 可以使用传参的方式实现 []中的第一个值 对应$1 []中的第二个值 对应$2 以此类推
UserParameter=file.num[*], ls $1 | wc -l

# 测试
 zabbix_get -s 10.10.10.13 -k file.num[/usr]
 zabbix_get -s 10.10.10.13 -k file.num[/]
```

![image-20220111142655010](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111142655010.png)

方式二：通过脚本的方式实现

```shell
# 修改zabbix-agent配置文件 这里使用引用的方式
# 只不过这里 后面接的是一个脚本 脚本中 echo 出 要监控的数据
vim /etc/zabbix/zabbix_agentd.d/data.num.conf
UserParameter=root.mem, /tmp/rootMem.sh

# 编写一个简单的脚本 监控/的使用情况
 vim /tmp/rootMem.sh
 
#!/bin/bash
mem=$(df -hT | awk -F " " '/\/$/{print $6}')
echo $mem

#给权限 
chmod +x /tmp/rootMem.sh

# 重启测试
zabbix_get -s 10.10.10.13 -k root.mem
10%
```

![image-20220111150159428](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111150159428.png)

上图虽然有错误，但是并不是我们配置错了，而是没有对数据进行处理

如何处理数据：zabbix为我们提供了一个进程的可选项，就是用来处理数据的

![image-20220111144355357](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111144355357.png)

再测试就没有之前的问题了

![image-20220111144542324](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111144542324.png)

![image-20220111144524957](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111144524957.png)

### 触发器详解

通常是一个阈值，当到达阈值是就会被触发

注意点：当我们再触发一个事件时，通常会取多次的一个平均值，因为有时候会因为网络波动或其他问题造成误报，所以为了保险起见，我们一般会去一个平均值

**触发器的依赖关系**

很多时候我们的报警都有一个依赖的关系，比方说我们有很多设备都依赖于一个中心设备，当我们的中心设备故障时，如果没有依赖关系，会导致被依赖的设备的触发器也会被触发，而本身他们是没有问题的，严重时还会导致我们的监控项报错，致使我们难以很快的确定问题本身，这是我们不想看到的情况。所以zabbx提供了 触发器的依赖关系。

示例：

nginx的连接数和活跃数等一些指标，都依赖与nginx服务本身是否运行。

1）环境准备

```shell
# nginx配置
location = /stat {
            stub_status on;
            access_log off;
        }


# agent
UserParameter=nginx.stat[*], /tmp/nginxStat.sh $1

# 脚本 里面不做默认值的处理
#!/bin/bash
if [ "$1" == "stat" ];then
    netstat -tnl | grep ":80 " &>/dev/null && echo "ok" || echo "no"
elif [ "$1" == "Active" ];then
     curl http://127.0.0.1/stat 2>/dev/null | awk 'NR==1{print $3}'
elif [ "$1" == "requests" ];then
     curl http://127.0.0.1/stat 2>/dev/null | awk 'NR==3{print $3}'
fi
```

2）创建触发器 不建立依赖关系

![image-20220111164714041](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111164714041.png)

3）模拟宕机 后 我们可以发现有两个 报错 就是因为 依赖于nginx的监控项 拿不到值所以报错

![image-20220111165308094](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111165308094.png)

4）为了解决上面的问题 我们来建立依赖关系

![image-20220111165452545](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111165452545.png)

5）再次启停nginx

![image-20220111165904670](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111165904670.png)

![image-20220111170116714](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111170116714.png)

6）观察结果，可以发现上面的报错没有了，在监控面板中也只报了nginx宕机这个主要的问题

---



### zabbix 动作（Action）详解

前面讲了发送邮件的其实就是一个动作

zabbix的动作可以分为两大类：

- 发送信息（用于告警）
- 执行命令（重启）

实验一：通过企业微信进行告警

1）通讯录 -- 创建子部门 --- 添加成员

3）拿到如下信息

- 帐号 

  ![image-20220111190630448](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111190630448.png) 



- AgentId和Secret

  ![image-20220111191605457](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111191605457.png)

![image-20220111191432334](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111191432334.png)

- CorpID（企业ID）

![image-20220111191854419](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111191854419.png)

3）上传脚本 

```shell
# 查看脚本存放位置
grep 'script' /etc/zabbix/zabbix_server.conf
# Full path to location of custom alert scripts.
# AlertScriptsPath=${datadir}/zabbix/alertscripts
AlertScriptsPath=/usr/lib/zabbix/alertscripts

# 上传脚本
vim /usr/lib/zabbix/alertscripts/wechat.sh
#!/bin/bash 
#Wechat alert script for zabbix		显示帮助信息
if [ $# -eq 0 ] || [[ "$1" == "-h" || "$1" == "--help" ]];then
	echo "Usage of $0:"
	echo -e " --CorpID=string"
	echo -e " --Secret=string"
	echo -e " --AgentID=string"
	echo -e " --UserID=string"
	echo -e " --Msg=string"
	exit
fi

#ops=(-c -s -a -u)
#args=(CorpID Secret AgentID UserID)
#while [ $# -gt 0 ];do
#    [ "$1" == "-m" ] && Msg="$2" && shift 2
#    for i in {0..3};do
#        [ "$1" == "${ops[i]}" ] &&  eval ${args[i]}="$2"
#    done
#    shift 2
#done
for i in "$@";do
	echo $i|grep Msg &> /dev/null && msg=$(echo $i|sed 's/.*=//') && Msg="$msg" && continue
	eval "$(echo $i|sed 's/--//')"
done
#echo $CorpID
#echo $Secret
#echo $UserID
#echo $AgentID
#echo $Msg
#
GURL="https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$CorpID&corpsecret=$Secret" 
Token=$(/usr/bin/curl -s -G $GURL |awk -F \" '{print $10}') # 拿到 access_token
PURL="https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$Token"
Info(){
	printf '{\n'
	printf '\t"touser": "'"$UserID"\"",\n"
	printf '\t"msgtype": "text",\n'
	printf '\t"agentid": "'"$AgentID"\"",\n"
	printf '\t"text": {\n'
	printf '\t\t"content": "'"$Msg"\""\n"
	printf '\t},\n'
	printf '\t"safe":"0"\n'
	printf '}\n'
}

/usr/bin/curl --data-ascii "$(Info)" $PURL
echo

# 授权
chmod +x /usr/lib/zabbix/alertscripts/wechat.sh

# 测试 如果测试成功后续就不必纠结这里的问题 可以确定是配置问题 
# 参数都是之前拿到的


/usr/lib/zabbix/alertscripts/wechat.sh --CorpID=ww4dcd53927eed9047 --Secret=6JvioICyzSK109UO9gIwwITRlNGi2deIV1RAQGa6tCI --AgentID=1000002 --UserID=LiuHai --Msg="要发送的啥"

# 看到下面的信息成功一半 正真看到收到信息才算完成成功
{"errcode":0,"errmsg":"ok","msgid":"Dv0oBVNA9p2BIWPODPqgkvLS4p2_IQnBOryGAhuCcKzn0obhJRcGDRKzY-vR-2GQ37cyGJTlhmMUy4_ctxO4rg"}

7eckPaB3sVngw_CI6c2trKPm5aozjH7Pj7eNq5ADQ6_TDlD_GoIIwOd3YRerxICHHG50xZ3GUaigXgZyF5kgsNZq_sdoPVF1CQJNtZDcNJQ-4xYwVBhzCO6HL0Yg6LjGBDBu3iChvTkQuQImu1ULumpMvMNdADdZLXkKMpcOJMlSbs0Mv-sj8fgCytB7Boxi971oISqd9zkIq8HDT1JZkw

```

4）将上面配置的企业微信创建为一个报警媒介

![image-20220111195927745](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111195927745.png)

![image-20220111195948063](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111195948063.png)

5）配置用户 就是需要一个用户来干这件事（发送消息）

![image-20220111200154433](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111200154433.png)

6）和之前一样配置动作

![image-20220111200415662](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111200415662.png)

7）模拟宕机，测试是否能够收到宕机提醒

![image-20220111201213728](https://gitee.com/Xmowan/ims/raw/master/ims/image-20220111201213728.png)
