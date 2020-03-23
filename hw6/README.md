#### Установка CentOS7 с LVM
В VirtualBox устанавливаю на вирутальную машину CentOS7 с LVM. При разметке диска выбираю автоматическое создание LVM, таким образом получаю:
```console
/boot - находится на физическом разделе /dev/sda1
/ и swap - находятся в разделе LVM с именем VG centos, на физическом разделе /dev/sda2 
```
Вывод lsblk
```console
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0    5G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0    4G  0 part 
  ├─centos-root 253:0    0  3,5G  0 lvm  /
  └─centos-swap 253:1    0  512M  0 lvm  [SWAP]
sr0              11:0    1 1024M  0 rom
```
#### Попасть в систему без пароля несколько способами
Загружаю систему, при попадании в GRUB со список ядер нажимаю на "e" (что означает edit) и попадаю в меню изменений параметров загрузки
##### Способ №1 init=/bin/sh
***
Данный способ указывает, что оболочка (в данном случае /bin/sh) должна быть запущена сразу после загрузки ядра и initrd. Это полезный вариант, но не лучший, потому что в некоторых случаях можно потерять консольный доступ или пропустить другие функции.
***
Ход действий:
* В конце строки, которая начинается с linux16 добавляю init=/bin/sh и нажимаю сtrl-x для
загрузки в систему
* Таким образом попадаю в систему, однако корневая система смонтирована в режиме read-only, об этом говорит аргумент ro, который можно заметить в списке аргументов, который мы видим в строке, начинающейся с linux16. Монтирую систему в режим read-write:
```console
sh-4.2# mount -o remount,rw /
```
* Убеждаюсь, анализируя вывод введенной ниже команды
```console
sh-4.2# mount | grep root
/dev/mapper/centos-root on / type xfs (rw,relatime,attr2,inode64,noqouta)
```
Наблюдаю аргумент rw, что говорит о том, что система в режиме read-write

##### Способ №2 rd.break
***
Данный способ останавливает процедуру загрузки, пока она еще находится в стадии initramfs. Опция полезна, если нет пароля для root.
***
Ход действий:
* В конце строки, которая начинается с linux16 добавляю аргумент загрузки ядра rd.break и нажимаю сtrl-x для
загрузки в систему
* Попадаю в emergency mode. Так как аргумент ro не изменялся на rw, корневая файловая, как и в предыдущем случае, смонтирована в режиме read-only. Однако я не нахожусь в системе. Далее попадаю в нее:
```console
switch_root:# mount -o remount,rw /sysroot
switch_root:# chroot /sysroot
```
Меняю пароль для пользователя root:
```console
sh-4.2# passwd root
```
Так как не отключен SELinux, ввожу следующую команду, чтобы новый файл /etc/shadow, который был создан при изменении пароля был принят в качестве нового файла:
```console
sh-4.2# touch /.autorelabel
```
* Перезагружаюсь и захожу в систему с новым паролем. 
##### Способ №3 rw init=/sysroot/bin/bash
***
Данный способ указывает, что система должны быть смонтирована в режиме read-write и запущена с оболочкой /bin/bash сразу после загрузки ядра и initrd.
***
Ход действий:
* В строке, которая начинается с linux16 заменяю аргумент ro на rw и дописываю init=/sysroot/bin/sh, после чего нажимаю сtrl-x
для загрузки в систему
* Загружаюсь в систему с оболочкой /bin/bash и смонтированной в режим read-write

#### Переименовать VG
* Для записи действий запускаю утилиту script, которая записала в итоге следующий файл - [typescript]()
* Смотрю текущее состояние системы:
```console
[root@localhost ~]# vgs
  VG     #PV #LV #SN Attr   VSize  VFree
  centos   1   2   0 wz--n- <7,00g    0
```
* Во второй строке вижу название VG - centos
* Переименовываю:
```console
[root@localhost ~]# vgrename centos OtusRoot
  Volume group "centos" successfully renamed to "OtusRoot"
```
* Далее правлю /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяю старое название (centos) на новое (OtusRoot). 
* Пересоздаю  initrd image, чтобы он знал новое название Volume Group
```console
[root@localhost ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
...
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-957.12.2.el7.x86_64.img' done ***
```
* После выполненных действий перезагружаюсь и проверяю успешно ли выполнены действия:
```console
[root@localhost ~]# vgs
  VG       #PV #LV #SN Attr   VSize  VFree
  OtusRoot   1   2   0 wz--n- <7,00g    0 
 ```
#### Добавить модуль в initrd
* Продолжаю записывать дейстия при помощи утилиты script в файл typescript
```console
[root@localhost ~]# script -a typescript
```
* Скриптý модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Создаю там директорию 01test для создания своего модуля:
```console
[root@localhost ~]# mkdir /usr/lib/dracut/modules.d/01test
```
В нее помещаю два скрипта:
1. [module-setup.sh](https://github.com/xeniaweber/otus/blob/master/hw6/module_setup.sh) - устанавливает модуль и вызывает скрипт test.sh
2. [test.sh](https://github.com/xeniaweber/otus/blob/master/hw6/test.sh) - вызываемый скрипт, в нём у нас рисуется пингвинчик с надписью otus
* Пересобираю образ initrd
```console
[root@localhost ~]# dracut -f -v
```



