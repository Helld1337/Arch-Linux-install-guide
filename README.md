# 1. Подготовка
`nano /etc/pacman.d/mirrorlist (Комментруй все ctrl + /, расскоминтируй Russia)`

`nano /etc/pacman.conf (Найди ParallelDownloads и поставь 15`

`Найди и расскоминтируй [multilib] `
   
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

`pacstrap /mnt f2fs-tools`

```mkdir -p /mnt/other && mount /dev/nvme0n1p3 /mnt/other```
   
# 4. Установка базы
```
pacstrap /mnt base base-devel linux linux-firmware linux-headers nano git bash-completion grub efibootmgr amd-ucode networkmanager nftables doas
```

# 5. Chroot
```genfstab -U /mnt >> /mnt/etc/fstab```

```arch-chroot /mnt``` `(Переход в установленную систему)`

```systemctl enable NetworkManager```

```pacman -Rs sudo```

# 6. Пользователи
```passwd root``` `(Установка пароля для root.)`

```useradd -m [user]``` `(Создание юзера)`

```passwd [user]``` `(Установка пароля пользователя)`

`usermod -aG wheel [user]`

`nano /etc/doas.conf`
```permit :wheel```

`chown -R [user]:[user] /other`

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

# 10. Установка AUR (yay)
`git clone https://aur.archlinux.org/yay.git`

`cd yay`

`makepkg -si`

# 11. Полезные утилиты
`doas pacman -S mesa lib32-mesa lib32-gamemode vulkan gamemode vulkan-radeon libva-mesa-driver gamescope zram-generator`

```doas nano /etc/systemd/zram-generator.conf``` `Настройка zram-generator`

```
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
swap-priority = 100
```

`doas systemctl daemon-reload`

`doas systemctl start /dev/zram0`

`zramctl` `Проверка работы`

# 12. Отключение usb wakeup
`doas nano /etc/systemd/system/wakeup.service`

```
[Unit]
Description=Disable usb wake-up
 
[Service]
Type=oneshot
ExecStart=/bin/sh -c "\
    for dev in XHC0 PTXH GPP0 GPP8 PT23 GP12 GP13; do \
        if grep -q \"$dev.*enabled\" /proc/acpi/wakeup; then \
            echo $dev > /proc/acpi/wakeup; \
        fi; \
    done"
RemainAfterExit=yes
 
[Install]
WantedBy=multi-user.target
```

`doas systemctl daemon-reload`

`doas systemctl enable wakeup.service`

`doas systemctl start wakeup.service`

# 13. Установка zen kernel
`doas pacman -S linux-zen linux-zen-headers`

`doas grub-mkconfig -o /boot/grub/grub.cfg`

# 14. Установка окружения
`yay -S wayfire sfwbar`

`doas pacman -S swaybg swayidle mako desktop-portal-wlr xdg-desktop-portal wl-clipboard slurp grim fuzzel brightnessctl wireplumber`

# 15. Установка программ
`doas pacman -S kitty firefox steam`

`yay -S proton-ge-custom-bin koala-clash-bin vesktop-git`

# 16. DPI Bypass
```git clone https://github.com/Sergeydigl3/zapret-discord-youtube-linux.git```
