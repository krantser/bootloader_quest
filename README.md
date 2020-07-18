# **Вход в систему без пароля** 

1. После загрузки меню загрузчика, нажатием клавиши E заходим в конфигурацию Grub,
после чего на экране появятся настройки в текстовом виде.
В конце строки начинающейся с linux16 дописываем путь к интерпретатору командной 
строки init=/bin/bash. После чего нажимаем сочетание клавишь Ctrl + X и мы попадём
в командную строку bash. Далее перемонтируем файловую систему в режиме для 
чтения\записи. Теперь проверим, работает ли запись в файловую систему, для этого
выполним простую команду и запишем в файл модель процессора использующегося в 
системе:
```
cat /proc/cpuinfo | grep name > /home/user01/cpu
```
Проверим результат выполнения команды:
```
cat /home/user01/cpu
```

Вывод команды:
```
model name      : Intel(R) Core(TM) i7-6500U CPU @ 2.50GHz
```

2. В строке убираем quiet, меняем ro на rw и доисываем в конце строки rd.break.
После чего нажимаем сочетание клавишь Ctrl + X и мы попадём в командную строку 
bash. И далее вводим следующие команды для смены пароля пользователя root:
```
chroot /sysroot
passwd root
touch /.autorelabel
exit
reboot
```
Входим с новым паролем.

3. В конце строки начинающейся с linux16 дописываем путь к интерпретатору командной 
строки init=/sysroot/bin/bash. После чего нажимаем сочетание клавишь Ctrl + X и 
мы попадём в командную строку bash.



[root@localhost user01]# vgs
  VG     #PV #LV #SN Attr   VSize VFree
  centos   1   2   0 wz--n- 7,67g 4,00m

[root@localhost user01]# vgrename centos bofh_vol
  Volume group "centos" successfully renamed to "bofh_vol"

Приводим fstab к следующему виду:
```
#
# /etc/fstab
# Created by anaconda on Thu Jul  9 19:14:08 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/bofh_vol-root /                       xfs     defaults        0 0
UUID=d1c57486-ff51-4cab-af01-ca1714397915 /boot                   xfs     defaults        0 0
/dev/mapper/bofh_vol-swap swap                    swap    defaults        0 0
```

В /etc/default/grub правим
```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto spectre_v2=retpoline rd.lvm.lv=bofh_vol/root rd.lvm.lv=bofh_vol/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

В /boot/grub2/grub.cfg правим строку в соответствии с видом:
```
linux16 /vmlinuz-3.10.0-1127.el7.x86_64 root=/dev/mapper/bofh_vol-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=bofh_vol/root rd.lvm.lv=bofh_vol/swap rhgb LANG=ru_RU.UTF-8
```

```
[root@localhost user01]# mkinitrd -f -v /boot/initramfs-3.10.0-1127.el7.x86_64.img $(uname -r)
```

Сокращённый вывод команды:
```
*** Creating initramfs image file '/boot/initramfs-3.10.0-1127.el7.x86_64.img' done ***
```

[root@localhost user01]# reboot

[root@localhost user01]# vgs
  VG       #PV #LV #SN Attr   VSize VFree
  bofh_vol   1   2   0 wz--n- 7,67g 4,00m

