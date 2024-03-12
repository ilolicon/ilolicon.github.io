---
title: Build installation package on linux
description: Build installation package on Debian series or Redhat series.
date: 2024-03-11 19:53:00+08:00
categories: 
- Linux
tags:
- dkpg
- rpm
- debian
- ubuntu
- redhat
- centos
---

## rpm

### rpm包构建

[RPM打包原理、示例、详解及备查](https://www.cnblogs.com/yipianchuyun/p/15442896.html)

### rpm配置文件处理

```bash
# .spec文件参考配置 

Name:           pkg name
Version:        %{version}
Release:        %(date +%%y%%m%%d%%0k)
Summary:        pkg summary

License:        xxx
URL:            xxx
Source0:        %{name}-%{version}.tar.gz


%description
pkg description.


%prep
%setup -q


%install
mkdir -p %{buildroot}%{_bindir}
mkdir -p %{buildroot}%{_sysconfdir}/%{name}
mkdir -p %{buildroot}%{_sysconfdir}/default
mkdir -p %{buildroot}%{_unitdir}

install -m 0755 %{_builddir}/%{name}-%{version}/.build/%{_os}-amd64/%{name} %{buildroot}%{_bindir}/%{name}
install -m 0644 %{_builddir}/%{name}-%{version}/dist/%{name}.env %{buildroot}%{_sysconfdir}/default/%{name}
install -m 0644 %{_builddir}/%{name}-%{version}/config.yml %{buildroot}%{_sysconfdir}/%{name}/config.yml
install -m 0644 %{_builddir}/%{name}-%{version}/dist/web-config.yml %{buildroot}%{_sysconfdir}/%{name}/web-config.yml
install -m 0644 %{_builddir}/%{name}-%{version}/dist/%{name}.service %{buildroot}%{_unitdir}


# 以下为配置文件处理参考 重点:$config(noreplace)
%files
%defattr(-,root,root,-)
%{_bindir}/%{name}
%config(noreplace) %{_sysconfdir}/%{name}
%config(noreplace) %{_sysconfdir}/default
%{_unitdir}/%{name}.service


%post
systemctl daemon-reload


%changelog
* Wed May 24 2023 xxx <xxx@xx.com>
- initial

```

### 问题处理

- Q: `GO`源码包构建遇到`RPM build warnings: Missing build-id in /bin/xxx`问题
- A: 参考该文档解决 [也谈GO的可移植性](https://tonybai.com/2017/06/27/an-intro-about-go-portability/)

## deb

### deb包构建

[从零开始制作deb文件](https://hedzr.com/packaging/deb/creating-deb-file-from-scratch/#debiancontrol)

[Debian新维护者手册](https://www.debian.org/doc/manuals/maint-guide/)

### deb配置文件处理

[How to mark some file in debian package as config?](https://askubuntu.com/questions/473354/how-to-mark-some-file-in-debian-package-as-config)

[Debian Policy Manual](https://www.debian.org/doc/debian-policy/ap-pkg-conffiles.html)

针对上面链接说的第三种情况: 本地安装 和 包维护者 都修改过配置文件的情况
升级会提示用户相关问题 需要用户自行解决差异
如果安装的时候 不想进入交互模式(比如批量安装的时候) 则可以使用dpkg的以下参数解决:

- --force-confold: 保留当前配置文件 将新配置文件安装为`.dpkg-dist`扩展名 需要手动合并
- --force-confnew: 覆盖当前配置文件，但将旧配置文件备份为`.dpkg-old`扩展名
- --force-confdef: 当用户未指定具体选项时 采取默认行为`保留、覆盖或交互`

```bash
sudo dpkg -i --force-confnew package_name.deb  # dpkg
apt install --force-confnew package_name       # apt

```
