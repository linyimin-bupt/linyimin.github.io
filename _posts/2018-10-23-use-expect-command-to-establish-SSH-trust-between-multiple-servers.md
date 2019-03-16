## 使用expect自动化完成集群SSH信任关系的建立

1. 将以下脚本存成文件，并修改其权限
```bash
$ chmod 775 establish-ssh.sh
```
2. 在同一目录下新建servers.txt文件，并填入相关IP
```bash
linyimin@:capra(master)$ cat servers.txt
192.168.121.10
192.168.121.11
192.168.121.12
192.168.121.13
192.168.121.14
192.168.121.15
```
3. 执行establish-ssh.sh文件
```bash
$./establish-ssh.sh
```
**脚本文件**
```bash
#!/bin/bash
# The name who want to ssh without password
USER_UID=linyimin
# The System user home directory
USER_DIR=/home/linyimin
# Password of the other server user
USER_PASSWORD='password'

#################################### main #####################################
if [ -f servers.txt ]
then
  :
else
  echo
  echo "########### Please touch File: servers.txt ##########################"
  echo "########### servers.txt For example #################################"
  echo "########### 192.168.1.10 ############################################"
  exit 1
fi

# if expect is not installed then install it
dpkg --get-selections | grep -q expect
if [ $? -eq 0 ]
then
  :
else
  echo "########## Please install expect #####################################"
  exit 1
fi

# if id_rsa.pub not exist,
if [ -f $USER_DIR/.ssh/id_rsa.pub ]
then
  :
else
  expect -c "
  spawn ssh-keygen
  expect \"Enter file in which to save the key*\"
  send \"\r\"
  expect \"Enter passphrase*\"
  send \"\r\"
  expect \"Enter same passphraseagain:\"
  send \"\r\"
  "
fi
for SSH_IP in `cat servers.txt`
do
  echo "$SSH_IP"
  expect -c "
  spawn ssh-copy-id -i $USER_DIR/.ssh/id_rsa.pub $SSH_IP
  expect \"*yes/no*\"
  send \"yes\r\"
  expect \"*password*\"
  send \"${USER_PASSWORD}\"
  "
  if [ $? -eq 0 ]
  then
    echo "------------- $SSH_IP is ok. -----------------------"
  else
    echo "------------- $SSH_IP is fail ----------------------"
  fi
done
```

**存在问题**
在使用的时候发现IP对应的主机不存在的时候，打印的还是`------------- $SSH_IP is ok. -----------------------`，
也就是expect命令执行完成之后`$?`始终是0