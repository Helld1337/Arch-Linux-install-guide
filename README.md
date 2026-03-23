1. Подготовка сетевой среды и менеджера пакетов
Перед началом установки убедитесь в наличии активного интернет-соединения (проводная сеть Ethernet настраивается автоматически через DHCP).
1.1. Настройка зеркал и оптимизация Pacman
Отредактируйте список зеркал для повышения скорости загрузки:
nano /etc/pacman.d/mirrorlist

Оптимизируйте работу менеджера пакетов, включив многопоточную загрузку:
nano /etc/pacman.conf

В разделе [options] раскомментируйте или добавьте строку:
ParallelDownloads = 15
2. Подготовка дискового пространства
Идентификация целевого накопителя NVMe:
lsblk

2.1. Разметка диска (Таблица разделов GPT)
Используйте утилиту cfdisk для создания структуры разделов на /dev/nvme0n1:
 * Раздел EFI: 512 MiB (Type: EFI System).
 * Системный раздел: Не менее 40–60 GiB (Type: Linux root x86-64).
 * Игровой раздел: Оставшееся пространство (Type: Linux filesystem).
2.2. Создание файловых систем
Выполните форматирование созданных томов:
mkfs.vfat -F 32 /dev/nvme0n1p1      # EFI
mkfs.ext4 /dev/nvme0n1p2           # Root (Система)
mkfs.f2fs -f /dev/nvme0n1p3        # Games (Оптимизировано для Flash)

3. Монтирование целевой иерархии
Установите соответствие разделов точкам монтирования в среде LiveCD:
mount /dev/nvme0n1p2 /mnt              # Корень
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi     # Загрузчик
mkdir -p /mnt/mnt/games
mount /dev/nvme0n1p3 /mnt/mnt/games    # Игровой раздел

4. Установка базовых компонентов системы
Развертывание ядра и необходимых системных утилит:
pacstrap /mnt base base-devel linux linux-firmware linux-headers f2fs-tools nano vi bash-completion grub efibootmgr networkmanager xorg sddm ttf-liberation ttf-dejavu

> Примечание: При использовании видеокарт NVIDIA добавьте пакет nvidia (или nvidia-lts в зависимости от ядра) в конец списка pacstrap.
> 
5. Конфигурирование установленной системы
Сгенерируйте таблицу файловых систем и войдите в окружение chroot:
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

5.1. Активация системных служб
Обеспечьте автоматический запуск сетевого менеджера и графического входа:
systemctl enable NetworkManager
systemctl enable sddm

5.2. Управление пользователями
Установите пароли для администратора и создайте учетную запись пользователя:
passwd root
useradd -m -G wheel [имя_пользователя]
passwd [имя_пользователя]

Для предоставления привилегий sudo выполните EDITOR=nano visudo и раскомментируйте строку: %wheel ALL=(ALL:ALL) ALL.
5.3. Локализация и временные зоны
Настройте региональные стандарты:
 * В файле /etc/locale.gen раскомментируйте en_US.UTF-8 UTF-8 и ru_RU.UTF-8 UTF-8.
 * Примените изменения:
   locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

 * Установите часовой пояс:
   ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime

6. Установка и настройка загрузчика (GRUB)
Инсталляция загрузчика на NVMe-накопитель:
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

Для отображения логов загрузки отредактируйте /etc/default/grub, удалив параметр quiet из строки GRUB_CMDLINE_LINUX_DEFAULT.
Генерация финальной конфигурации:
grub-mkconfig -o /boot/grub/grub.cfg

7. Завершение работы
Выход из среды окружения и корректное размонтирование:
exit
umount -R /mnt
reboot

Установка завершена.
Желаете, чтобы я подготовил список команд для быстрой установки драйверов видеокарты или настройки графической оболочки (Plasma/GNOME) после пе
рвого запуска?
