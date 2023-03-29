0. Домашнее задание
	Работа с mdadm
	
1. Установлен VirtualBox и Vagrant на локальную машину ещё на Уроке 1.
2. Запускаем garik-VirtualBox$vagrant -v
	Vagrant 2.2.19
3. Берём с начального стенда образ: https://github.com/erlong15/otus-linux
4. Добавляем в файл Vagrantfile:
  :sata4 =>
#  {
#    :dfile => './sata4.vdi',
#    :size => 250, # Megabytes
#    :port => 4
#  },
  :sata5 =>
  {
    :dfile => './sata5.vdi', # Путь, по которому будет создан файл диска
    :size => 250, # Размер диска в мегабайтах
    :port => 5 # Номер порта на который будет зацеплен диск
  },
  :sata6 =>
  {
	:dfile => './sata6.vdi', # Путь, по которому будет создан файл диска
	:size => 250, # Размер диска в мегабайтах
	:port => 6 # Номер порта на который будет зацеплен диск
  },

5. Выполняем : 
	vagrant up
	vagrant ssh
   Добавились указанные устройства :
	sudo lshw -short | grep disk
garik-VirtualBox$vagrant ssh
Last login: Mon Mar 27 03:00:37 2023 from 10.0.2.2
[vagrant@otuslinux ~]$ sudo lshw -short | grep disk
/0/100/1.1/0.0.0    /dev/sda  disk        42GB VBOX HARDDISK
/0/100/d/0          /dev/sdb  disk        262MB VBOX HARDDISK
/0/100/d/1          /dev/sdc  disk        262MB VBOX HARDDISK
/0/100/d/2          /dev/sdd  disk        262MB VBOX HARDDISK
/0/100/d/3          /dev/sde  disk        262MB VBOX HARDDISK
/0/100/d/0.0.0      /dev/sdf  disk        262MB VBOX HARDDISK

	vagrant@otuslinux ~]$ sudo fdisk -l
                                                                                                                                                                             
Disk /dev/sda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0009ef1a

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048    83886079    41942016   83  Linux

Disk /dev/sdb: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdc: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdd: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sde: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sdf: 262 MB, 262144000 bytes, 512000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

[vagrant@otuslinux ~]$
Готовы 5 дисков для создания RAID.

6. Обнуляем суперблоки и создаём рейд:
[vagrant@otuslinux ~]$ sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde
mdadm: Unrecognised md component device - /dev/sdf

[vagrant@otuslinux ~]$ sudo mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 253952K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[vagrant@otuslinux ~]$

7. Проверяем массив :
[vagrant@otuslinux ~]$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]

unused devices: <none> 

[vagrant@otuslinux ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Mon Mar 27 04:20:34 2023
        Raid Level : raid6
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Mon Mar 27 04:20:45 2023
             State : clean
    Active Devices : 5
   Working Devices : 5
    Failed Devices : 0
     Spare Devices : 0 

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : c9372fa4:84f34792:07abbf64:f40fa4a5
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       3       8       64        3      active sync   /dev/sde
       4       8       80        4      active sync   /dev/sdf
[vagrant@otuslinux ~]$

8. Создаем файл mdadm.conf:
	[vagrant@mdadm ~]$ echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
	[vagrant@mdadm ~]$ mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf

9. Выводим из строяч одно устройство из RAID массива :
	[vagrant@mdadm ~]$ sudo mdadm /dev/md0 --fail /dev/sde
		mdadm: set /dev/sde faulty in /dev/md0                                                                                                                                       
	[vagrant@otuslinux mdadm]$   
10. [vagrant@otuslinux mdadm]$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid6 sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
      761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]

unused devices: <none>
[vagrant@otuslinux mdadm]$ mdadm -D /dev/md0
mdadm: must be super-user to perform this action
[vagrant@otuslinux mdadm]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Mon Mar 27 04:20:34 2023
        Raid Level : raid6
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 5
     Total Devices : 5
       Persistence : Superblock is persistent

       Update Time : Mon Mar 27 05:13:37 2023
             State : clean, degraded
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : c9372fa4:84f34792:07abbf64:f40fa4a5
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       -       0        0        3      removed
       4       8       80        4      active sync   /dev/sdf

       3       8       64        -      faulty   /dev/sde
[vagrant@otuslinux mdadm]$

11. Создаем GPT раздел с пятью партициями:

[vagrant@otuslinux mdadm]$ sudo parted -s /dev/md0 mklabel gpt
[vagrant@otuslinux mdadm]$ sudo parted /dev/md0 mkpart primary ext4 0% 20%
Information: You may need to update /etc/fstab.
[vagrant@otuslinux mdadm]$ sudo parted /dev/md0 mkpart primary ext4 20% 40%
Information: You may need to update /etc/fstab.
[vagrant@otuslinux mdadm]$ sudo parted /dev/md0 mkpart primary ext4 40% 60%
Information: You may need to update /etc/fstab.
[vagrant@otuslinux mdadm]$ sudo parted /dev/md0 mkpart primary ext4 60% 80%
Information: You may need to update /etc/fstab.
[vagrant@otuslinux mdadm]$ sudo parted /dev/md0 mkpart primary ext4 80% 100%
Information: You may need to update /etc/fstab.

12. Создаем на этих партициях файловую систему и подключаем к каталогам:
	[vagrant@otuslinux mdadm]$ for i in $(seq 1 5); do sudo mount /dev/md0p$i /raid/part$i; done
mke2fs 1.42.9 (28-Dec-2013)                                                                                                                                                  
Filesystem label=                                                                                                                                                            
OS type: Linux                                                                                                                                                               
Block size=1024 (log=0)                                                                                                                                                      
Fragment size=1024 (log=0)                                                                                                                                                   
Stride=512 blocks, Stripe width=1536 blocks                                                                                                                                  
37696 inodes, 150528 blocks                                                                                                                                                  
7526 blocks (5.00%) reserved for the super user                                                                                                                              
First data block=1                                                                                                                                                           
Maximum filesystem blocks=33816576                                                                                                                                           
19 block groups                                                                                                                                                              
8192 blocks per group, 8192 fragments per group                                                                                                                              
1984 inodes per group                                                                                                                                                        
Superblock backups stored on blocks:                                                                                                                                         
        8193, 24577, 40961, 57345, 73729                                                                                                                                     
                                                                                                                                                                             
Allocating group tables: done                                                                                                                                                
Writing inode tables: done                                                                                                                                                   
Creating journal (4096 blocks): done                                                                                                                                         
Writing superblocks and filesystem accounting information: done                                                                                                              
                                                                                                                                                                             
mke2fs 1.42.9 (28-Dec-2013)                                                                                                                                                  
Filesystem label=                                                                                                                                                            
OS type: Linux                                                                                                                                                               
Block size=1024 (log=0)                                                                                                                                                      
Fragment size=1024 (log=0)                                                                                                                                                   
Stride=512 blocks, Stripe width=1536 blocks                                                                                                                                  
38152 inodes, 152064 blocks                                                                                                                                                  
7603 blocks (5.00%) reserved for the super user                                                                                                                              
First data block=1                                                                                                                                                           
Maximum filesystem blocks=33816576                                                                                                                                           
19 block groups                                                                                                                                                              
8192 blocks per group, 8192 fragments per group                                                                                                                              
2008 inodes per group                                                                                                                                                        
Superblock backups stored on blocks:                                                                                                                                         
        8193, 24577, 40961, 57345, 73729                                                                                                                                     
                                                                                                                                                                             
Allocating group tables: done                                                                                                                                                
Writing inode tables: done                                                                                                                                                   
Creating journal (4096 blocks): done                                                                                                                                         
Writing superblocks and filesystem accounting information: done                                                                                                              
                                                                                                                                                                             
mke2fs 1.42.9 (28-Dec-2013)                                                                                                                                                  
Filesystem label=                                                                                                                                                            
OS type: Linux                                                                                                                                                               
Block size=1024 (log=0)                                                                                                                                                      
Fragment size=1024 (log=0)                                                                                                                                                   
Stride=512 blocks, Stripe width=1536 blocks                                                                                                                                  
38456 inodes, 153600 blocks                                                                                                                                                  
7680 blocks (5.00%) reserved for the super user                                                                                                                              
First data block=1                                                                                                                                                           
Maximum filesystem blocks=33816576                                                                                                                                           
19 block groups                                                                                                                                                              
8192 blocks per group, 8192 fragments per group                                                                                                                              
2024 inodes per group                                                                                                                                                        
Superblock backups stored on blocks:                                                                                                                                         
        8193, 24577, 40961, 57345, 73729                                                                                                                                     
                                                                                                                                                                             
Allocating group tables: done                                                                                                                                                
Writing inode tables: done                                                                                                                                                   
Creating journal (4096 blocks): done                                                                                                                                         
Writing superblocks and filesystem accounting information: done                                                                                                              
                                                                                                                                                                             
mke2fs 1.42.9 (28-Dec-2013)                                                                                                                                                  
Filesystem label=                                                                                                                                                            
OS type: Linux                                                                                                                                                               
Block size=1024 (log=0)                                                                                                                                                      
Fragment size=1024 (log=0)                                                                                                                                                   
Stride=512 blocks, Stripe width=1536 blocks                                                                                                                                  
38152 inodes, 152064 blocks                                                                                                                                                  
7603 blocks (5.00%) reserved for the super user                                                                                                                              
First data block=1                                                                                                                                                           
Maximum filesystem blocks=33816576                                                                                                                                           
19 block groups                                                                                                                                                              
8192 blocks per group, 8192 fragments per group                                                                                                                              
2008 inodes per group                                                                                                                                                        
Superblock backups stored on blocks:                                                                                                                                         
        8193, 24577, 40961, 57345, 73729                                                                                                                                     
                                                                                                                                                                             
Allocating group tables: done                                                                                                                                                
Writing inode tables: done                                                                                                                                                   
Creating journal (4096 blocks): done                                                                                                                                         
Writing superblocks and filesystem accounting information: done                                                                                                              
                                                                                                                                                                             
mke2fs 1.42.9 (28-Dec-2013)                                                                                                                                                  
Filesystem label=                                                                                                                                                            
OS type: Linux                                                                                                                                                               
Block size=1024 (log=0)                                                                                                                                                      
Fragment size=1024 (log=0)                                                                                                                                                   
Stride=512 blocks, Stripe width=1536 blocks                                                                                                                                  
37696 inodes, 150528 blocks                                                                                                                                                  
7526 blocks (5.00%) reserved for the super user                                                                                                                              
First data block=1                                                                                                                                                           
Maximum filesystem blocks=33816576                                                                                                                                           
19 block groups                                                                                                                                                              
8192 blocks per group, 8192 fragments per group                                                                                                                              
1984 inodes per group                                                                                                                                                        
Superblock backups stored on blocks:                                                                                                                                         
        8193, 24577, 40961, 57345, 73729                                                                                                                                     
                                                                                                                                                                             
Allocating group tables: done                                                                                                                                                
Writing inode tables: done                                                                                                                                                   
Creating journal (4096 blocks): done                                                                                                                                         
Writing superblocks and filesystem accounting information: done                                                                                                              
[vagrant@otuslinux mdadm]$ for i in $(seq 1 5); do sudo mount /dev/md0p$i /raid/part$i; done                                                                                 

[vagrant@otuslinux raid]$ ls -la                                                                                                                                             
total 5                                                                                                                                                                      
drwxr-xr-x.  7 root root   71 Mar 27 05:37 .                                                                                                                                 
dr-xr-xr-x. 19 root root  267 Mar 27 05:37 ..                                                                                                                                
drwxr-xr-x.  3 root root 1024 Mar 27 05:31 part1                                                                                                                             
drwxr-xr-x.  3 root root 1024 Mar 27 05:31 part2                                                                                                                             
drwxr-xr-x.  3 root root 1024 Mar 27 05:31 part3                                                                                                                             
drwxr-xr-x.  3 root root 1024 Mar 27 05:31 part4                                                                                                                             
drwxr-xr-x.  3 root root 1024 Mar 27 05:31 part5                                                                                                                             

13. Делаем исправления в Vagrantfile и запускаем его. 
	Отработано успешно. Вывод на экран в файле "vagrant up.jpg".