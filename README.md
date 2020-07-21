## Вход в систему без пароля 

1. После загрузки меню загрузчика, нажатием клавиши E заходим в конфигурацию Grub,
после чего на экране появятся настройки в текстовом виде.
В конце строки начинающейся с `linux16` дописываем путь к интерпретатору командной 
строки `init=/bin/bash`. После чего нажимаем сочетание клавишь Ctrl + X и мы попадём
в командную строку bash. Далее перемонтируем файловую систему в режиме для 
чтения\записи. Теперь проверим, работает ли запись в файловую систему, для этого
выполним простую команду и запишем в файл модель процессора использующегося в 
системе:
```sh
[root@localhost user01]# cat /proc/cpuinfo | grep name > /home/user01/cpu
```
Проверим результат выполнения команды:
```sh
[root@localhost user01]# cat /home/user01/cpu
```

Вывод команды:
```
model name      : Intel(R) Core(TM) i7-6500U CPU @ 2.50GHz
```

2. В строке убираем `quiet`, меняем `ro` на `rw` и доисываем в конце строки `rd.break`.
После чего нажимаем сочетание клавишь Ctrl + X и мы попадём в командную строку 
bash. И далее вводим следующие команды для смены пароля пользователя root:
```sh
bash-4.2# chroot /sysroot
bash-4.2# passwd root
bash-4.2# touch /.autorelabel
bash-4.2# exit
bash-4.2# reboot
```
Входим с новым паролем.

3. В конце строки начинающейся с `linux16` дописываем путь к интерпретатору командной 
строки `init=/sysroot/bin/bash`. После чего нажимаем сочетание клавишь Ctrl + X и 
мы попадём в командную строку bash.


## Переименовать Volume Group

Далее переименуем группу томов LVM. И для начала посмотрим наличие существующих
групп в системе:
```sh
[root@localhost user01]# vgs
```

Вывод команды:
```
  VG     #PV #LV #SN Attr   VSize VFree
  centos   1   2   0 wz--n- 7,67g 4,00m
```

Далее переименуем группу с помощью команды `vgrename`, где centos имя группы,
которую хотим переименовать, а bofh_vol - новое имя группы:
```sh
[root@localhost user01]# vgrename centos bofh_vol
```

Вывод команды:
```
  Volume group "centos" successfully renamed to "bofh_vol"
```

Теперь нужно в конфигурационных файлах системы, использующихся при загрузке ОС,
поменять старое имя группы томов на новое.
Сперва приводим конфигурацию fstab к следующему виду:
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

Далее правим /etc/default/grub и приводим к виду:
```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto spectre_v2=retpoline rd.lvm.lv=bofh_vol/root rd.lvm.lv=bofh_vol/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

В /boot/grub2/grub.cfg редактируем строки в соответствии с видом:
```
linux16 /vmlinuz-3.10.0-1127.el7.x86_64 root=/dev/mapper/bofh_vol-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=bofh_vol/root rd.lvm.lv=bofh_vol/swap rhgb LANG=ru_RU.UTF-8
```

Пересоздадим образ initrd с новым именем Virtual Group для томов:
```sh
[root@localhost user01]# mkinitrd -f -v /boot/initramfs-3.10.0-1127.el7.x86_64.img $(uname -r)
```

Сокращённый вывод команды:
```
*** Creating initramfs image file '/boot/initramfs-3.10.0-1127.el7.x86_64.img' done ***
```

Теперь можно перезагрузить систему:
```sh
[root@localhost user01]# reboot
```

После загрузки проверим имя Volume Group:
```sh
[root@localhost user01]# vgs
```

Вывод команды:
```
  VG       #PV #LV #SN Attr   VSize VFree
  bofh_vol   1   2   0 wz--n- 7,67g 4,00m
```

Как видим группа успешно переименована.

## Добавляем модуль в initrd

Для начала создадим два файла - module-setup.sh и tux.sh, первый файл необходим 
для установки модуля, а второй соответственно скрипт модуля.
Содержание module-setup.sh:
```
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/tux.sh"
} 
```

Содержание tux.sh:
```
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'
Hello! You are in dracut module!
 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...." 
```

Создаём директорию, в которую поместим оба наших скрипта:
```sh
[root@localhost user01]# mkdir /usr/lib/dracut/modules.d/01tux/
```

Теперь пересоберём образ initramfs:
```sh
[root@localhost user01]# dracut -f -v
```

Сокращённый вывод команды:
```
*** Creating initramfs image file '/boot/initramfs-3.10.0-1127.el7.x86_64.img' done ***
```

Просмотрим список модулей входящих в initramfs:
```sh
[root@localhost user01]# lsinitrd -m /boot/initramfs-3.10.0-1127.el7.x86_64.img
```

Сокращённый вывод команды:
```
dracut modules:
bash
tux
nss-softokn
i18n
```

Как видим в списке присутсвует наш модуль tux. Далее отредактируем конфигурационный
файл загрузчика:
```sh
[root@localhost user01]# vi /boot/grub2/grub.cfg
```

И после строк начинающихся с linux16 уберём директивы quiet и rghb и далее
перезагрузим систему. После этого видим на экране вместе с загрузкой демонов
рисунок пингвина с надписью. На этом работа завершена.

