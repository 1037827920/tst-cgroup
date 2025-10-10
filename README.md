
- [引言](#引言)
- [tst-cgroup测试套使用文档](#tst-cgroup测试套使用文档)
  - [运行测试](#运行测试)
  - [操作指南](#操作指南)
  - [其他](#其他)

# 引言

本项目始于[2024腾讯犀牛鸟开源人才计划](https://opensource.tencent.com/summer-of-code)OpenCloudOS社区的[Testing sig发布的issue](https://gitee.com/OpenCloudOS/contributor_rhino-bird/issues/IA76YL?from=project-issue)。

代码提交在gitee仓库地址：https://gitee.com/opencloudos-testing/tst-cgroup

# tst-cgroup测试套使用文档

## 运行测试

### 克隆到本地

```bash
git clone https://github.com/1037827920/tst-cgroup.git
```

### 安装必要的工具

```bash
sudo bash install_package.sh
```

### 编译cgroup util中的程序

在tst_lib/cgroup_util目录下输入 `make` 命令编译程序，生成的可执行文件在tst_lib/cgroup_util/bin中

### 运行单个测试用例

在根目录下，以root权限运行

```bash
sudo ./tsuite run testcase/$TESTCACE
```

`TESTCASE` 是具体的测试用例名称

### 运行整个测试套

在根目录下，以root权限运行

```bash
sudo ./tsuite run
```

运行结果在logs/.ts.sysdir/run.result文件中

## 操作指南

### 切换cgroup版本

切换到cgroup v1，在/etc/default/grub的GRUB_CMDLINE_LINUX添加`systemd.unified_cgroup_hierarchy=0`

切换到cgroup v2，在/etc/default/grub的GRUB_CMDLINE_LINUX添加`systemd.unified_cgroup_hierarchy=1`

然后更新配置：

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 启用zswap

以下测试用例需要启用系统的zswap：

- cgroup-v2-memory-zswap

修改方式：在/etc/default/grub的GRUB_CMDLINE_LINUX添加`zswap.enabled=1`

然后更新配置：

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 在qemu上测试

以下测试用例需要在有两个NUMA节点及以上的系统中测试：

- cgroup-v1-cpuset-mems
- cgroup-v1-cpuset-mem_exclusive
- cgroup-v1-cpuset-memory_migrate-001
- cgroup-v1-cpuset-memory_migrate-002
- cgroup-v1-cpuset-memory_spread_page-001
- cgroup-v1-cpuset-memory_spread_page-002
- cgroup-v2-cpuset-mems

OpenCloudOS 8：下载qcow2镜像格式

```bash
qemu-system-x86_64 -machine q35 \
-smp 8,sockets=2,cores=4,threads=1 \
-m 8G \
-cpu host \
-enable-kvm \
-drive file=OpenCloudOS-8.8-5.4.119-20.0009.29-20230817.1028-x86_64.qcow2,format=qcow2 \
-object memory-backend-ram,size=4G,id=ram-node0 \
-object memory-backend-ram,size=4G,id=ram-node1 \
-numa node,nodeid=0,cpus=0-3,memdev=ram-node0 \
-numa node,nodeid=1,cpus=4-7,memdev=ram-node1 \
-net nic -net user,hostfwd=tcp::2222-:22  \
-vnc :0
```

OpenCloudOS 9: 下载minimal镜像格式

```bash
qemu-img create -f qcow2 OpenCloudOS9 20G 
```

```bash
qemu-system-x86_64 -machine q35 \
-smp 8,sockets=2,cores=4,threads=1 \
-m 8G \
-cpu host \
-enable-kvm \
-drive file=OpenCloudOS9,format=qcow2 \
-cdrom OpenCloudOS-9.2-x86_64-minimal.iso \
-object memory-backend-ram,size=4G,id=ram-node0 \
-object memory-backend-ram,size=4G,id=ram-node1 \
-numa node,nodeid=0,cpus=0-3,memdev=ram-node0 \
-numa node,nodeid=1,cpus=4-7,memdev=ram-node1 \
-net nic -net user,hostfwd=tcp::2222-:22 \
-vnc :0
```

OpenCloudOS Stream 23: 下载minimal镜像格式

```bash
qemu-img create -f qcow2 OpenCloudOS-Stream-23 20G 
```

```bash
qemu-system-x86_64 -machine q35 \
-smp 8,sockets=2,cores=4,threads=1 \
-m 8G \
-cpu host \
-enable-kvm \
-drive file=OpenCloudOS-Stream-23,format=qcow2 \
-cdrom OpenCloudOS-Stream-23-20240304-minimal-x86_64.iso \
-object memory-backend-ram,size=4G,id=ram-node0 \
-object memory-backend-ram,size=4G,id=ram-node1 \
-numa node,nodeid=0,cpus=0-3,memdev=ram-node0 \
-numa node,nodeid=1,cpus=4-7,memdev=ram-node1 \
-net nic -net user,hostfwd=tcp::2222-:22 \
-vnc :0
```

### 切换hugepage大小

以下测试用例需要修改系统hugepage默认大小：

- cgroup-v1-hugetlb-limit_in_bytes-003
- cgroup-v1-hugetlb-limit_in_bytes-004
- cgroup-v1-hugetlb-rsvd-limit_in_bytes-003
- cgroup-v1-hugetlb-rsvd-limit_in_bytes-004
- cgroup-v2-hugetlb-max-003
- cgroup-v2-hugetlb-max-004
- cgroup-v2-hugetlb-rsvd-max-003
- cgroup-v2-hugetlb-rsvd-max-004

修改方式：在/etc/default/grub的GRUB_CMDLINE_LINUX添加`default_hugepagesz=1G hugepagesz=1G`

然后更新配置：

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 设置swapfile

创建一个空文件：

```
sudo fallocate -l 4G /swapfile
```

设置正确的权限：

```
sudo chmod 600 /swapfile
```

格式化为swap：

```
sudo mkswap /swapfile
```

启用Swap文件：

```
sudo swapon /swapfile
```

永久启用：

```
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## 其他

### 添加新的测试用例

```bash
./tsuite new case sh|c|py cgroup-v1-cpuset-cpus-001
```

### 显示所有测试用例

```bash
./tsuite list
```

### 参考

- [Control Groups version 1 — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v1/index.html)
- [Control Group v2 — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html#)
- [linux/tools/testing/selftests/cgroup at master · torvalds/linux](https://github.com/torvalds/linux/tree/master/tools/testing/selftests/cgroup)
- [[译\] Control Group v2（cgroupv2 权威指南）（KernelDoc, 2021）](https://arthurchiao.art/blog/cgroupv2-zh/#hugetlb-interface-files)
- [第 3 章 子系统和可调参数 | Red Hat Product Documentation](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/6/html/resource_management_guide/ch-subsystems_and_tunable_parameters#ch-Subsystems_and_Tunable_Parameters)
- [cgroup 子系统之 net_cls 和 net_prio | ggaaooppeenngg](https://ggaaooppeenngg.github.io/zh-CN/2017/05/19/cgroup-子系统之-net-cls-和-net-prio/)



