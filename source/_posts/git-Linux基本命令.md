---
title: git&Linux基本命令
tags: [git,Linux]
---

git和Linux的一些基本命令操作

<!-- more -->

##### git

###### 基本

```shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"

$ git init

$ git add readme.txt
$ git commit -m "wrote a readme file"

$ git status
$ git diff readme.txt 

$ git log		$ git log --pretty=oneline

$ git reset --hard HEAD^
$ git reset --hard 1094a
$ git reflog
```



###### 远程

```shell
$ ssh-keygen -t rsa -C "youremail@example.com"
$ git remote add origin https://.....($ git remote add origin git@github.com:michaelliao/learngit.git)
$ git push -u origin(远程仓库名) master(本地分支名) master(远程分支名)  (-u首次建立关联)
$ git push origin master
$ git clone git@github.com:michaelliao/gitskills.git
git remote rm origin
git remote -v

$ git push origin  master(本地分支名) master(远程分支名)
$ git pull
$ git branch --set-upstream-to=origin/dev dev（绑定本地分支）
```

###### 分支

```shell
$ git checkout -b dev（$ git switch -c dev）===
$ git branch dev
$ git checkout dev（$ git switch master）
$ git merge dev
$ git branch -d dev（not merge  -D）
$ git log --graph --pretty=oneline --abbrev-commit
$ git rebase（直线提交历史）


```

###### 隐藏

```shell
$git stash
$ git stash list
$ git stash pop
$git stash apply(恢复不删除)
$ git cherry-pick 4c805e2
```

###### 标签

```shell
$ git tag v0.9 [f52c633]
$ git tag [标签名]
$ git show v0.9
$ git tag -d v0.9		删除标签
$ git push origin v1.0
$ git push origin --tags	推送所有标签
$ git push origin :refs/tags/v0.9
```

###### 多关联

```shell
git remote add github git@github.com:michaelliao/learngit.git
git remote add gitee git@gitee.com:liaoxuefeng/learngit.git
git push github master
git push gitee master
```

###### 别名

```shell
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

##### Linux

###### 基本命令

```she'l
ls: 列出目录	
cd：切换目录	(Change Directory)
pwd：显示目前的目录	(Print Working Directory)
mkdir：创建一个新的目录	 (make directory)
rmdir：删除一个空的目录
cp: 复制文件或目录
rm: 移除文件或目录
mv: 移动文件与目录，或修改文件与目录的名称
dhclient 自动分配地址[BOOTPROTO=STATIC	ONBOOT=YES	
IPADDR=192.168.XX.XXX	NETMASK=255.255.255.0 	GATEWAY=192.168.XX.X
DNS1=119.29.29.29]
```

###### 用户/组管理

```shell
useradd [user]	useradd [-g]  组名 [user]
passwd [user] 
userdel [-r] [user]
id [user]
su [user]

groupadd [group]
groupdel [group]
gpasswd  -d student root 将用户student从root组删除
usermod [-g] 组名 [user]
```

###### 常用

```shell
cat [dir] | grep [..]
ls -l /home |grep "!-"|wc -l 查看某文件夹下文件的个数
stat [file]

[ls,cat,echo...] >>追加
[ls,cat,echo...] >覆盖

head/tail -n [num] [file]     		tail -f [file]

ln -s [dir] [name] 

date "+%Y %M %D....."
cal

find [scope] -name [name]
find [scope] -user [user]
find [scope] -size  [+-n]

tar -zcvf [name].tar.gz [files] 打包
tar -zxvf [files] -C [dir] 解压

rpm [-qa | -qi | qf]
rpm -e [--nodeps(强删)] [name]
rpm -ivh		i=install  v=verbose提示  h=hash进度条
```

###### 文件权限管理

```shell
chown/chgrp [user/group] [file]
chmod u=[rwx],g=[rwx],o=[rwx] [file]
chmod u[+-][rwx],g[+-][rwx],o[+-][rwx] [file]
chomd [777] [file]     (4=r,2=w,1=x)
```

###### 任务调度

```shell
crontab [-e -l -r]  ([写入 查看 删除])
*/1 * * * * [执行文件/脚本] 
```

###### 磁盘管理

```shell
df -h
du -axh	文件夹占用
lsblk -f  	查看是否已分配
fdisk /dev/dba	分区
mkfs -t ext4 /dev/sdb1   格式化
mount  /dev/sdb1   /home/newdisk	挂载
/dev/fstab  设置开机自动挂载
```

###### 进程管理

```shell
ps -aux | grep ...
ps -ef 		-e显示所有进程 -f全格式
kill [-9] 进程号	-9强制
killall 进程名称
top [-d | -i | -p]	-d秒数 -i不显示闲置或僵死 -p指定进程id监控
```

###### 服务管理

```shell
systemctl [status | stop | start | restart | reload ]firewalld
ll /etc/init.d/	
netstat -anp 	-an按一定顺序 -p显示哪个在调用   查看网络服务
```

###### Shell编程

	  #!/bin/bash开头
	  A=100;
	  unset A --->A=
	位置参数变量
	$n	第n个参数
	$*	整体
	$@	分别
	$#	参数个数
	=====================

```shell
预定义变量
$$	当前进程PID
$!	后台运行最后一个PID
$?	最后一次命名的返回状态 0√
========================

运算(2+3)*4
#方式一$()
RESULT1=$(((2+3)*4))
#方式二$[]  推荐方式
RESULT2=$[(2+3)*4]
#方式三 expr
................
==========================
条件判断
if[ condition ] 
then
	echo""
elif[ condirion ] 
	echo""
fi
 -lt小于 -le小于等于  -gt大于 -ge大于等于  -eq等于 -ne不等于
-r/-w/-x 有读/写/执行的权限
-f文件存在并且时一个常规文件 -e文件存在 -d文件存在并是目录
------------------------
case $变量名 in
"值1")
echo"1"
;;
"值2")
echo"2"
;;
*)
echo"other"
;;
esac
==================

循环
for 变量 in 值1 值2 值3...
do
	程序
done 
-------------------------
for ((i=0;i<100;i++))
do
	程序
done 
--------------------------
while[ condition ]
do
	程序
done 
===============

read读取控制台输入
read -p  "读取时的提示符" -t指定读取时等待的时间(秒)	NUM

================
函数
function getSum(){
	SUM=$[$n1+$n2]
}
read -p "输入n1" n1
read -p "输入n2" n2
#调用
getSum $n1 $n2
```