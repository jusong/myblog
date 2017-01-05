---
layout: post
title:  "gentoo安装vmware-workstation-12.5"
date:   2017-01-06 02:28:47 +0800
categories: linux gentoo vmware
---

**下载vmware workstation的二进制安装包**
```
$ wget https://download3.vmware.com/software/wkst/file/VMware-Workstation-Full-12.5.0-4352439.x86_64.bundle
```

**用root用户执行安装**
```
$ chmod +x VMware-Workstation-Full-12.5.0-4352439.x86_64.bundle
$ ./VMware-Workstation-Full-12.5.0-4352439.x86_64.bundle
```

**运行vmware**
```
$ vmware
```
会提示让编译vmware的模块到内核，编译过程中如果出现如下错误
```
/tmp/modconfig-NRdC2o/vmci-only/linux/driver.c:1682:4: 错误：隐式声明函数‘vmalloc’ [-Werror=implicit-function-declaration]
    data_buffer = vmalloc(data_buffer_size);
    ^
/tmp/modconfig-NRdC2o/vmci-only/linux/driver.c:1682:16: 警告：赋值时将整数赋给指针，未作类型转换
    data_buffer = vmalloc(data_buffer_size);
                ^
/tmp/modconfig-NRdC2o/vmci-only/linux/driver.c:1690:7: 错误：隐式声明函数‘vfree’ [-Werror=implicit-function-declaration]
       vfree(data_buffer);
       ^
/tmp/modconfig-NRdC2o/vmci-only/linux/driver.c: 在函数‘vmci_exit’中:
/tmp/modconfig-NRdC2o/vmci-only/linux/driver.c:2486:14: 错误：void 值未如预期地被忽略
       retval = misc_deregister(&linuxState.misc);
              ^
/tmp/modconfig-NRdC2o/vmci-only/linux/vmciKernelIf.c: 在函数‘__VMCIMemcpyToQueue’中:
/tmp/modconfig-NRdC2o/vmci-only/linux/vmciKernelIf.c:1205:10: 错误：隐式声明函数‘memcpy_fromiovec’ [-Werror=implicit-function-declaration]
          err = memcpy_fromiovec((uint8 *)va + pageOffset, iov, toCopy);
          ^
/tmp/modconfig-NRdC2o/vmci-only/linux/vmciKernelIf.c: 在函数‘__VMCIMemcpyFromQueue’中:
/tmp/modconfig-NRdC2o/vmci-only/linux/vmciKernelIf.c:1280:10: 错误：隐式声明函数‘memcpy_toiovec’ [-Werror=implicit-functio
```
则需要对vmware的模块代码包进行打补丁，这是在arch社区里找到的补丁
```
$ cd /tmp
$ git clone https://aur.archlinux.org/vmware-patch.git
```

解压vmware的模块代码包进行打补丁，再打包好放回去

vmware的模块代码包在/usr/lib/vmware/modules/source/下面 
```
$ ls /usr/lib/vmware/modules/source/
vmblock.tar  vmci.tar  vmmon.tar  vmnet.tar  vsock.tar
```

此处根据之前编译的错误提示找出需要进行打补丁的包，如
```
$ tar xf /usr/lib/vmware/modules/source/vmci.tar -C /tmp/
$ tar xf /usr/lib/vmware/modules/source/vsock.tar -C /tmp/
```

然后进行打补丁
```
$ patch -p0 < vmware-patch/vmci-12.0.0-4.2.patch
$ patch -p0 < vmware-patch/vsock-11.1.2-4.2.patch
```
补丁里包含多个版本，选择对应版本的即可

最后进行重新打包放回原处
```
$ tar cf /usr/lib/vmware/modules/source/vmci.tar vmci-only 
$ tar cf /usr/lib/vmware/modules/source/vsock.tar  vsock-only
```

重新执行vmware进行编译模块
```
$ vmware
```

如果无法启动，查看错误日志，提示找不到version.h文件，则需要创建软连接
```
$ ln -s /lib/modules/`uname -r`/build/include/generated/uapi/linux/version.h /lib/modules/`uname -r`/build/include/linux/version.h
```
然后再次启动vmware就可以了
