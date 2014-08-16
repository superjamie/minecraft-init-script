minecraft-init-script by Super Jamie <jamie@superjamie.net>

This is an initscript to run a Minecraft or CraftBukkit server on CentOS, Fedora, and Ubuntu.

## Features ##

* Start, stop, restart CraftBukkit as a system service
* Automatic (via cron) and manual logfile rotation
* Automatic (via cron) and manual backups
* Backup compression and rotation (keeps 7 days worth of backups)
* Check latest Recommended Build and update to it if required
* Information display including Java path, current memory usage, current TCP connections
* Able to run multiple separate instances of the server at once

## Features I'm considering adding ##

* Simplified backup restoration
* Log security alerts emailed to admins via sendmail/postfix

## Supported Distributions ##

* CentOS 6, CentOS 5, Fedora 14 (probably works on Fedora Core 6 and later, untested)
* Ubuntu Server 12.04 LTS

Other distros which use SysV Init or Upstart (Debian, Mint) will probably work.
Distros using systemd (Fedora 15+, Arch Linux, etc) will not work.

## Requirements ##

* screen, rsync
  (you may need to install these)
* bash, chkconfig or sysv-rc, coreutils, cronie, curl, diffutils, grep, initscripts, net-tools, procps, tar
  (these should all be installed by default)
* Oracle Java 7
* Enough disk space to save your map twice, plus another ~5 times for a week of compressed backup space.
  If your map is 1Gb then you probably need at least 7Gb, plus any space your plugins require and any additional backups you'll be making.

## Installation ##

As the root user:

* Install Sun Java (CentOS/Fedora)
  Download the RPM from http://www.java.com/
  yum localinstall jre-<version>.rpm
* Install Sun Java (Ubuntu)
  sudo add-apt-repository ppa:webupd8team/java
  sudo apt-get update
  sudo apt-get install oracle-java7-installer
* Confirm your JVM installation with
  java -version
* Create a new user with a home directory
  useradd -m bukkit
* Save the script as /etc/init.d/minecraft and make it executable
  chmod +x /etc/init.d/minecraft
* Copy between the <<COMMENT and COMMENT lines and place the copy at /etc/default/minecraft
  If you need to edit settings, edit the /etc/default/minecraft file, not the initscript
* Allow the bukkit user to run the init script without needing root access
  Type `visudo` and add this line to the bottom:
  bukkit localhost=NOPASSWD:/etc/init.d/minecraft*
* Create an alias so you only have to type "minecraft" to run the script
  Add the following line to both root and bukkit's ~/.bashrc file:
  alias minecraft='/etc/init.d/minecraft'
* Start the server on system boot if desired (CentOS/Fedora)
  chkconfig --add minecraft
  chkconfig minecraft on
* Start the server on system boot if desired (Ubuntu)
  update-rc.d minecraft defaults

As the regular user, bukkit:

* Make the required paths
  mkdir -p ~/backups && mkdir -p ~/craftbukkit
* Put your craftbukkit.jar, world, plugins, server.properties, etc into ~/craftbukkit

## Backups ##

* Create cron jobs to do regular backups and rotations around 4am
  Type `crontab -e` to open the cron interface and add the following
   0 4 * * * /etc/init.d/minecraft backup              # backup world at 4:00am
   5 4 * * * /etc/init.d/minecraft logrotate           # rotate logs at 4:05am
  15 4 * * * /etc/init.d/minecraft removeoldbackups    # remove old backups at 4:30am

* If you have multiple worlds, you can pass the worldname as a parameter to the regular backup
   0 4 * * * /etc/init.d/minecraft backup world1       # backup world1 at 4:00am
   5 4 * * * /etc/init.d/minecraft logrotate           # rotate logs at 4:05am
  15 4 * * * /etc/init.d/minecraft backup world2       # backup world2 at 4:15am
  30 4 * * * /etc/init.d/minecraft removeoldbackups    # remove old backups at 4:30am

## Multiple Instances ##

It is possible to run multiple instances, for example a Creative server and a Survival server, on the same OS.

* Copy the script to two new files
  cp /etc/init.d/minecraft /etc/init.d/minecraft-creative
  cp /etc/init.d/minecraft /etc/init.d/minecraft-survival
* Edit the Provides section on Line 6 to the same as the new filename
  # Provides: minecraft-creative
  # Provides: minecraft-survival
* Create a settings file for each instance in /etc/default/ using the same name as the script
  /etc/default/minecraft-creative
  /etc/default/minecraft-creative
* Set an alias for each server in ~/.bashrc
  alias creative='/etc/init.d/minecraft-creative'
  alias survival='/etc/init.d/minecraft-survival'
* Add the new scripts to chkconfig or update-rc.d
* Set the paths of the separate maps in each script
  MCPATH="/home/bukkit/craftbukkit-creative"
  MCPATH="/home/bukkit/craftbukkit-survival"
* Change your screen session names in each script
  SCRNAME="creative"
  SCRNAME="survival"

## Usage ##

* Start the server
  minecraft start
* Stop the server
  minecraft stop
* Restart the server
  minecraft restart
* Back up the map and executable (the compressed backup is done when the .md5 file appears in the backup directory)
  minecraft backup
* Back up multiple maps
  minecraft backup world1
  minecraft backup world2
* Check the server is running with
  minecraft status
* Get some more info with
  minecraft info

  * CraftBukkit (pid 9037) is running...
  - Java Path : /usr/java/jre1.6.0_31/bin/java
  - Start Command : java -Xms512M -Xmx3584M -jar craftbukkit.jar nogui
  - Server Path : /home/bukkit/craftbukkit
  - World Name : world
  - Process ID : 9037
  - Memory Usage : 742796 kb
  - Active Connections :
  Proto Recv-Q Send-Q Local Address Foreign Address State
  tcp 0 0 0.0.0.0:25565 0.0.0.0:* LISTEN
  tcp 0 0 192.168.2.99:25565 192.168.2.69:55507 ESTABLISHED
* Broadcast a message to the server with
  minecraft say
  (note that some punctuation like 'apostrophes' will not work)

## License ##

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

## Credits ##

Thanks to these people whose work I have used in the making of this

* http://forums.bukkit.org/threads/tutorial-centos-bukkit-installation.56371/
* http://www.minecraftwiki.net/wiki/M3tal_Warrior_Server_Startup_Script
* http://forums.bukkit.org/threads/admin-linux-init-script-for-bukkit.53235/
