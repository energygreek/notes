---
title: arduino-cli usage
comments: true
date: 2024-12-13
tags: [arduino]
---

# arduino-cli 使用
在[esp32](esp32.md)中，使用了arduino-cli来编译和上传代码。这里记录下arduino-cli操作arduino nano的方法和问题

## 编译
说也非常奇怪，Linux的java GUI版本arduino编译提示找不到头文件'SoftwareSerial.h'， 这个库是arduino自带的。但是不使用GUI的编译按钮而用GUI的输出中的命令编译又正常。 

这是从GUI的IDE的编译上传的输出日志，得到的编译上传命令， 手动执行却正常。
```sh
cd ~/Arduino
BUILD_PATH=$(mktemp -d)
arduino-builder -verbose -compile -hardware /usr/share/arduino/hardware -tools /usr/share/arduino/hardware/tools/avr -libraries /
./Arduino/libraries  -build-cache ./arduino_cache -build-path $BUILD_PATH  -fqbn=arduino:avr:uno  -prefs=build.warn_data_percentage=75 ./TestSerial/TestSerial.ino
avrdude -C/etc/avrdude.conf -v -patmega328p -carduino -P/dev/ttyUSB0 -b115200 -D -Uflash:w:${BUILD_PATH}/TestSerial.ino.hex:i 
```

用arduino-cli编译也正常
```
arduino-cli compile --fqbn  arduino:avr:nano   TestSerial 
```

## 上传失败

但是用arduino-cli编译正常，上传失败，提示
```
arduino-cli upload --fqbn arduino:avr:uno --port /dev/ttyUSB0 TestSerial                                                                                                            
avrdude: error at /etc/avrdude.conf:402: syntax error                                                                                                                                           
avrdude: error reading system wide configuration file "/etc/avrdude.conf"                                                                                                                       
Failed uploading: uploading error: exit status 1   
```

后来用verbos输出对比GUI版本的命令，发现两个命令用的avrdude不一样。 一个是/usr/bin/avrdude， 一个是$HOME/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/bin/avrdude。两个的版本不一样， 而后者使用了前者的配置文件 /etc/avrdude.conf。所以出现`syntax error`
```
arduino-cli upload -v  --fqbn arduino:avr:uno --port /dev/ttyUSB0 TestSerial                                                                                                        
"/home/jimery/.arduino15/packages/arduino/tools/avrdude/6.3.0-arduino17/bin/avrdude" "-C/etc/avrdude.conf" -v -V -patmega328p -carduino "-P/dev/ttyUSB0" -b115200 -D "-Uflash:w:/home/jimery/.ca
che/arduino/sketches/57AA4FA906132DC236F48CCE34943FA0/TestSerial.ino.hex:i"                                                                                                                     
                                                                                                                                                                                                
avrdude: Version 6.3-20190619                                                                                                                                                                   
         Copyright (c) 2000-2005 Brian Dean, http://www.bdmicro.com/                                                                                                                            
         Copyright (c) 2007-2014 Joerg Wunsch                                                   
                                                                                                                                                                                                
         System wide configuration file is "/etc/avrdude.conf"                                                                                                                                  
avrdude: error at /etc/avrdude.conf:402: syntax error                                                                                                                                           
avrdude: error reading system wide configuration file "/etc/avrdude.conf"                                                                                                                       
Failed uploading: uploading error: exit status 1 
```

最后全部卸载了GUI包括avrdude， 再将$HOME目录的avrdude配置复制到/etc/avrdude.conf， 问题解决了。