<pre>

<h1>  Control SELinux File Contexts </h1>
[root@vm-rhel9 ~]# mkdir /custom
[root@vm-rhel9 ~]# ls -ltr
total 4
-rw-------. 1 root root 1049 Feb  7 09:29 anaconda-ks.cfg
drwxr-xr-x. 3 root root   19 Feb  7 10:53 rhel9-git
[root@vm-rhel9 ~]# echo 'This is SERVERA.' > /custom/index.html
[root@vm-rhel9 ~]#

vim /etc/httpd/conf/httpd.conf

#
# DocumentRoot: The directory out of which you will serve your
# documents. By default, all requests are taken from this directory, but
# symbolic links and aliases may be used to point to other locations.
#
DocumentRoot "/custom"

#
# Relax access to content within /var/www.
#
<Directory "/custom">
    AllowOverride None
    # Allow open access:
    Require all granted
</Directory>

[root@vm-rhel9 ~]# systemctl enable --now httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service                        .

[root@vm-rhel9 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
     Active: active (running) since Fri 2024-02-09 22:02:47 PST; 39s ago
       Docs: man:httpd.service(8)
   Main PID: 10792 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 213 (limit: 23116)
     Memory: 40.3M
        CPU: 124ms
     CGroup: /system.slice/httpd.service
             ├─10792 /usr/sbin/httpd -DFOREGROUND
             ├─10795 /usr/sbin/httpd -DFOREGROUND
             ├─10796 /usr/sbin/httpd -DFOREGROUND
             ├─10797 /usr/sbin/httpd -DFOREGROUND
             └─10798 /usr/sbin/httpd -DFOREGROUND

Feb 09 22:02:46 vm-rhel9.example.com systemd[1]: Starting The Apache HTTP Server...
Feb 09 22:02:47 vm-rhel9.example.com httpd[10792]: Server configured, listening on: port 80
Feb 09 22:02:47 vm-rhel9.example.com systemd[1]: Started The Apache HTTP Server.
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# ls -ldZ /custom/
drwxr-xr-x. 2 root root unconfined_u:object_r:default_t:s0 24 Feb  9 21:59 /custom/
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# semanage fcontext -a -t httpd_sys_content_t '/custom(/.*)?'
[root@vm-rhel9 ~]# ls -ldZ /custom/
drwxr-xr-x. 2 root root unconfined_u:object_r:default_t:s0 24 Feb  9 21:59 /custom/
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# restorecon -Rv /custom
Relabeled /custom from unconfined_u:object_r:default_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
Relabeled /custom/index.html from unconfined_u:object_r:default_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
[root@vm-rhel9 ~]#

Relabeled /custom/index.html from unconfined_u:object_r:default_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
[root@vm-rhel9 ~]# ls -ldZ /custom/
drwxr-xr-x. 2 root root unconfined_u:object_r:httpd_sys_content_t:s0 24 Feb  9 21:59 /custom/
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# curl http://localhost
This is SERVERA.
[root@vm-rhel9 ~]#


</pre>



