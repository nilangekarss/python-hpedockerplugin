# Default groups and users in persona configuration:

# CSSOS-SSA06 cli% showfsuser
```
Username	       UID           ---------------------SID---------------------     Primary_Group       Enabled
Administrator   10500           S-1-5-21-3407317619-3829948340-1570492076-500       Local Users         false
Guest  	        10501           S-1-5-21-3407317619-3829948340-1570492076-501       Local Users         false
-----------------------------------------------------------------------------------------------------------------------------------------------------
            2 total
```

# CSSOS-SSA06 cli% showfsgroup
```
GroupName            GID       ---------------------SID---------------------			Group Members	
Local Users          10800     S-1-5-21-3407317619-3829948340-1570492076-800			Administrator,	Guest
Administrators       10544     S-1-5-32-544			                                  Administrator	
Users                10545     S-1-5-32-545			                                  Administrator	
Guests               10546     S-1-5-32-546			                                  Guest	
Backup Operators     10551     S-1-5-32-551			                                  --	
-----------------------------------------------------------------------------------------------------------------------------------------------------
            5 total
```			

# createfshare:

# SYNTAX
```
    createfshare {smb|nfs|obj|ftp} [options <arg>] <vfs> <sharename>
```

# OPTIONS
    The following options are for all subcommands:

    -fpg <fpgname>
        Specifies the file provisioning group (FPG) that <vfs> belongs.
        If this is not specified, the command will find out the FPG based
        on the specified <vfs>. However, if <vfs> exists under multiple
        FPGs, -fpg must be specified.

    -fstore <fstore>
        Specifies the file store under which the share will be created. If this
        is not specified, the command uses the <sharename> as the file store
        name. The file store will be created if it does not exist. If you
        specify this option to create a file share, you will have to specify it
        when you set or remove the share using setfshare/removefshare.

    -sharedir <sharedir>
        Specifies the directory path to share. It can be a full path starting
        from "/", or a relative path under the file store. If this is not
        specified, the share created will be rooted at the file store. If this
        option is specified, option -fstore must be specified.

    -comment <comment>
        Specifies any comments or additional information for the share. The
        comment can be up to 255 characters long. Unprintable characters are
        not allowed.

    -f
        Specifies that the command is forced. When creating a share of a
        second protocol type for a given file store, if this option is not used,
        the command requires confirmation before proceeding with its
        operation.


# nfs
        -options <options>
            Specifies options to use for the share to be created. Standard
            NFS export options except "no_subtree_check" are supported.
            Do not enter option "fsid", which is provided. If not specified,
            the following options will be automatically set:
            sync, auth_nlm, wdelay, sec=sys, no_all_squash, crossmnt, secure,
            subtree_check, hide, root_squash, ro.

            See linux exports(5) man page for detailed information.

        -audit {operation1:value1[,operation2:value2]...}
            Following operations can be audited on a share level for NFS
            protocol on specified VFS.
            Acceptable value is either on or off. Default value is "off".
            Note: Operations are case insensitive.
            Below operations will generate event when:
            ------------------------------------------
            audit_open:  Open a file. This is NFSv4 only.
            audit_close: Close a file. This is NFSv4 only.
            audit_meta:  Rename a file or directory, creating a hardlink for a
                         file, creating a file or directory and removing a
                         file or a directory.
            audit_attr:  Changing file/directory attributes.
            audit_read:  Read of a file/directory. Subsequent file reads are
                         suppressed for a specific time (currently 10m).
                         Note, reads from a directory are never suppressed.
            audit_write: Write to a file. Subsequent writes are suppressed for a
                         specific time (currently 10m).

            For more details, please refer to the File Persona User Guide.

        -clientip <clientlist>
            Specifies the clients that can access the share. The NFS client
            can be specified by the name (for example, sys1.example.com), the
            name with a wildcard (for example, *.example.com), or by its IP
            address. Use comma to separate the IP addresses. If this is not
            specified, the default is "*".

# Note:
# When a share is created, it will have following permissions and owners by default:

```
CSSOS-SSA06 cli% showfshare nfs -d swapshare2
Share Name              : swapshare2
File Provisioning Group : swap_fpg2
Virtual File Server     : vfs_fpg2
File Store              : fpg2_fstore1
Share Directory         : swapdir2
Full Directory Path     : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir2
State                   : normal
Clients                 : *
Options                 : ro, no_all_squash, secure, subtree_check, root_squash, auth_nlm, sec=sys, crossmnt, wdelay, hide, sync
Comment                 : ---

-File access audit NFS options-
audit_close:off audit_open:off
audit_acl:off   audit_meta:off
audit_read:off  audit_write:off
audit_attr:off


CSSOS-SSA06 cli% showfshare nfs -dirperm -fstore fpg2_fstore1 -vfs vfs_fpg2 swapshare2
Share Name              : swapshare2
Sharepath               : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir2
Owner                   : root
Group                   : Administrators@B-LOCAL_CLUSTER
Modebits                : 770
---------------ACL----------------
Type Flags Principal Permissions
A          OWNER@    rwaDxtTnNcCoy
A    g     GROUP@    rwaDxtTnNcy
A          EVERYONE@ tcy

```
# Note: When Options are set to root_squash, as a root user when we mount swapshare2 to /opt/data, we dont have root privillages and we get following error if we try to enter the mounted share
```
Ex:
mount -t nfs 192.168.98.2:/swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir2 /opt/data
root@COSSOSBE03-B08:/opt# cd data/
-bash: cd: data/: Permission denied
root@COSSOSBE03-B08:/opt# ls -ltr
total 12
drwxr-xr-x 3 root root  4096 Mar  5 05:43 hpe
drwxrwx--- 2 root 10544 8192 Mar  5 23:43 data

==========================================================================================================================================

Same is the case when we have rw access to the share and Options has root_squash set

Ex:
CSSOS-SSA06 cli% createfshare nfs -fpg swap_fpg2 -fstore fpg2_fstore1 -sharedir swapdir3 -options rw -f vfs_fpg2 swapshare3
CSSOS-SSA06 cli% showfshare nfs -dirperm -fstore fpg2_fstore1 -vfs vfs_fpg2 swapshare3
Share Name              : swapshare3
Sharepath               : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir3
Owner                   : root
Group                   : Administrators@B-LOCAL_CLUSTER
Modebits                : 770
---------------ACL----------------
Type Flags Principal Permissions
A          OWNER@    rwaDxtTnNcCoy
A    g     GROUP@    rwaDxtTnNcy
A          EVERYONE@ tcy

CSSOS-SSA06 cli% showfshare nfs -d swapshare3
Share Name              : swapshare3
File Provisioning Group : swap_fpg2
Virtual File Server     : vfs_fpg2
File Store              : fpg2_fstore1
Share Directory         : swapdir3
Full Directory Path     : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir3
State                   : normal
Clients                 : *
Options                 : rw, no_all_squash, secure, subtree_check, root_squash, auth_nlm, sec=sys, crossmnt, wdelay, hide, sync
Comment                 : ---

-File access audit NFS options-
audit_close:off audit_open:off
audit_acl:off   audit_meta:off
audit_read:off  audit_write:off
audit_attr:off


root@COSSOSBE03-B08:/opt# mount -t nfs 192.168.98.2:/swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir3 /opt/data3
root@COSSOSBE03-B08:/opt# cd data3/
-bash: cd: data3/: Permission denied

========================================================================================================================
```
# When we create a share with no_root_squash, root has privillages of root:
```
CSSOS-SSA06 cli% createfshare nfs -fpg swap_fpg2 -fstore fpg2_fstore1 -sharedir swapdir4 -options rw,no_root_squash -f vfs_fpg2 swapshare4
CSSOS-SSA06 cli% showfshare nfs -d swapshare4
Share Name              : swapshare4
File Provisioning Group : swap_fpg2
Virtual File Server     : vfs_fpg2
File Store              : fpg2_fstore1
Share Directory         : swapdir4
Full Directory Path     : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir4
State                   : normal
Clients                 : *
Options                 : rw, no_all_squash, secure, no_root_squash, subtree_check, auth_nlm, sec=sys, crossmnt, wdelay, hide, sync
Comment                 : ---

-File access audit NFS options-
audit_close:off audit_open:off
audit_acl:off   audit_meta:off
audit_read:off  audit_write:off
audit_attr:off

CSSOS-SSA06 cli% showfshare nfs -dirperm -fstore fpg2_fstore1 -vfs vfs_fpg2 swapshare4
Share Name              : swapshare4
Sharepath               : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir4
Owner                   : root
Group                   : Administrators@B-LOCAL_CLUSTER
Modebits                : 770
---------------ACL----------------
Type Flags Principal Permissions
A          OWNER@    rwaDxtTnNcCoy
A    g     GROUP@    rwaDxtTnNcy
A          EVERYONE@ tcy

Ex:
root@COSSOSBE03-B08:/opt# mount -t nfs 192.168.98.2:/swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir4 /opt/data4
root@COSSOSBE03-B08:/opt# ls -ltr
total 28
drwxr-xr-x 3 root root  4096 Mar  5 05:43 hpe
drwxrwx--- 2 root 10544 8192 Mar  5 23:43 data
drwxrwx--- 2 root 10544 8192 Mar  6  2019 data3
drwxrwx--- 2 root 10544 8192 Mar  6  2019 data4
root@COSSOSBE03-B08:/opt# cd data4/
root@COSSOSBE03-B08:/opt/data4# id
uid=0(root) gid=0(root) groups=0(root)
root@COSSOSBE03-B08:/opt/data4#

=====================================================================================================================
```


# Points on chaning ownership and ACLs:

# When a share is created it belongs to owner root, group Administrators and mode bits are set to 770
```
Ex:
CSSOS-SSA06 cli% showfshare nfs -dirperm -fstore fpg2_fstore1 -vfs vfs_fpg2 swapshare4
Share Name              : swapshare4
Sharepath               : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir4
Owner                   : root
Group                   : Administrators@B-LOCAL_CLUSTER
Modebits                : 770
---------------ACL----------------
Type Flags Principal Permissions
A          OWNER@    rwaDxtTnNcCoy
A    g     GROUP@    rwaDxtTnNcy
A          EVERYONE@ tcy


```

As shown above each mode bit is mapped to some permissions.
Here is the table for the same:

# Put the table

Approaches we can take for changing owners and permissions of share:

1. Using 3par commands: when we want to change owner/group/permissions we can use setfshare command
```
Ex:
CSSOS-SSA06 cli% setfshare nfs -owner docker -group docker -f -fpg swap_fpg2 -fstore fpg2_fstore1 vfs_fpg2 swapshare2
User not found
CSSOS-SSA06 cli% setfshare nfs -owner abc -group docker -f -fpg swap_fpg2 -fstore fpg2_fstore1 vfs_fpg2 swapshare2
Group not found

1. Now create user abc with id 1000 and make it a part of Local Users group using ssmc 
2. Create group docker with id 1000 

Now users and groups have following info:

CSSOS-SSA06 cli% showfsuser
Username        UID ---------------------SID---------------------- Primary_Group Enabled
Administrator 10500 S-1-5-21-3407317619-3829948340-1570492076-500  Local Users   false
Guest         10501 S-1-5-21-3407317619-3829948340-1570492076-501  Local Users   false
abc            1000 S-1-5-21-3407317619-3829948340-1570492076-5009 Local Users   false
----------------------------------------------------------------------------------------
            3 total
CSSOS-SSA06 cli% showfgroup
invalid command name "showfgroup"
CSSOS-SSA06 cli% showfsgroup
GroupName          GID ---------------------SID----------------------
Local Users      10800 S-1-5-21-3407317619-3829948340-1570492076-800
Administrators   10544 S-1-5-32-544
Users            10545 S-1-5-32-545
Guests           10546 S-1-5-32-546
Backup Operators 10551 S-1-5-32-551
docker            1000 S-1-5-21-3407317619-3829948340-1570492076-5005
---------------------------------------------------------------------
               6 total
CSSOS-SSA06 cli% showfsgroup -d Administrators
Groupname     : Administrators
GID           : 10544
SID           : S-1-5-32-544
Group members : Administrator, abc

CSSOS-SSA06 cli% showfsuser -d abc
UserName         : abc
UID              : 1000
SID              : S-1-5-21-3407317619-3829948340-1570492076-5009
Primary Group    : Local Users
Secondary Groups : Administrators
Enabled          : false


CSSOS-SSA06 cli% setfshare nfs -owner 1000 -group 1000 -f -fpg swap_fpg2 -fstore fpg2_fstore1 vfs_fpg2 swapshare2
User not found
```

Note: setfshare only works with username/groupname but not with uid/gid
Here is the example of setfshare command:

```
CSSOS-SSA06 cli% showfsuser
Username        UID ---------------------SID---------------------- Primary_Group Enabled
Administrator 10500 S-1-5-21-3407317619-3829948340-1570492076-500  Local Users   false
Guest         10501 S-1-5-21-3407317619-3829948340-1570492076-501  Local Users   false
abc            1000 S-1-5-21-3407317619-3829948340-1570492076-5009 Local Users   true
----------------------------------------------------------------------------------------
            3 total
CSSOS-SSA06 cli% showfsgroup
GroupName          GID ---------------------SID----------------------
Local Users      10800 S-1-5-21-3407317619-3829948340-1570492076-800
Administrators   10544 S-1-5-32-544
Users            10545 S-1-5-32-545
Guests           10546 S-1-5-32-546
Backup Operators 10551 S-1-5-32-551
docker            1000 S-1-5-21-3407317619-3829948340-1570492076-5010
---------------------------------------------------------------------
               6 total
CSSOS-SSA06 cli% showfshare nfs -dirperm swapshare2
Option -dirperm requires -fstore, -vfs and share name.
CSSOS-SSA06 cli% showfshare nfs -dirperm -fstore fpg2_fstore1 -vfs vfs_fpg2 swapshare2
Share Name              : swapshare2
Sharepath               : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir2
Owner                   : root
Group                   : Administrators@B-LOCAL_CLUSTER
Modebits                : 770
---------------ACL----------------
Type Flags Principal Permissions
A          OWNER@    rwaDxtTnNcCoy
A    g     GROUP@    rwaDxtTnNcy
A          EVERYONE@ tcy

CSSOS-SSA06 cli% setfshare nfs -owner abc -group docker -f -fpg swap_fpg2 -fstore fpg2_fstore1 vfs_fpg2 swapshare2
CSSOS-SSA06 cli% showfshare nfs -dirperm -fstore fpg2_fstore1 -vfs vfs_fpg2 swapshare2
Share Name              : swapshare2
Sharepath               : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir2
Owner                   : abc@LOCAL_CLUSTER
Group                   : docker@LOCAL_CLUSTER
Modebits                : 770
---------------ACL----------------
Type Flags Principal Permissions
A          OWNER@    rwaDxtTnNcCoy
A    g     GROUP@    rwaDxtTnNcy
A          EVERYONE@ tcy

```
2. Use chown and chmode commands on mounted sharedirectory

```
root@COSSOSBE03-B08:/opt# mount -t nfs 192.168.98.2:/swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir4 /opt/data4
root@COSSOSBE03-B08:/opt# ls -ltr
total 28
drwxr-xr-x 3 root root  4096 Mar  5 05:43 hpe
drwxrwx--- 2 root 10544 8192 Mar  5 23:43 data
drwxrwx--- 2 root 10544 8192 Mar  6  2019 data3
drwxrwx--- 2 root 10544 8192 Mar  6  2019 data4
root@COSSOSBE03-B08:/opt# cd data4/
root@COSSOSBE03-B08:/opt/data4# id
uid=0(root) gid=0(root) groups=0(root)

root@COSSOSBE03-B08:/opt/data4# chown 1000:1000  -R /opt/data4

```
This will change the owner and group to abc and docker at 3par file persona

Now, user administrator:administrator having id 1000:1000 will have access to this share on mounted directory:
```
administrator@COSSOSBE03-B08:/opt$ id
uid=1000(administrator) gid=1000(administrator) groups=1000(administrator),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(sambashare),126(libvirtd),127(lpadmin)

administrator@COSSOSBE03-B08:/opt$ ls -ltr /opt
total 28
drwxr-xr-x 3 root          root          4096 Mar  5 05:43 hpe
drwxrwx--- 2 root                  10544 8192 Mar  5 23:43 data
drwxrwx--- 2 root                  10544 8192 Mar  6 02:25 data3
drwxrwx--- 2 administrator administrator 8192 Mar  6 02:28 data4

```
On 3par side the owner and group information for this share:

```
CSSOS-SSA06 cli%  showfshare nfs -dirperm -fstore fpg2_fstore1 -vfs vfs_fpg2 swapshare4
Share Name              : swapshare4
Sharepath               : /swap_fpg2/vfs_fpg2/fpg2_fstore1/swapdir4
Owner                   : abc@LOCAL_CLUSTER
Group                   : docker@LOCAL_CLUSTER
Modebits                : 770
---------------ACL----------------
Type Flags Principal Permissions
A          OWNER@    rwaDxtTnNcCoy
A    g     GROUP@    rwaDxtTnNcy
A          EVERYONE@ tcy
```

# Caution:
When we use chown and chmode to change owners and modes of share, eventhough user and group with these ids doesn't exist on 3PAR 
owners and permissions of this particular share will get changed but whereas in case of setfshare command execution on 3PAR file share 
ownership and permissions will not get changed if respective owner and group doesn't exist on 3PAR.



My Take would be use chown and chmode instead of setfshare only if passed ids exist on 3par (by checking existing fsusers and fsgroups)


# Note:

If we use setfshare we can use ACL permissions but it will increase the number of options to be passed to the create command:

Ex:

1. If we want to change ownership of share and mode permissions of share1 withought using setfshare we can use following docker create command:
```
docker volume create -d hpe --name share1 -o fsOwner="1000:1000" fsMode="754"
and 
$ docker run -it -v VOLUME:/dir --rm --user 1000:1000 --volume-driver hpe busybox /bin/sh
```

If ACLs needs to be used for providing access of a share to a particular owner/group with specific permissions then docker command create
command will be as shown below:
```
docker volume create -d hpe --name share1 -o fsOwner="1000:1000" fsMode="type:flag:principal:permissions"
where each filed can have values as shown below:
type        A           D         U         L
meaning   allow     delete      audit     alarm
          
flag        f                d                     p                      i               s                   F                 g
meaning   file-inherit     directory-inherit    no-propagate-inherit    inherit-only    successful-access    failed-access    group
          
principal   
          OWNER@      GROUP@      EVERYONE@
          
permissions r                     w                  a                        x           d         D                   t             T             n                 N                   c           C           o             y
            read-data             write-data         append-data              execute     delete    delete-child        read-attrs    write-attrs   read-named-attrs  write-named-attrs   read-ACL    write-ACL   write-owner   synchronize
            or list-directory     or create-file     or create-subdirectory                         (directories only)
            

Ex:
A:fd:OWNER@:rwax,A:fdg:GROUP@:rwax
```






