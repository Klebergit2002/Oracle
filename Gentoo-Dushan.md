### GENTOO LINUX /I3 - DUSHANS

#### CRIANDO AS PARTIÇÕES

```bash
$ sudo su
# Como root
$ parted -a optimal /dev/sda
(parted) mklabel gpt
(parted) unit mib
(parted) mkpart primary 1 512
(parted) name 1 EFI
(parted) mkpart primary 512 1012
(parted) name 2 boot
(parted) mkpart primary 1012 17396
(parted) name 3 swap
(parted) mkpart primary 17396 58356 # 178356
(parted) name 4 root
(parted) mkpart primary 58356 -1
(parted) name 5 home
(parted) print
(parted) q
```

#### FORMATANDO AS PARTIÇÕES

```bash
# Como root
# Instalar mkfs.xfs
$ apt update && apt upgrade
$ sudo apt-get install xfsprogs
#
$ mkfs.vfat -F 32 /dev/sda1
$ mkfs.ext2 /dev/sda2
$ mkswap /dev/sda3
$ mkfs.xfs /dev/sda4
$ mkfs.xfs -f /dev/sda4 # Se precisar forçar - overwrite
$ mkfs.ext4 /dev/sda5
```

#### BAIXANDO O STAGE 3

```bash
# Como root
$ mkdir /mnt/gentoo
$ mount /dev/sda4 /mnt/gentoo
$ cd /mnt/gentoo

# Baixar o stage3 64bits
$ apt install links
$ links gentoo.org # Stage 3 openrc 2021-05-23 194 MiB - Verificar no rodape da pagina
$ tar xpfv stage3* --xattrs-include='*.*' --numeric-owner
$ nano -w /mnt/gentoo/etc/portage/make.conf
```

#### ALTERANDO O MAKE.CONF

```ini
# Como root
# Alterar as seguintes linhas do arquivo
COMMON_FLAGS="-march=native -02 -pipe"
## COMMON_FLAGS="-march=ivybridge -02 -pipe"
## Adicionar apos FFLAGS
CPU_FLAGS_X86="mmx sse sse2"
## USE="alsa glamour"
USE="alsa"
## MAKEOPTS="-j5" 
MAKEOPTS="-j2" 
EMERGE_DEFAULT_OPTS=" --jobs 3 --with-bdeps=y --quiet --keep-going=y"
PORTAGE_TMPDIR="/var/tmp"
ACCEPT_LICENSE="*"
#VIDEO_CARDS="intel i915"
EDITOR=/usr/bin/nano
GRUB_PLATFORMS="efi-64"
```



```bash
# Como root
$ emerge -a mirrorselect
$ mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
$ mkdir --parents /mnt/gentoo/etc/portage/repos.conf

$ cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
$ cat /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
$ cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
#
$ mount --types proc /proc /mnt/gentoo/proc
$ mount --rbind /dev /mnt/gentoo/dev
$ mount --rbind /sys /mnt/gentoo/sys
$ test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
$ mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
$ chmod 1777 /dev/shm

# ENTRANDO NO NOVO AMBIENTE
$ chroot /mnt/gentoo/ /bin/bash
$ source /etc/profile
$ export PS1="(chroot) ${PS1}"
$ mkdir /boot # Se necessário
$ mount /dev/sda2 /boot
#
$ emerge-webrsync
$ emerge --sync # --quiet
$ eselect profile list
$ eselect profile set 5 # [5] default/linux/amd64/17.1/desktop (stable)
$ emerge -aDNuv @world # yes
#
$ ls /usr/share/zoneinfo/
# $ echo "Europe/Belgrade" > /etc/timezone
$ echo "America/Recife" > /etc/timezone
$ emerge --config sys-libs/timezone-data
# $ emerg --ask vim
$ nano -w /etc/locale.gen # Descomentar a linguagem que vai usar
$ locale-gen
$ eselect locale list
$ eselect locale set 3
$ nano -w /etc/env.d/02locale #LANG="en_US" --> LANG="en_US.utf8" e add LC_COLLATE="C"
$ eselect locale list
$ eselect locale set 4 # Deu erro
$ env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
$ emerge -a cpuid2cpuflags # yes
#
$ cpuid2cpuflags >> /etc/portage/make.conf #Editar make.conf e fazer as correções
$ nano /etc/portage/make.conf
# Fazer as correções em CPU_FLAGS_X86 e acrescentar a linha
# GENTOO_MIRRORS="http://mirror.switch.ch/ftp/mirror/gentoo/ http://mirrors.evowise.com/gentoo/ http://lug.mtu.edu/gentoo/"
$ emerge -a gentoo-sources genkernel usbutils dosfstools pciutils gentoolkit ufed eix axel xfsprogs # yes
# Deu problema com a instalação do genkernel
# O MIRROR http://mirror.switch.ch/ftp/mirror/gentoo/ TA QUEBRADO

##############
# PAREI AQUI
##############

$ etc-update # -3 yes yes (min 41)
$ vim /etc/genkernel.conf # Alterar MENUCONFIG="yes", CLEAN="no", MRPROPER="no" e descomentar MAKEOPTS e TMPDIR
$ vim /etc/fstab 
# Add linhas no fstab
## /dev/cdrom	/mnt/cdrom	ext4	noauto,user		0 0
# /dev/sda1		/boot/efi	vfat	noauto,noatime 	0 2
# /dev/sda2		/boot		ext2	defaults		0 2
# /dev/sda3		none		swap	sw				0 0
# /dev/sda4		/			xfs		noatime			0 1
# /dev/sda5		/home		ext4	noatime			0 2
# tmpfs 		/var/tmp	tmpfs 	rw,nosuid,noatime,nodev,size=8G,mode=1777 0	0
# tmpfs /var/tmp/portage	tmpfs 	rw,nosuid,noatime,nodev,size=8G,mode=775,uid=portage,gid=portage,x-mount.mkdir=775 0 0
```

#### CONFIGURANDO/COMPILANDO O KERNEL

```bash
# Como root
$ genkernel all
# Deixar o padrão

# Não fazer nada por enquanto
# Abre um menu de configurações do kernel
# Entrar na opção: 
# Processor type and features
#    Preemption Model (Preemtible Kernel (Low-Latency Desktop))
#       (X) Preemptible kernel (Low-Latency Desktop)
#    	Peemptible Kernel (Low-Latency Desktop)
#    [ ] AMD Secure Memory Encryption (SME) support (Desmarcar)
#    [ ] AMD microcode loading support (Desmarcar)
# Enable the block layer
# 	 Partition types
#
$ vim /etc/conf.d/modules
# Adicionar a linha
# modules="overlay"
$ emerge -a linux-firmware # yes
```



```bash
# Como root
$ vim /etc/conf.d/hostname # Alterar: hostname="dushan"
$ emerge --ask --noreplace net-misc/netifrc # yes
$ ifconfig # Ver o enp...
$ vim /etc/conf.d/net # Add: config_enp0s25="dhcp" e dns_domain_lo="homenetwork"
$ cd /etc/init.d
$ ln -s net.lo net.enp0s25
$ rc-update add net.enp0s25 default
$ cd
$ vim /etc/hosts # Alterar: 127.0.0.1 dushan.homenetwork dushan localhost
$ passwd
$ vim /etc/conf.d/hwclock # Alterar: clock="local"
$ emerge -a syslog-ng cronie mlocate logrotate dhcpcd # yes
$ rc-update add syslog-ng default
$ rc-update add cronie default
$ rc-update add sshd default
#
$ mkfs.fat -F32 /dev/sda1
$ cd /boot
$ mkdir efi/
$ mount /dev/sda1 /boot/efi
$ mount -o remount,rw /boot/efi
$ cd /etc/portage/package.use/
$ ls # Aparece: zz-autounmask
$ echo 'sys-boot/grub:2 mount truetype' >> /etc/portage/package.use/grub:2
$ ls # Aparece: grub:2 zz-autounmask
$ vim grub\:2 # Conferir os dados do echo
#
$ emerge --ask --newuse --deep --verbose sys-boot/grub:2 os-prober # yes
$ mount -o remount,rw /sys/firmware/efi/efivars
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable
$ grub-mkconfig -o /boot/grub/grub.cfg
$ useradd -m -G users,wheel,video,audio,root,sys,disk,adm,bin,daemon,tty,portage,console,usb,input,lp,uucp -s /bin/bash dushan
$ passwd dushan
$ lspci grep | -i VGA
$ vim /etc/portage/make.conf # Descomentar: VIDEO_CARDS
$ eselect profile list
$ eselect profile set 16 # [16] default/linux/amd64/17.0/desktop (stable)
# thats it for now, and now i will reboot to finish instalation witn xorg-server and i3 
# window manager 
$ reboot
```

#### INSTALANDO A INTERFACE GRÁFICA - I3

```bash
# Como dushan
$ eselect profile list
$ eselect profile set 16
$ emerge --ask xorg-server i3 i3status # yes
$ etc-update # -3 yes
$ emerge --ask xorg-server i3 i3status # yes
#
$ cd /home
$ su 
$ mkhomedir_helper dushan
$ exit
$ cd dushan
$ ls -la
$ vim .xintrc # Add linha: exec i3
$ xinit
# Como root
$ su
$ Xorg-configure
$ cd /root
$ ls # Aparece: xorg.conf.new
$ mv xorg.conf.new /etc/X11/xorg.conf
$ ls /etc/X11
$ exit
# Como dushan
$ xinit
# Como root
$ su
$ emerge  --ask xterm dmenu # yes
$ exit
# Como dushan
$ startx
```

