## k8s ansible自动化部署
### 1. ansible安装
ansible主控机安装ansible, shell如下
```shell script
# 安装
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install ansible
```
### 执行步骤
1. ansible-playbook -i hosts init-site.yml
2. ansible-playbook -i hosts site.yml
### 1. 内核升级配置
组配置文件`group_vars/kernelservers`

配置项
```yaml
---
# 内核索引 可通过  awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg 查询
kernel_index: 0
```

### 2. ssh免密配置
组配置文件`group_vars/sshservers`
```yaml
---
# 登录ssh主机列表的用户名和密码， 若部分主机密码不一致，可在 host_vars目录下新建与主机同名的文件，独配置
ansible_ssh_name: root
ansible_ssh_pass: 123456

# 是否删除旧的ssh目录
delete_old_ssh_dir: 0

# ansible会将所有主机公钥copy至ansible主控机，之后ansible控制机再将免密文件分发至控制机
ansible_master: 192.168.2.49
ansible_master_ssh_pass: 123456
```

### 3. harbor安装
harbor配置，默认配置如下
```yaml
---
harbor_version: "1.10.5"
harbor_home: "/usr/local"
```
同时需要修改harbor`roles/harbor/files/harbor.yml` 必需修改项`hostname`和`https`,若hostname不可公网访问，则需要在/etc/hosts中配置映射


### 4. nfs安装配置
若安装多台nfs server则可以在`host_vars`目录下新建与主机同名的文件，文件内容为
```yaml
---
nfs_server: 0

# nfs数据共享目录
nfs_data_dirs: []
```
同时修改nfs-server目录导出配置`oles/nfs/files/exports ` , 默认如下
```text
/data/nfs 192.168.2.*(rw,sync,no_root_squash)
```

### 5. 安装 nginx配置
此处安装的nginx是供k8s集群使用，安装过程会读取配置的`k8s_master_servers`组下所有主机列表，并且nginx所在主机需包含k8s集群主机域名映射
默认配置中已将nginxservers做为k8sservers组的子组存在

### 6. 安装docker
k8s集群所有主机均需要安装docker组件，默认dockerservers组已做为k8sservers组的子组存在， 同时需要为docker添加配置, 配置文件为`group_vars/dockerservers`具体配置项如下:
```yaml
---
# vars file for docker
# docker 镜像一列表
mirrors: 
  - "http://registry.cnegroup.com"
  - "https://docker.mirrors.ustc.edu.cn"
  - "http://f1361db2.m.daocloud.io"
  - "https://registry.docker-cn.com"
# 信任仓库列表
repositories: 
  - "registry.cnegroup.com"

# 开启tcp端口
enable_tcp: 0
```

### 7. k8s集群安装
k8s配置文件`k8sservers`, 具体配置项如下
```yaml
---
# 集群主机域名映射
host_mappings: 
  - "192.168.2.46 ansible-01"
  - "192.168.2.47 ansible-02"
  - "192.168.2.48 ansible-03"
  - "192.168.2.49 cne-k8s.com"
  - "192.168.2.49 ansible-master"
```
