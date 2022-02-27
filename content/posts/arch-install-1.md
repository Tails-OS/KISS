+++
title = "Установка ArchLinux BTRFS + UEFI"
description = "В этой части мануала мы научимся устанавливать базовый ArchLinux + BTRFS с минимальной болью на систему с UEFI."
type = ["posts","post"]
tags = [
    "archlinux",
    "kiss",
    "archway",
    "unixway",
]
date = "2022-02-27"
categories = [
    "ArchLinux"
]
series = ["ArchLinux"]
[ author ]
  name = "Th3_Fox"
+++

## Введение
BTRFS (B-Tree Filesystem) — файловая система для Unix-подобных операционных систем, основанная на технике «Copy on Write» (CoW), призванная обеспечить легкость масштабирования файловой системы, высокую степень надежности и сохранности данных, гибкость настроек и легкость администрирования, сохраняя при этом высокую скорость работы.

### Основные возможности btrfs:
* Максимальный размер файла 2^64 байт
* Динамическая таблица inode
* Дедупликация данных
* Эффективное хранение файлов как очень малых, так и очень больших размеров
* Создание сабвольюмов и снапшотов
* Квоты на размеры сабвольюмов
* Контрольные суммы для данных и метаданных
* Возможность объединить несколько накопителей в единую файловую систему
* Создание RAID конфигурации на уровне файловой системы
* Сжатие данных
* Дефрагментация данных на лету

#### Или если говорить вкратце
BTRFS - Т1000 из мира файловых систем. Является наследником идей EXT2-3, и прекрасно подходит для SSD носителей, ибо имеет модули автодетекта, что позволяет не сильно париться с настройкой TRIM и флагов монтирования. Скорость чтения сопоставима, а иногда (Особенно при высоких нагрузках) превышают показатели EXT4. Идеальный выбор для игровой/домашней системы на базе Linux.

## Подготовка к установке
Для начала нам нужно скачать сам образ. Для этого перейдем на официальный [сайт](https://archlinux.org/) ArchLinux и скачаем образ с ближайшего к вам зеркала.
{{< image src="/img/gallery/guide/1.png" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
{{< image src="/img/gallery/guide/2.png" alt="ArchLinux Mirror" position="center" style="border-radius: 8px; margin-top: 4px" >}}
Далее, нужно записать данный образ на флешку и для этого советую использовать [Balena Etcher](https://www.balena.io/etcher/)

## Установка
Подключаем вашу флешку к компьютеру и грузимся с нее.
{{< image src="/img/gallery/guide/3.png" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Выбираем первую строку и жмём Enter.
#### Подключение к интернету
Если у вас проводное соединение, то делать ничего не нужно, но если вы хотите подключиться через WiFi, то вам нужно использовать следующие комманды:
Для того чтобы узнать имя вашего устройства:
``` Bash
iwctl device list
```
Чтобы просканировать сеть, где `wlan0` - имя вашего устройства:
``` Bash
iwctl station wlan0 scan
```
Чтобы вывести список доступных сетей(точек):
``` Bash
iwctl station wlan0 get-networks
```
Подключение к вашей точке доступа:
``` Bash
iwctl station wlan0 connect "Имя вашей точки" 
```
Далее, нужно проверить наше подключение:
``` Bash
ping -c 3 ya.ru 
```
Если пинг идет, то все хорошо, идем дальше.

#### Разделы диска
Далее, нам нужна разбить наш диск на разделы, но для начала нам нужно узнать имя самого диска:
``` Bash
fdisk -l
```
{{< image src="/img/gallery/guide/4.png" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Время форматировать наш диск. Для этого введем следующее:
``` Bash
gdisk /dev/"имя вашего диска"
```
У меня это `/dev/sda`:
``` Bash
gdisk /dev/sda
```
Для перехода в экспертный режим введите:
``` Bash
X
```
Затем для удаления GPT введите:
``` Bash
Z
```
Дважды согласитесь, чтобы полностью очистить диск.
{{< image src="/img/gallery/guide/5.jpg" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Можно ещё раз проверить вывод команды, чтобы убедиться, что изменения сохранены:
```Bash
fdisk -l
```
{{< image src="/img/gallery/guide/6.jpg" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Время размечать наш диск. Используем команду `cfdisk` для этого и выбираем `gpt` формат:
```Bash
cfdisk /dev/sda
```
{{< image src="/img/gallery/guide/7.png" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
>**Напомню, что вместо `sda` вы пишите имя своего диска**

Используя стрелочки создаём 2 раздела на диске:

* `/dev/sda1` размером 512M места под UEFI
* `/dev/sda2` все остальное оставшееся место.
Нажимаем `New` и устанавливаем размер равный 512M и жмем `Enter`
Указываем тип:
{{< image src="/img/gallery/guide/8.jpg" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Нам нужно выбрать EFI System:
{{< image src="/img/gallery/guide/9.jpg" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Переходим к оставшейся свободной области (стрелочка вниз), опять нажимаем `New`, выбираем весь незанятый размер, в качестве типа ставим `Linux root (x86-64)`: 
{{< image src="/img/gallery/guide/10.jpg" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Теперь выбираем Write, чтобы записать сделанные изменения.
 Пишем `yes` и выходим: 
 {{< image src="/img/gallery/guide/11.jpg" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Форматируем наши разделы:
1. Форматируем тот раздел, который мы выделили под UEFI:
```Bash
mkfs.fat -F32 /dev/sda1
```
2. Форматируем тот раздел, которому мы выделили все оставшееся место:
```Bash
mkfs.btrfs -f /dev/sda2
```
Теперь монтируем второй раздел для создание подразделов:
```Bash
mount /dev/sda2 /mnt
```
Приступим к созданию подразделов:
```Bash
btrfs su cr /mnt/@
```
```Bash
btrfs su cr /mnt/@home
```
```Bash
btrfs su cr /mnt/@root
```
```Bash
btrfs su cr /mnt/@srv
```
```Bash
btrfs su cr /mnt/@log
```
```Bash
btrfs su cr /mnt/@cache
```
```Bash
btrfs su cr /mnt/@tmp
```
Посмотрим созданные подразделы:
```Bash
btrfs su li /mnt
```
 {{< image src="/img/gallery/guide/12.png" alt="ArchLinux Download" position="center" style="border-radius: 8px;" >}}
Теперь отмонтируем диск:
```Bash
umount /mnt
```
Теперь создадим папки для монтирования подразделов:
```Bash
mkdir -p /mnt/{boot,home,root,srv,var/log,var/cache,tmp}
```
Начнем сам процесс монтирования:
```Bash
mount /dev/sda1 /mnt/boot
```
```Bash
mount -o subvol=@,defaults,noatime,compress=zstd:3 /dev/sda2 /mnt
```
```Bash
mount -o subvol=@home,defaults,noatime,compress=zstd:3 /dev/sda2 /mnt/home
```
```Bash
mount -o subvol=@root,defaults,noatime,compress=zstd:3 /dev/sda2 /mnt/root
```
```Bash
mount -o subvol=srv,defaults,noatime,compress=zstd:3 /dev/sda2 /mnt/srv
```
```Bash
mount -o subvol=@log,defaults,noatime,compress=zstd:3 /dev/sda2 /mnt/var/log
```
```Bash
mount -o subvol=@cache,defaults,noatime,compress=zstd:3 /dev/sda2 /mnt/var/cache
```
```Bash
mount -o subvol=@tmp,defaults,noatime,compress=zstd:3 /dev/sda2 /mnt/tmp
```
#### Установка
Начинается самая долгая часть, потому что нужно будет много скачать. Устанавливаем все основные пакеты.
```Bash
pacstrap /mnt base base-devel linux linux-firmware linux-headers btrfs-progs ntfs-3g exfat-utils iw dialog wpa_supplicant nano vim wget zsh intel-ucode
```
`intel-ucode` заменить на `amd-ucode`, если у вас процессор AMD.
После завершения установки пакетов создадим  fstab файл:
```Bash
genfstab -U /mnt >> /mnt/etc/fstab
```
#### Настройка установленной системы
Теперь воспользуемся командой arch-chroot, которая позволяет временно подменить корневой каталог на любой другой, в котором есть структура корневой файловой системы Linux. При этом программы, которые мы оттуда запустим, не будут знать о том, что снаружи ещё что-то существует. Мы практически окажемся в нашей новой системе с правами администратора:
```Bash
arch-chroot /mnt
```
Настраиваем дату и и время:
```Bash
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
```
Где `Europe/Moscow` - ваша временная зона.
Чтобы увидеть все временные зоны:
```Bash
ls /usr/share/zoneinfo
```
Чтобы увидеть подкатегории:
```Bash
ls /usr/share/zoneinfo/Europe
```
Включаем использование systemd-timesyncd для синхронизации времени:
```Bash
timedatectl set-ntp true
```
Устанавливаем аппаратные часы на UTC:
```Bash
hwclock --systohc --utc
```
Установим русский язык. В файле locale.gen нам нужно раскомментировать две строки, для этого:
```Bash
nano /etc/locale.gen
```
Ищем там и раскомментируем строки: `en_US.UTF-8 UTF-8`
`ru_RU.UTF-8 UTF-8`
После этого выполняем:
```Bash
locale-gen
```
Создадим locale.conf:
```Bash
nano /etc/locale.conf
```
И туда впишем: 
>LANG=ru_RU.UTF-8

Далее, настроим язык консоли:
```Bash
nano /etc/vconsole.conf
```

И туда впишем следующее:
>KEYMAP=ru  
>FONT=cyr-sun16

Зададим имя нашему ПК:
```Bash
nano /etc/hostname 
```
и туда вписываем любое имя:
>CyberNet

Установим сетевой менеджер:
```Bash
pacman -S networkmanager
```
И включим его:
```Bash
systemctl enable NetworkManager
```
Зададим пароль на рут:
```Bash
passwd
```
Далее вводим пароль.
Установим загрузчик:
```Bash
bootctl install
```
Редактируем содержимое файла:
```Bash
nano /boot/loader/loader.conf
```
Удалите все то, что там есть и впишите туда:
>default  arch

Создайте конфигурационный файл для добавления пункта Arch Linux в менеджер systemd-boot:
```Bash
nano /boot/loader/entries/arch.conf
```
Содержимое файла должно быть следующим: 
>title  ArchLinux  
>linux  /vmlinuz-linux  
>initrd  /intel-ucode.img  
>initrd  /initramfs-linux.img  
>options  root=/dev/sda2 rootflags=subvol=@ rw quiet

>**Обратите внимание на /dev/sda2 — это путь до моего диска с системой, замените на свой.** 
Выйдем из chroot и перезагрузимся:
```Bash
exit  
reboot
```
#### Создание пользователя
Заходим под root и создаем пользователя:
```Bash
useradd -m -g users -G audio,video,network,wheel,storage,rfkill -s /bin/zsh th3_fox
```
>**th3_fox - меняете на свое имя пользователя**

Задаем для него пароль:
```Bash
passwd th3_fox
```
Чтобы пользователи из группы wheel могли выполнять команды с sudo, выполним следующее:
В файле `/etc/sudoers` найдем и раскомментируем строку 
>%wheel ALL=(ALL) ALL
```Bash
nano /etc/sudoers
```
Далее, просто выходим из рут и заходим уже под нашим пользователем. 
## Итог
Вот и все, вы подошли к завершению данного мануала и успешно установили ArchLinux на свой ПК.  
В следующем руководстве, мы будем устанавливать графическое окружение рабочего стола.
