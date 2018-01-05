Hardware
========

















## LEGACY

Linux
=====

change window title of terminals
--------------------------------
in ~/.zshrc:
  termtitle () {
    echo -en "\033]0;"$@"\a"
  }

simple webserver with python
----------------------------
start a webserver with directory listing in the current directory:
  python2 -m SimpleHTTPServer 8000

ubuntu libgl errors
-------------------
To avoid erros with bumblebee hybrid graphic on 32bit applications (e.g. steam)
do the following:

sudo vim /etc/ld.so.conf.d/optimus.conf

Add next two lines to file:
  /usr/lib32
  /usr/lib/i386-linux-gnu/mesa

Then execute:
  sudo ldconfig


transcode movie to HQ h264 via ffmpeg
-------------------------------------
map 0 makes sure that all audio tracks will be present in output file aswell.
without "veryslow" profile will be great quality, too.

  ffmpeg -i INPUT.mkv -map 0 -vcodec libx264 -preset veryslow -acodec	libmp3lame OUTPUT.mkv

get and set date via timestamp
------------------------------
get:
  date +%s

set:
  date +%s -s "@1384261338"

boblight-X11
------------
depends: subversion

### compile
  svn checkout http://boblight.googlecode.com/svn/trunk/ boblight
  ./configure --prefix=/usr --without-portaudio

### start
  boblight-X11 -o speed=42 -o value=5 

### config
/etc/boblight.conf

TODO << insert file

suspend
-------
### suspend command
echo -n mem > /sys/power/state

### patch kernel
in ./kernel/power/power.h:70 change "mode = 0644" to:
  #define power_attr(_name) \
  static struct kobj_attribute _name##_attr = {   \
          .attr   = {                             \
                  .name = __stringify(_name),     \
                  .mode = 0777,                   \
          },                                      \
          .show   = _name##_show,                 \
          .store  = _name##_store,                \
  }


disable guest login on ubuntu
-----------------------------
tested on ubuntu 12.04 and ubuntu 14.04.
edit/create /etc/lightdm/lightdm.conf:
  [SeatDefaults]
  user-session=ubuntu
  greeter-session=unity-greeter
  allow-guest=false


create a bootable USB stick with Windows7 under Linux
-----------------------------------------------------
depends: ntfs3g

### create the partition
  parted /dev/sdX
  mklabel msdos
  mkpart primary ntfs 1 -1
  set 1 boot on
  quit

  mkfs.ntfs -f /dev/sdX1

### compile and install the bootloader
  # download http://ms-sys.sourceforge.net/
  tar xvzf ms-sys-2.*.tar.gz
  cd ms-sys-2.*
  make
  cd bin
  ./ms-sys -7 /dev/sdX

### copy the files
  mkdir -p /mnt/usb
  mkdir -p /mnt/iso

  mount -o loop /path/to/iso /mnt/iso
  mount /dev/sdX1 /mnt/usb

  cp -av /mnt/iso* /mnt/usb

### finish
  sync
  umount /mnt/usb
  umount /mnt/iso

official nvidia optimus
-----------------------
this describes how to setup the original nvidia mechanisms for nvidia optimus chips
on gentoo.

add in /etc/make.conf:
  VIDEO_CARDS="intel nvidia modesetting"

depends: 
  nvidia-drivers >= 304
  xrandr >= 1.4
  xorg-server >= 1.13
theese are all available in gentoo stable


in /etc/xorg.conf:
  Section "ServerLayout"
      Identifier "layout"
      Screen 0 "nvidia"
      Inactive "intel"
  EndSection
  
  Section "Device"
      Identifier "nvidia"
      Driver "nvidia"
      BusID "PCI:1:0:0"
  EndSection
  
  Section "Screen"
      Identifier "nvidia"
      Device "nvidia"
      Option "UseDisplayDevice" "none"
  EndSection
  
  Section "Device"
      Identifier "intel"
      Driver "modesetting"
  EndSection
  
  Section "Screen"
      Identifier "intel"
      Device "intel"
  EndSection

in ~/.xinitrc:
  xrandr --setprovideroutputsource modesetting NVIDIA-0
  xrandr --auto

support for CEC adaptors under gentoo
-------------------------------------
to enable cec support for devices such as the
pulseeigt-cec adaptor you'll need the following:

in /etc/portage/make.conf:
  USE="cec"

add the user to uucp:
  gpasswd -a <username> uucp

enable in kernel CDC-ACM devices:
  kernel: USB Modem (CDC ACM) support (CONFIG_USB_ACM)

h264 hardware encoding with gstreamer
-------------------------------------
The following command is used to convert any movie to a matroska(mkv) container,
with a h264 video (scaled down to 720p) and faac audio (with a bitrate of 40000).
The whole video encoding is hardware accelerated and requires a working gstreamer-vaapi backend.
The hardware must be able to do h264 encoding (newer intels, nvidia,..)

  gst-launch-1.0 filesrc location="~/input.mp4" ! \
    decodebin name=demux ! \
    videoscale ! "video/x-raw, width=1280, height=720" ! \
    vaapiencode_h264 rate-control=1 tune=high-compression cabac=true ! \
    matroskamux name=mux ! \
    filesink location="~/output.mkv" demux. ! \
    audioconvert ! faac bitrate=40000 ! aacparse ! mux.

Scroll in GNU/Screen in gnome-terminals
---------------------------------------
into ~/.screenrc:
  # enable scrolling
  termcapinfo xterm ti@:te@
  # bind page up/down
  bindkey -m "^[[5;2~" stuff ^b
  bindkey -m "^[[6;2~" stuff ^f 

KDE enable samba for dolphin
----------------------------
In most cases, dolphin tries to download files from samba shares, instead of
streaming them / giving the path to the application (vlc, mplayer). 
To tell KDE to open the files directly, put the following in the applicaions dir:

  cp /usr/share/applications/mplayer.desktop /home/$USER/.local/share/applications

add the following to /home/$USER/.local/share/applications/mplayer.desktop:
  X-KDE-Protocols=http,ftp,smb

works with mplayer and vlc and maybe some others too.

Add the same timestamp to all files
-----------------------------------
timestamp could be 201403270000
  find . | xargs touch -h -t $timestamp

Resize DD-Image to max-size
---------------------------
  bzcat image-file.img > /dev/mmcblk0
  sync
eject sdcard, insert sdcard
to make sure every partition gets reloaded
  fdisk /dev/mmcblk0
  p
  d
  2
  n
  p
  2
  <return>
  <return>
  w
  q
  sync
eject sdcard, insert sdcard
to make sure every partition gets reloaded
  resize2fs /dev/sdb2
  sync

List all modules needed by lspci
--------------------------------
this prints all modules which are known drivers 
for lspci detected hardware:
  LC_ALL=c lspci -mvk | grep ^Driver | awk '{print $2}' | uniq

Basic pipe-knowledge
--------------------
as the "<" looks often confusing for some people:
  vim - < filename
the same as:
  cat filename | vim -

sed basics
----------
### replace
  sed 's/alt/neu/g' filename > changed_file
  sed -i 's/alt/neu/g' filename

### delete
  sed -i '/this will be deleted/d' filename

disable kernel sysrq calls in userland
--------------------------------------
  sysctl -w kernel.sysrq = 0

GTK Scrollbars
--------------
get ubuntu-like scrollbars (so called overlay-scrollbars):
source: https://launchpad.net/ayatana-scrollbar
gentoo: http://gpo.zugaina.org/x11-misc/overlay-scrollbar/

	layman -Sa stuff
	emerge -av overlay-scrollbar

gnome3 on gentoo
----------------
[[ deprecated, gnome3 is in stable ]]
packages: layman [git]
gnome3 is still "keyworded" in gentoo [06/2013],
the easiest way to install, is to use the gnome-overlay:

    layman -Sa gnome
    ln -s /var/lib/layman/gnome/status/portage-configs/package.use.gnome3 /etc/portage/package.use/gnome3.use
    ln -s /var/lib/layman/gnome/status/portage-configs/package.use.mask.gnome3 /etc/portage/package.use/mask-gnome3.use
    ln -s /var/lib/layman/gnome/status/portage-configs/package.keywords.gnome3 /etc/portage/package.keywords/gnome3.keywords
    ln -s  /var/lib/layman/gnome/status/portage-configs/package.unmask.gnome3 /etc/portage/package.unmask/gnome3.unmask
    eselect profile set 4
    emerge -avuND world gnome

gentoo with systemd
-------------------
add "systemd" to USE-Flags or
select an profile like gnome/systemd:
  eselect profile set 5

rebuild everything:
  emerge -avuND world

use systemd as default init:
  for REPLACE in runlevel reboot shutdown poweroff halt telinit; do
   ln -sfv /usr/bin/systemctl /sbin/${REPLACE}
  done

  ln -sfv /usr/lib/systemd/systemd /sbin/init

configure systemd basics:
  hostnamectl set-hostname surtur
  localectl set-keymap de
  localectl set-x11-keymap de
  localectl set-locale LANG=de_DE.utf8

disable auto suspend on laptop lid close
edit the file /etc/systemd/logind.conf:
  HandleLidSwitch=ignore

use old names for network devices:
  ln -s /dev/null /etc/udev/rules.d/80-net-setup-link.rules

gnome3 gsettings
----------------
show settings:
  gsettings list-recursively org.gnome.desktop.media-handling 

deactivate automount:
  gsettings set org.gnome.desktop.media-handling automount 'false'
  gsettings set org.gnome.desktop.media-handling automount-open 'false' 

resize with rigth click:
  gsettings set org.gnome.desktop.wm.preferences resize-with-right-button true

desktop font:
  gsettings set org.nemo.desktop font 'Sans 8'

keyboard shortcuts:
  gsettings set org.gnome.desktop.wm.keybindings minimize "['<Alt>s']"
  gsettings set org.gnome.desktop.wm.keybindings close "['<Alt>c']"
  gsettings set org.gnome.desktop.wm.keybindings toggle-maximized "['<Alt>v']"
  gsettings set org.gnome.desktop.wm.keybindings toggle-fullscreen "['<Alt>f']"
  gsettings set org.gnome.desktop.wm.keybindings move-to-workspace-left "['<Shift><Alt>Left']"
  gsettings set org.gnome.desktop.wm.keybindings move-to-workspace-right "['<Shift><Alt>Right']"
  gsettings set org.gnome.settings-daemon.plugins.media-keys screensaver "['<Super>l']"
  gsettings set org.gnome.settings-daemon.plugins.media-keys terminal "['<Super>1']"
  gsettings set org.gnome.settings-daemon.plugins.media-keys www "['<Super>2']"
  gsettings set org.gnome.settings-daemon.plugins.media-keys home "['<Super>3']"

Gnome Terminal Internal Padding
-------------------------------
add the following to ~/.config/gtk-3.0/gtk.css
   TerminalScreen {
     -VteTerminal-inner-border: 10px 10px 10px 10px;
   }

Cinnamon Gsettings
------------------
update: since version 2.0+ cinnamon is using its own config strings, 
the gnome settings from above will not work anymore
update: cinnamon 2.6.x changed the gsetting paths (keybindings updated)

show settings:
  gsettings list-recursively org.cinnamon.desktop.media-handling

deactivate automount:
  gsettings set org.cinnamon.desktop.media-handling automount 'false'
  gsettings set org.cinnamon.desktop.media-handling automount-open 'false' 

desktop font:
  gsettings set org.nemo.desktop font 'Sans 8'

keyboard shortcuts:
  gsettings set org.cinnamon.desktop.keybindings.media-keys screensaver "['<Super>l']"
  gsettings set org.cinnamon.desktop.keybindings.media-keys terminal "['<Super>1']"
  gsettings set org.cinnamon.desktop.keybindings.media-keys www "['<Super>2']"
  gsettings set org.cinnamon.desktop.keybindings.media-keys home "['<Super>3']"
  gsettings set org.cinnamon.desktop.keybindings.wm panel-run-dialog "['<Super>0']"
  gsettings set org.cinnamon.desktop.keybindings.wm minimize "['<Alt>s']"
  gsettings set org.cinnamon.desktop.keybindings.wm close "['<Alt>c']"
  gsettings set org.cinnamon.desktop.keybindings.wm toggle-maximized "['<Alt>v']"
  gsettings set org.cinnamon.desktop.keybindings.wm toggle-fullscreen "['<Alt>f']"
  gsettings set org.cinnamon.desktop.keybindings.wm move-to-workspace-left "['<Shift><Alt>Left']"
  gsettings set org.cinnamon.desktop.keybindings.wm move-to-workspace-right "['<Shift><Alt>Right']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-1 "['<Alt>F1']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-2 "['<Alt>F2']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-3 "['<Alt>F3']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-4 "['<Alt>F4']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-5 "['<Alt>F5']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-down "['<Control><Alt>Down']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-up "['<Control><Alt>Up']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-left "['<Control><Alt>Left']"
  gsettings set org.cinnamon.desktop.keybindings.wm switch-to-workspace-right "['<Control><Alt>Right']"
  gsettings set org.cinnamon.desktop.keybindings.wm move-to-workspace-1 "['<Alt><Shift>F1']"
  gsettings set org.cinnamon.desktop.keybindings.wm move-to-workspace-2 "['<Alt><Shift>F2']"
  gsettings set org.cinnamon.desktop.keybindings.wm move-to-workspace-3 "['<Alt><Shift>F3']"
  gsettings set org.cinnamon.desktop.keybindings.wm move-to-workspace-4 "['<Alt><Shift>F4']"
  gsettings set org.cinnamon.desktop.keybindings.wm move-to-workspace-5 "['<Alt><Shift>F5']"

Gnome Terminal Force Quit
-------------------------
This removes the "really-quit" dialog on gnome-terminal:
  gconftool-2 --set --type bool /apps/gnome-terminal/global/confirm_window_close false

Cinnamon Change Desktop Colors
------------------------------
This is to change the desktop text color in Cinnamon, 
simply edit in your gtk CSS file the nemo desktop settings.

  ~/.config/gtk-3.0/gtk.css:
  .nemo-desktop.nemo-canvas-item {
    color: #CCCCCC;
    text-shadow: 1px 1px @desktop_item_text_shadow;
  }

Cinnamon Systray Icon Size
--------------------------
the tray icon size is defined in:
/usr/share/cinnamon/applets/systray@cinnamon.org/applet.js

Install RabbitVCS on Linux Mint
-------------------------------
A simple copy-paste to install RabbitVCS on Linux Mint, in the Cinnamon Desktop Environment:

  add-apt-repository ppa:rabbitvcs/ppa
  apt-get install nemo-rabbitvcs
  nemo -q

Install a Webserver on Linux Mint
---------------------------------
To install a Lighttpd Webserver with PHP, MySQL(MariaDB) and PHPMyAdmin,
use one of the following instructions as root.

### Install all at once
  apt-get install lighttpd php5-cgi php5-mysql php5-mcrypt mariadb-server phpmyadmin
  lighty-enable-mod fastcgi 
  lighty-enable-mod fastcgi-php
  php5enmod mcrypt 
  service lighttpd force-reload
  ln -s /usr/share/phpmyadmin/ /var/www

### Install step by step
lighttpd webserver:
  apt-get install lighttpd

lighttpd php & mysql bindings:
  apt-get install php5-cgi php5-mcrypt php5-mysql

enable php:
  lighty-enable-mod fastcgi 
  lighty-enable-mod fastcgi-php
  php5enmod mcrypt 

start/restart lighttpd:
  service lighttpd force-reload

mariadb/mysql server + frontend:
  apt-get install mariadb-server
  apt-get install phpmyadmin

move phpmyadmin to htdocs:
  ln -s /usr/share/phpmyadmin/ /var/www

toggle window manager decorations
---------------------------------
A python script to toggle window decorations in 
windowmanagers like marco/metacity:

  #!/usr/bin/python
  from gtk.gdk import *
   
  w=window_foreign_new((get_default_root_window().property_get("_NET_ACTIVE_WINDOW")[2][0]))
  state = w.property_get("_NET_WM_STATE")[2]
  maximized='_NET_WM_STATE_MAXIMIZED_HORZ' in state and '_NET_WM_STATE_MAXIMIZED_VERT' in state
   
  if w.get_decorations() != 0 :
      w.set_decorations(0)
  else:
      w.set_decorations(DECOR_ALL)
  window_process_all_updates()

Xdefaults
---------
colored manpages:
  *colorIT:      #BEC040
  *colorBD:      #728CA6
  *colorUL:      #73C040

show xrdb colors
----------------
Display colors of xrdb compliant terminals like urxvt and xterm

  #!/bin/bash
  xrdb -l ~/.Xdefaults
  colors=($(xrdb -query | sed -n 's/.*color\([0-9]\)/\1/p' | sort -nu | cut -f2))
  for i in {0..7}; do echo -en "\e[0m \e[$((30+$i))m \xE2\x96\x88\xE2\x96\x88 ${colors[i]} \e[0m"; done
  echo
  for i in {8..15}; do echo -en "\e[0m \e[1;$((22+$i))m \xE2\x96\x88\xE2\x96\x88 ${colors[i]} \e[0m"; done
  echo

Screen
------
  shelltitle "$ |terminal"
  msgwait 2
  vbell off

  altscreen on
  term screen-256color
  
  bindkey "^[Od" prev  # change window with ctrl-left
  bindkey "^[Oc" next  # change window with ctrl-right

  hardstatus alwayslastline "%{= w}%-w%{= bw} %n %t %{-}%+w %-=%{b}%c:%s"

zsh youtube
-----------
1. installation
package: youtube-dl (git), mplayer

2. zshrc

  youtube() {
    mplayer -cache 2048 `youtube-dl "$1" -g` 2>&1
  }

mplayer-webcam
--------------
package: mplayer
use flag: v4l2

record:
  mencoder -tv driver=v4l2:fps=25:height=600:width=800 -ovc raw -vf scale=800:600 -o $1 tv:// -nosound

view:
  mplayer tv:// -tv driver=v4l2:width=800:height=600:device=/dev/video0
 
minidlna
--------
package: minidlna

/etc/conf.d/minidlna:
  # /etc/conf.d/minidlna 

  # Should minidlna rescan the entire collection on startup?
  # Warning: This may take a long time!
  RESCAN="false"

  # The location of the config file
  #CONFIG="/etc/minidlna.conf"

  # Specify the user/group minidlna should run as
  M_USER="eric"
  M_GROUP="users"


/etc/minidlna.conf:
  port=8200
  #network_interface=eth0
  #   + "A" for audio  (eg. media_dir=A,/home/jmaggard/Music)
  #   + "V" for video  (eg. media_dir=V,/home/jmaggard/Videos)
  #   + "P" for images (eg. media_dir=P,/home/jmaggard/Pictures)
  media_dir=/mnt/storage/public/
  friendly_name=calypso
  db_dir=/home/eric/minidlna/
  album_art_names=Cover.jpg/cover.jpg/AlbumArtSmall.jpg/albumartsmall.jpg/AlbumArt.jpg/albumart.jpg
  inotify=yes
  enable_tivo=no
  strict_dlna=no
  notify_interval=900
  serial=12345678
  model_number=1

manual linux router-setup
-------------------------
### hostap-daemon
package: hostapd

/etc/hostapd/hostapd.conf:
  interface=wlan0
  driver=nl80211

  ssid=YOUR_ESSID_GOES_HERE
  channel=YOUR_CHANNEL
  wpa_passphrase=YOURPASSWORD

  ignore_broadcast_ssid=0
  country_code=DE
  ieee80211d=1
  hw_mode=g
  ieee80211n=1
  beacon_int=100
  dtim_period=2
  macaddr_acl=0
  max_num_sta=255
  rts_threshold=2347
  fragm_threshold=2346
  logger_syslog=-1
  logger_syslog_level=2
  logger_stdout=-1
  logger_stdout_level=2
  dump_file=/tmp/hostapd.dump
  ctrl_interface=/var/run/hostapd
  ctrl_interface_group=0
  auth_algs=3
  wmm_enabled=1
  wpa=2
  rsn_preauth=1
  rsn_preauth_interfaces=wlan0
  wpa_key_mgmt=WPA-PSK
  rsn_pairwise=CCMP
  wpa_group_rekey=600
  wpa_ptk_rekey=600
  wpa_gmk_rekey=86400

### DHCP-Server
package: dnsmasq

/etc/dnsmasq.conf:
  # DHCP-Server aktiv für Interface
  interface=wlan0

  # DHCP-Server nicht aktiv für Interface
  no-dhcp-interface=eth0

  # IP-Adressbereich / Lease-Time
  dhcp-range=interface:wlan0,10.4.4.2,10.4.4.200,infinite

  # static ips
  #dhcp-host=<MAC-Adresse>,<Name>,<IP-Adresse>,infinite
  #dhcp-host=f1:f1:f1:f1:f1:f1,,10.4.4.2,infinite

### Routing
package: iptables
NAT eth0 through wlan0:
  iptables -F
  iptables -t nat -F
  iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

  sysctl -w net.ipv4.ip_forward=1
  sysctl -p

debian
------
### debootstrap (cross)
creating rootfs-tarball for foreign architecture:

  debootstrap --foreign --no-check-gpg --arch=armhf testing /debian-armhf http://ftp.debian.org/debian/ 
... boot ...
  /debootstrap/debootstrap --second-stage

### debootstrap (native)
creating rootfs-tarball for native architecture:
  debootstrap --no-check-gpg testing /debian http://ftp.debian.org/debian/ 

### apt-get source
build packages out of deb-src repositories:
  apt-get source <package>
  cd <package>/
  vim <stuff>
  apt-get -b source <package>
  dpkg -i <package>

debootstrap for ubuntu
---------------------
  debootstrap trusty ./trusty http://archive.ubuntu.com/ubuntu/

color-changing urxvt
--------------------
instant color switching in rxvt-unicode:
change cursor color (blue):
  printf '\33]12;12\007'
  
change background color (dark-grey):
  printf '\33]11;%s\007' "#333333"

### auto-change
.zshrc:

  appearance() {
	  if (( EUID != 0 )); then
		  printf '\33]12;12\007' # blue user cursor
	  else
		  printf '\33]12;9\007' # red root cursor
	  fi
  }
  appearance
   
   
  su() {
	  [ -n "$1" ]
		  printf '\33]12;8\007' # changes cursor color
		  /bin/su $@
		  appearance
  }
   
  ssh() {
	  [ -n "$1" ]
		  printf '\33]12;10\007' # changes cursor colo
		  /usr/bin/ssh $@
		  appearance
  }
   
  chroot() {
	  [ -n "$1" ]
		  printf '\33]12;13\007' # changes cursor color	
		  /bin/chroot $@
		  appearance
  }


awesome irssi notify
--------------------

### irssi plugin
~/.irssi/scripts/autorun/awnotify.pl:
  use strict;
  use Irssi;
   
  sub hilight {
	  my ($dest, $text, $stripped) = @_;
	  if ($dest->{level} & MSGLEVEL_HILIGHT) {
		  filewrite($dest->{target}. " " .$stripped );
	  }
  }
   
  sub filewrite {
	  my ($text) = @_;
	  $text =~ s/\n/ /;
	  $text =~ s/[<@&]//g;
   
	  my @values = split(' ', $text, 4);
   
	  `echo '$values[1] $values[3]' >> /var/tmp/irssi-notify`;
	  `notify-send '<span color="#aaaaaa">$values[1]</span> $values[3]'`;
  }
   
  sub del_notify {
	  `echo > /var/tmp/irssi-notify`;
  }
   
  Irssi::signal_add_last("print text", "hilight");
  Irssi::signal_add('gui key pressed', 'del_notify');



### notify-daemon
~/.config/autorun/awnotify.sh:
  #/bin/sh 
  while true; do
  echo -n `echo "irssiwidget.text = ' [<span color=\"#aaaaaa\">im </span><span color=\"#ffffff\">"``cat /var/tmp/irssi-notify | grep -c ">"`"</span>] '"  | awesome-client
  sleep 1
  done &


### awesome-config
~/.config/awesome/rc.lua:

  irssiwidget = widget({ type = "textbox" })
   
  mywibox[s].widgets = {
	 
	  [...]
	  irssiwidget,
	  [...]
  }

archlinux aur bash/zsh function
-------------------------------
dependency: wget
add the following to ~/.bashrc or ~/.zshrc:
  function aur() {
    if [ "$1" = "-Ss" ]; then
      wget "https://aur.archlinux.org/packages/?O=0&K=$2" -O /tmp/parse.html &> /dev/null
      grep "\/packages\/" /tmp/parse.html | grep -v "?K=" | cut -d \> -f 3 | cut -d \< -f 1 | grep "$2" --color=always
      rm /tmp/parse.html
    elif [ "$1" = "-S" ]; then
      mkdir ~/aur &> /dev/null || true
      cd ~/aur
      wget https://aur.archlinux.org/packages/${2:0:2}/$2/$2.tar.gz -N &> $2.tar.gz.log
      tar -xzf $2.tar.gz
      cd $2
      makepkg -si
    fi
  }
  
the alx network driver
----------------------
On some distributions the ALX network driver of my Dell XPS One 27 PC wont work.
This will install the correct driver:

  wget https://www.kernel.org/pub/linux/kernel/projects/backports/2013/03/04/compat-drivers-2013-03-04-u.tar.bz2 
  ./scripts/driver-select alx
  make
  su -c make install


top 10 commands
---------------
  history | awk '{CMD[$2]++;count++;}END { for (a in CMD)print CMD[a] " " CMD[a]/count*100 "% " a;}' | grep -v "./" | column -c3 -s " " -t | sort -nr | nl | head -n10
  
KDE Kmix Tray Scroll Step
-------------------------
in [Globals] in ~/.kde4/share/config/kmixrc
  VolumePercentageStep=2
  
MP3 remove silence
------------------
depends on package: sox
Remove silence on the beginning and the end of MP3 files:
  #!/bin/bash
  echo `date` > /tmp/trimp3.log
  for i in *.mp3; do
	  echo -n "removing silence from: $i"
	  sox "$i" "$i".tmp.mp3 silence 1 0.1 0.1% reverse silence 1 0.1 0.1% reverse &>> /tmp/trimp3.log
	  mv "$i".tmp.mp3 "$i"
	  echo "  ... done"
  done
  
Xorg Display Resolution Scaling
-------------------------------
this will fake a FullHD resolution for a display with only 1600x900:
  xrandr --output LVDS1 --mode 1600x900 --scale 1.2x1.2

git cheatsheet
--------------
### add alias for unstaging
  git config --global alias.unstage "reset HEAD"

### default git prune
  git config --global fetch.prune true

for default on pull:
  git config --global pull.rebase true

### show database statistics
  git count-objects -v

### compress database
  git repack

### verify files in git
  git fsck --full

### diff HEAD to previous HEAD
  git diff HEAD HEAD^
or:
  git diff HEAD HEAD~1

### .gitignore
  /absolute/path
  relative/path
  *.todo
  !exception for this.todo

### replace all local files by HEAD
  git reset --hard HEAD

### replace only one file by HEAD
  git checkout HEAD foo.txt

### change last commit
  git commit --amend

### show ALL git commits and revert to deleted commit
  git reflog
  git reset --hard e6b0203

### create branch
  git branch <NAME>

### switch branch
  git checkout <NAME>

### create and switch
  git checkout -b <NAME>

### merge
  git merge <branch>

### conflict
  git checkout --theirs <file>
  git checkout --ours <file>

### create git bare repository
  git clone --bare <myfiles> newrepo.git

### rebase
  git rebase <branch>


### interactive rebase
  git rebase -i <branch>

### apply single commit to branch
  git cherry-pick <REF>

### git show refs (e.g. bitbake recipes)
  git show-ref

### push tags
  git push --tags

### delete local tag / branch
  git tag -d <name>
  git branch -d <branch>

### delete remote tag / branch
  git push origin :refs/tags/v1.0
  git push origin :refs/heads/testing

### stash (+ untracked files)
  git stash -u
  git stash pop
  git stash list
  git stash pop stash@{1}

boost mdadm raid resync
-----------------------
### set read-ahead to 4096
  blockdev --setra 4096 /dev/md127
### set stripe cache to 8192
  echo 8192 > /sys/block/md127/md/stripe_cache_size

pacman restore corrupted btrfs files
------------------------------------
On my notebook i got alot of corrupted files after powerloss.
I was on a upgrade (pacman -Syu) when the system shut off without syncing.
The result where alot of files (all new pacman installed files) with size "0".
btrfs check hasn't found any corruption and pacman thought everything is installed correctely.

This is how i solved it:
  pacman -S pkgfile
  pkgfile --update
  pacman -S --force $(for i in $(find /usr -size 0 -type f; do pkgfile $i; done | sort | uniq)

usb issues on some arm devices
------------------------------
On some arm devices (pandaboard, odroid-x) i noticed some issues,
with usb initialisation of my sat receiver. 
Adding this to the cmdline solved the problem:
  vram=16M coherent_pool=6M

udev rule for static serial interfaces
--------------------------------------
the output of this script must be copied to: /etc/udev/rules.d/<new_rule>.rules
  echo "=== enter your data ==="; \
  echo -n "tty (e.g: /dev/ttyUSB0): "; \
  read tty; \
  echo -n "name (e.g: beagle): "; \
  read name; \
  echo "=== copy the following rule to /etc/udev/rules.d/serial.rules ==="; \
  idVendor=$(udevadm info -a -n $tty | grep '{idVendor}' | head -n1 | cut -d \" -f 2); \
  idProduct=$(udevadm info -a -n $tty | grep '{idProduct}' | head -n1 | cut -d \" -f 2); \
  idSerial=$(udevadm info -a -n $tty | grep '{serial}' | head -n1 | cut -d \" -f 2); \
  echo "SUBSYSTEM==\"tty\", ATTRS{idVendor}==\"$idVendor\", ATTRS{idProduct}==\"$idProduct\", ATTRS{serial}==\"$idSerial\", SYMLINK+=\"serial_$name\""

copy & paste for urxvt
----------------------
Support for Ctrl-Shift-C and Ctrl-Shift-V in (u)rxvt.
install: xsel
download the clipboard perl script:
  wget https://github.com/muennich/urxvt-perls/raw/master/clipboard -O /usr/lib/urxvt/perl/clipboard 

.Xdefaults:
	URxvt.perl-ext-common: default,matcher,clipboard
	URxvt.keysym.Shift-Control-C: perl:clipboard:copy
	URxvt.keysym.Shift-Control-V: perl:clipboard:paste

Add Useless-Gap support for awesomewm (3.5+)
--------------------------------------------
Apply this patch file to: /usr/share/awesome/lib/awful/layout/suit/tile.lua

  --- tile.lua	2013-01-29 17:01:13.461898665 -0600
  +++ uselessgap.lua	2013-01-30 12:59:41.794402805 -0600
  @@ -63,11 +63,46 @@
	  geom[height] = math.floor(unused * fact[i] / total_fact) - cls[c].border_width * 2
	  geom[x] = group.coord
	  geom[y] = coord
  -        geom = cls[c]:geometry(geom)
	  coord = coord + geom[height] + cls[c].border_width * 2
	  unused = unused - geom[height] - cls[c].border_width * 2
	  total_fact = total_fact - fact[i]
	  used_size = math.max(used_size, geom[width] + cls[c].border_width * 2)
  +
  +        -- Useless gap.
  +        if useless_gap > 0
  +        then
  +            -- Top and left clients are shrinked by two steps and
  +            -- get moved away from the border. Other clients just
  +            -- get shrinked in one direction.
  +
  +            top = false
  +            left = false
  +
  +            if geom[y] == wa[y] then
  +                top = true
  +            end
  +
  +            if geom[x] == 0 or geom[x] == wa[x] then
  +                left = true
  +            end
  +
  +            if top then
  +                geom[height] = geom[height] - 2 * useless_gap
  +                geom[y] = geom[y] + useless_gap
  +            else
  +                geom[height] = geom[height] - useless_gap
  +            end
  +
  +            if left then
  +                geom[width] = geom[width] - 2 * useless_gap
  +                geom[x] = geom[x] + useless_gap
  +            else
  +                geom[width] = geom[width] - useless_gap
  +            end
  +        end
  +        -- End of useless gap.
  +
  +        geom = cls[c]:geometry(geom)
      end
    
      return used_size

in your rc.lua set a fefault value for the gap:
  useless_gap = 15

and keybinds to change it:
  awful.key({ modkey,           }, "Up",   function ()  
						useless_gap = useless_gap + 5 
						awful.tag.incmwfact(0) -- only for repaint...
					    end),
  awful.key({ modkey,           }, "Down", function ()  
						useless_gap = useless_gap - 5    
						awful.tag.incmwfact(0) -- only for repaint...
					    end),
					    
Guides
======

Install Archlinux with FullDisk Encryption
------------------------------------------
### Setup the live environment
loadkeys de-latin1
wifi-menu

### Setup the partitions
cfdisk /dev/sda
mkfs.ext4 /dev/sda1 -L boot
cryptsetup -c aes-xts-plain64 -y -s 512 luksFormat /dev/sdX2
cryptsetup luksOpen /dev/sda2 rootfs
mkfs.ext4 /dev/mapper/rootfs -L system

mount /dev/mapper/rootfs /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot

### Main installation
pacstrap /mnt base base-devel
pacstrap /mnt grub-bios
pacstrap /mnt <your software>

genfstab -p -U /mnt > /mnt/etc/fstab

### System configuration
arch-chroot /mnt
vi /etc/locale.gen
  de_DE.UTF-8 UTF-8
  en_GB.UTF-8 UTF-8
locale-gen
echo LANG=de_DE.UTF-8 > /etc/locale.conf
export LANG=de_DE.UTF-8

vi /etc/vconsole.conf
  KEYMAP="de-latin1-nodeadkeys"
  FONT=Lat2-Terminus16
  FONT_MAP=

ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
echo HOSTNAME > /etc/hostname

### Modify the initramfs 
vi /etc/mkinitcpio.conf
- Add "keymap" and "encrypt" before "filesystems" in the HOOKS
mkinitcpio -p linux

### Bootloader
grub-install /dev/sda
vi /etc/default/grub
  GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:rootfs"
grub-mkconfig -o /boot/grub/grub.cfg

### Finish the installation
passwd
exit
reboot

Install Archlinux on Multimedia PC
----------------------------------
This describes howto install archlinux with xbmc and hardware video decoding
for an AMD APU (E-350) using the opensource radeon drivers.

### Setup the live environment
loadkeys de-latin1
wifi-menu

### Setup partitions
cfdisk /dev/sda
mkfs.ext4 /dev/sda1 -L system
mount /dev/sda1 /mnt

### Main installation
pacstrap /mnt base base-devel
pacstrap /mnt syslinux

genfstab -p -U /mnt > /mnt/etc/fstab
arch-chroot /mnt

### System configuration
vi /etc/locale.gen
  de_DE.UTF-8 UTF-8
  en_GB.UTF-8 UTF-8
locale-gen
echo LANG=de_DE.UTF-8 > /etc/locale.conf
export LANG=de_DE.UTF-8

vi /etc/vconsole.conf
  KEYMAP="de-latin1-nodeadkeys"
  FONT=Lat2-Terminus16
  FONT_MAP=

ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
ln -s /dev/null /etc/udev/rules.d/80-net-name-slot.rules
echo HOSTNAME > /etc/hostname

### Software installation
pacman -S xorg xorg-xinit xbmc libva-vdpau-driver wicd
systemctl enable wicd

### Create and setup a new user
passwd
useradd eric -m -G users,audio,video,games,network,optical,storage,wheel
passwd eric
su -l eric -c "echo xbmc-standalone > /home/eric/.xinitrc"

### Make use of the video decoding on the new OSS radeon drivers
vi /etc/profile.d/radeon.sh
  export LIBVA_DRIVER_NAME=vdpau
  export VDPAU_DRIVER=r600
chmod +x /etc/profile.d/radeon.sh

### Bootloader
syslinux-install_update -i -a -m
vi /boot/syslinux/syslinux.cfg
  PROMPT 1
  TIMEOUT 50 
  DEFAULT arch
  
  LABEL arch
      LINUX ../vmlinuz-linux
      APPEND root=/dev/sda1 rw quiet radeon.dpm=1 radeon.audio=1 clocksource=hpet hpet=enable
      INITRD ../initramfs-linux.img

### Finish the installation
exit
reboot

### Blu-Ray
for bluray support follow this instructions:
http://vlc-bluray.whoknowsmy.name/ 
this will make it possible to watch some encrypted blurays.

Install Trinity Core on Debian/Ubuntu
-------------------------------------

### Guides
http://collab.kpsn.org/display/tc/How-to_Linux
http://collab.kpsn.org/display/tc/How-to_First_step_into_Trinity_Core


### Downloads
server: git://github.com/TrinityCore/TrinityCore.git 
database: http://www.trinitycore.org/f/files/download/5-tdb-full-updates/


### Install dependencies
sudo apt-get install build-essential autoconf libtool gcc g++ make cmake git-core patch wget links zip unzip unrar
sudo apt-get install openssl libssl-dev libmysqlclient15-dev libmysql++-dev libreadline6-dev zlib1g-dev libbz2-dev libncurses5-dev libace-dev


### Build the Server
  git clone git://github.com/TrinityCore/TrinityCore.git 
  mkdir TrinityCore/build
  cd TrinityCore/build
  cmake ../ -DSERVERS=1 -DTOOLS=1 -DPREFIX=/home/trinity/server -DSCRIPTS=1
  make -j5
  make install


### Build Library for Map extraction
  sudo apt-get install automake1.10
  cd ~/TrinityCore/dep/libmpq/
  sh ./autogen.sh
  ./configure
  make
  sudo make install


### Map Generation
  cd <your WoW client directory>
  mkdir vmaps mmaps
  /home/<username>/server/bin/mapextractor
  /home/<username>/server/bin/vmap4extractor
  /home/<username>/server/bin/vmap4assembler Buildings vmaps
  cp Buildings/* ./vmaps
  /home/<username>/server/bin/mmaps_generator
  cp dbc maps mmaps vmaps /home/<username>/server/data


### Database
  mysql -u root -p < ./TrinityCore/sql/create/create_mysql.sql
  mysql -u root -p auth < ./TrinityCore/sql/base/auth_database.sql 
  mysql -u root -p characters < ./TrinityCore/sql/base/characters_database.sql

  mysql -u root -p world < ./path/to/your/tdb.sql
  cd ./TrinityCore/sql/updates/world
  for i in *; do mysql -u trinity --password=trinity world < $i; done


### Configure
  cp worldserver.conf.dist worldserver.conf
  cp authserver.conf.dist authserver.conf

worldserver.conf:
  LoginDatabaseInfo = "127.0.0.1;3306;trinity;trinity;auth"     
  WorldDatabaseInfo = "127.0.0.1;3306;trinity;trinity;world"     
  CharacterDatabaseInfo = "127.0.0.1;3306;trinity;trinity;characters"

  vmap.enableLOS = 1
  vmap.enableHeight = 1
  vmap.petLOS = 1
  vmap.enableIndoorCheck = 1
  mmap.enablePathFinding = 0

authserver.conf:
  LoginDatabaseInfo = "127.0.0.1;3306;trinity;trinity;auth"


### Start Server
  cd ./server/data
  ../bin/authserver
  ../bin/worldserver


### Add Accounts
  account create <user> <pass>
  account set gmlevel <user> 3 -1


### GM Commands
check quests:
  .lookup quest questname
  .quest complete questid

