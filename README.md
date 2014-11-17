This will ultimately be a vagrant setup for SS to study for the RHCE

#Set SELinux to enforcing mode
	sestatus/getenforce
	lokkit --selinux=enforcing
	vi /etc/selinux/config
	SELINUX=enforcing

#Create a repo
	vi /etc/yum.repo.d/base.repo
	[base]
	name=example base repo
	baseurl=ftp://server.example.com/pub/packages
	enabled=1
	gpgcheck=0

#Configure your host to forward ipv4 packets
	vi /etc/sysctl.conf
	# Controls IP packet forwarding
	net.ipv4.ip_forward = 1

#Set up a mail server w/ the following conditions
- Natasha's mail should be spooled to /var/spool/mail/natasha
- The server should accept mail remotely
- All mail sent to admin should be received by natasha
	yum install -y postfix*
	vi /etc/postfix/main.cf
	queue_directory = /var/spool/mail
	inet_address=all
	myhostname=host.example.com
	mydomain=example.com
	/etc/init.d/postfix restart
	chkconfig postfix on
	mail -v natasha@<ip> this is a test .
	vi /etc/aliases
	admin: natasha
	newaliases
	mail -v admin@<ip> this ia a test .

#Write a script in bash such that:
- Aurg python
- OP perl
- Aurg perl
- OP python
		vim /root/script.sh
		#!/bin/bash
		if [ $# -ne 1 ]; then
			echo "Invalid Aurgument"
			exit 1
		fi
	
		case $1 in
			python) echo "perl"
			;;
			perl) echo "python"
			;;
			*) echo "python|perl"
			;;
		esac

#Configure an FTP server such that
- natasha can login via ftp 
- anon enabled
- users can download
- access allowed from example.com and denied from bad.com
	yum -y install vsftpd*
	vi /etc/vsftpd/vsftpd.conf  (Verify for anonymous access/tcp wrappers)                  
	anonymous_enable=yes
	local_enable=yes
	no_anon_password=yes
	tcp_wrapper_enable=yes	
	vi /etc/hosts.deny
	vsftpd: .bad.com
	chkconfig vsftpd on
	service vfstpd restart
	setsebool -P ftp_home_dir 1
	getsebool -a | grep ftp_home_dir
	ftp as yourself to test

#Set up an FTP server such that
- /common is exported and only accessible by example.com
	yum -y install nfs*
	vi /etc/exports
	/common *.example.com(rw,sync)
	/etc/init.d/portmap restart
	/etc/init.d/nfs restart
	chkconfig nfs on
	chkconfig portmap on
	chkconfig nfslock on
	showmount -e

#Mount the ISO /root/boot.iso to /disk this mount should be persistent across reboots.
	vi /etc/fstab
	/root/boot.iso /disk auto defaults,loop 0 0
	mount -av
	df -h

#Setup an ssh server such that only users from example.com are allowed.
	yum -y install openssh*
	vi /etc/hosts.deny
	sshd: ALL EXCEPT .example.com
	/etc/init.d/sshdb restart
	chkconfig sshd on
	netstat -antp | grep sshd

#Create a website by your hostname ie "http://stationx.example.com"
- Copy station.html from server1.example.com/pub/
- Rename this as index.html
- Move it to the standard document root of apache
- Pre-res is provided by DNS
	yum -y install httpd*
	vi /etc/httpd/conf/httpd.conf
	ServerName stationx.example.com
	NameVirtualHost stationx.example.com
	<VirtualHost station11.example.com>
	ServerAdmin webmaster@station11.example.com
	DocumentRoot /var/www/html
	ServerName station11.example.com
	ErrorLog logs/station11.example.com-error_log
	CustomLog logs/station11.example.com-access_log common
	</VirtualHost>	
	/etc/init.d/httpd restart
	chkconfig httpd on
	httpd -t
	gftp
	mv stationx.html /var/www/html/index.html
	restorecon -R /var/www/html/index.html
	browse from host

#Extend your server to host virtual site wwwx.example.com
- Doc Root should be in /var/www/virtual
- copy from dir server1/pub/www.html as index.html
- Harry should be able to write contents to /var/www/virtual
	mkdir /var/www/virtual
	gftp www.html
	mv www.html /var/www/virtual/index.html
	vi /etc/httpd/conf/httpd.conf #add new virtual host section
	<VirtualHost www11.example.com:80>
	ServerAdmin webmaster@www11.example.com
	DocumentRoot /var/www/virtual
	ServerName www11.example.com
	ErrorLog logs/www11.example.com-error_log
	CustomLog logs/www11.example.com-access_log common
	</VirtualHost>
	/etc/init.d/httpd restart
	httpd -t
	restorecon -R /var/www/virtual/index.html
	chcon -R --reference=/var/www/html /var/www/virtual
	setfacl -m "u:harry:rwx" /var/www/virtual
	getfacl
	browse to page

#Import an ISCSI disk from the server server1.example.com such that
- the disk must be mounte as /mnt/iscsi
- this mount should be persistent	
	rpm -qa | grep iscsi
	yum install isci-initiator-utils
	iscsiadm -m discovery -t st -p server1.example.com
	iscsiadm -m node -T <iqn> -p server1.example.com -l
	tailf /var/log/messages to get device type
	fdisk -cu /dev/sd?
	mkfs.ext4 /dev/sd?1
	blkid /dev/sda1 #UUID
	vi /etc/fstab
	UUID=<uuid> /mnt/iscsi ext4 defaults,_netdev 0 0

#Create a Samba share /common such that:
- Harry can only read the contents of /common
- harry can be asked for auth
- workgroup should be set to STAFF
- The share /common should be accessible and browseable only from .example.com
- password for harry is "password"
	yum -y install samba*
	vi /etc/samba/smb.conf
	workgroup = STAFF
	encrypt passwords = yes
	security = user
	[common]
	path = /common
	read list = harry
	browseable = yes
	hosts allow = .example.com
	/etc/init./smb restart
	chkconfig smb on
	testparm
	smbpasswd -a harry
	pbedit -L
	setsebool -P samba_enable_home_dirs on
	chcon -t samba_share_t/common
	smbclient //stationx.example.com -U harry

#The user jean should not be allowed to add a cron job for himself
	vi /etc/cron.deny
	jean

#Copy the file boot.iso to /var/www/html/secure. Secure the file and make it available to only local hosts over apache webserver
	cp boot.iso /var/www/html/secure
	vi /etc/httpd/conf/httpd.conf
	/cgi-bin
	<Directory "/var/www/html/secure/boot.iso">
	Allow Override None
	Order deny,allow
	Allow from .example.com
	Deny from All
	</Directory>

#Pass a parameter sysvctl=1 to the kernel at boot time changes should be persistent.
	vi /boot/grub/grub.conf
	/KEYTABLE 
	sysvctl=1

#Build an RPM that packages a single file rpm.txt
	yum -y install rpmdevtools
	rpmdev-setuptree
	mkdir rpm-1.0
	cp rpm.txt rpm-1.0
	tar czf rpm-1.0.tar.gz rpm-1.0
	cp rpm-1.0.tar.gz rpmbuild/SOURCES
	rpmdev-newspec rpmbuild/SPECS/rpm.spec
	vi rpmbuild/SPECS/rpm.spec
