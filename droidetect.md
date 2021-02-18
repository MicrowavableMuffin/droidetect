# droidetect  
## Synopsis:  

Spawns droidcam on Samsung device plugin. Kills droidcam on unplug.

### Installation:  

Optional:  
`sudo ln -s /path/to/droidetect/droidetect /usr/bin/droidetect`  
_If you don't do this 'optional' step you will have to use `path/to/droidetect/droidetect &`
to launch_

### Usage:  

`droidcam &`

#### Example(s):  

> Notification: 📱'Samsung Connected' 'Starting Droidcam'  
_adb and droidcam silently start in the background_  

> Notification: 📱 'Samsung Disconnected' 'Terminating Droidcam'  
_ALL instances of droidcam are closed_  

### Description:  
 Use Droidstyle Icon in the Notification, set the local repo-path  
 Ensure baseline exists as-is  
 Delete the USB Change stub if it already exists  
 Function to ensure adb is started and launch droidcam (both silently)  
 Function to Update baseline file so USB plug-change is not repeated  
 Function to Close Droidcam  
 Watcher Loop, runs every 5 seconds  
 Copy connected USB devices to the compare-file  
 Create USB change stub by comparing compare-file and baseline  
 When connected, notify the user and start droidcam if its not running  
 Remember the connection, instead of notifying every 5 seconds  
 When disconnected, notify the user and kill active droidcam sessions  
 Remember that device has been disconnected instead of notifying every 5 seconds  
 Wait 5 seconds before repeating  
  
## __Code__  
```
#!/bin/bash

icon='/usr/share/icons/Humanity/devices/24/phone-motorola-droid.svg'
DIR=`dirname $(readlink $0)`

[ -f $DIR/baselineusb ] || lsusb > "$DIR/baselineusb"
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
}

while true
do
  lsusb > /tmp/compareusb
  diff /tmp/compareusb $DIR/baselineusb >> /tmp/deltausb
  if [[ $(grep '<' /tmp/deltausb && grep 'Samsung' /tmp/deltausb) ]]
  then
    notify-send -i $icon -t 15000 'Samsung Connected' 'Starting Droidcam'
    [[ $(ps|grep droidcam) ]]||start_droidcam
    update_baseline
  elif [[ $(grep '>' /tmp/deltausb && grep 'Samsung' /tmp/deltausb) ]]
  then
    notify-send -i $icon -t 15000 'Samsung Disconnected' 'Terminating Droidcam'
    kill_droidcam && update_baseline
  fi
  sleep 5
done
```
