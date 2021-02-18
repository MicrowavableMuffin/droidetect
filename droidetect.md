# droidetect  
### Installation:

`sudo ln -s /path/to/droidetect/droidetect /usr/bin/droidetect`  
This script will run without doing this, but you will need to type the path to droidetect 
in order to start.

### Usage:  

`droidetect &`

### Description:  

Spawns droidcam on device-plug, kills on unplug

## __Code__  
```
#!/bin/bash

icon='/usr/share/icons/Humanity/devices/24/phone-motorola-droid.svg'
DIR=`dirname $(readlink $0)`

[ -f /tmp/deltausb ] && rm -f /tmp/deltausb

function start_droidcam() {
  adb devices > /tmp/adb_devices 2>&1 &
  droidcam > /dev/null 2>&1 &
}

function update_baseline() {
  cp -f /tmp/compareusb $DIR/baselineusb
  rm /tmp/deltausb
}

function kill_droidcam() {
  while [[ $(ps|grep droidcam) ]]
  do
    killall droidcam
  done
  cp -f /tmp/compareusb $DIR/baselineusb
  rm /tmp/deltausb
}

while true
do
  lsusb > /tmp/compareusb
  diff /tmp/compareusb $DIR/baselineusb >> /tmp/deltausb
  if [[ $(grep '<' /tmp/deltausb && grep 'Samsung' /tmp/deltausb) ]]
  then
    notify-send -i $icon -t 15000 'Samsung Connected' 'Starting Droidcam'
    start_droidcam
    update_baseline
  elif [[ $(grep '>' /tmp/deltausb && grep 'Samsung' /tmp/deltausb) ]]
  then
    notify-send -i $icon -t 15000 'Samsung Disconnected' 'Terminating Droidcam'
    kill_droidcam
  fi
  sleep 5
done
```