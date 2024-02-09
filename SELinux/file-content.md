<pre>
<h1> Control SELinux File Contexts </h1>

[root@vm-rhel9 rhel9-git]# getenforce
Enforcing
[root@vm-rhel9 rhel9-git]# grep ^[^#] /etc/selinux/config
SELINUX=enforcing
SELINUXTYPE=targeted
[root@vm-rhel9 rhel9-git

[root@vm-rhel9 rhel9-git]# setenforce 0
[root@vm-rhel9 rhel9-git]# getenforce
Permissive
[root@vm-rhel9 rhel9-git]#

[root@vm-rhel9 rhel9-git]# setenforce 1
[root@vm-rhel9 rhel9-git]# getenforce
Enforcing
[root@vm-rhel9 rhel9-git]#

[root@vm-rhel9 ~]# touch /tmp/file1 /tmp/file2
[root@vm-rhel9 ~]# ls -Z /tmp/file*
unconfined_u:object_r:user_tmp_t:s0 /tmp/file1  unconfined_u:object_r:user_tmp_t:s0 /tmp/file2
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# ls -lZ /tmp/file*
-rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 0 Feb  9 10:30 /tmp/file1
-rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 0 Feb  9 10:30 /tmp/file2
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# ls -Zd /var/www/html/
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# mv /tmp/file1 /var/www/html/
[root@vm-rhel9 ~]# cp /tmp/file2 /var/www/html/
[root@vm-rhel9 ~]# ls -lZ /var/www/html/file*
-rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0          0 Feb  9 10:30 /var/www/html/file1
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 0 Feb  9 10:33 /var/www/html/file2
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# mkdir /virtual
[root@vm-rhel9 ~]# ls -Zd /virtual/
unconfined_u:object_r:default_t:s0 /virtual/

[root@vm-rhel9 ~]# chcon -t httpd_sys_content_t /virtual/
[root@vm-rhel9 ~]# ls -Zd /virtual/
unconfined_u:object_r:httpd_sys_content_t:s0 /virtual/
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# restorecon -v /virtual/
Relabeled /virtual from unconfined_u:object_r:httpd_sys_content_t:s0 to unconfined_u:object_r:default_t:s0
[root@vm-rhel9 ~]# ls -Zd /virtual/
unconfined_u:object_r:default_t:s0 /virtual/

[root@vm-rhel9 ~]# ls -lZ /var/www/html/file*
-rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0          0 Feb  9 10:30 /var/www/html/file1
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 0 Feb  9 10:33 /var/www/html/file2
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# semanage fcontext -l | grep http
/etc/WebCalendar(/.*)?                             all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/apache(2)?(/.*)?                              all files          system_u:object_r:httpd_config_t:s0
/etc/apache-ssl(2)?(/.*)?                          all files          system_u:object_r:httpd_config_t:s0
/etc/cherokee(/.*)?                                all files          system_u:object_r:httpd_config_t:s0
/etc/drupal.*                                      all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/glpi(/.*)?                                    all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/horde(/.*)?                                   all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/htdig(/.*)?                                   all files          system_u:object_r:httpd_sys_content_t:s0
/etc/httpd(/.*)?                                   all files          system_u:object_r:httpd_config_t:s0
/etc/httpd/.*                                      symbolic link      system_u:object_r:etc_t:s0
/etc/httpd/alias(/.*)?                             all files          system_u:object_r:cert_t:s0
/etc/httpd/conf/keytab                             regular file       system_u:object_r:httpd_keytab_t:s0
/etc/init\.d/cherokee                              regular file       system_u:object_r:httpd_initrc_exec_t:s0
/etc/lighttpd(/.*)?                                all files          system_u:object_r:httpd_config_t:s0
/etc/mock/koji(/.*)?                               all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/nextcloud(/.*)?                               all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/nginx(/.*)?                                   all files          system_u:object_r:httpd_config_t:s0
/etc/opt/rh/rh-nginx18/nginx(/.*)?                 all files          system_u:object_r:httpd_config_t:s0
/etc/owncloud(/.*)?                                all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/rc\.d/init\.d/httpd                           regular file       system_u:object_r:httpd_initrc_exec_t:s0
/etc/rc\.d/init\.d/lighttpd                        regular file       system_u:object_r:httpd_initrc_exec_t:s0
/etc/rt(/.*)?                                      all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/thttpd\.conf                                  regular file       system_u:object_r:httpd_config_t:s0
/etc/vhosts                                        regular file       system_u:object_r:httpd_config_t:s0
/etc/z-push(/.*)?                                  all files          system_u:object_r:httpd_sys_rw_content_t:s0
/etc/zabbix/web(/.*)?                              all files          system_u:object_r:httpd_sys_rw_content_t:s0
/home/[^/]+/((www)|(web)|(public_html))(/.*)?/\.htaccess regular file       unconfined_u:object_r:httpd_user_htaccess_t:s0
/home/[^/]+/((www)|(web)|(public_html))(/.*)?/logs(/.*)? all files          unconfined_u:object_r:httpd_user_ra_content_t:s0
/home/[^/]+/((www)|(web)|(public_html))(/.+)?      all files          unconfined_u:object_r:httpd_user_content_t:s0
/home/[^/]+/((www)|(web)|(public_html))/cgi-bin(/.+)? all files          unconfined_u:object_r:httpd_user_script_exec_t:s0
/opt/.*\.cgi                                       regular file       system_u:object_r:httpd_sys_script_exec_t:s0
/opt/dirsrv/var/run/dirsrv/dsgw/cookies(/.*)?      all files          system_u:object_r:httpd_var_run_t:s0
/srv/([^/]*/)?www(/.*)?                            all files          system_u:object_r:httpd_sys_content_t:s0
/srv/([^/]*/)?www/logs(/.*)?                       all files          system_u:object_r:httpd_log_t:s0
/srv/gallery2(/.*)?                                all files          system_u:object_r:httpd_sys_content_t:s0
/srv/gallery2/smarty(/.*)?                         all files          system_u:object_r:httpd_sys_rw_content_t:s0
/usr/.*\.cgi                                       regular file       system_u:object_r:httpd_sys_script_exec_t:s0
/usr/bin/htsslpass                                 regular file       system_u:object_r:httpd_helper_exec_t:s0
/usr/bin/httpd                                     regular file       system_u:object_r:publicfile_exec_t:s0
/usr/bin/mongrel_rails                             regular file       system_u:object_r:httpd_exec_t:s0
/usr/lib(64)?/nagios/plugins/check_http            regular file       system_u:object_r:nagios_services_plugin_exec_t:s0
/usr/lib/apache(/.*)?                              all files          system_u:object_r:httpd_modules_t:s0
/usr/lib/apache(2)?/suexec(2)?                     regular file       system_u:object_r:httpd_suexec_exec_t:s0
/usr/lib/apache-ssl/.+                             regular file       system_u:object_r:httpd_exec_t:s0
/usr/lib/apache2/modules(/.*)?                     all files          system_u:object_r:httpd_modules_t:s0
/usr/lib/cgi-bin(/.*)?                             all files          system_u:object_r:httpd_sys_script_exec_t:s0
/usr/lib/cgi-bin/(nph-)?cgiwrap(d)?                regular file       system_u:object_r:httpd_suexec_exec_t:s0
/usr/lib/cherokee(/.*)?                            all files          system_u:object_r:httpd_modules_t:s0
/usr/lib/httpd(/.*)?                               all files          system_u:object_r:httpd_modules_t:s0
/usr/lib/lighttpd(/.*)?                            all files          system_u:object_r:httpd_modules_t:s0
/usr/lib/systemd/system/httpd.*                    regular file       system_u:object_r:httpd_unit_file_t:s0
/usr/lib/systemd/system/nginx.*                    regular file       system_u:object_r:httpd_unit_file_t:s0
/usr/lib/systemd/system/php-fpm.*                  regular file       system_u:object_r:httpd_unit_file_t:s0
/usr/lib/systemd/system/thttpd.*                   regular file       system_u:object_r:httpd_unit_file_t:s0
/usr/libexec/httpd-ssl-pass-dialog                 regular file       system_u:object_r:httpd_passwd_exec_t:s0
/usr/local/nagios/sbin(/.*)?                       all files          system_u:object_r:httpd_sys_script_exec_t:s0
/usr/sbin/apache(2)?                               regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/apache-ssl(2)?                           regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/apachectl                                regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/cherokee                                 regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/htcacheclean                             regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/httpd(\.worker)?                         regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/httpd\.event                             regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/lighttpd                                 regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/nginx                                    regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/php-fpm                                  regular file       system_u:object_r:httpd_exec_t:s0
/usr/sbin/rotatelogs                               regular file       system_u:object_r:httpd_rotatelogs_exec_t:s0
/usr/sbin/suexec                                   regular file       system_u:object_r:httpd_suexec_exec_t:s0
/usr/sbin/thttpd                                   regular file       system_u:object_r:httpd_exec_t:s0
/usr/share/doc/ghc/html(/.*)?                      all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/drupal.*                                all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/glpi(/.*)?                              all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/htdig(/.*)?                             all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/icecast(/.*)?                           all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/joomla(/.*)?                            all files          system_u:object_r:httpd_sys_rw_content_t:s0
/usr/share/munin/plugins/http_loadtime             regular file       system_u:object_r:services_munin_plugin_exec_t:s0
/usr/share/nginx/html(/.*)?                        all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/ntop/html(/.*)?                         all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/openca/htdocs(/.*)?                     all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/selinux-policy[^/]*/html(/.*)?          all files          system_u:object_r:httpd_sys_content_t:s0
/usr/share/system-config-httpd/system-config-httpd regular file       system_u:object_r:bin_t:s0
/usr/share/wordpress-mu/wp-config\.php             regular file       system_u:object_r:httpd_sys_script_exec_t:s0
/usr/share/wordpress-mu/wp-content(/.*)?           all files          system_u:object_r:httpd_sys_rw_content_t:s0
/usr/share/wordpress/.*\.php                       regular file       system_u:object_r:httpd_sys_script_exec_t:s0
/usr/share/wordpress/wp-content/upgrade(/.*)?      all files          system_u:object_r:httpd_sys_rw_content_t:s0
/usr/share/wordpress/wp-content/uploads(/.*)?      all files          system_u:object_r:httpd_sys_rw_content_t:s0
/usr/share/wordpress/wp-includes/.*\.php           regular file       system_u:object_r:httpd_sys_script_exec_t:s0
/usr/share/z-push(/.*)?                            all files          system_u:object_r:httpd_sys_content_t:s0
/var/cache/httpd(/.*)?                             all files          system_u:object_r:httpd_cache_t:s0
/var/cache/lighttpd(/.*)?                          all files          system_u:object_r:httpd_cache_t:s0
/var/cache/mason(/.*)?                             all files          system_u:object_r:httpd_cache_t:s0
/var/cache/mediawiki(/.*)?                         all files          system_u:object_r:httpd_cache_t:s0
/var/cache/mod_.*                                  all files          system_u:object_r:httpd_cache_t:s0
/var/cache/mod_gnutls(/.*)?                        all files          system_u:object_r:httpd_cache_t:s0
/var/cache/mod_proxy(/.*)?                         all files          system_u:object_r:httpd_cache_t:s0
/var/cache/mod_ssl(/.*)?                           all files          system_u:object_r:httpd_cache_t:s0
/var/cache/nginx(/.*)?                             all files          system_u:object_r:httpd_cache_t:s0
/var/cache/php-.*                                  all files          system_u:object_r:httpd_cache_t:s0
/var/cache/php-eaccelerator(/.*)?                  all files          system_u:object_r:httpd_cache_t:s0
/var/cache/php-mmcache(/.*)?                       all files          system_u:object_r:httpd_cache_t:s0
/var/cache/rt(3|4)(/.*)?                           all files          system_u:object_r:httpd_cache_t:s0
/var/cache/ssl.*\.sem                              regular file       system_u:object_r:httpd_cache_t:s0
/var/lib/cacti/rra(/.*)?                           all files          system_u:object_r:httpd_sys_content_t:s0
/var/lib/cherokee(/.*)?                            all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/dav(/.*)?                                 all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/dokuwiki(/.*)?                            all files          system_u:object_r:httpd_sys_rw_content_t:s0
/var/lib/drupal.*                                  all files          system_u:object_r:httpd_sys_rw_content_t:s0
/var/lib/ganglia(/.*)?                             all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/glpi(/.*)?                                all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/graphite-web(/.*)?                        all files          system_u:object_r:httpd_sys_rw_content_t:s0
/var/lib/htdig(/.*)?                               all files          system_u:object_r:httpd_sys_content_t:s0
/var/lib/httpd(/.*)?                               all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/ipsilon(/.*)?                             all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/lighttpd(/.*)?                            all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/mod_security(/.*)?                        all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/moodle(/.*)?                              all files          system_u:object_r:httpd_sys_rw_content_t:s0
/var/lib/nextcloud(/.*)?                           all files          system_u:object_r:httpd_sys_rw_content_t:s0
/var/lib/nginx(/.*)?                               all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/openshift/\.httpd\.d(/.*)?                all files          system_u:object_r:httpd_config_t:s0
/var/lib/openshift/\.log/httpd(/.*)?               all files          system_u:object_r:httpd_log_t:s0
/var/lib/owncloud(/.*)?                            all files          system_u:object_r:httpd_sys_rw_content_t:s0
/var/lib/php(/.*)?                                 all files          system_u:object_r:httpd_var_lib_t:s0
/var/lib/php/session(/.*)?                         all files          system_u:object_r:httpd_var_run_t:s0
/var/lib/php/wsdlcache(/.*)?                       all files          system_u:object_r:httpd_var_run_t:s0
/var/lib/phpMyAdmin(/.*)?                          all files     

[root@vm-rhel9 ~]# ls -lZ /var/www/html/
total 8
-rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0            0 Feb  9 10:30 file1
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0   0 Feb  9 10:33 file2
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 158 Feb  8 09:17 hosts
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0  11 Feb  8 09:09 index.html
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# restorecon -Rv /var/www/
Relabeled /var/www/html/file1 from unconfined_u:object_r:user_tmp_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
[root@vm-rhel9 ~]# ls -lZ /var/www/html/file*
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 0 Feb  9 10:30 /var/www/html/file1
-rw-r--r--. 1 root root unconfined_u:object_r:httpd_sys_content_t:s0 0 Feb  9 10:33 /var/www/html/file2
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# mkdir /virtual2
[root@vm-rhel9 ~]# touch /virtual2/index.html
[root@vm-rhel9 ~]# ls -Zd /virtual2/
unconfined_u:object_r:default_t:s0 /virtual2/
[root@vm-rhel9 ~]# ls -Z /virtual2
unconfined_u:object_r:default_t:s0 index.html
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# semanage fcontext -a -t httpd_sys_content_t '/virtual2(/.*)?'
[root@vm-rhel9 ~]# ls -Z /virtual2
unconfined_u:object_r:default_t:s0 index.html
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# restorecon -RFvv /virtual2/
Relabeled /virtual2 from unconfined_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
Relabeled /virtual2/index.html from unconfined_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# ls -Z /virtual2
system_u:object_r:httpd_sys_content_t:s0 index.html
[root@vm-rhel9 ~]#

[root@vm-rhel9 ~]# ls -Zd /virtual2
system_u:object_r:httpd_sys_content_t:s0 /virtual2
[root@vm-rhel9

[root@vm-rhel9 ~]# semanage fcontext -l -C
SELinux fcontext                                   type               Context

/virtual2(/.*)?                                    all files          system_u:object_r:httpd_sys_content_t:s0
[root@vm-rhel9 ~]#

</pre>

