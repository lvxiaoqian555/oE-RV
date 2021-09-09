1.映射端口
==
创建容器，配置端口映射，将container的22端口映射到D1的5678端口
```
docker run -p 5678:22-it riscv64/debian:experimental /bin/bash
```

2.debian container安装ssh服务
==
在debian container中安装openssh-server
```
apt-get update
apt-get upgrade
apt-get install openssh-server
apt-get install vim
```

修改/etc/ssh/sshd_config
```
PermitRootLogin yes
```

重启服务
```
/etc/init.d/ssh restart
```

修改root密码
```
passwd root
```
此时可以通过其他主机通过密码ssh container
```
ssh -p 5678 root@10.8.8.12 
```
此时通过用户名密码可以登录成功container说明ssh服务已经配置好

3.配置公钥登录
==
在其他主机上面运行如下命令，生成密钥对
```
ssh-keygen
```
回车三次，运行结束以后，在$HOME/.ssh/目录下，会新生成两个文件：id_rsa.pub和id_rsa。前者是你的公钥，后者是你的私钥。

将id_rsa.pub scp到D1
```
scp .ssh/id_rsa.pub root@10.8.8.12:/root
```
回到D1，如果目前在container里面，使用exit退出bash，container也会exit，使用docker ps -a查看container id
```
docker cp /root/id_rsa.pub container_id:/root
```
不管容器有没有启动，拷贝命令都会生效

启动容器并进入，如果没有.ssh目录就创建一个，将pub文件放到.ssh目录下，将内容追加在authorized_keys里面，并修改权限
```
docker start container_id
docker exec container_id
mkdir .ssh
mv id_rsa.pub .ssh/
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
chmod 700 ~/.ssh
```

修改/etc/ssh/sshd_config文件
```
PubkeyAuthentication yes
```
重启服务
```
/etc/init.d/ssh restart
```
此时通过刚才那台主机可以ssh到container，并且不需要输入密码
```
ssh -p 5678 root@10.8.8.12 
```
