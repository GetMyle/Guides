### XFS based quota on a directory in Linux

This guide is based on [XFS quota management](https://solidlinux.wordpress.com/2012/12/09/xfs-quota-managament/) document.
It seems to be the most convinient way to restrict an available space in a directory in the Linux file system.
The other way how to achieve the same goal will be mentioned at the end of this document.

---

_XFS_ file system has the feature to create a quota per project that can be used to limit available space in a directory. In CentOS 6 _XFS_ is fully supported, but root file system is _ext4_. In CentOS 7 XFS is a default file system, and supported as a root file system.
By default it is mounted without the quota enabled. First, we need to mount _XFS_ with the enable quota.
In the example below I use the mount point /home mounted to /
In the file /etc/fstab the record to mount _XFS_ with the project's quota enebled should look like that:

`
/dev/mapper/vg_01-lv_home /home                   xfs     rw,prjquota        1 2
`

Online remount like

`
    #mount -o remount,pquota /home
`

may not always work. It is better to modify /etc/fstab, and then execute 

```
    #umount /home
    #mount /home
```
The result of the command

`
    #mount
`

for /home should be like 

> /dev/sde1 on /home type xfs (rw,**prjquota**)

We should see **prjquota** in the mounting options.

Let's create the test directories for user *myle*.
```
    #mkdir /home/pq_test
    #mkdir /home/pq_test/01
    #mkdir /home/pq_test/02
    #chmod 755 /home/pq_test/*
    #chown myle:myle /home/pq_test/*
```

Now we will create and edit two files used by XFS to find the _projects_ quota, and to map these _projects_ named 01 and 02 on the directories.

```
    #touch /etc/projects
    #touch /etc/projid
    #echo 01:/home/pq_test/01 >> /etc/projects
    #echo 02:/home/pq_test/02 >> /etc/projects
    #echo 01:01 >> /etc/projid
    #echo 02:02 >> /etc/projid
```

The next step is to create quotas for these two directories

```
    #xfs_quota -x -c 'project -s 01' /home
    #xfs_quota -x -c 'project -s 02' /home
    #xfs_quota -x -c 'limit -p bhard=10m 01' /home
    #xfs_quota -x -c 'limit -p bhard=10m 02' /home
```

The result of the command

`
    #xfs_quota -x -c 'report' /home
`

should be like 

>Project quota on /home (/dev/sde1)
>                               Blocks
>Project ID       Used       Soft       Hard    Warn/Grace
>    ---------- --------------------------------------------------
>
>01                  0          0      10240     00 [--------]
>
>02                  0          0      10240     00 [--------]


And the result of the command

`
    #df -h /home/pq_test/01
`

should be like 

>Filesystem            Size  Used Avail Use% Mounted on
>
>/dev/sde1              **10M**     0   10M   0% /home
>

So now we have limited the *size* of /home/pq_test/01 to 10 Mb.
It should be noted that it doesn't limit a user only to this directory. This solution protect the _managed_ code in the directory /home/pq_test/01 from accidentaly using more than 10 Mb, but if a user can change to some other directory where there is no quota then the user will not be the space limited.
The other solutions to limit the size of a directory involves the real file system mounting to every directory. We can or format the file with the file system (virtual file system), or use LVM, but it is more cumbersom solution.

[Implementing Disk Quotas on Linux](http://souptonuts.sourceforge.net/quota_tutorial.html)

We also can combine the qoutas per a directory with the quotas per user if we need more sophisticated qouta management.


