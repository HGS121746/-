# **运维自动化**
    Ansible是一个同时管理多个远程主机的软件，必须可以通过ssh登录

    inventory file 主机列表文件

    playbook 脚本运行内容

    只能是通过ssh协议登录的主机，就可以完成ansible自动化部署操作

- 批量文件分发
- 批量数据复制
- 批量数据修改，删除
- 批量自动化安装软件服务
- 批量服务启停
- 脚本化，自动批量服务部署

**ansible特点**
- 无需安装客户端软件
- 使用yaml配置文件语法
- 能出色完成各种任务配置管理

**基于Python开发，有paramiko以及PyYAML模块**
- 安装部署简单
- 管理主机便捷，支持多台主机并行管理
- 无需安装被管理节点的客户端（no agent），且无需占用客户端的其他端口，仅仅使用ssh服务即可
- 不仅只支持python，还支持其他语言的二次开发
- 不用root权限也可以执行，降低系统权限

**ansible实践部署**

    准备3个linux虚拟机，设置静态地址

**准备ansible管理机器**
- 选择yum自动化安装，阿里云yum，epel源提前配置好
  
  yum install epel-release -y

  yum install ansible libselinux-python -y

- 检查ansible安装情况
  rpm -ql ansible|grep -E "^/etc|^/usr/bin"
  查询配置文件和可执行命令操作

- 检查ansible版本
  ansible --version

**准备被管理机器**

    安装ansible依赖模块
    yum install epel-release libselinux-python -y
**ansible管理方式**
- 传统的ssh密码验证
- 密钥管理
  配置好ansible的配置文件，添加被管理机器的ip地址或者主机名称

  1.备份现有的配置文件

  cp /etc/ansible/hosts{,.ori}

  2.添加ansible需要管理的机器

        [chaoge]

        192.168.5.213

        192.168.5.212

**ssh密码认证方式管理机器**

ansible直接利用linux本地的ssh服务，以及一些远程的ssh操作，一般情况下客户机的远程ssh都是开启的无需额外操作

1.在1号机器上执行如下命令

-m 指定功能模块，默认就是command模块

-a 告诉模块需要执行的参数

-k 询问密码验证

-u 指定运行的用户

在管理机器上，告诉其他被管理的机器，你要执行什么命令

ansible 主机列表 -m command -a "hostname" -k -u root

2.如上操作，一般情况下会有错误提示，需要按照如下动作执行，只需要手动ssh对机器进行过一次连接就好，记录指纹数据

3.此时可以再次执行ansible命令，在管理机器上让被管理的机器显示我们想要的结果

**配置免密登录**

每次执行ansible命令的时候都需要输入ssh认证的密码，如果不同的主机密码不一致，还得多次数据密码才行
- ansible自带的密码认证参数
可在hosts文件中定义好密码，即可实现快速的认证

        参数

        ansible_host    主机地址

        ansible_port    端口，默认是22

        ansible_user    认证的用户

        ansible_ssh_pass    用户认证的密码

**使用hosts文件的参数形式来实现ssh认证**

1.修改hosts文件

    [chaoge]
    192.168.5.213 ansible_user=root ansible_ssh_pass=111333
    192.168.5.212 ansible_user=root ansible_ssh_pass=111333

2.此时不需要密码即可ssh自动验证

ansible chaoge -m command -a "hostname"

**ssh密钥方式批量管理主机**

1.在管理机器上创建密钥对

    [root@192 ansible]# ssh-keygen -f ~/.ssh/id_rsa -P "" > /dev/null 2>&1

2.检查公私钥对文件

    [root@192 ansible]# cd ~/.ssh/
    [root@192 .ssh]# ls
    id_rsa  id_rsa.pub  known_hosts

**编写公钥分发脚本**

    #!/bin/bash
    rm -rf ~/.ssh/id_rsa*
    ssh-keygen -f ~/.ssh/id_rsa -P "" > /dev/null 2>&1
    SSH_Pass=111333
    Key_Path=~/.ssh/id_rsa.pub
    for ip in 212 213
    do
            sshpass -p$SSH_Pass ssh-copy-id -i $Key_Path "-o StrictHostKeyChecking=no" 192.168.5.$ip
    done
此时在管理机器上再连接客户端机器就无需再使用账号密码了，可以尝试使用ansible连接

# 总结

在生产环境中，ansible的连接方式，二选一即可，最好是配置ssh公私钥免密登录

如果生产环境的要求更高，可以先用普通用户登录再提权操作


# amsible模式与命令
ansible实现批量化主机管理的模式，主要有两种
- 利用ansible的纯命令行实现的批量管理,ad-hoc模式------好比简单的shell命令管理
- 利用ansible的playbook剧本来实现批量管理，playbook剧本模式----复杂的shell脚本管理

## ad-hoc模式
ansible的ad-hoc模式是ansible的命令行模式，也就是处理一些临时的，简单的任务，可以直接使用ansible的命令行来操作
- 临时批量查看机器运行情况
- 临时文件分发

## playbook模式
ansible的palybook模式是针对比较具体，且比较大的任务，那么你就得写好剧本操作
- 一键部署rsync备份服务器
- 一键部署lnmp环境

## ansible的ad-hoc命令行解析
让被管理机器返回主机名

    [root@m01 ~]# ansible chaoge -m command -a "hostname"
    192.168.5.212 | CHANGED | rc=0 >>
    nfs01
    192.168.5.213 | CHANGED | rc=0 >>
    rsync01
ad-hoc命令解释
- ansible---自带提供的命令操作
- chaoge---/etc/ansible/hosts文件中定义的主机组，还可以写主机ip，以及通配符*
- -m command ansible的指定模块的参数，以及指定了command模块
- -a 指定给command模块什么参数，hostname，uname -r

**ansible-doc命令**
列出所有的ansible支持的模块

    ansible-doc -l|grep ^command
查看某个模块的具体用法

    ansible-doc -s command
## ansible模块精讲
**command模块**

    作用：在远程节点上执行一个命令
ansble-doc -s command  查看该模块支持的参数

    chdir 在执行命令之前，先通过cd进入该参数指定的目录
    creates 在创建一个文件之前，判断该文件是否存在，如果存在了则跳过前面的东西，如果不存在则执行前面的动作
    free_form 该参数可以输入任何的系统命令，实现远程执行和管理
    removes 定义一个文件是否存在，如果存在则执行前面的动作，如果不存在则跳过动作
command模块式ansible的基本模块，可以省略不写，但是要注意如下的坑
- 使用command模块，不得出现shell变量$name,也不得出现特殊符号><|;&这些符号command模块都不认识，如果你想使用前面指定的变量，特殊符号，请使用shell模块，command不适用

让客户端机器，先切换到/tmp目录下，然后打印当前的工作目录

    [root@m01 ~]# ansible chaoge -m command -a "pwd chdir=/tmp/"
    192.168.5.212 | CHANGED | rc=0 >>
    /tmp
    192.168.5.213 | CHANGED | rc=0 >>
    /tmp

练习creates参数

该参数的作用是判断文件是否存在，存在则跳过，不存在则执行

    ansible 192.168.5.212 -m command -a "pwd creates=/opt"
    [root@m01 ~]# ansible 192.168.5.212 -m command -a "pwd creates=/opt"
    192.168.5.212 | SUCCESS | rc=0 >>
    skipped, since /opt exists

参数removes实践，存在则执行，不存在则跳过

    [root@m01 ~]# ansible chaoge -a "ls /op removes=/op"
    192.168.5.213 | SUCCESS | rc=0 >>
    skipped, since /op does not exist
    192.168.5.212 | SUCCESS | rc=0 >>
    skipped, since /op does not exist

warn参数，是否提供告警信息

    [root@m01 ~]# ansible chaoge -m command -a "chmod 000 /etc/hosts"
    [WARNING]: Consider using the file module with mode rather than running 'chmod'.  If
    you need to use command because file is insufficient you can add 'warn: false' to this
    command task or set 'command_warnings=False' in ansible.cfg to get rid of this
    message.
    192.168.5.213 | CHANGED | rc=0 >>

    192.168.5.212 | CHANGED | rc=0 >>

    [root@m01 ~]# ansible chaoge -m command -a "chmod 000 /etc/hosts warn=False"
    192.168.5.213 | CHANGED | rc=0 >>

    192.168.5.212 | CHANGED | rc=0 >>

# shell模块

作用：在远程机器上执行命令（复杂的命令）

了解模块用法的渠道
- linux命令行里面通过ansible-doc
- ansible官网查看帮助信息

    shell模块支持的参数与解释
    chdir       在执行命令之前，通过cd进入该参数指定的目录
    creates     定义一个文件是否存在，如果存在则不执行该命令，不存在则执行
    free_form   参数信息中可以输入任何系统指令，实现远程管理
    removes     顶一个一个文件是否存在，如果存在则执行命令

shell模块案例

    批量查询进程信息
    amsible chaoge -m shell -a "ps -ef|grep xx"
    批量写入文件
    amsible chaoge -m shell -a "echo 111> /tmp/111.txt"

## 批量远程执行脚本

    需要执行的脚本，必须在客户机上存在，否则会报文件不存在，这是shell模块的特点，是因为还有一个专门执行脚本的script模块
    注意的是这个脚本必须在客户端机器上存在才行
    1.创建文件夹
    2.创建sh脚本文件，还要写入脚本内容
    3.赋予脚本可执行权限
    4.执行脚本
    5.忽略warning信息

思路分析

    最好所有的操作都是在 管理机器上，也就是（管理）机器上m01上进行远程的批量化操作

    ansible chaoge -m shell -a "mkdir -p /server/myscripts/;echo 'hostname' > /server/myscripts/hostname.sh;chmod +x /server/myscripts/hostname.sh;bash /server/myscripts/hostname.sh warn=False"

## script模块

功能：把管理机器上的脚本远程的传输到被管理节点上去执行

比起shell模块，script模块功能更强大，在管理机器上有一份脚本，就可以在所有被管理节点上去运行

ansible-doc -s script

远程的执行脚本，在客户端机器上不需要存在该脚本

ansible chaoge -m script -a "/myscripts/local_hostname.sh"
