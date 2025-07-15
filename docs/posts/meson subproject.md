---
draft: true
comments: true
date: 2025-06-23
tags: [meson]
---

# meson subprojects
meson添加外部依赖的方式好几种，1.添加子目录和修改'meson.build'  2. subprojects 结果是第一种方式能自动编译，但是链接失败。第二种方式成功。

## subprojects 方式
1. 在项目根目录创建新目录'subprojects'，在'subprojects'目录下创建warp文件例如'libxml2.wrap'
```
[wrap-file]
directory = libxml2-2.14.3
patch_url = libxml2-2.14.3.tar.gz
source_filename = libxml2-2.14.3.tar.gz
[provide]
libxml-2.0 = xml_dep
```

patch_url可以是http文件，也可以是本地文件


2. 修改根目录的 'meson.build' 文件
```
subproject('libxml2', default_options: ['c_std=c11','python=disabled','minimum=true','writer=enabled'])
xml_dep = dependency('libxml-2.0')
```

3. 添加'xml_dep' 到编译可执行文件的依赖列表

## 添加子目录方式（失败）

1. 下载解压libxml-2.0的压缩档到跟目录或者其他相关目录
2. 修改父目录的'meson.build', 添加`subdir('doc')`， 声明依赖 `libxml2_dep = dependency('libxml-2.0', fallback: ['libxml2', 'libxml_dep'])` 
3. 添加'libxml2_dep' 到编译可执行文件的依赖列表