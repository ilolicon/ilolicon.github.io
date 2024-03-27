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

### rpm包私有仓库构建

- 所有构建好的.rpm包 全部放到准备好的目录`$BASE_DIR`
- 进入目录`cd $BASE_DIR` 执行命令`createrepo .`即可(或不进目录 跟绝对路径)
- 搭建http/ftp等服务进行访问 配置.repo文件即可进行安装

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

### deb私有仓库制作

借助reprepro设置仓库

[DebianRepoFormat](https://wiki.debian.org/DebianRepository/Format)

[SetupWithReprepro](https://wiki.debian.org/DebianRepository/SetupWithReprepro)

[creating-your-own-signed-apt-repository-and-debian-packages](https://scotbofh.wordpress.com/2011/04/26/creating-your-own-signed-apt-repository-and-debian-packages/)

- 安装软件包

```bash
sudo apt install -y gnupg reprepro
```

- 创建GPG密钥

```bash
# 两个命令都可以 查看man手册获取区别
gpg --gen-key
gpg --full-gen-key # 执行之后 按照提示操作即可

gpg -k, list-keys # 查看所有keys
```

- 配置包的存储库`BASE_DIR` 这里以nginx为例

```nginx
# 增加配置 私有仓库自行根据需求添加认证
server {
  listen 80;

  access_log /var/log/nginx/repo-error.log;
  error_log /var/log/nginx/repo-error.log;

  location / {
    root /srv/repos/apt;  # $BASE_DIR
    autoindex on;
  }

  location ~ /(.*)/conf {
    deny all;
  }

  location ~ /(.*)/db {
    deny all;
  }
}

# nginx -t
# systemctl reload nginx

```

- 配置reprepro [具体参考](https://wiki.debian.org/DebianRepository/SetupWithReprepro)

```bash
# create a reprepro configuration directory
mkdir -p /srv/repos/apt/debian/conf

# create the conf/distributions
Origin: Your project name
Label: Your project name
Codename: <osrelease>
Architectures: source i386 amd64
Components: main
Description: Apt repository for project x
SignWith: <key-id>

# <key-id>获取
$ gpg --list-secret-key --with-subkey-fingerprint
pub   rsa4096 2010-09-23 [SC]
      E123D55E623D56323D65E123655E623D563D5831
uid           [ultimate] Joe User (Some organization) <joe.user@example.com>
sub   rsa4096 2010-09-23 [E]
      F24957412415744F1495F149571F2495F2495714  

# Here <keyid> (fingerprint) for the OpenPGP key is F24957412415744F1495F149571F2495F2495714 (that's technically the subkey, which is recommended to be used for this sort of signing purpose).

# add an options file to make daily life with reprepro command-line a little easier. This file is in /srv/repos/apt/debian/conf/options:
verbose
basedir /srv/repos/apt/debian
ask-passphrase

# 执行命令创建仓库树
reprepro export  # 完整命令 reprepro --ask-passphrase -Vb /srv/repos/apt/debian export

# 把.deb包加入仓库
# 完整命令也需要加上 --ask-passphrase -Vb /srv/repos/apt/debian 
# 更多命令行用户参阅man手册: man reprepro
reprepro includedeb codename .deb-filename

# 把.deb移除仓库
reprepro remove codename package-names

# 导出OpenPGP key
gpg --armor --output whatever.gpg.key --export-options export-minimal --export <key-id>

# Copy this to a webserver so that users can download it and add it to their OpenPGP keychains similarly to this (as root):
# 注意: 该方法已弃用 参考下面`client端相关配置`
wget -O - http://www.example.com/repos/apt/conf/<whatever>.gpg.key | apt-key add -

```

### client端相关配置

http(s) deb仓库搭建完成之后 即可以进行相关包下载

- 添加key

```bash
# wget或curl获取key内容 再通过apt-key add -命令导入会有warning提示
# Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
# 下面命令废弃 不推荐
wget -O - http://www.example.com/repos/apt/conf/<whatever>.gpg.key | apt-key add -

```

- 参考下面回答添加key和source.list

[warning-apt-key-is-deprecated-manage-keyring-files-in-trusted-gpg-d-instead](https://stackoverflow.com/questions/68992799/warning-apt-key-is-deprecated-manage-keyring-files-in-trusted-gpg-d-instead)

```bash
sudo mkdir -m 0755 -p /etc/apt/keyrings/

curl -fsSL https://example.com/EXAMPLE.gpg |
    sudo gpg --dearmor -o /etc/apt/keyrings/EXAMPLE.gpg

echo "deb [signed-by=/etc/apt/keyrings/EXAMPLE.gpg] https://example.com/apt stable main" |
    sudo tee /etc/apt/sources.list.d/EXAMPLE.list > /dev/null

# Optional (you can find the email address / ID using `apt-key list`)
sudo apt-key del support@example.com

# Optional (not necessary on most systems)
sudo chmod 644 /etc/apt/keyrings/EXAMPLE.gpg
sudo chmod 644 /etc/apt/sources.list.d/EXAMPLE.list

```

- 如果http服务器添加过认证 需要使用apt-auth管理 否则有warning 参考下面内容
- [how-to-access-local-apt-repository-that-requires-http-authentication](https://stackoverflow.com/questions/18941517/how-to-access-local-apt-repository-that-requires-http-authentication)
- `man apt_auth.conf`查看具体信息

```bash
EXAMPLE
       Supplying login information for a user named apt with the password debian for the sources.list(5) entry

           deb https://example.org/debian bookworm main

       could be done in the entry directly:

           deb https://apt:debian@example.org/debian bookworm main

       Alternatively an entry like the following in the auth.conf file could be used:

           machine example.org
           login apt
           password debian

       Or alternatively within a single line:

           machine example.org login apt password debian

       If you need to be more specific all of these lines will also apply to the example entry:

           machine example.org/deb login apt password debian
           machine example.org/debian login apt password debian
           machine example.org/debian/ login apt password debian

       On the other hand neither of the following lines apply:

           machine example.org:443 login apt password debian
           machine example.org/deb/ login apt password debian
           machine example.org/ubuntu login apt password debian
           machine example.orga login apt password debian
           machine example.net login apt password debian

```
