# 1. Подготовка
`nano /etc/pacman.d/mirrorlist (Комментруй все ctrl + /, расскоминтируй Russia)`

`nano /etc/pacman.conf (Найди ParallelDownloads и поставь 15`
   
# 2. Разметка диска
`cfdisk /dev/nvme0n1 (Создание таблицы разделов GPT)`
   ```
   1 nvme0n1p1 512MB EFI System
   2 nvme0n1p2 60G Linux filesystem
   3 nvme0n1p3 ост. Linux Filesystem
   ```

   ```
 mkfs.vfat /dev/nvme0n1p1
 mkfs.ext4 /dev/nvme0n1p2
 mkfs.f2fs /dev/nvme0n1p3
   ```

# 3. Монтирование
```mount /dev/nvme0n1p2 /mnt```

```mkdir -p /mnt/boot/efi && mount /dev/nvme0n1p1 /mnt/boot/efi```

```mkdir -p /mnt/mnt/games && mount /dev/nvme0n1p3 /mnt/mnt/games```
   
# 4. Установка базы
```
pacstrap /mnt base base-devel linux linux-firmware linux-headers f2fs-tools nano git bash-completion grub efibootmgr networkmanager sddm
```

# 5. Chroot
```genfstab -U /mnt >> /mnt/etc/fstab```
```arch-chroot /mnt``` `(Переход в установленную систему)`
```systemctl enable NetworkManager```
```systemctl enable sddm```
   
# 6. Пользователи
```passwd root``` `(Установка пароля для root.)`
```useradd -m [user]``` `(Создание юзера)`
```passwd [user]``` `(Установка пароля пользователя)`
`/etc/sudoers (Найди строку "root ALL=(ALL:ALL) ALL", под ней напиши [user] ALL=(ALL:ALL) ALL`

# 7. Локализация и время
`nano /etc/locale.gen (расскоминтируй "ru_RU.UTF-8" и "en_US.UTF-8")`
```locale-gen```
```echo "LANG=en_US.UTF-8" > /etc/locale.conf```
```ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime```
   
# 8. Инициализация загрузчика
```grub-install /dev/nvme0n1```
```grub-mkconfig -o /boot/grub/grub.cfg```
   
# 9. Завершение
```exit```
```umount -R /mnt```
```reboot```
