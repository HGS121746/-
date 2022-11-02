# zabbix专有词汇
- zabbix serever，服务端，收集数据，写入数据
- zabbix agent，部署在被监控的机器上，是一个进程，和zabbix server进行交互，以及负责执行命令
- Host，服务器的概念，指zabbix监控的实体，服务器，交换机等
- Hosts，主机组
- Application，应用
- Events，事件
- Media，发送通知的渠道
- Remote command，远程命令
- Template，模板
- item，对于某一个指标的监控，称之为items，如某台服务器的内存使用情况，就是一个item监控项
- Trigger，触发器，定义报警的逻辑，有正常，异常，未知三个状态
- Action，当Trigger符合设定值后，zabbix指定的动作，如邮件发送

# zabbix程序组件
- Zabbix_server,服务端守护进程
- Zabbix_agentd，agent守护进程
- Zabbix_proxy,代理服务器
- Zabbix_database，储存系统，mysql，pgsql
- Zabbix_web，web GUI图形化界面
- Zabbix_get，命令行工具，测试向agent发送数据
- Zabbix_sender，命令行工具，测试向server发送数据
- Zabbix_java_gateway，java网关

# 安装zabbix5.0

php版本最低7.2.0

1.准备好一台linux服务器,环境初始化
    ifconfig ens33|awk "NR==2{print $2}"    #打印当前服务器ip地址
    # 关闭防火墙
    sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
    systemctl disable --now firewalld

2.zabbix-server内存设置尽量大，4G好

3.获取zabbix的下载源
    rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

4.更换zabbix.repo源为阿里的
    sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo

5.清空缓存,下载zabbix服务端
    yum clean all
    yun makecache

    yum install zabbix-server-mysql zabbix-agent -y

6.安装Software Collections，可以在机器上使用多个版本的软件，并且不会影响到整个机器的安装环境
    yum install centos-release-scl -y

7.修改zabbix-front前端源，修改如下参数
    [zabbix-frontend]
    name=Zabbix Official Repository frontend - $basearch
    baseurl=heeps://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/$basearch/frontend
    enabled=1 # 开启这里的参数
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

8.安装zabbix前端环境
    yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y

9.安装zabbix所需的数据库
    yum install mariadb-server -y

10.配置数据库，设置数据库开机自启
    systemctl enable --now mariadb


11.初始化数据库设置密码
    mysql_secure_installation

12.添加数据库用户，添加zabbix所需的数据库
    create database zabbix character set utf8 collate utf8_bin;
    # 创建用户名以及密码
    create user zabbix@localhost identified by '111333';
    # 授权
    grant all privileges on zabbix.* to zabbix@localhost;
    # 刷新授权表
    flush privileges;
    # 退出
    exit

13.使用zabbix-mysql命令，导入数据库信息
    # mysql -u用户名 -p 数据库名
    zcat /usr/share/doc/zabbix-server-mysql-5.0.28/create.sql.gz | mysql -u zabbix -p zabbix

14.修改zabbix server配置文件，修改密码
    vim /etc/zabbix/zabbix_server.conf
    grep "^DBPa" /etc/zabbix/zabbix_server.conf

15.修改zabbix的php配置文件
    # 更改时区
    vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
    # 更改为上海
    php_value[date.timezone] = Asia/Shanghai

16.启动zabbix相关服务器
    systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
    systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm

17.访问zabbix入口

18.安装成功后默认的账号
    Admin
    zabbix


# 部署zabbix客户端

agent2新版本采用golang语言开发，适应多核心机器

由于是go语言开发，部署比较方便，与之前不同

agent2默认使用10050端口
- 旧版本的客户端是，zabbix-anent1
- go语言新版本的客户端是，zabbix-agent2

    1.机器环境准备，两台zabbix客户端
    192.168.5.212
    192.168.5.213

    2.更新系统时间
    yum install ntpdate -y
    ntpdate -u ntp.aliyun.com
    date

    3.时区的同一配置
    #备份时区文件
    mv /etc/localtime{,.bak}
    #设置当前时区为上海
    ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

具体的zabbix-agent2部署流程
    #提前配置好zabbix仓库源
    rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
    sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo
    #安装agent2
    yum install zabbix-agent2 -y

    # 查看配置文件
    /etc/zabbix/zabbix_agent2.conf
    # 启动命令
    -rwxr-xr-x. 1 root root 15992160 Sep 19 17:48 /usr/sbin/zabbix_agent2
    # 设置开机并立即启动
    systemctl enable --now zabbix-agent2

    # 修改agent2配置文件，查看配置信息，根据自己的配置修改主机ip和主机名
    PidFile=/var/run/zabbix/zabbix_agent2.pid
    LogFile=/var/log/zabbix/zabbix_agent2.log
    LogFileSize=0
    Server=192.168.5.211
    ServerActive=192.168.5.211
    Hostname=rsync01
    Include=/etc/zabbix/zabbix_agent2.d/*.conf
    ControlSocket=/tmp/agent.sock

    # 重启
    systemctl restart zabbix-agent2

# 验证zabbix-agent2的连通性
    #回复为1则表示连通
    zabbix_get -s '192.168.5.212' -p 10050 -k 'agent.ping'

# 解决zabbix-server查看的乱码问题

    zabbix默认检测服务端本身，存在乱码问题
    1.安装字体文件
    yum -y install way-microhei-fonts
    2.复制字体，复制完成后界面自动刷新
    \cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf

