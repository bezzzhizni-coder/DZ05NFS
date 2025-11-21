# DZ05NFS
> Основная часть:  
запустить 2 виртуальных машины (сервер NFS и клиента);  
на сервере NFS должна быть подготовлена и экспортирована директория;   
в экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё;   
экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);  
монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3.  
Для самостоятельной реализации:   
настроить аутентификацию через KERBEROS с использованием NFSv4.  
```
root@testsrv:/home/gor# rpcinfo | grep nfs
    100003    3    tcp       0.0.0.0.8.1            nfs        superuser
    100003    4    tcp       0.0.0.0.8.1            nfs        superuser
    100227    3    tcp       0.0.0.0.8.1            nfs_acl    superuser
    100003    3    tcp6      ::.8.1                 nfs        superuser
    100003    4    tcp6      ::.8.1                 nfs        superuser
    100227    3    tcp6      ::.8.1                 nfs_acl    superuser

root@testsrv:/home/gor# mkdir -p /pool01/share/upload
root@testsrv:/home/gor# chown -R nobody:nogroup /pool01/share/
root@testsrv:/home/gor# chmod 0777 /pool01/share/upload/
root@testsrv:/home/gor# cat << EOF > /etc/exports
> /pool01/share/ 172.16.100.98/32(rw,sync,root_squash)
> EOF
root@testsrv:/home/gor# exportfs -r
exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "172.16.100.98/32:/pool01/share/".
  Assuming default behaviour ('no_subtree_check').
  NOTE: this default has changed since nfs-utils version 1.0.x

/pool01/share/ 172.16.100.0/24(rw,sync,root_squash,no_subtree_check)
root@testsrv:/pool01/share# exportfs -a
```
```
root@testsrv2:/home/gor# apt install nfs-common

/etc/fstab
172.16.100.98:/pool01/share /mnt nfs vers=3,noauto,x-systemd.automount 0 0

root@testsrv2:/home/gor# mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=67,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=4255)
root@testsrv2:/home/gor# mount -t nfs 172.16.100.98:/pool01/share /mnt/
root@testsrv2:/home/gor# mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=67,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=4255)
172.16.100.98:/pool01/share on /mnt type nfs (rw,relatime,vers=3,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=172.16.100.98,mountvers=3,mountport=36648,mountproto=udp,local_lock=none,addr=172.16.100.98)
172.16.100.98:/pool01/share on /mnt type nfs4 (rw,relatime,vers=4.2,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=172.16.100.167,local_lock=none,addr=172.16.100.98)

/etc/fstab
172.16.100.98:/pool01/share /mnt nfs vers=3,auto 0 0

root@testsrv2:/home/gor# systemctl daemon-reload
root@testsrv2:/home/gor# systemctl restart remote-fs.target
root@testsrv2:/home/gor# mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=67,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=4255)
172.16.100.98:/pool01/share on /mnt type nfs (rw,relatime,vers=3,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=172.16.100.98,mountvers=3,mountport=36648,mountproto=udp,local_lock=none,addr=172.16.100.98)
```

