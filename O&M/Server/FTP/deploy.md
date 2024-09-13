##### 安装与基本配置
```bash
sudo dnf install -y vsftpd
sudo systemctl start vsftpd
sudo mkdir -p /etc/vsftpd/vuser_conf
## 配置文件路径/etc/vsftpd/vsftpd.conf
sudo vi /etc/vsftpd/vsftpd.conf

## 注意以下参数 如过你使用的是RHEL系统那么应该已经配置完毕了
anonymous_enable=NO #禁用匿名用户
local_enable=YES #启用本地用户
write_enable=YES #启用写权限
guest_enable=YES #启用虚拟用户
guest_username=ftp #同上
chroot_local_user=YES #chroot 限制
allow_writeable_chroot=YES #同上
user_config_dir=/etc/vsftpd/vuser_conf #虚拟用户独立配置文件
pam_service_name=vsftpd #PAM认证
listen_ipv6=NO #IPV6 禁用
listen=YES #启用IPV4
```
##### 虚拟用户模式配置
###### 创建虚拟用户数据库文件
格式为:
```text
virtualuser1
password1
virtualuser2
password2
```
使用 `db_load` 工具将其转换为 Berkeley DB 格式的数据库文件：
```bash
sudo db_load -T -t hash -f virtual_users.txt /etc/vsftpd/virtual_users.db
```
###### 编辑PAM 文件
```bash
sudo cp -a /etc/pam.d/vsftpd /etc/pam.d/vsftpd.bak

# 修改配置文件内容为
#%PAM-1.0
auth required pam_userdb.so db=/etc/vsftpd/virtual_users
account required pam_userdb.so db=/etc/vsftpd/virtual_users
```

##### 安全配置
```bash
#防火墙开端口
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --reload

#selinux
sudo semanage fcontext -a -t public_content_rw_t "/home/fonzo/Public/Ftp(/.*)?"

```

##### 一个配置文件
```bash
`local_root=/home/fonzo/Public/Ftp write_enable=YES anon_world_readable_only=NO anon_upload_enable=YES anon_mkdir_write_enable=NO anon_other_write_enable=NO`

此配置将：

- 将用户的根目录设置为 `/home/fonzo/Public/Ftp`。
- 允许虚拟用户在此目录下进行写入（如上传文件）。
- 禁止创建新目录和其他写操作（如删除文件或更改文件权限）
```