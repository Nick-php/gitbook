#### ssh 公私钥登录配置

第一步，在客户端生成公钥
```
ssh-keygen -t rsa #生成的公钥匙在 ~/.ssh/ 目录下
```
第二步，把公钥上传到服务器端
```
scp id_rsa.pub root@ip地址:文件保存路径
cat id_rsa.pub >> /root/.ssh/authorized_keys 追加到文件中

```

第三步，修改服务器端 ssh 配置文件 sshd_config

```
RSAAuthentication yes #开启RSA验证
PubkeyAuthentication yes #是否使用公钥验证
PasswordAuthentication no #禁止使用密码验证登录
chmod 600 /root/.ssh/authorized_keys  #为了安全把文件修改权限
```

第四步，服务器端重启ssh服务
```
service sshd restart #重启 sshd 服务
```

---

#### 禁止 ping 服务器配置

第一种方法：临时生效配置
```
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1 #临时生效禁止 ping 作用，重启服务器后恢复

sudo sysctl -w net.ipv4.icmp_echo_ignore_all=0 #解除禁止 ping 作用
```

第二种方法：永久生效配置

编辑 /etc/sysctl.conf 文件
```
sudo vim /etc/sysctl.conf 
```
在配置文件中添加 
```
net.ipv4.icmp_echo_ignore_all=1 
``` 
如果已经有 `net.ipv4.icmp_echo_ignore_all` 这一行了，直接修改=号后面的值即可的。（0表示允许，1表示禁止）
修改完成后执行 `sysctl -p` 使新配置生效

第三种方法：通过 iptables 配置
```
sudo iptables -A INPUT -p icmp --icmp-type 8 -s 0/0 -j DROP #往防火墙添加规则

删除已添加的iptables规则
sudoi ptables -L -n --line-numbers #将所有iptables以序号标记显示

sudo iptables -D INPUT 8 #删除INPUT里序号为8的规则
```