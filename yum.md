# Yum 学习笔记

## 背景

[Yum](http://linux.duke.edu/projects/yum/)

## 学习教程

- [yum cookbook](https://github.com/chef-cookbooks/yum)

## 学习内容

Yellowdog Updater, Modified

### [常用命令](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/ch-yum)

-  查看可用的更新

  ```bash
  yum check-update
  ```

- 更新单个 rpm

  ```bash
  yum update package_name
  ```

- 安装所有的更新

  ```bash
  yum update
  yum update-minimal --security
  ```

- 定时更新包

  ```bash
  yum install yum-cron
  service yum-cron start
  service yum-cron status
  ```

-  查找包

  ```bash
  yum search vim gvim emacs
  yum search all vim gvim emacs
  ```

- 列出包

  ```bash
  yum list *rmc*
  yum list all
  yum list installed
  yum list available
  yum grouplist
  yum repolist
  ```

- 查看包的详情

  ```bash
  yum info package_name(rpm -q --info package_name)
  ```

-  安装包

  ```bash
  yum install package_name
  yum install package_name package_name…
  yum groupinstall group_name
  yum groupinstall groupid
  ```

  - 查看特定文件属于哪个软件包

  ```bash
  yum provides "*bin/named"
  ```

- 卸载包

  ```bash
  yum remove package_name…
  yum groupremove group
  ```

- 查看 yum 历史

  ```bash
  yum history list
  yum history package-list *rmc*
  yum history addon-info last

  yum history new #新建历史记录
  ```

| Action     | Abbreviation | Description                                                   |
| ---------- | ------------ | ------------------------------------------------------------- |
| Downgrade  | D            | At least one package has been downgraded to an older version. |
| Erase      | E            | At least one package has been removed.                        |
| Install    | I            | At least one new package has been installed.                  |
| Obsoleting | O            | At least one package has been marked as obsolete.             |
| Reinstall  | R            | At least one package has been reinstalled.                    |
| Update     | U            | At least one package has been updated to a newer version.     |

| Symbol | Description                                                                                                                  |
| ------ | ---------------------------------------------------------------------------------------------------------------------------- |
| <      | Before the transaction finished, the rpmdb database was changed outside Yum.                                                 |
| >      | After the transaction finished, the rpmdb database was changed outside Yum.                                                  |
| \*     | The transaction failed to finish.                                                                                            |
| #      | The transaction finished successfully, but yum returned a non-zero exit code.                                                |
| E      | The transaction finished successfully, but an error or a warning was displayed.                                              |
| P      | The transaction finished successfully, but problems already existed in the rpmdb database.                                   |
| s      | The transaction finished successfully, but the --skip-broken command-line option was used and certain packages were skipped. |

- 回滚操作

  ```bash
  yum history undo id
  ```

- 重做

  ```bash
  yum history redo id
  yum -q history addon-info id saved_tx > file_name
  yum load-transaction file_nam
  ```

- 掉电异常处理

  ```bash
  yum-complete-transaction
  yum-complete-transaction --cleanup-only
  ```

### repo  [配置项](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-configuring_yum_and_yum_repositories)

- 全局配置文件/etc/yum.conf

  ```bash
  cachedir=/var/cache/yum/$basearch/$releasever #缓存地址
  keepcache=0 #1：安装成功后本地仍保留包 0：不保留
  debuglevel=2 #范围1-10,级别越高，日志越详细,0:disable
  logfile=/var/log/yum.log #日志文件
  exactarch=1 #是否检查架构 0：不检查 1：检查（32位和64位不能互相兼容）
  obsoletes=1 # 1:更新时，执行老包废弃逻辑
  gpgcheck=1 #是否进行gpg检查 0：不检查 1：检查
  plugins=1 # 0:不可用所有插件 1：可用
  installonly_limit=5 #限制同时安装的rpm个数
  ```

-  子仓库配置

- 目录：/etc/yum.repos.d/

  ```bash
  [xxx-base-12]
  name=xxx-base-12
  enabled=1
  gpgcheck=0
  baseurl=http://xxx.mmb.com/dev/os/centos6.7_64
  ```

- 配置文件中的变量

  - $releasever 发行版，对应 conf 中的 distroverpkg.例如 6Client 或 6Server
  - $arch,CPU 架构， i686 或 x86_64
  - $basearch,系统基础架构，i386 或 x86_64
  - $YUM0-9

- 创建自己的$变量

  ```bash
  echo "Red Hat Enterprise Linux" > /etc/yum/vars/osname
  ```

### yum 是怎么工作的？

yum install jdk

1. 获取  库中所有可用的 package 的列表
2. 计算 yum 是否可以获取到你所需要  包的依赖
3. 如果 2 满足，则在用户确认后，下载相关的依赖包,并安装

### [创建一个 yum 源仓库 createRepo](http://yum.baseurl.org/wiki/RepoCreate)

-  创建私有 repo

  ```bash
  yum install createrepo
  mkdir -p  /home/privateRepo
  createrepo --database /home/privateRepo #privateRepo中会生成yum的元数据repodata,sqlite数据库会提高yum操作速度

  createrepo --unique-md-filenames /home/privateRepo #所有元数据的文件名必须唯一,这个选项通常用在镜像库
  ```

- 一个库中有大量的  包，只有  小部分增加或者  改变时，耗时太长。

  ```bash
  createrepo --update /srv/my/repo
  ```

- 排除库中的某些包

  ```bash
    createrepo -x filename -x filename2 -x filename* /srv/my/repo
  ```

- 改变 checksum

  ```bash
  createrepo --checksum /srv/my/repo
  ```

### 详解[repodata](https://www.slashroot.in/yum-repository-and-package-management-complete-tutorial)

- filelists.xml.gz

  - pkgid 包 id(唯一)
  - name 包名
  - arch 架构
  - version 版本号
  - file 将要被安装的文件名

  ```xml
  <package pkgid="deee52b24486906ee52576ee471b57061ccd5544" name="php-mbstring" arch="i386">
    <version epoch="0" ver="5.1.6" rel="32.el5" />
    <file>
      /etc/php.d/mbstring.ini
    </file>
    <file>
      /usr/lib/php/modules/mbstring.so
    </file>
  </package>
  ```

- primary.xml.gz

  - 基本信息
  - sha hash
  - 概述

  ```xml
  <package type="rpm">
    <name>
      php53-mbstring
    </name>
    <arch>
      i386
    </arch>
    <version epoch="0" ver="5.3.3" rel="5.el5" />
    <checksum type="sha" pkgid="YES">
      e4d153d1ac6f71fa50bb6587cf13b324ee44537c
    </checksum>
    <summary>
      A module for PHP applications which need multi-byte string handling
    </summary>
    <description>
      The php-mbstring package contains a dynamic shared object that will add
    </description>
  </package>
  ```

- repomd.xml

  - 定位 primary.xml.gz,filelists.xml.gz,other.xml.gz
  - 修改的时间戳
  - checksum

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <repomd xmlns="http://linux.duke.edu/metadata/repo">
  <data type="other">
    <location href="repodata/other.xml.gz"/>
    <checksum type="sha">53349b063023303c5976135a7e485b0aa932ba6f</checksum>
    <timestamp>1360494599</timestamp>
    <open-checksum type="sha">2245d5a3cf0c3398a28b0c63948ecdf58668041e</open-checksum>
  </data>
  <data type="filelists">
    <location href="repodata/filelists.xml.gz"/>
    <checksum type="sha">35d6d80d5b115f0da3452534c1f5cf9c0efcef3f</checksum>
    <timestamp>1360494599</timestamp>
    <open-checksum type="sha">5d1f2791794e1cc20590099a962cf5260f942b7a</open-checksum>
  </data>
  <data type="primary">
    <location href="repodata/primary.xml.gz"/>
    <checksum type="sha">afe11b0976096135357e11e2ff1707d88ebef6fd</checksum>
    <timestamp>1360494599</timestamp>
    <open-checksum type="sha">976baa466007d496accfa24922f2f0722c56f1dd</open-checksum>
  </data>
  </repomd>
  ```

- other.xml.gz
  - 作者
  - 版本信息
  - 修改记录
  - 之前版本的 bugfix 信息等
