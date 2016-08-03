# ansible的安装配置，应用初学

## 安装：
ubantu:`apt-get install ansible`

centos:`yum install ansible -y` 

## Ansible配置及测试
Ansible 配置主机文件路径为：·/etc/ansible/hosts·
打开后有默认的配置样例，如下：
```
    # Ex 1: Ungrouped hosts, specify before any group headers.

    #green.example.com
    #blue.example.com
    #192.168.100.1
    #192.168.100.10

    # Ex 2: A collection of hosts belonging to the 'webservers' group

    #[webservers]
    #alpha.example.org
    #beta.example.org
    #192.168.1.100
```
参照样例添加我们自己的主机文件，如下：
```
    [olly_servers]
    192.168.57.133
    192.168.57.130
    192.168.57.131
```
使用命令`ssh-keygen -t rsa`，在主控机上创建一个密钥对

使用ssh-copy-id命令把公钥拷贝到目标受控主机上(拷贝过程输入密码)
```
    ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.57.133
    ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.57.130
    ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.57.131
```

做一个简单测试，看配置是否生效，如下：（-m命令标识 引入ansible的模块）
```
    root@ubuntu:~# ansible olly_servers -m ping
    192.168.57.133 | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }
    192.168.57.131 | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }
    192.168.57.130 | SUCCESS => {
        "changed": false,
        "ping": "pong"
    }

```
Ansible提供了很多的功能模块，Commands（命令行）、Files（文件管理）、Monitoring（监控管理）、Network（网络管理）、Source Control（版本控制）、System（系统服务）、等等，
更多模块介绍见...

http://ansibleworks.com/docs/modules.html
## Ansible常用的模块
### command模块：
可以实现远程shell命令运行。command作为Ansible的默认模块，可以运行远程权限范围所有的shell命令，
例如批量查询机器状态：
```
    root@ubuntu:~# ansible olly_servers -m command -a "free -m"
    192.168.57.133 | SUCCESS | rc=0 >>
                  total        used        free      shared  buff/cache   available
    Mem:           7264        1628        4213           9        1422        5371
    Swap:          4095           0        4095

    192.168.57.130 | SUCCESS | rc=0 >>
                  total        used        free      shared  buff/cache   available
    Mem:           3791         134        3445           8         211        3447
    Swap:          1023           0        1023

    192.168.57.131 | SUCCESS | rc=0 >>
                  total        used        free      shared  buff/cache   available
    Mem:           3791         135        3448           8         207        3448
    Swap:          1023           0        1023

```
例如，批量为机器安装工具
```
root@ubuntu:~# ansible olly_servers -m command -a "yum install bc -y"
192.168.57.131 | SUCCESS | rc=0 >>
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.163.com
 * extras: mirrors.163.com
 * updates: mirrors.163.com
Resolving Dependencies
--> Running transaction check
---> Package bc.x86_64 0:1.06.95-13.el7 will be installed
--> Finished Dependency Resolution

```
### script模块：
script功能是在远程主机执行主控端存储的shell脚本文件，相当于scp+shell组合
例如：getcpuinfo.sh在主控机本地目录下，可以让远程主机执行该脚本：
```
root@ubuntu:~# ls -l
total 8
-rwxr-xr-x 1 root root 702 Aug  3 02:08 getcpuinfo.sh
-rw-r--r-- 1 root root 521 Jul 31 23:28 test.py
root@ubuntu:~# ansible olly_servers -m script -a "./getcpuinfo.sh"
192.168.57.133 | SUCCESS => {
    "changed": true,
    "rc": 0,
    "stderr": "",
    "stdout": ".951%\r\n",
    "stdout_lines": [
        ".951%"
    ]
}
192.168.57.130 | SUCCESS => {
    "changed": true,
    "rc": 0,
    "stderr": "",
    "stdout": ".148%\r\n",
    "stdout_lines": [
        ".148%"
    ]
}
192.168.57.131 | SUCCESS => {
    "changed": true,
    "rc": 0,
    "stderr": "",
    "stdout": ".099%\r\n",
    "stdout_lines": [
        ".099%"
    ]
}
```
###shell模块:
shell功能是执行在远程主机的shell脚本文件。