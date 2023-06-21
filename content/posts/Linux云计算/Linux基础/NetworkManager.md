
遇到过三个问题和这个服务有关系，均需要把这个服务关掉就好了
- 虚拟机启动慢
   > 关掉这个服务立马就快了，看系统日志可以看到这个服务好像去拉去什么网络信息导致虚拟机启动很慢
```bash
service network start 启动报LSB错误 关掉服NetManager后网络启动正常
```

使用云服务器，ping不通外网 第一次遇到，重启了天翼云服务器 第二次遇到，变聪明了，重启了网络服务解决，希望下次不要再出现连不上的情况

```bash
Nov 24 09:41:41 ecs-5295 NetworkManager[626]: <info>  [1606182101.5430] manager: NetworkManager state is now CONNECTED
Nov 24 09:41:41 ecs-5295 NetworkManager[626]: <info>  [1606182101.5479] manager: NetworkManager state is now CONNECTED
Nov 24 09:41:41 ecs-5295 NetworkManager[626]: <info>  [1606182101.5481] policy: set 'eth0' (eth0) as default for IPv4
Nov 24 09:41:41 ecs-5295 NetworkManager[626]: <info>  [1606182101.5516] device (eth0): Activation: successful, device
Nov 24 09:41:41 ecs-5295 NetworkManager[626]: <info>  [1606182101.5522] manager: NetworkManager state is now CONNECTED
```

```bash
chkconfig NetworkManager off #永久关闭
```

注意，在标准的centos8中，如果禁止开机自启动，把它关了。会导致获取不到ip地址。自研的云桌面服务器，虽然把这个服务禁用了，自制的ISO中是带了network这个服务的，所以不影响。

在centos8中，<font color="#ffc000">做bond使用nmcli命令很方便，这个打开还是很有用的</font>。
```bash
service NetworkManager stop
chkconfig NetworkManager off
chkconfig network on
service network start
```