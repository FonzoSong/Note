#### 命令补全
```bash
openstack complete >> /etc/bash_completion.d/complete
echo "source /etc/bash_completion.d/complete" >> ~.bashrc
```
#### `ftp` 服务器设定默认路径
```shell
echo "anon_root=/opt/" >> /etc/vsftpd/vsftpd.conf
```

#### `gpt` 格式分区
```shell
parted /dev/sdb
mklabel gpt
mkpart
```
#### 脏数据回写调优
```shell
echo "60" > /proc/sys/vm/dirty_background_ratio
```

#### 用户创建
```shell
openstack user create \
--domain default \
--password tompassword123 \
--email tom@example.com tom
```

#### 镜像创建
```shell
openstack image create "cirros_0.3.4" \
--file chinaskills_cloud_iaas_v2.0.3.iso \
--disk-format qcow2 \
--container-format bare \
--public
```

#### 云主机类型创建
```shell
openstack flavor create m1 --id 56 --ram 2048 --disk 20 --vcpus 2
```
#### 网络创建
```shell
openstack network create --external ext-net

openstack subnet create --network ext-net \
  --subnet-range 192.168.200.0/24 \
  --gateway 192.168.200.1 \
  --allocation-pool start=192.168.200.100,end=192.168.200.200 \
  ext-subnet
```

#### 公网访问
```shell
cat /etc/nova/nova.conf | grep novncproxy_base_url
novncproxy_base_url = http://192.168.100.10:6080/vnc_auto.html
#novncproxy_base_url=http://127.0.0.1:6080/vnc_auto.html
将192.168.100.10改为公网IP
```

#### 数据库备份
```shell
mysqldump -u'root' -p '000000' --all-databases > /root/openstack.sql
```
#### 数据库用户创建
```sql
create user 'examuser'@'localhost';
ALTER USER 'examuser'@'localhost' IDENTIFIED BY '000000';
```
#### 安全组创建
[User-Facing Operations — operations-guide 2013.2.1.dev1200 documentation (openstack.org)](https://docs.openstack.org/operations-guide/ops-user-facing-operations.html#security-groups)
```shell
openstack security group create group_web --description "Custom security group"
openstack security group rule create --protocol icmp --ingress --ethertype IPv4 --dst-port -1 --remote-ip 0.0.0.0/0 group_web
openstack security group rule create --protocol tcp --ingress --ethertype IPv4 --dst-port 22 --remote-ip 0.0.0.0/0 group_web
```
#### 项目管理
```bash
openstack project create shop --description "Hello shop"
openstack project set --disable shop
openstack project show shop
```
#### 用户管理
```shell
openstack quota show admin
openstack quota set --instances 13 admin
openstack quota show admin
```

#### heat模板管理
```bash
openstack flavor create m2.flavor --id 1234 --ram 1024 --disk 20 --vcpus 1
```

#### 后端配置文件
```
[root@controller ~]# cat /etc/glance/glance-api.conf | grep _quota
# ``image_property_quota`` configuration option.
#     * image_property_quota
#image_member_quota = 128
#image_property_quota = 128
#image_tag_quota = 128
#image_location_quota = 10
#user_storage_quota = 0
[root@controller ~]# vim /etc/glance/glance-api.conf
[root@controller ~]# sudo systemctl restart openstack-glance-api
[root@controller ~]# sudo systemctl restart openstack-glance-registry
[root@controller ~]# cat /etc/glance/glance-api.conf | grep _quota
# ``image_property_quota`` configuration option.
#     * image_property_quota
#image_member_quota = 128
#image_property_quota = 128
#image_tag_quota = 128
#image_location_quota = 10
user_storage_quota = 10GB
```

#### 存储服务管理
```bash
openstack volume type create lvm
cinder type-key lvm set volume_backend_name=lvm
openstack volume create --size 1 --type lvm lvm_test
cinder show lvm_test
```
#### 存储管理
