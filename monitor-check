#!/bin/bash

#script to detect if monitor is connected and
#set the correct xorg.conf file.
#huge shoutout to KF7VUT for his work on getting
#this working
#20JULY2023 KM4ACK

MYPATH="$(cd "$(dirname "${BASH_SOURCE[0]}")" >/dev/null 2>&1 && pwd)"
VERSION=1

INSTALL(){
sudo apt update
#set root as owner & move this script to correct dir
chmod +x $MYPATH/monitor-check
sudo chown root:root $MYPATH/monitor-check
sudo mv $MYPATH/monitor-check /usr/local/bin/

#install x11vnc if not found
if ! hash x11vnc 2>/dev/null; then
    echo "########################################"
    echo "# x11vnc not installed. installing now #"
    echo "########################################"
    sudo apt install x11vnc
fi

#install video dummy driver if not found
ck=$(apt list --installed | grep dummy)
if [ -z "$ck" ]; then
    echo "#############################################"
    echo "# video dummy not installed. installing now #"
    echo "#############################################"
    sudo apt install xserver-xorg-video-dummy
fi

#modify /etc/systemd/system/display-manager.service
ck=$(grep monitor-check /etc/systemd/system/display-manager.service)
if [ -z "$ck" ]; then
    echo "######################################"
    echo "# modify display-manager.service now #"
    echo "######################################"
    sudo sed -i '/^\[Service\]/a #check for monitor\nExecStartPre=/bin/sh /usr/local/bin/monitor-check' /etc/systemd/system/display-manager.service
fi
echo;echo "Next you will be asked to supply a VNC password"
echo "It is recommended that you use your sudo password"
/usr/bin/x11vnc --storepasswd
echo "Creating service file"
cat <<EOF > /run/user/$UID/x11vnc.service
[Unit]
Description=Start X11VNC at startup.
After=multi-user.target
 
[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/$USER/.vnc/passwd -rfbport 5900 -shared
 
[Install]
WantedBy=multi-user.target
EOF
sudo mv /run/user/$UID/x11vnc.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc.service

echo "Installation complete. Reboot recommended"
echo "73, de KM4ACK"
exit 0
}

UNINSTALL(){
clear;echo;echo
echo "Uninstall should not be run over a VNC connection"
echo "since the VNC server will be uninstalled and you"
echo "will lose your connection. Press ctrl+c to stop"
echo;echo "Will proceed in 15 seconds"
sleep 15
echo "###############"
echo "removing x11vnc"
echo "###############"
sudo apt purge x11vnc
echo "###########################"
echo "removing video dummy driver"
echo "###########################"
sudo apt purge xserver-xorg-video-dummy
echo "#############################"
echo "removing monitor check script"
echo "#############################"
sudo rm /usr/local/bin/monitor-check
sudo sed -i '/#check for monitor/d' /etc/systemd/system/display-manager.service
sudo sed -i '/ExecStartPre=\/bin\/sh \/usr*/d' /etc/systemd/system/display-manager.service
sudo rm /etc/systemd/system/x11vnc.service
echo "uninstall complete. reboot recommended"
echo "73, de KM4ACK"
exit 0
}

if [ "$1" = 'install' ]; then
    INSTALL
elif [ "$1" = 'uninstall' ]; then
    UNINSTALL
fi

CREATE(){
#create the xorg.conf file 
cat <<EOF > /tmp/xorg.conf
Section "Monitor"
  Identifier "Monitor0"
  HorizSync 28.0-80.0
  VertRefresh 48.0-75.0
  # https://arachnoid.com/modelines/
  # 1920x1080 @ 60.00 Hz (GTF) hsync: 67.08 kHz; pclk: 172.80 MHz
  Modeline "1920x1080_60.00" 172.80 1920 2040 2248 2576 1080 1081 1084 1118 -HSync +Vsync
EndSection
Section "Device"
  Identifier "Card0"
  Driver "dummy"
  VideoRam 256000
EndSection
Section "Screen"
  DefaultDepth 24
  Identifier "Screen0"
  Device "Card0"
  Monitor "Monitor0"
  SubSection "Display"
    Depth 24
    Modes "1920x1080_60.00"
  EndSubSection
EndSection

EOF
}

x=0
for file in /sys/class/drm/*; do
    if [ -f $file/status ]; then
        CK=$(grep -w connected $file/status)
    else
        CK=""
    fi

    if [ -n "$CK" ]; then
        echo "monitor connected"
        x=1
        if [ -f /etc/X11/xorg.conf ]; then
            mv /etc/X11/xorg.conf /etc/X11/xorg.conf.bkup
        fi
        break
    fi
done

    #copy xorg.conf file
    if [ $x = 0 ]; then
        CREATE
        cp /tmp/xorg.conf /etc/X11/
    fi
