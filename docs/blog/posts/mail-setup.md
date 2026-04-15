---
date: 
  created: 2023-07-18
  update: 2026-02-02
tags: [email, mutt]
---

# Linux下的邮件配置
这里记录Linux（debian）下如何使用Mutt查看邮件，用fetchmail接收邮件，用msmtp发送邮件以及使用maildrop来筛选邮件。与其他邮件客户端相比，如thunderbird，他们使用的在线登陆方式，没有下载邮件来离线访问。而mutt支持离线访问本地的邮件，也可以配置为访问邮件服务器，查看服务器上的邮件。 在线方式除了离线不可用的缺点之外，每次打开mutt需要登陆，非常慢。
我这里的配置是比较简陋，适合公司少量邮件的场景。又因为目前使用的Byobu-terminal终端，其邮件提醒是基于/var/mail/spool/$USER文件。邮件比较多或者附件比较大的话，相较于用文件夹来存储邮件，这种方式会比较慢。

## 软件安装
需要安装 fetchmail, maildrop, mutt

## 接收配置
在家目录下创建文件.fetchmailrc
```
poll imap.exmail.qq.com
  protocol IMAP
  user "example@example.com"
  password "password"
  keep
  #fetchall
  ssl

mimedecode
mda "/usr/bin/maildrop"
```
keep 方式不会删除服务器上的邮件。然后启动定时任务，自动检查新邮件
```
systemctl --user enable --now fetchmail.service
```

service 内容
```
# /usr/lib/systemd/user/fetchmail.service
# or any other location per man:systemd.unit(5)
[Unit]
Description=Fetchmail Daemon
Documentation=man:fetchmail(1)

[Service]
ExecStart=fetchmail --nodetach --daemon 300
ExecStop=fetchmail --quit
Restart=always

[Install]
WantedBy=default.target
```

## 发送配置

在家目录创建文件.msmtproc，填入发送的验证信息
```
defaults
auth           on
tls            on
tls_starttls off
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log

# exmail
account        office
host           smtp.exmail.qq.com
port           465
from           example@example.com
user           example@example.com
password       password
```

## mutt配置
由于我不想删除/var/mail/里面的邮件，这样才能在byobu-terminal中提醒，所以没有配置maildrop。最后需要配置mutt读取/var/mail/$USER。在家目录或者xdg配置目录（~/.config/mutt）创建文件.muttrc/muttrc
```
set spoolfile=/var/spool/mail/jimery
set header_cache=~/.cache/mutt
set send_charset="utf-8"
```

这样就能查看邮件了。 在mutt里面手动管理邮件，存档或者删除。

## mutt使用
其实不用mutt, 直接用mail也可以，这个默认访问/var/mail/$USER,是Linux默认的邮件工具。 mutt能很方便调用msmtproc来发送邮件。


## maildrop in Arch
又换回Arch了，发现maildrop在AUR里，而且还依赖好几个AUR, 而自己编译只依赖courier-unicode库。

1. 下载courier-unicode源码
2. ./configure --prefix=$HOME/.local --enable-shared=no --with-pic  # 静态编译，指定安装到用户根目录
3. make && make install # 必须安装否则编译maildrop的时候找不到这个库，即使指定路径也不行。 
4. 下载maildrop源码
5. ./configure --prefix=/home/hst/.local --enable-shared=no # 静态编译，指定安装到用户根目录
6. make
7. 验证是静态链接的courier-unicode，确认`ldd ./libs/maildrop/maildrop` 里面没有'courier-unicode'

Arch里面创建用户时默认没有对应的邮箱文件，直接手动创建`sudo touch "/var/spool/mail/$(whoami)"`, 还有chmod

## neomutt
mutt 中浏览pdf等非text的附件时，mutt应该保存文件到指定目录，然后使用第三方软件打开的。但浏览器提示文件不存在。 用neomutt就能解决。

### foxmail适配
Foxmail邮件subject的字符编码默认是GBK, 除非在设置里修改。 邮箱协议最初协议默认是ASCII收发subject。现代的大部分邮箱普遍使用UTF-8收发。 
收和发不一致的问题，本应该像html那样，用`content-type`来显式表明发邮件一方使用的字符编码。  
可是邮箱协议并没有为subject定义content-type字段， 只有MIME有方法设置content-type和charset。不过可以通过下面的方式为subject设置charset  
```
Subject: =?charset?encoding?encoded-text?=
```

但是foxmail没有这么做，而是之间将GBK编码塞到了subject位置，所以乱码。
```
Date: Tue, 3 Feb 2026 10:24:34 +0800
From:
To:
Cc: 
Subject: »Ø¸´: Èý·½Í¨»°²âÊÔÎÊÌâ-ÈýÐÇÖÕ¶ËÍ¬Ñù²¦²â³¡¾°Ò»´ÎÕý³££¬Ò»´ÎÈý·½¶¼Ã»ÓÐÉùÒô
References:
X-Priority: 3
X-GUID: 9BF12FA7-4B7A-4CC5-B07D-1B3C61294575
X-Has-Attach: no
X-Mailer: Foxmail 7.2.9.156[cn]
Mime-Version: 1.0
Message-ID:
Content-Type: multipart/related;
------=_002_NextPart574464278560_=----
Content-Type: text/plain;
        charset="GB2312"
Content-Transfer-Encoding: base64
```

解决办法：要么要求发件人修改foxmail配置，要么该收件人配置，在neomutt的配置增加`set assumed_charset="gbk:utf-8"`就能解决这个问题
