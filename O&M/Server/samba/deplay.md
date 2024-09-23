##### 安装
```bash
sudo dnf install  samba
```
配置文件
```conf

[global]
	workgroup = SAMBA
	security = user
	passdb backend = tdbsam

	printing = cups
	printcap name = cups
	load printers = yes
	cups options = raw

[public]
	comment = Public Directory for each user
	path = %H/Public   # 指向每个用户的 ~/Public 目录
	valid users = %S    # 仅允许用户访问自己的 public 目录
	browseable = Yes    # 使共享可见
	read only = No     # 设置为只读
	create mask = 0755  # 确保文件权限为只读
	directory mask = 0755
	inherit acls = Yes

[printers]
	comment = All Printers
	path = /var/tmp
	printable = Yes
	create mask = 0600
	browseable = No

[print$]
	comment = Printer Drivers
	path = /var/lib/samba/drivers
	write list = @printadmin root
	force group = @printadmin
	create mask = 0664
	directory mask = 0775

```

##### 创建并启用本地用户

```bash
sudo useradd -M -s /sbin/nologin smb 
sudo passwd smb 
smbpasswd -a smb 
sudo smbpasswd -a smb 
sudo smbpasswd -e smb
```