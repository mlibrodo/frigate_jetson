pwd and ROBOFLOW_API_KEY are found in (google doc)[https://docs.google.com/document/d/1Ijb5Ih6niiNby-Oxc1I2EeX49zCnluoEAXFwS41Kqhk/edit?usp=sharing]

```
ssh michael@192.168.55.1  # pwd 
```

```
michael@michael-desktop:~/frigate$ lsusb
Bus 002 Device 002: ID 0bda:0489 Realtek Semiconductor Corp. 4-Port USB 3.0 Hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 003: ID 13d3:3549 IMC Networks Bluetooth Radio
Bus 001 Device 004: ID 046d:c52b Logitech, Inc. Unifying Receiver
Bus 001 Device 006: ID 046d:082d Logitech, Inc. HD Pro Webcam C920
Bus 001 Device 005: ID c0f4:0201 Hengchangtong  HCT USB Entry Keyboard
Bus 001 Device 002: ID 0bda:5489 Realtek Semiconductor Corp. 4-Port USB 2.0 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
michael@michael-desktop:~/frigate$ ls -l /dev/video*
crw-rw----+ 1 root video 81, 0 Jan 20 14:58 /dev/video0
crw-rw----+ 1 root video 81, 1 Jan 20 14:58 /dev/video1
michael@michael-desktop:~/frigate$ sudo apt install -y v4l-utils
[sudo] password for michael: 
Sorry, try again.
[sudo] password for michael: 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
v4l-utils is already the newest version (1.22.1-2build1).
0 upgraded, 0 newly installed, 0 to remove and 6 not upgraded.
michael@michael-desktop:~/frigate$ v4l2-ctl --list-devices
NVIDIA Tegra Video Input Device (platform:tegra-camrtc-ca):
    /dev/media0

HD Pro Webcam C920 (usb-3610000.usb-2.3):
    /dev/video0
    /dev/video1
    /dev/media1

… Setup frigate to access the C920 webcam …
michael@michael-desktop:~/frigate$ curl -s "http://127.0.0.1:5000/api/c920/latest.jpg" -o /tmp/c920_latest.jpg
michael@michael-desktop:~/frigate$ ls -lh /tmp/c920_latest.jpg

…
michael@michael-desktop:~/frigate$ curl -s -X POST "http://127.0.0.1:9001/ember-training-poc/1" \
  -H "Authorization: Bearer $ROBOFLOW_API_KEY" \
  -F "file=@/tmp/c920_latest.jpg" \
  | head -c 400; echo
{"inference_id":"365e502a-5b4c-45eb-8089-31dedca77be5","time":0.04878015500071342,"image":{"width":1280,"height":720},"predictions":[{"class":"no_ember","class_id":1,"confidence":0.9159}],"top":"no_ember","confidence":0.9159}


…
export ROBOFLOW_API_KEY="XXX"
export MODEL_ID="ember-training-poc/1"
export FRIGATE_CAMERA="c920"

# optional tuning
export PUMP_FPS="2"
export EMIT_ON_CHANGE="1"
export EMBER_LABEL="ember"
export EMBER_THRESHOLD="0.85"
export DEBOUNCE_HITS="3"
export DEBOUNCE_WINDOW="5"

python3 pump.py

```

starting just the ROBOFLOW inference server

```
# 0) Go to your home directory
cd /home/michael
docker rm -f inference-server 2>/dev/null || true
# (optional but recommended) Export your Roboflow API key
export ROBOFLOW_API_KEY="<REDACTED>"


# 1) Stop and remove any existing inference container
docker rm -f inference-server 2>/dev/null || true


# 2) Start Roboflow Inference Server (Jetson, offline-ready)
docker run -d \
  --name inference-server \
  --runtime nvidia \
  -p 9001:9001 \
  -e MODEL_CACHE_DIR=/tmp/cache \
  -e MPLCONFIGDIR=/tmp \
  --tmpfs /tmp:rw,noexec,nosuid,size=1g \
  -v inference-cache:/tmp/cache \
  roboflow/roboflow-inference-server-jetson-6.2.0:latest


# (optional sanity check)
docker ps --filter name=inference-server


# 3) Run your inference script (primes cache if online, works offline after)
python3 main.py

inference infer -i ~/Downloads/ember-test.jpg \
>  -m ember-training-poc/1 \
    --api-key B5p60bLPJYURpEpoGHcc


```


Setting up jetson as a wifi hotspot so that we can connect the camera

```
michael@michael-desktop:~$ nmcli dev wifi hotspot \
  ifname wlP1p1s0 \
  ssid WildfireNet \
  password StrongPass123
Error: Failed to setup a Wi-Fi hotspot: Not authorized to control networking.
michael@michael-desktop:~$ sudo nmcli dev wifi hotspot   ifname wlP1p1s0   ssid WildfireNet   password StrongPass123
[sudo] password for michael:
Device 'wlP1p1s0' successfully activated with '6200d275-4b0d-4b79-8192-6d2f6d4d4e75'.
Hint: "nmcli dev wifi show-password" shows the Wi-Fi name and password.
```


## HEADLESS
# connect jetson -> usbc -> laptop
```OSX
01:34:24:ltm-1920:~:$ssh -4 -o IPQoS=none michael@192.168.55.1
michael@192.168.55.1's password:
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.148-tegra aarch64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

Expanded Security Maintenance for Applications is not enabled.

211 updates can be applied immediately.
181 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

39 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm

Last login: Thu Apr 23 08:53:10 2026 from 192.168.55.100

```


06  sudo nmcli dev wifi connect "wc-public"
  607  iw dev
  608  iw list | grep -A15 "valid interface combinations"
  609  iw list
  610  nmcli dev wifi list
  611  ip addr show wlP1p1s0
  612  nmcli connection down "wc-public"
  613  sudo nmcli connection down "wc-public"
  614  touch wifisetup
  615  sudo nmcli dev wifi hotspot   ifname wlP1p1s0   ssid WildfireNet   password StrongPass123
  616  ip addr show wlP1p1s0
  617  ip neigh
  618  sudo iw dev wlP1p1s0 station dump
  619  ip neigh
  620  nc -vz 10.42.0.142 554
  621  ip neigh
  622  nc -vz 10.42.0.142 554
  623  ip neigh
  624  ping 10.42.0.142
  625  nc -vz 10.42.0.142 554
  626  ping 10.42.0.142
  627  history
