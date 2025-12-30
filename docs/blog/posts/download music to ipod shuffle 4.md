---
draft: false
comments: true
date: 2025-08-28
tags: [music,apple]
---

# download music to ipod shuffle 4


## install ipod-shuffle-4g.py

Python script for building the Track and Playlist database for the newer gen IPod Shuffle.
Forked from the [shuffle-db-ng project](https://code.google.com/p/shuffle-db-ng/)
and improved my nims11 and NicoHood.

## automatically or manually mount ipod  
mount /dev/sdb /mnt

## copy mp3 or m4a file into ipod

/mnt/iPod_Control/Music/F00/

## command of update play list

sudo ./ipod-shuffle-4g.py -d 1 /mnt/      


## copy music to iPhone in Linux

```
sudo apt update
sudo apt install ifuse libimobiledevice-utils
```

pair iphone, unlock and trust pair in the iphone
```
idevicepair pair
```

list supported app installed in the iPhone
```
ifuse --list-apps
```

mount vlc's app directory to Linux
```
ifuse --documents org.videolan.vlc-ios ~/iphone_vlc
```

copy Music file to ~/iphone_vlc

finally fusermount -u ~/iphone_vlc
