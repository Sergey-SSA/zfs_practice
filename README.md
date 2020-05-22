# zfs_practice
Установил ZFS на CentOS Linux release 7.8.2003 (Core)
Добавил официальный репозиторий

`sudo yum install http://download.zfsonlinux.org/epel/zfs-release.el7_8.noarch.rpm`

```
Total size: 2.9 k
Installed size: 2.9 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : zfs-release-1-7.8.noarch                                                                                                                                                                                                  1/1
  Verifying  : zfs-release-1-7.8.noarch                                                                                                                                                                                                  1/1

Installed:
  zfs-release.noarch 0:1-7.8

Complete!
```

Отключил репозиторий ZFS на основе DKMS и включил репозиторий ZFS.
Открыл конфигурационный файл yum ZFS

`sudo nano /etc/yum.repos.d/zfs.repo`

выключил ZFS на основе DKMS и включил ZFS-kmod

```
[zfs]
name=ZFS on Linux for EL7 - dkms
baseurl=http://download.zfsonlinux.org/epel/7.8/$basearch/
**enabled=0**
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux

[zfs-kmod]
name=ZFS on Linux for EL7 - kmod
baseurl=http://download.zfsonlinux.org/epel/7.8/kmod/$basearch/
**enabled=1**
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
```

и установил ZFS

`sudo yum install zfs`

```
Installed:
  zfs.x86_64 0:0.8.4-1.el7

Dependency Installed:
  kmod-zfs.x86_64 0:0.8.4-1.el7      libnvpair1.x86_64 0:0.8.4-1.el7     libuutil1.x86_64 0:0.8.4-1.el7     libzfs2.x86_64 0:0.8.4-1.el7     libzpool2.x86_64 0:0.8.4-1.el7     lm_sensors-libs.x86_64 0:3.4.0-8.20160601gitf9185e5.el7
  sysstat.x86_64 0:10.1.5-19.el7

Complete!
```

После проверки проверил установку

`sudo lsmod | grep zfs`

Вывода небыло.
Загрузил модуль ядра вручную

`sudo modprobe zfs`

и ещё раз проверил

```
zfs                  3986613  0
zunicode              331170  1 zfs
zlua                  147429  1 zfs
zcommon                89551  1 zfs
znvpair                94388  2 zfs,zcommon
zavl                   15167  1 zfs
icp                   301854  1 zfs
spl                   104299  5 icp,zfs,zavl,zcommon,znvpair
```

Всё готово можно работать с ZFS.
Проверил версию ZFS

`zfs version`

```
zfs-0.8.4-1
zfs-kmod-0.8.4-1
```

Задача - ○ создать 4 файловых системы на каждой применить свой алгоритм сжатия
Создал пул на диске sdb

`sudo zpool create pool1 sdb`

проверил

`sudo zpool list`

```
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
pool1  4.50G   212K  4.50G        -         -     0%     0%  1.00x    ONLINE  -
```

На созданном пуле создал 4 файловые системы

`sudo zfs create pool1/data1`

`sudo zfs create pool1/data2`

`sudo zfs create pool1/data3`

`sudo zfs create pool1/data4`

Проверил все точки монтирования

`df -h`

```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        489M     0  489M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M  6.7M  489M   2% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
/dev/sda1        40G   13G   28G  33% /
tmpfs           100M     0  100M   0% /run/user/1000
pool1           4.4G  128K  4.4G   1% /pool1
pool1/data1     4.4G  128K  4.4G   1% /pool1/data1
pool1/data2     4.4G  128K  4.4G   1% /pool1/data2
pool1/data3     4.4G  128K  4.4G   1% /pool1/data3
pool1/data4     4.4G  128K  4.4G   1% /pool1/data4
```

Для созданных файловых систем установил типы сжатий

`sudo zfs set compression=gzip-9 pool1/data1`

`sudo zfs set compression=zle pool1/data2`

`sudo zfs set compression=lzjb pool1/data3`

`sudo zfs set compression=lz4 pool1/data4`

Скачал тестовый файл

`wget -O War_and_Peace.txt http://www.gutenberg.org/ebooks/2600.txt.utf-8`

И разместил его на файловых системах

`sudo cp War_and_Peace.txt /pool1/data1`

`sudo cp War_and_Peace.txt /pool1/data2`

`sudo cp War_and_Peace.txt /pool1/data3`

`sudo cp War_and_Peace.txt /pool1/data4`

**Вывод размера файла**

`sudo du -k /pool1/data{1..4}/War_and_Peace.txt`

```
1185    /pool1/data1/War_and_Peace.txt
1186    /pool1/data2/War_and_Peace.txt
1192    /pool1/data3/War_and_Peace.txt
1186    /pool1/data4/War_and_Peace.txt
```
_Как видно выше в данном случае сжатие лучше работает на файловых системах gzip-9 и lz4_

**Следующей командой получил степерь сжатия и тип**

`zfs get compression,compressratio`

```
NAME         PROPERTY       VALUE     SOURCE
pool1        compression    off       default
pool1        compressratio  1.08x     -
pool1/data1  compression    gzip-9    local
pool1/data1  compressratio  1.08x     -
pool1/data2  compression    zle       local
pool1/data2  compressratio  1.08x     -
pool1/data3  compression    lzjb      local
pool1/data3  compressratio  1.07x     -
pool1/data4  compression    lz4       local
pool1/data4  compressratio  1.08x     -
```

Следующая задача - Определить настройки pool’a
Загрузить архив с файлами, распокавать и собрать pool ZFS.
Восстановил пул

`wget --no-check-certificate -O file.tar.gz 'https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download'`

`tar -xvf file.tar.gz`

```
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
```

`cd zpoolexport`

`ls -l`

>-rw-r--r--. 1 vagrant vagrant 524288000 May 15 05:00 filea
>-rw-r--r--. 1 vagrant vagrant 524288000 May 15 05:00 fileb

`mkdir recovery`

`sudo zpool import -d filea`

```
   pool: otus
     id: 6554193320433390805
  state: DEGRADED
 status: One or more devices are missing from the system.
 action: The pool can be imported despite missing or damaged devices.  The
        fault tolerance of the pool may be compromised if imported.
   see: http://zfsonlinux.org/msg/ZFS-8000-2Q
 config:

        otus                                 DEGRADED
          mirror-0                           DEGRADED
            /home/vagrant/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb          UNAVAIL  cannot open
```

`sudo zpool import -d fileb`

```
   pool: otus
     id: 6554193320433390805
  state: DEGRADED
 status: One or more devices are missing from the system.
 action: The pool can be imported despite missing or damaged devices.  The
        fault tolerance of the pool may be compromised if imported.
   see: http://zfsonlinux.org/msg/ZFS-8000-2Q
 config:

        otus                                 DEGRADED
          mirror-0                           DEGRADED
            /root/zpoolexport/filea          UNAVAIL  cannot open
            /home/vagrant/zpoolexport/fileb  ONLINE
```

`sudo zpool import -d ./ otus`

Проверил

`sudo zpool list`

```
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus    480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -
pool1  4.50G  4.84M  4.50G        -         -     0%     0%  1.00x    ONLINE  -
```

Определили размер хранилища

`sudo zfs get recordsize /otus`

```
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
```

Определил тип пула и степерь сжатия

`sudo zfs get compression,compressratio`

```
NAME            PROPERTY       VALUE     SOURCE
otus            compression    zle       local
otus            compressratio  1.00x     -
otus/hometask2  compression    zle       inherited from otus
otus/hometask2  compressratio  1.00x     -
```

Информация о пуле

`sudo zpool status -v otus`

```
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                                 STATE     READ WRITE CKSUM
        otus                                 ONLINE       0     0     0
          mirror-0                           ONLINE       0     0     0
            /home/vagrant/zpoolexport/filea  ONLINE       0     0     0
            /home/vagrant/zpoolexport/fileb  ONLINE       0     0     0
```

Выяснил какая используется контрольная сумма

`sudo zfs get checksum`

```
NAME            PROPERTY  VALUE      SOURCE
otus            checksum  sha256     local
otus/hometask2  checksum  sha256     inherited from otus
```

Задача - из востановленного снапшота прочитать сообщение от преподавателей в файле *secret_message*

Скачал файл

`wget --no-check-certificate -O otus_task2.file 'https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download'`

Восстановил снапшот

`sudo zfs receive otus/task2 < otus_task2.file`

`ls -l /otus/task2/`

```
total 2590
-rw-r--r--. 1 root    root          0 May 15 06:46 10M.file
-rw-r--r--. 1 root    root     309987 May 15 06:39 Limbo.txt
-rw-r--r--. 1 root    root     509836 May 15 06:39 Moby_Dick.txt
-rw-r--r--. 1 root    root    1209374 May  6  2016 War_and_Peace.txt
-rw-r--r--. 1 root    root     727040 May 15 07:08 cinderella.tar
-rw-r--r--. 1 root    root         65 May 15 06:39 for_examaple.txt
-rw-r--r--. 1 root    root          0 May 15 06:39 homework4.txt
drwxr-xr-x. 3 vagrant vagrant       4 Dec 18  2017 task1
-rw-r--r--. 1 root    root     398635 May 15 06:45 world.sql
```

Файла *secret_message* нет
