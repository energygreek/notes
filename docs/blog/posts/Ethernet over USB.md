---
draft: false
comments: true
date: 2025-12-12
tags: [netowrk, USB, 'NCM Gadget', '树莓派']
---

# Ethernet over USB
NCM 全称“Network Control Model”，是USB定义的用来交换以太网帧。也就是USB网卡，但是相比Windows的RNDS,速度快多了。实测在300MBps以上。而且听说是不安全导致Linux废弃了RNDS的支持。

## 树莓派
在网上买了一个USB供电板，免焊接的。用大模型生成了USB网卡配置脚本
```sh
#!/bin/bash
set -ex

CONFIGFS=/sys/kernel/config
GADGET=usb_gadget/g1
GADGET_DIR=$CONFIGFS/$GADGET
UDC=$(ls /sys/class/udc | head -n1)

# 1. Load libcomposite and mount configfs
modprobe libcomposite
mount -t configfs none $CONFIGFS || true

# 2. Create gadget directory
mkdir -p "$GADGET_DIR"
cd "$GADGET_DIR"

# 3. Set vendor/product IDs (example values—you can adjust)
echo 0x1d6b > idVendor     # Linux Foundation
echo 0x0104 > idProduct    # Multifunction Composite Gadget

# 4. Create English strings
mkdir -p strings/0x409
echo "serial1234" > strings/0x409/serialnumber
echo "MyManuf" > strings/0x409/manufacturer
echo "My NCM Gadget" > strings/0x409/product

# 5. Add NCM function
mkdir -p functions/ncm.usb0
# You may optionally set MACs, interface name, qmult, etc.
# echo "12:34:56:78:9a:bc" > functions/ncm.usb0/host_addr
# echo "12:34:56:78:9a:bd" > functions/ncm.usb0/dev_addr
# echo "usb0" > functions/ncm.usb0/ifname

#!/bin/bash
set -ex

CONFIGFS=/sys/kernel/config
GADGET=usb_gadget/g1
GADGET_DIR=$CONFIGFS/$GADGET
UDC=$(ls /sys/class/udc | head -n1)

# 1. Load libcomposite and mount configfs
modprobe libcomposite
mount -t configfs none $CONFIGFS || true

# 2. Create gadget directory
mkdir -p "$GADGET_DIR"
cd "$GADGET_DIR"

# 3. Set vendor/product IDs (example values—you can adjust)
echo 0x1d6b > idVendor     # Linux Foundation
echo 0x0104 > idProduct    # Multifunction Composite Gadget

# 4. Create English strings
mkdir -p strings/0x409
echo "serial1234" > strings/0x409/serialnumber
echo "MyManuf" > strings/0x409/manufacturer
echo "My NCM Gadget" > strings/0x409/product

# 5. Add NCM function
mkdir -p functions/ncm.usb0
# You may optionally set MACs, interface name, qmult, etc.
# echo "12:34:56:78:9a:bc" > functions/ncm.usb0/host_addr
# echo "12:34:56:78:9a:bd" > functions/ncm.usb0/dev_addr
# echo "usb0" > functions/ncm.usb0/ifname

# 6. Create config and assign function
mkdir -p configs/c.1
mkdir -p configs/c.1/strings/0x409
echo "Config NCM" > configs/c.1/strings/0x409/configuration
ln -s functions/ncm.usb0 configs/c.1/

# 7. Bind to UDC (activate gadget)
echo "$UDC" > UDC

echo "NCM gadget activated"

# Wait for interface to appear
for i in {1..10}; do
    if ip link show usb0 &>/dev/null; then
        break
    fi
    sleep 0.5
done

# move to usb_ncm_up.service
#nmcli connection up usb0-shared || true
```

## NetworkManager 配置

```sh
nmcli connection add type ethernet ifname usb0 con-name usb0-shared ipv4.method shared
nmcli connection up usb0-shared
```


## 配置systemd自动运行和拉起网口
设备上的USB网卡配置脚本要在NetworkManager启动前执行，而拉起设备要在启动NetworkManager后执行，所以分拆成两个服务。

usb_ncm_gadget.service
```
[Unit]
Description=Setup USB NCM Gadget
Before=NetworkManager.service
After=local-fs.target sysfs.mount

[Service]
Type=oneshot
ExecStart=/usr/local/bin/usb_ncm_gadget.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

等NetworkManager启动后，拉起网口

usb_ncm_gadget_up.service
```
[Unit]
Description=Bring Up NM connection for usb0
After=NetworkManager.service
Requires=NetworkManager.service
Requires=usb_ncm_gadget.service
Wants=sys-subsystem-net-devices-usb0.device
After=sys-subsystem-net-devices-usb0.device

[Service]
Type=oneshot
ExecStart=/usr/bin/nmcli connection up usb0-shared

[Install]
WantedBy=multi-user.target
```
