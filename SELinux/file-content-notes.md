<pre>

Control SELinux File Contexts
Objectives
Manage the SELinux policy rules that determine the default context for files and directories with the semanage fcontext command and apply the context defined by the SELinux policy to files and directories with the restorecon command.

Initial SELinux Context
All resources, such as processes, files, and ports, are labeled with an SELinux context. SELinux maintains a file-based database of file labeling policies in the /etc/selinux/targeted/contexts/files/ directory. New files obtain a default label when their file name matches an existing labeling policy.

When a new file's name does not match an existing labeling policy, the file inherits the same label as the parent directory. With labeling inheritance, all files are always labeled when created, regardless of whether an explicit policy exists for a file.

When files are created in default locations that have an existing labeling policy, or when a policy exists for a custom location, then new files are labeled with a correct SELinux context. However, if a file is created in an unexpected location without an existing labeling policy, then the inherited label might not be correct for the new file's intended purpose.

Furthermore, copying a file to a new location can cause that file's SELinux context to change, where the new context is determined by the new location's labeling policy, or from parent directory inheritance if no policy exists. A file's SELinux context can be preserved during copying to retain the context label that was determined for the file's original location. For example, the cp -﻿p command preserves all file attributes where possible, and the cp --preserve=context command preserves only SELinux contexts, during copying.

NOTE
Copying a file always creates a file inode, and that inode's attributes, including the SELinux context, must be initially set, as previously discussed.

However, moving a file does not typically create an inode if the move occurs within the same file system, but instead moves the existing inode's file name to a new location. Because the existing inode's attributes do not need to be initialized, a file that is moved with mv preserves its SELinux context unless you set a new context on the file with the -Z option.

After you copy or move a file, verify that it has the appropriate SELinux context and set it correctly if necessary.

The following example demonstrates how this process works.

Create two files in the /tmp directory. Both files receive the user_tmp_t context type.

Move the first file, and copy the second file, to the /var/www/html directory.

The moved file retains the file context that was labeled from the original /tmp directory.

The copied file has a new inode and inherits the SELinux context from the destination /var/www/html directory.

The ls -Z command displays the SELinux context of a file. Observe the label of the files that are created in the /tmp directory.

[root@host ~]# touch /tmp/file1 /tmp/file2
[root@host ~]# ls -Z /tmp/file*
unconfined_u:object_r:user_tmp_t:s0 /tmp/file1
unconfined_u:object_r:user_tmp_t:s0 /tmp/file2
The ls -Zd command displays the SELinux context of the specified directory. Note the label on the /var/www/html directory and the files inside it.

[root@host ~]# ls -Zd /var/www/html/
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
[root@host ~]# ls -Z /var/www/html/index.html
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
Move one file from the /tmp directory to the /var/www/html directory. Copy the other file to the same directory. Note the resulting label on each file.

[root@host ~]# mv /tmp/file1 /var/www/html/
[root@host ~]# cp /tmp/file2 /var/www/html/
[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:user_tmp_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
The moved file retained its original label and the copied file inherited the destination directory label. The unconfined_u is the SELinux user role, object_r is the SELinux role, and s0 is the (lowest possible) sensitivity level. Advanced SELinux configurations and features use these values.

Change the SELinux Context
You can manage the SELinux context on files with the semanage fcontext, restorecon, and chcon commands.

The recommended method to change the context for a file is to create a file context policy by using the semanage fcontext command, and then to apply the specified context in the policy to the file by using the restorecon command. This method ensures that you can relabel the file to its correct context with the restorecon command whenever necessary. The advantage of this method is that you do not need to remember what the context is supposed to be, and you can correct the context on a set of files.

The chcon command changes the SELinux context directly on files, without referencing the system's SELinux policy. Although chcon is useful for testing and debugging, changing contexts manually with this method is temporary. Although file contexts that you can change manually survive a reboot, they might be replaced if you run restorecon to relabel the contents of the file system.

IMPORTANT
When an SELinux system relabel occurs, all files on a system are labeled with their policy defaults. When you use restorecon on a file, any context that you change manually on the file is replaced if it does not match the rules in the SELinux policy.

The following example creates a directory with a default_t SELinux context, which it inherited from the / parent directory.

[root@host ~]# mkdir /virtual
[root@host ~]# ls -Zd /virtual
unconfined_u:object_r:default_t:s0 /virtual
The chcon command sets the file context of the /virtual directory to the httpd_sys_content_t type.

[root@host ~]# chcon -t httpd_sys_content_t /virtual
[root@host ~]# ls -Zd /virtual
unconfined_u:object_r:httpd_sys_content_t:s0 /virtual
Running the restorecon command resets the context to the default value of default_t. Note the Relabeled message.

[root@host ~]# restorecon -v /virtual
Relabeled /virtual from unconfined_u:object_r:httpd_sys_content_t:s0 to unconfined_u:object_r:default_t:s0
[root@host ~]# ls -Zd /virtual
unconfined_u:object_r:default_t:s0 /virtual
Define SELinux Default File Context Policies
The semanage fcontext command displays and modifies the policies that determine the default file contexts. You can list all the file context policy rules by running the semanage fcontext -l command. These rules use extended regular expression syntax to specify the path and file names.

When viewing policies, the most common extended regular expression is (/.*)?, which is usually appended to a directory name. This notation is humorously called the pirate, because it looks like a face with an eye patch and a hooked hand next to it.

This syntax is described as "a set of characters that begin with a slash and followed by any number of characters, where the set can either exist or not exist". Stated more simply, this syntax matches the directory itself, even when empty, and also matches almost any file name that is created within that directory.

For example, the following rule specifies that the /var/www/cgi-bin directory, and any files in it or in its subdirectories (and in their subdirectories, and so on), have the system_u:object_r:httpd_sys_script_exec_t:s0 SELinux context, unless a more specific rule overrides this one.

/var/www/cgi-bin(/.*)?  all files  system_u:object_r:httpd_sys_script_exec_t:s0
NOTE
The all files field option from the previous example is the default file type that semanage uses when you do not specify one. This option applies to all file types that you can use with semanage; they are the same as the standard file types as in the Control Access to Files chapter in the Red Hat System Administration I (RH124) course. You can get more information from the semanage-fcontext(8) man page.

Basic File Context Operations
The following table is a reference for the semanage fcontext command options to add, remove, or list SELinux file context policies.

Table 5.1. The semanage fcontext Command

Option	Description
-a, --add	Add a record of the specified object type.
-d, --delete	Delete a record of the specified object type.
-l, --list	List records of the specified object type.

To manage SELinux contexts, install the policycoreutils and policycoreutils-python-utils packages, which contain the restorecon and semanage commands.

To reset all files in a directory to the default policy context, first use the semanage fcontext -l command to locate and verify that the correct policy exists with the intended file context. Then, use the restorecon command on the wildcarded directory name to reset all the files recursively. In the following example, view the file contexts before and after using the semanage and restorecon commands.

First, verify the SELinux context for the files:

[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:user_tmp_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
Then, use the semanage fcontext -l command to list the default SELinux file contexts:

[root@host ~]# semanage fcontext -l
...output omitted...
/var/www(/.*)?       all files    system_u:object_r:httpd_sys_content_t:s0
...output omitted...
The semanage command output indicates that all the files and subdirectories in the /var/www/ directory have the httpd_sys_content_t context by default. Running restorecon command on the wildcarded directory restores the default context on all files and subdirectories.

[root@host ~]# restorecon -Rv /var/www/
Relabeled /var/www/html/file1 from unconfined_u:object_r:user_tmp_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
[root@host ~]# ls -Z /var/www/html/file*
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file1
unconfined_u:object_r:httpd_sys_content_t:s0 /var/www/html/file2
The following example uses the semanage command to add a context policy for a new directory. First, create the /virtual directory with an index.html file inside it. View the SELinux context for the file and the directory.

[root@host ~]# mkdir /virtual
[root@host ~]# touch /virtual/index.html
[root@host ~]# ls -Zd /virtual/
unconfined_u:object_r:default_t:s0 /virtual
[root@host ~]# ls -Z /virtual/
unconfined_u:object_r:default_t:s0 index.html
Next, use the semanage fcontext command to add an SELinux file context policy for the directory.

[root@host ~]# semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
Use the restorecon command on the wildcarded directory to set the default context on the directory and all files within it.

[root@host ~]# restorecon -RFvv /virtual
Relabeled /virtual from unconfined_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
Relabeled /virtual/index.html from unconfined_u:object_r:default_t:s0 to system_u:object_r:httpd_sys_content_t:s0
[root@host ~]# ls -Zd /virtual/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /virtual/
[root@host ~]# ls -Z /virtual/
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html
Use the semanage fcontext -l -C command to view any local customizations to the default policy.

[root@host ~]# semanage fcontext -l -C
SELinux fcontext     type         Context

/virtual(/.*)?       all files    system_u:object_r:httpd_sys_content_t:s0











</pre>
