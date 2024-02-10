

<pre>

[root@vm-rhel9 SELinux]# getsebool -a | grep httpd
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_manage_courier_spool --> off
httpd_can_network_connect --> off
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> off

[root@vm-rhel9 SELinux]# getsebool httpd_enable_homedirs
httpd_enable_homedirs --> off
[root@vm-rhel9 SELinux]# semanage boolean -l | grep httpd_enable_homedirs
httpd_enable_homedirs          (off  ,  off)  Allow httpd to enable homedirs
[root@vm-rhel9 SELinux]#

[root@vm-rhel9 SELinux]# setsebool httpd_enable_homedirs on
[root@vm-rhel9 SELinux]# semanage boolean -l | grep httpd_enable_homedirs
httpd_enable_homedirs          (on   ,  off)  Allow httpd to enable homedirs
[root@vm-rhel9 SELinux]#

[root@vm-rhel9 SELinux]# getsebool httpd_enable_homedirs
httpd_enable_homedirs --> on
[root@vm-rhel9 SELinux]#

[root@vm-rhel9 SELinux]# semanage boolean -l -C
SELinux boolean                State  Default Description

virt_sandbox_use_all_caps      (on   ,   on)  Allow virt to sandbox use all caps
virt_use_nfs                   (on   ,   on)  Allow virt to use nfs
[root@vm-rhel9 SELinux]#

<b> persistant value </b>
[root@vm-rhel9 SELinux]# setsebool -P httpd_enable_homedirs on
[root@vm-rhel9 SELinux]# semanage boolean -l | grep httpd_enable_homedirs
httpd_enable_homedirs          (on   ,   on)  Allow httpd to enable homedirs
[root@vm-rhel9 SELinux]#

[root@vm-rhel9 SELinux]# semanage boolean -l -C
SELinux boolean                State  Default Description

httpd_enable_homedirs          (on   ,   on)  Allow httpd to enable homedirs
virt_sandbox_use_all_caps      (on   ,   on)  Allow virt to sandbox use all caps
virt_use_nfs                   (on   ,   on)  Allow virt to use nfs
[root@vm-rhel9 SELinux]#



# must have permissions of 711, ~userid/public_html must have permissions
# of 755, and documents contained therein must be world-readable.
# Otherwise, the client will only receive a "403 Forbidden" message.
#
<IfModule mod_userdir.c>
    #
    # UserDir is disabled by default since it can confirm the presence
    # of a username on the system (depending on home directory
    # permissions).
    #
 #   UserDir disabled    <----------- COMMENTS 

    #
    # To enable requests to /~user/ to serve the user's public_html
    # directory, remove the "UserDir disabled" line above, and uncomment
    # the following line instead:
    #
   UserDir public_html     <----------
</IfModule>

#
# Control access to UserDir directories.  The following is an example
# for a site where the

root@vm-rhel9 conf.d]# systemctl enable --now httpd
[root@vm-rhel9 conf.d]# mkdir ~/public_html
[root@vm-rhel9 conf.d]# echo 'This is student content on SERVERA.' > \
~/public_html/index.html
[root@vm-rhel9 conf.d]#


<h1>  Investigate and Resolve SELinux Issues </h1>

[root@vm-rhel9 public_html]# sealert -a /var/log/audit/audit.log
100% done
found 1 alerts in /var/log/audit/audit.log
--------------------------------------------------------------------------------

SELinux is preventing /usr/sbin/httpd from getattr access on the file /web/index.html.

*****  Plugin catchall_labels (83.8 confidence) suggests   *******************

If you want to allow httpd to have getattr access on the index.html file
Then you need to change the label on /web/index.html
Do
# semanage fcontext -a -t FILE_TYPE '/web/index.html'
where FILE_TYPE is one of the following: NetworkManager_exec_t, NetworkManager_log_t, NetworkManager_priv_helper_exec_t, NetworkManager_tmp_t, abrt_dump_oops_exec_t, abrt_etc_t, abrt_exec_t, abrt_handle_event_exec_t, abrt_helper_exec_t, abrt_retrace_coredump_exec_t, abrt_retrace_spool_t, abrt_retrace_worker_exec_t, abrt_tmp_t, abrt_upload_watch_tmp_t, abrt_var_cache_t, abrt_var_log_t, abrt_var_run_t, accountsd_exec_t, acct_data_t, acct_exec_t, admin_crontab_tmp_t, admin_passwd_exec_t, afs_logfile_t, aide_exec_t, aide_log_t, alsa_exec_t, alsa_tmp_t, amanda_exec_t, amanda_log_t, amanda_recover_exec_t, amanda_tmp_t, amtu_exec_t, anacron_exec_t, anon_inodefs_t, antivirus_exec_t, antivirus_log_t, antivirus_tmp_t,  

*****  Plugin catchall (17.1 confidence) suggests   **************************

If you believe that httpd should be allowed getattr access on the index.html file by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# ausearch -c 'httpd' --raw | audit2allow -M my-httpd
# semodule -X 300 -i my-httpd.pp


Additional Information:
Source Context                system_u:system_r:httpd_t:s0
Target Context                unconfined_u:object_r:default_t:s0
Target Objects                /web/index.html [ file ]
Source                        httpd
Source Path                   /usr/sbin/httpd
Port                          <Unknown>
Host                          <Unknown>
Source RPM Packages           httpd-core-2.4.57-5.el9.x86_64
Target RPM Packages
SELinux Policy RPM            selinux-policy-targeted-34.1.43-1.el9.noarch
Local Policy RPM              selinux-policy-targeted-34.1.43-1.el9.noarch
Selinux Enabled               True
Policy Type                   targeted
Enforcing Mode                Enforcing
Host Name                     vm-rhel9.example.com
Platform                      Linux vm-rhel9.example.com
                              5.14.0-162.6.1.el9_1.x86_64 #1 SMP PREEMPT_DYNAMIC
                              Fri Sep 30 07:36:03 EDT 2022 x86_64 x86_64
Alert Count                   2
First Seen                    2024-02-08 12:05:24 PST
Last Seen                     2024-02-08 12:05:24 PST
Local ID                      a52882d4-45f4-407c-9215-382e1b4683a3

Raw Audit Messages
type=AVC msg=audit(1707422724.754:232): avc:  denied  { getattr } for  pid=4712 comm="httpd" path="/web/index.html" dev="dm-0" ino=134295306 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:default_t:s0 tclass=file permissive=0


type=SYSCALL msg=audit(1707422724.754:232): arch=x86_64 syscall=newfstatat success=no exit=EACCES a0=ffffff9c a1=7f747800a5b0 a2=7f748ce387c0 a3=100 items=0 ppid=4708 pid=4712 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm=httpd exe=/usr/sbin/httpd subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=newfstatat AUID=unset UID=apache GID=apache EUID=apache SUID=apache FSUID=apache EGID=apache SGID=apache FSGID=apache

Hash: httpd,httpd_t,default_t,file,getattr

[root@vm-rhel9 public_html]#



root@vm-rhel9 public_html]# touch /root/mypage
[root@vm-rhel9 public_html]# mv /root/mypage /var/www/html
[root@vm-rhel9 public_html]# systemctl start httpd
[root@vm-rhel9 public_html]# curl http://localhost/mypage
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
[root@vm-rhel9 public_html]#

[root@vm-rhel9 public_html]# tail /var/log/audit/audit.log
type=BPF msg=audit(1707546943.212:162): prog-id=0 op=UNLOAD
type=BPF msg=audit(1707546943.212:163): prog-id=0 op=UNLOAD
type=BPF msg=audit(1707546943.213:164): prog-id=60 op=LOAD
type=BPF msg=audit(1707546943.213:165): prog-id=0 op=UNLOAD
type=BPF msg=audit(1707546943.214:166): prog-id=61 op=LOAD
type=BPF msg=audit(1707546943.214:167): prog-id=0 op=UNLOAD
type=BPF msg=audit(1707546943.214:168): prog-id=62 op=LOAD
type=BPF msg=audit(1707546943.214:169): prog-id=63 op=LOAD
type=BPF msg=audit(1707546943.214:170): prog-id=0 op=UNLOAD
type=BPF msg=audit(1707546943.214:171): prog-id=0 op=UNLOAD
[root@vm-rhel9 public_html]#
[root@vm-rhel9 web]# ls -lZ
total 4
-rw-r--r--. 1 root root unconfined_u:object_r:default_t:s0 10 Feb  8 12:02 index.html
[root@vm-rhel9 web]# semanage fcontext -a \
-t httpd_sys_content_t '/web(/.*)?'
[root@vm-rhel9 web]# restorecon -Rv /web
Relabeled /web from unconfined_u:object_r:default_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
Relabeled /web/index.html from unconfined_u:object_r:default_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
[root@vm-rhel9 web]# ls -lZ
total 4
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 10 Feb  8 12:02 index.html
[root@vm-rhel9 web]#

[root@vm-rhel9 web]# semanage login -l

Login Name           SELinux User         MLS/MCS Range        Service

__default__          unconfined_u         s0-s0:c0.c1023       *
root                 unconfined_u         s0-s0:c0.c1023       *
[root@vm-rhel9 web]#


[root@vm-rhel9 web]# aureport --avc

AVC Report
===============================================================
# date time comm subj syscall class permission obj result event
===============================================================
1. 2024-02-08 12:05:24 httpd system_u:system_r:httpd_t:s0 262 file getattr unconfined_u:object_r:default_t:s0 denied 231
2. 2024-02-08 12:05:24 httpd system_u:system_r:httpd_t:s0 262 file getattr unconfined_u:object_r:default_t:s0 denied 232
[root@vm-rhel9 web]# aureport --avc --summary

Avc Object Summary Report
=================================
total  obj
=================================
2  unconfined_u:object_r:default_t:s0
[root@vm-rhel9 web]#








</pre>



