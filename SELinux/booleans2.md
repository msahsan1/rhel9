<pre>
<h1>Boolean <h2>
[root@vm-rhel9 ~]# mkdir /common/
[root@vm-rhel9 ~]# ls -ldZ /common/
drwxr-xr-x. 2 root root unconfined_u:object_r:default_t:s0 6 Feb 10 08:53 /common/
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# id natasha
id: ‘natasha’: no such user
[root@vm-rhel9 ~]# useradd  -m -s /sbin/nologin natasha
[root@vm-rhel9 ~]# id natasha
uid=1001(natasha) gid=1001(natasha) groups=1001(natasha)
[root@vm-rhel9 ~]#

apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
natasha:x:1001:1001::/home/natasha:/sbin/nologin
[root@vm-rhel9 ~]# su - natasha
This account is currently not available.
[root@vm-rhel9 ~]#


[root@vm-rhel9 ~]# groupadd  smbgroup
[root@vm-rhel9 ~]# usermod -g smbgroup natasha
[root@vm-rhel9 ~]# smbpasswd -a natasha
New SMB password:
Retype new SMB password:
Added user natasha.
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# semanage fcontext -a -t samba_share_t "/common(./*)?"
[root@vm-rhel9 ~]# ls -dZ /common/
unconfined_u:object_r:default_t:s0 /common/
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# restorecon -vvFR /common/
Relabeled /common from unconfined_u:object_r:default_t:s0 to system_u:object_r:samba_share_t:s0
[root@vm-rhel9 ~]# ls -dZ /common/
system_u:object_r:samba_share_t:s0 /common/

[root@vm-rhel9 ~]# ausearch -m AVC -ts recent
<no matches>
[root@vm-rhel9 ~]# getsebool -a | grep smb
smbd_anon_write --> off
[root@vm-rhel9 ~]# semanage boolean -l | grep smb
smbd_anon_write                (off  ,  off)  Allow smbd to anon write
[root@vm-rhel9 ~]# setsebool smbd_anon_write on
[root@vm-rhel9 ~]# semanage boolean -l | grep smb
smbd_anon_write                (on   ,  off)  Allow smbd to anon write  <---- Not persistant


[root@vm-rhel9 ~]# setsebool -P smbd_anon_write on
[root@vm-rhel9 ~]# semanage boolean -l | grep smb
smbd_anon_write                (on   ,   on)  Allow smbd to anon write <-- Persistant both on)


<h2> Senarios </h2>
[root@vm-rhel9 ~]# vim /etc/httpd/conf.d/userdir.conf
[root@vm-rhel9 ~]# systemctl restart httpd
[root@vm-rhel9 ~]# useradd anna
[root@vm-rhel9 ~]# chmod -R 755 /home/anna/
[root@vm-rhel9 ~]# su - anna
[anna@vm-rhel9 ~]$ mkdir public_html
[anna@vm-rhel9 ~]$ echo "Hello ann "> public_html/index.html
[anna@vm-rhel9 ~]$

[root@vm-rhel9 ~]# sesearch -s httpd_t -t httpd_user_content_t -A
allow domain file_type:blk_file map; [ domain_can_mmap_files ]:True
allow domain file_type:chr_file map; [ domain_can_mmap_files ]:True
allow domain file_type:file map; [ domain_can_mmap_files ]:True
allow domain file_type:lnk_file map; [ domain_can_mmap_files ]:True
allow httpd_t file_type:dir { getattr open search };
allow httpd_t file_type:filesystem getattr;
allow httpd_t httpd_user_content_type:dir { ioctl lock read }; [ httpd_enable_homedirs ]:True
allow httpd_t httpd_user_content_type:file map; [ httpd_enable_homedirs ]:True
allow httpd_t httpd_user_content_type:file { getattr ioctl lock open read }; [ httpd_enable_homedirs                                                                ]:True
allow httpd_t httpd_user_content_type:lnk_file { getattr read }; [ httpd_enable_homedirs ]:True
allow httpd_t httpdcontent:dir { add_name create ioctl link lock read remove_name rename reparent rmd                                                               ir setattr unlink watch watch_reads write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_en                                                               able_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scri                                                               pting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scri                                                               pting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scri                                                               pting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scri                                                               pting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scri                                                               pting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:dir { add_name ioctl lock read remove_name write }; [ ( httpd_builtin_scri                                                               pting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:file { append create getattr ioctl link lock open read rename setattr unli                                                               nk watch watch_reads write }; [ ( httpd_builtin_scri
pting && httpd_unified && httpd_enable_cgi ) ]:Tr                                                               ue
allow httpd_t httpdcontent:file { execute execute_no_trans getattr ioctl map open read }; [ ( httpd_b                                                               uiltin_scripting && httpd_unified && httpd_enable_cgi ) ]:True
allow httpd_t httpdcontent:lnk_file { append create getattr ioctl link lock read rename setattr unlin                                                               k watch watch_reads write }; [ ( httpd_builtin_scripting && httpd_unified && httpd_enable_cgi ) ]:Tru                                                               e
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# setsebool -P httpd_enable_homedirs on

<h2>

VSFTP </h2>

[root@vm-rhel9 ~]# ls -dZ /var/ftp
system_u:object_r:public_content_t:s0 /var/ftp
[root@vm-rhel9 ~]# mkdir /var/ftp/pub
[root@vm-rhel9 ~]# ls -dZ /var/ftp
system_u:object_r:public_content_t:s0 /var/ftp
[root@vm-rhel9 ~]# semanage fcontext -a -t public_content_rw_t "/var/ftp/pub(/.*)?"
[root@vm-rhel9 ~]# ls -dZ /var/ftp
system_u:object_r:public_content_t:s0 /var/ftp
[root@vm-rhel9 ~]# restorecon -RV /var/ftp/pub/
restorecon: invalid option -- 'V'
usage:  restorecon [-iIDFmnprRv0xT] [-e excludedir] pathname...
usage:  restorecon [-iIDFmnprRv0xT] [-e excludedir] -f filename
[root@vm-rhel9 ~]# restorecon -Rv /var/ftp/pub/
Relabeled /var/ftp/pub from unconfined_u:object_r:public_content_t:s0 to unconfined_u:object_r:public_content_rw_t:s0
[root@vm-rhel9 ~]# ls -dZ /var/ftp
system_u:object_r:public_content_t:s0 /var/ftp
[root@vm-rhel9 ~]# ls -dZ /var/ftp/pub/
unconfined_u:object_r:public_content_rw_t:s0 /var/ftp/pub/
[root@vm-rhel9 ~]# chmod 777 /var/ftp/pub/
[root@vm-rhel9 ~]#



[root@vm-rhel9 ~]# vim /etc/vsftpd/vsftpd.conf
[root@
# Example config file /etc/vsftpd/vsftpd.conf
#
# The default compiled in settings are fairly paranoid. This sample file
# loosens things up a bit, to make the ftp daemon more usable.
# Please see vsftpd.conf.5 for all compiled in defaults.
#
# READ THIS: This example file is NOT an exhaustive list of vsftpd options.
# Please read the vsftpd.conf.5 manual page to get a full idea of vsftpd's
# capabilities.
#
# Allow anonymous FTP? (Beware - allowed by default if you comment this out).
anonymous_enable=yes ### <----------------
#
# Uncomment this to allow local users to log in.
local_enable=YES
#
# Uncomment this to enable any form of FTP write command.
write_enable=YES
#
# Default umask for local users is 077. You may wish to change this to 022,
# if your users expect that (022 is used by most other ftpd's)
local_umask=022
#
# Uncomment this to allow the anonymous FTP user to upload files. This only
# has an effect if the above global write enable is activated. Also, you will
# obviously need to create a directory writable by the FTP user.
# When SELinux is enforcing check for SE bool allow_ftpd_anon_write, allow_ftpd_full_access
anon_upload_enable=YES ## <-------------------
#
# Uncomment this if you want the anonymous FTP user to be able to create
# new directories.
#anon_mkdir_write_enable=YES
#
# Activate directory messages - messages given to remote users when they
# go into a certain directory.

<h1> SElinux TroubleShoot </h1>

<b>[root@vm-rhel9 ~]# grep AVC /var/log/audit/audit.log </b>
type=AVC msg=audit(1707422724.754:231): avc:  denied  { getattr } for  pid=4712 comm="httpd" path="/web/index.html" dev="dm-0" ino=134295306 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:default_t:s0 tclass=file permissive=0
type=AVC msg=audit(1707422724.754:232): avc:  denied  { getattr } for  pid=4712 comm="httpd" path="/web/index.html" dev="dm-0" ino=134295306 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:default_t:s0 tclass=file permissive=0
[root@vm-rhel9 ~]#



</pre>
