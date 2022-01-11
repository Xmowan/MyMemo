## Jenkins+Gitlab+apache

作者：Liuhai

环境准备：

jenkins	CentOS 7 4G内存2CPU	IP：10.10.10.11

Gitlab	CentOS 7 4G内存2CPU	IP：10.10.10.12

webServer	CentOS 7 2G内存2CPU	IP：10.10.10.13

#### 一、Gitlab 安装部署

安装版本：社区版

官网：<https://about.gitlab.com/installation>    

国内镜像源：<https://mirrors.tuna.tsinghua.edu.cn> 

**安装**

下载GitLab安装包，如果没有网络需要提前准备好rpm包

```shell
wget https://omnibus.gitlab.cn/el/7/gitlab-jh-14.6.0-jh.0.el7.x86_64.rpm
```

安装下载的rpm包

```shell
rpm -Uvh gitlab-jh-14.6.0-jh.0.el7.x86_64.rpm 
# 或
# 推荐会自动解决依赖问题
yum -y localinstall gitlab-jh-14.6.0-jh.0.el7.x86_64.rpm
```

配置Girlab

```shell
vim /etc/gitlab/gitlab.rb

external_url 'http://10.0.0.12'  #修改此行为自己的IP
#增加下面行，可选邮件通知设置
gitlab_rails['smtp_enable'] = true	# 开启邮箱
gitlab_rails['smtp_address'] = "liuhai.qq.com" # SMTP邮件服务器
gitlab_rails['smtp_port'] = 25	# SMTP邮件服务器端口
gitlab_rails['smtp_user_name'] = "liuhai@qq.com"	# 自己的邮箱
gitlab_rails['smtp_password'] = "授权码"	# 自己邮箱的授权码
gitlab_rails['smtp_domain'] = "qq.com"	# SMTP服务器的域
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
```

执行初始化并启动服务

```shell
gitlab-ctl reconfigure 
```

常用命令

```shell
gitlab-rails #用于启动控制台进行特殊操作，如修改管理员密码、打开数据库控制台( gitlab-rails dbconsole)等
gitlab-psql #数据库命令行
gitlab-rake #数据备份恢复等数据操作

gitlab-ctl  #客户端命令行操作行
gitlab-ctl stop #停止gitlab
gitlab-ctl start #启动gitlab
gitlab-ctl restar #重启gitlab
gitlab-ctl  status #查看组件运行状态
gitlab-ctl  tail nginx #查看某个组件的日志
```

访问Gitlab

http://10.10.10.12

第一次登录使用root用户，密码在/etc/gitlab/initial_root_password中，完成登录后及时修改密码！！

#### 二、jenkins 安装部署

**安装jdk环境**

jdk版本可自由选择，官方推荐的是jdk11

```shell
# 解压
tar -xf jdk-8u181-linux-x64.tar.gz -C /usr/local/
# 创建一个软链接 方便版本的切换
ln -s /usr/local/jdk1.8.0_181 /usr/local/java
# 编辑环境变量
vim /etc/profile.d/java.sh
export JAVA_HOME=/usr/local/java
export PATH=$JAVA_HOME/bin:$PATH
# 使环境生效
. /etc/profile.d/java.sh
# 验证
java --version
```

**安装**

方案一：

官网的方案，会因为网络问题而失败

```shell
# 建立源
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
# 下载密钥
rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
# 直接安装 
yum -y install jenkins 
```

方案二：

先下载好rpm包，更推荐这种方式

```shell
# 安装epel源
yum -y install epel-relesae
# 本地安装
yum -y localinstall jenkins-2.319.1-1.1.noarch.rpm
```

**配置**

指定java路径

```shell
vim /etc/sysconfig/jenkins
JENKINS_JAVA_CMD="/usr/local/java/bin/java"
```

**启动服务**

```shell
systemctl restart jenkins
```

**访问**

http://10.10.10.11:8080

将插件一遍一遍的重复重试完成插件的安装，配置管理员密码重启就可以了!!

### 三、web服务器

简单的apach就行了,安装启动

```shell
yum -y install httpd
systemctl start httpd
```

---

**到这里环境准备就完成了 **

---

### 四、配置jenkins拉取Gitlab中的项目并推送到web服务器

**1、jenkins全局环境搭建**

1）设置编码 系统设置-->系统配置-->全局属性，做如下配置

![全局属性UTF-8](https://gitee.com/Xmowan/ims/raw/master/ims/%E5%85%A8%E5%B1%80%E5%B1%9E%E6%80%A7UTF-8.png)

2）系统设置-->全局工具配置-->JDK，做如下配置

![jdk配置](https://gitee.com/Xmowan/ims/raw/master/ims/jdk%E9%85%8D%E7%BD%AE.png)

**2、配置拉取Gitlab中的项目**

1）在Gitlab中建立一个管理员用户用于在jenkins中拉取项目，后面会使用；然后使用任意一个用户创建一个项目

实验中就直接使用jenkins用户创建了，然后创建一个仓库，放入一个测试文件index.html

![Gitlab中创建一个仓库](https://gitee.com/Xmowan/ims/raw/master/ims/Gitlab%E4%B8%AD%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E4%BB%93%E5%BA%93.png)

2）添加凭据，也就是说Jenkins使用什么身份拉取Gitlab中项目

![添加凭据3](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%B7%BB%E5%8A%A0%E5%87%AD%E6%8D%AE3.png)3）然后在jenkins中新建一个任务-，任务名称自定义，选择自由风格的点击确认，然后做如下配置

![拉取2](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%8B%89%E5%8F%962.png)

![拉取1](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%8B%89%E5%8F%961.png)

4）测试构建---点击立即构建，除了使用图上的观察结果外还可以也可在/var/lib/jenkins/workspace/下查看！

到这一步我们已经将开发写的代码拉取到了jenkins这中

![拉取完成](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%8B%89%E5%8F%96%E5%AE%8C%E6%88%90.png)

---

**3、配置推送项目到web服务器**

方案一：

配置Publish Over SSH，用于将拉取下来的代码推送到web服务器中，是基于ssh传输的

1）先要安装插件 Publish Over SSH

插件安装：系统管理 --> 插件管理 -->可选插件 --> 搜索Publish Over SSH --> 安装即可 （其他的插件也一样）

2）然后在Jenkins中生产公私钥文件，将公钥传到web服务器中

3）在系统管理-->系统配置中做找到Publish SSH做如下配置

![推送配置1](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%8E%A8%E9%80%81%E9%85%8D%E7%BD%AE1.png)

方案二：

采用ansible的方式进行推送，也就是说可以不使用 Publish Over SSH，而使用ansible的copy模块

1）ansible配置

```shell
vim /etc/ansible/ansible.cfg
# uncomment this to disable SSH key host checking
# 不要询问指纹信息
host_key_checking = False
# 设置以root用户执行ansible
remote_user = root 	
```

2）然后在/var/lib/jenkins/.ssh中生成公私钥文件，然后将公钥文件拷贝到web服务器中 

3）这里的脚本是在，jenkins机器上执行行的，而且实在我们成功拉取项目后执行的。要明白的目的就是将拉取下来的项目传输到web服务起的网站发布目录下。

![angsible推送](https://gitee.com/Xmowan/ims/raw/master/ims/angsible%E6%8E%A8%E9%80%81.png)

再在构建设置中选择执行shell的方式

![报错](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%8A%A5%E9%94%99.png)

报如下错误：

解决方案：

 https://www.cnblogs.com/rwxwsblog/p/5658703.html

4）然后打开web服务器的网页，能够看到 index.html中的内容就算是完成了

http://10.10.10.13:80

---

**到这里jenkins的基本使用就算是完成了，我们可以实现一键构建了（限于html、php这样不需要编译的）**

---

### 五、实现对Java项目的构建，然后发布到Tomcat中

**1、在jenkins服务器上准备maven环境**

Maven是基于项目对象模型，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。 

Maven采用纯Java编写, 它采用了一种被称之为Project Object Model(POM)概念来管理项目，所有的项目配置信息都被定义在一个叫做POM.xml的文件中, 通过该文件Maven可以管理项目的整个生命周期，包括清除、编译，测试，报告、打包、部署等等。 

```shell
# 解压
tar -xf apache-maven-3.5.4-bin.tar.gz -C /usr/local/
# 创建一个软链接 方便版本的切换
ln -s /usr/local/maven-3.5.4 /usr/local/maven3
# 编辑环境变量
vim /etc/profile.d/maven3.sh
export MAVEN_HOME=/usr/local/maven3
export PATH=$MAVEN_HOME/bin:$PATH
# 使环境生效
. /etc/profile.d/maven3.sh
# 验证 可以看到版本信息就代表成了
mav -v
# 配置加速
vim /usr/local/maven3/conf/settings.xml
# 找到mirrors标签，加入以下内容：                       
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>https://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

**2、需要准备一个项目直接从Gitee上下载一个然后push到我们的Gitlab中，就相当于开发提交了一次代码**

https://github.com/bingyue/easy-springmvc-maven

**3、准备一台Tomcat服务器，这里直接装在之前的web服务器上，安装过程略**

**4、这次采用私钥凭证拉取代码，将jenkins中的公钥复制到Gitlib中的ssh中，然后在jenkins中**

再将公钥加入到jenkins中的凭据中

![私钥拉取代码](https://gitee.com/Xmowan/ims/raw/master/ims/%E7%A7%81%E9%92%A5%E6%8B%89%E5%8F%96%E4%BB%A3%E7%A0%81.png)

![公钥拉取代码](https://gitee.com/Xmowan/ims/raw/master/ims/%E5%85%AC%E9%92%A5%E6%8B%89%E5%8F%96%E4%BB%A3%E7%A0%81.png)

**5、创建一个maven风格的任务，然后使用git的方式拉取项目，如果没有构建maven项目的那个选项，就是没有安装插件[Maven Integration](https://plugins.jenkins.io/maven-plugin) ，按照之前安装插件的方式安装即可，然后设置发布步骤**

![maven项目](https://gitee.com/Xmowan/ims/raw/master/ims/maven%E9%A1%B9%E7%9B%AE.png)

![使用key凭据](https://gitee.com/Xmowan/ims/raw/master/ims/%E4%BD%BF%E7%94%A8key%E5%87%AD%E6%8D%AE.png)

![传输wra包](https://gitee.com/Xmowan/ims/raw/master/ims/%E4%BC%A0%E8%BE%93wra%E5%8C%85.png)

**6、使用Publish over SSH的方式传输**

![war包传输](https://gitee.com/Xmowan/ims/raw/master/ims/war%E5%8C%85%E4%BC%A0%E8%BE%93.png)

**7、然后打开web服务器的网页，能够看到 网页就代表成功了**

http://10.10.10.13:8080

---

**之前的实验中我们都是通过手动点击构建来触发构建，下面就来讲一下利用触发器自动构建**

---

### 六、触发器

**1、在哪儿构建**

![构建触发器](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%9E%84%E5%BB%BA%E8%A7%A6%E5%8F%91%E5%99%A8.png)

**2、常见的触发方式**

- 定时构建(Build perriodocally)，在一个固定的时间，无条件自动构建
- 轮询构建(SCM)，在定时构建的基础上增加了条件，会自动检查仓库是否有变化，有才构建
- Push事件触发，当向仓库的某个分支（一般为master）成功push代码后，就会触发构建
- 远程触发通过预定URL来触发，一般会结合脚本使用

**3、Push事件触发器的使用**

安装GitLab插件

jenkins上设置：在任务中做设置，然后勾选上进行如下设置

![触发地址](https://gitee.com/Xmowan/ims/raw/master/ims/%E8%A7%A6%E5%8F%91%E5%9C%B0%E5%9D%80.png)

![触发分支](https://gitee.com/Xmowan/ims/raw/master/ims/%E8%A7%A6%E5%8F%91%E5%88%86%E6%94%AF.png)

![生成令牌](https://gitee.com/Xmowan/ims/raw/master/ims/%E7%94%9F%E6%88%90%E4%BB%A4%E7%89%8C.png)

在Gitlab上设置：

使用管理员身份登录，在设置中做如下设置

![打开外发请求](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%89%93%E5%BC%80%E5%A4%96%E5%8F%91%E8%AF%B7%E6%B1%82.png)

然后在项目中做如下设置

![添加触发事件](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%B7%BB%E5%8A%A0%E8%A7%A6%E5%8F%91%E4%BA%8B%E4%BB%B6.png)

最后点击测试观察是否能够自动构建

**4、远程触发**

远程触发需要设置匿名用户具有可读权限

系统管理 - 全局安全配置 - 授权策略 - 匿名用户具有可读权限 

![匿名用户可读](https://gitee.com/Xmowan/ims/raw/master/ims/%E5%8C%BF%E5%90%8D%E7%94%A8%E6%88%B7%E5%8F%AF%E8%AF%BB.png)

然后在任务管理页添加触发器

![远程触发构建](https://gitee.com/Xmowan/ims/raw/master/ims/%E8%BF%9C%E7%A8%8B%E8%A7%A6%E5%8F%91%E6%9E%84%E5%BB%BA.png)

通过访问：ip/job/easy-springmvc/build?token= 12346（身份令牌）

如果能够触发构建就代表成功了

---

**通过构建触发器我们可以实现，自动化的构建了，但是不够灵活，不能实现版本的回退，于是就有了参数化构建**

---

### 七、参数化构建

**1、简介**

参数化构建，可以让我们实现定制化的任务

参数名称 = 变量值

可以理解为我们定义了一个变量，在构建时给不同值，让jenkins为我们做不同的事，也可以利用在脚本中

**2、利用参数化和tag实现版本的切换**

1）首先得有打标签的版本

![分支](https://gitee.com/Xmowan/ims/raw/master/ims/%E5%88%86%E6%94%AF.png)

2）简单配置一下

![定义一个Git参数](https://gitee.com/Xmowan/ims/raw/master/ims/%E5%AE%9A%E4%B9%89%E4%B8%80%E4%B8%AAGit%E5%8F%82%E6%95%B0.png)

![指定tag分支](https://gitee.com/Xmowan/ims/raw/master/ims/%E6%8C%87%E5%AE%9Atag%E5%88%86%E6%94%AF.png)

3）测试构建

![tag测试](https://gitee.com/Xmowan/ims/raw/master/ims/tag%E6%B5%8B%E8%AF%95.png)

----

### 八、流水线

流水线（pipline）是指按顺序连接在一起的事件或作业组 

**1、相关概念**

 **pipline** - 定义整个构建过程，通常包括构建应用程序、测试和交付应用程序的阶段。 

**node** - 节点，执行流水线的机器 

**stage** - 阶段，定义阶段性的任务，是多个step的子集 

**step** - 步骤，定义单一的任务 

**3、创建的方法**

1）通过Jenkinsfile语法来创建，通常有两种语法 一种是脚本式的 另一种是声明式的。声明式更容易上手。

2）还有一种方式就是通过Blue Ocean（蓝色海洋） 这个插件来创建，非常适合初学者

**4、一个声明式语法的示例**

```shell
node {
    stage('build'){
     echo 'build';
    }
    
    stage('test'){
     echo 'test';
    }
    
    stage('deploy'){
     echo 'deploy';
    }
}
```

无论以那种方式创建一个流水线后，都会在仓库中自动生成一个jenkinsfile的文件

