# Fasi 1 — Base Arch uppsetning

**Markmið:** Bootable Arch kerfi á 120 GB Kingston SSD, með notanda `arnar`, NetworkManager, og Wi-Fi tilbúið. Engin GUI ennþá — það kemur í Fasa 2.

**Forsendur:**
- Komin að root prompt á Arch live USB (`root@archiso ~#`)
- Wi-Fi SSID + lykilorð við hendina
- Hefur staðfest að einu disk-inn á NUC sé Kingston A400 (`/dev/sda` venjulega)

**⚠️ Eyðileggjandi skref:** Fasi 1 þurrkar út diskinn. Aldrei keyra `dd`, `mkfs`, `cfdisk` skipanir án þess að lesa þær fyrst.

---

## 1. Athuga UEFI mode og lyklaborð

```bash
# Á að gefa lista af files. Ef tóm (eða villa) — bootaði í Legacy. Þá: reboot, fara í BIOS, kveikja á UEFI only.
ls /sys/firmware/efi/efivars

# Íslenskt lyklaborð í consolanum
loadkeys is-latin1

# Stærri letur ef þú átt erfitt að lesa
setfont ter-128b
```

## 2. Tengjast Wi-Fi með iwctl

```bash
iwctl
```

Innan iwctl prompt-sins:

```
[iwd]# device list
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect <SSID>
# Slær inn lykilorð þegar beðið er
[iwd]# exit
```

Staðfesta:

```bash
ping -c 3 archlinux.org
```

> Þrjú svör → tenging virkar. Engin svör → enduráætla.

## 3. Tími

```bash
timedatectl set-ntp true
timedatectl status
```

## 4. Disk skoðun og partitioning

```bash
lsblk -o NAME,SIZE,MODEL,TRAN
```

Kingston A400 á að birtast sem `/dev/sda` með stærð 111.8 G (eða svipað).
> Ef þú sérð fleiri en einn disk — staðfestu rétta nafnið! USB-inn er líka í `lsblk` (venjulega `/dev/sdb`, með `TRAN=usb`).

**Eyða gömlum partitions (eyðileggjandi!):**

```bash
wipefs --all /dev/sda
sgdisk --zap-all /dev/sda
```

**Búa til partitions með `sgdisk`** (skript-vænt, minna villuhætt en cfdisk):

```bash
# Partition 1: 1 GiB EFI System (FAT32)
sgdisk --new=1:0:+1G --typecode=1:ef00 --change-name=1:EFI /dev/sda

# Partition 2: rest of disk, Linux filesystem (Btrfs)
sgdisk --new=2:0:0 --typecode=2:8300 --change-name=2:ROOT /dev/sda

# Athuga
sgdisk --print /dev/sda
lsblk /dev/sda
```

Á að sýna:
```
/dev/sda1   1G   EFI System
/dev/sda2  ~111G Linux filesystem
```

## 5. Formatta

```bash
mkfs.fat -F32 -n EFI /dev/sda1
mkfs.btrfs -L AddiOS /dev/sda2
```

## 6. Búa til Btrfs subvolumes

```bash
# Mount root tímabundið
mount /dev/sda2 /mnt

# Búa til subvolumes
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@var_log
btrfs subvolume create /mnt/@var_cache
btrfs subvolume create /mnt/@swap

btrfs subvolume list /mnt

# Unmount
umount /mnt
```

## 7. Mount-a með compression

```bash
# Mount aðal subvolume
mount -o noatime,compress=zstd:1,subvol=@ /dev/sda2 /mnt

# Búa til mount points
mkdir -p /mnt/{home,boot,.snapshots,swap}
mkdir -p /mnt/var/{log,cache}

# Mount sub-mount points
mount -o noatime,compress=zstd:1,subvol=@home      /dev/sda2 /mnt/home
mount -o noatime,compress=zstd:1,subvol=@snapshots /dev/sda2 /mnt/.snapshots
mount -o noatime,subvol=@var_log                    /dev/sda2 /mnt/var/log
mount -o noatime,subvol=@var_cache                  /dev/sda2 /mnt/var/cache
mount -o noatime,subvol=@swap                       /dev/sda2 /mnt/swap

# EFI partition á /boot
mount /dev/sda1 /mnt/boot

# Staðfesta
findmnt /mnt
```

## 8. Pacstrap — grunnpakkar

```bash
pacstrap -K /mnt \
  base base-devel \
  linux linux-firmware intel-ucode \
  btrfs-progs \
  networkmanager \
  sudo nano vim git \
  man-db man-pages texinfo \
  zram-generator
```

Tekur 5–15 mínútur eftir nettengingu.

## 9. fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab

# Athuga — á að innihalda 6 subvolumes + EFI
cat /mnt/etc/fstab
```

> Ef eitthvað lítur skrítið út, ekki halda áfram. Spyrðu mig.

## 10. chroot inn í nýja kerfið

```bash
arch-chroot /mnt
```

Næsti prompt verður `[root@archiso /]#` — núna ertu inni í **nýja kerfinu**, ekki live USB lengur.

## 11. Tímabelti og klukka

```bash
ln -sf /usr/share/zoneinfo/Atlantic/Reykjavik /etc/localtime
hwclock --systohc
```

## 12. Locale

```bash
# Opna locale.gen
nano /etc/locale.gen
```

Finna og **un-comment-a** (taka burt `#`) þessar tvær línur:
```
en_US.UTF-8 UTF-8
is_IS.UTF-8 UTF-8
```

`Ctrl+O` til að vista, `Ctrl+X` til að loka.

```bash
locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "KEYMAP=is-latin1" > /etc/vconsole.conf
```

## 13. Hostname og hosts

```bash
echo "AddiOS" > /etc/hostname

cat > /etc/hosts <<'EOF'
127.0.0.1   localhost
::1         localhost
127.0.1.1   AddiOS.localdomain AddiOS
EOF
```

## 14. mkinitcpio

Default Arch HOOKS virka með Btrfs. Engar breytingar þarf.

```bash
mkinitcpio -P
```

Ef þú sérð warnings um vantar firmware — það er í lagi (er bara fyrir vélbúnað sem þú átt ekki).

## 15. Lykilorð fyrir root og notanda

```bash
# Root lykilorð
passwd

# Búa til arnar með bash sem default shell
useradd -m -G wheel,audio,video,storage,input,network -s /bin/bash arnar
passwd arnar
```

Sudo aðgangur fyrir `wheel` hópinn:

```bash
EDITOR=nano visudo
```

Finna og un-comment-a línuna:
```
%wheel ALL=(ALL:ALL) ALL
```

`Ctrl+O`, `Ctrl+X`.

## 16. systemd-boot

```bash
bootctl install
```

**Loader config:**

```bash
cat > /boot/loader/loader.conf <<'EOF'
default  arch.conf
timeout  3
console-mode max
editor   no
EOF
```

**Boot entry — sækja UUID fyrst:**

```bash
ROOT_UUID=$(blkid -s UUID -o value /dev/sda2)
echo "ROOT_UUID = $ROOT_UUID"
# Á að gefa UUID streng eins og: 1234abcd-...
```

**Aðal boot entry:**

```bash
cat > /boot/loader/entries/arch.conf <<EOF
title   AddiOS
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=UUID=$ROOT_UUID rootflags=subvol=@ rw quiet splash
EOF
```

**Fallback (öryggisnet ef aðal initramfs brotnar):**

```bash
cat > /boot/loader/entries/arch-fallback.conf <<EOF
title   AddiOS (fallback)
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux-fallback.img
options root=UUID=$ROOT_UUID rootflags=subvol=@ rw
EOF
```

**Athuga:**

```bash
bootctl list
cat /boot/loader/loader.conf
cat /boot/loader/entries/arch.conf
```

## 17. Virkja services

```bash
systemctl enable NetworkManager
systemctl enable systemd-timesyncd
systemctl enable fstrim.timer
```

## 18. Exit chroot, unmount, reboot

```bash
exit
umount -R /mnt
reboot
```

Þegar NUC-inn restartast → **dragðu USB-inn úr** áður en hann bootar aftur.

---

## Eftir reboot — fyrsta innskráning

Þegar SSD bootar:
1. systemd-boot menu birtist í 3 sekúndur (veldu „AddiOS" eða bíddu)
2. Login prompt á consolanum
3. Loggaðu inn sem `arnar` með lykilorðinu þínu

**Tengjast Wi-Fi á nýja kerfinu:**

```bash
nmcli device status
nmcli device wifi list
nmcli device wifi connect "<SSID>" password "<PASSWORD>"

# Staðfesta
ping -c 3 archlinux.org
```

**Lokaprófun:**

```bash
# Athuga RAM uppstillingu (sú spurning sem var óvíst í Phase 0)
sudo dmidecode -t memory | grep -E "Size|Speed|Locator"

# Athuga Btrfs subvolumes mountaða
findmnt | grep btrfs

# Athuga fría plássið
df -h /
btrfs filesystem usage /

# Athuga snapshot kerfið
btrfs subvolume list /
```

Þegar þú færð svör frá öllu þessu — segðu Claude Code og við förum í **Fasa 2** (KDE Plasma + dev tools + zram + snapper).

---

## Bilanaleit

**„No bootable device" eftir reboot:**
- Líklega er `bootctl install` ekki búin að keyra rétt eða EFI partition er ekki mountuð á `/boot` í chroot
- Boot inn í USB aftur, `arch-chroot /mnt` (eftir mount), `bootctl install` aftur

**Btrfs villa við boot („cannot find root"):**
- Athuga `arch.conf` — er `rootflags=subvol=@` rétt? Er UUID rétt?
- Boot fallback entry-uð til að sjá villuboð

**Wi-Fi virkar ekki eftir login:**
- `systemctl status NetworkManager` — er service-inn virkur?
- `rfkill list` — er Wi-Fi blocked? `rfkill unblock wifi`

**Locale villuboð („LC_ALL: cannot change locale"):**
- Líklega vantar `locale-gen` skref. Keyrðu aftur í chroot.
