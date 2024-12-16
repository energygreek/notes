---
title: mail setup
date: 2023-07-18 18:24:55
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
