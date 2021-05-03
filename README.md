# roger-skyline-1
This project, roger-skyline-1 let you install a Virtual Machine, discover the basics about system and network administration as well as a lots of services used on a server machine.

### 3 / Network and Security Part

You must install sudo
```
apt install sudo -y
```
You must create a non-root user to connect to the machine and work.
```
sudo adduser newuser
```

Use sudo, with this user, to be able to perform operation requiring special rights.
```
sudo usermod -aG sudo newuser
```

To switch to the newly created user use:
```
su newuser
```
_NB: while entering the password you won't see anything, it is normal._  

use ```passwd``` if you want to change the password to a more robust one.  



• We don’t want you to use the DHCP service of your machine. You’ve got to
configure it to have a static IP and a Netmask in \30.
```
```

You have to change the default port of the SSH service by the one of your choice. Let's change it to 57331.
```
sudo vi /etc/ssh/sshd_config
ucomment port: 22 and change the value to 57331
you can chose wthaever port you want but it's best to chose o port above 10000
as these ports are rarely used by your system.
```
then 
```
sudo systemctl restart sshd
```

SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT
be allowed directly, but with a user who can be root.

On your Computer:
```
$ ssh-keygen
# just hit enter to keep it simple.
```
then
```
$ ssh-copy-id -p 57331 -i ~/.ssh/id_rsa.pub newuser@host_ip_address
```

Check that the key is now present with:
```
.ssh/authorized_keys
```

You have to set the rules of your firewall on your server only with the services used outside the VM. You only need the SSH access
```
$ sudo apt install ufw -y
$ sudo ufw enable
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 57331/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 443/tcp
$ sudo ufw status verbose
```
_NB: use the correct ssh port that you previously changed_

Additionally you can disable the password login but make sure that ```.ssh/authorized_keys``` contains the private kay.
```
PasswordAuthentication no
```

You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.  

```
$ sudo install fail2ban -y
$ sudo echo "[dos_protect_http_s]" > /etc/fail2ban/jail.local
$ sudo echo "enable = true" >> /etc/fail2ban/jail.local
$ sudo echo "port = 80,443" >> /etc/fail2ban/jail.local
$ sudo echo "logpath = /var/log/apache2/access.log" >> /etc/fail2ban/jail.local
$ sudo echo "filter = dos_filter_http_s" >> /etc/fail2ban/jail.local
$ sudo echo "[definition]" > /etc/fail2ban/filter.d/dos_filter_http_s.conf
$ sudo echo "failregex = "^<HOST> .*" >> /etc/fail2ban/filter.d/dos_filter_http_s.conf
$ sudo echo "maxretry = 30" >> /etc/fail2ban/jail.local
$ sudo echo "findtime = 60" >> /etc/fail2ban/jail.local
$ sudo echo "bantime = 60" >> /etc/fail2ban/jail.local
$ sudo echo "" >> /etc/fail2ban/jail.local
$ sudo echo "[dos_protect_ssh]" >> /etc/fail2ban/jail.local
$ sudo echo "enable = true" >> /etc/fail2ban/jail.local
$ sudo echo "port = 57331" >> /etc/fail2ban/jail.local
$ sudo echo "logpath = /var/log/auth.log" >> /etc/fail2ban/jail.local
$ sudo echo "filter = dos_filter_ssh" >> /etc/fail2ban/jail.local
$ sudo echo "[definition]" > /etc/fail2ban/filter.d/dos_filter_ssh.conf
$ sudo echo "failregex = .* Bad .* from <HOST> .*" >> /etc/fail2ban/filter.d/dos_filter_ssh.conf
$ sudo echo "maxretry = 10" >> /etc/fail2ban/jail.local
$ sudo echo "findtime = 60" >> /etc/fail2ban/jail.local
$ sudo echo "bantime = 60" >> /etc/fail2ban/jail.local
```

You have to set a protection against scans on your VM’s open ports.
```
$ sudo echo "" >> /etc/fail2ban/jail.local
$ sudo echo "[scan_protect]" >> /etc/fail2ban/jail.local
$ sudo echo "enable = true" >> /etc/fail2ban/jail.local
$ sudo echo "logpath = /var/log/syslog" >> /etc/fail2ban/jail.local
$ sudo echo "filter = dos_filter_ssh" >> /etc/fail2ban/jail.local
$ sudo echo "[definition]" > /etc/fail2ban/filter.d/dos_filter_ssh.conf
$ sudo echo "failregex = .* Bad .* from <HOST> .*" >> /etc/fail2ban/filter.d/dos_filter_ssh.conf
$ sudo echo "maxretry = 10" >> /etc/fail2ban/jail.local
$ sudo echo "findtime = 60" >> /etc/fail2ban/jail.local
$ sudo echo "bantime = 60" >> /etc/fail2ban/jail.local
```

### Stop the services you don’t need for this project.

you can display with:
```
service --status-all
```

You need to keep the following ones:
```
ssh
ufw
```

### WIP (booh!)
Create a script that updates all the sources of package, then your packages and which logs the whole in a file named /var/log/update_script.log. Create a scheduled
task for this script once a week at 4AM and every time the machine reboots.
• Make a script to monitor changes of the /etc/crontab file and sends an email to
root if it has been modified. Create a scheduled script task every day at midnight.
