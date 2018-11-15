<h1 align ="center">Dual boot Arch Linux Windows 10 UEFI</h1>

Este guia destina-se a ajudar alguém a instalar a distribuição Arch Linux  em seu Computador. O guia pressupõe que você tenha alguma familiaridade com o sistema linux e esteja confortável, trabalhando a partir da linha de comando, mas isso não exige que você seja um especialista. Aprendemos muito fazendo e se você quiser saber mais sobre como o linux opera, o Arch Linux é uma excelente opção por muitas razões.




Para criar um USB bootable usando o comando (dd) no Linux:

```

# dd bs=4M if='/lugar_onde_esta_seu_iso' of='/lugar_para_o_qual_copiar' status=progress && sync

```

(Substitua o X pela letra do seu dispositivo ex: 'sdc' 'sdd') use: fdisk -l

```

Exemplo: # dd bs=4M if='/home/joao/archlinux-2018.08.01.iso' of='/dev/sdX' status=progress && sync

```

Executamos esse comando para verificar se estamos no modo de inicialização: ( UEFI ):

```

efivar -l

```

Antes de instalar o Arch Linux, verifique se o computador está conectado à Internet.

```

localectl set-x11-keymap br abnt2

```
```

dhcpcd (se estiver com cabo)

```

```

wifi-menu (Caso esteja no WI-FI)

```

```

ping archlinux.org

```

Atualize o relógio do sistema.

```

timedatectl set-ntp true

```

Verificar: (opcional)

 ```
 
 timedatectl status
 
 ```

<h2>Particionamento de Disco</h2>

Identificar los discos:

 ```
 
 lsblk
 
 ```
Verificar a tabela de partições:

```

gdisk /dev/sda

```
Estou usando GPT em vez de MBR 

Criar a partição swap :

```

gdisk /dev/sda
n
ENTER
ENTER
+2G
8200
W
Y

```
Criar a partição / :

```

gdisk /dev/sda
n
ENTER
ENTER
+10G
8304
W
Y

```
Criar a partiçãon /home :

```

gdisk /dev/sda
n
ENTER
ENTER
ENTER
8302
W
Y

```
Verificar:

```

lsblk

```
Formatar a partição SWAP :

```

mkswap /dev/sda5

```
Ativar a partição SWAP :

```

swapon /dev/sda5

```
Formatar a partição (/root) :

```

mkfs.ext4 /dev/sda6

```
Formatar a partição (/home):

```

mkfs.ext4 /dev/sda7

```

Montar  a partição (/root) em /mnt :

```

mount /dev/sda6 /mnt

```

Criar o diretório /home :

```

mkdir -p /mnt/home

```
Montar partición /home :

```

mount /dev/sda7 /mnt/home

```

Agora monte a partição: (/boot)   (UEFI)

```

mkdir -p /mnt/boot/efi && mount /dev/sdaX /mnt/boot/efi

```



Verifique as partições com este comando

```

lsblk /dev/sda

```

<h2>Instalar os pacotes base do Arch Linux</h2>

```

pacstrap -i /mnt base base-devel

```
Isto  iniciará uma instalação dos pacotes base (300 MiB aprox.)

<h2>Configurar fstab:</h2>

```

genfstab -U -p /mnt >> /mnt/etc/fstab

```
Você deve sempre verificar se a entrada fstab está correta ou não, que será capaz de inicializar em seu sistema. Para verificar a entrada fstab, execute:

```

cat /mnt/etc/fstab

```

Se tudo estiver OK você deve ver o root e o home montado.
Agora é hora de mudar para o diretório root recém-instalado para configurá-lo.

```

arch-chroot /mnt

```
Configurar KEYMAP:

A variável KEYMAP é especificada no arquivo /etc/vconsole.conf . Ele define qual layout de teclado, será usado nos consoles virtuais. Execute este comando:

```

echo -e "KEYMAP=br-abnt2\nFONT=Lat2-Terminus16\nFONT_MAP=" > /etc/vconsole.conf

```

<h2>Configurações de idioma e fuso horário</h2>

Para configurar o idioma do sistema, execute o seguinte comando:

```

sed -i  '/pt_BR/,+1 s/^#//' /etc/locale.gen

```
Agora execute:

```

locale-gen

```
```

echo LANG=pt_BR.UTF-8 > /etc/locale.conf

```
```

export LANG=pt_BR.UTF-8 

```
Para ver todos os fusos horários disponíveis da América:

```

ls /usr/share/zoneinfo/America

```

Agora você pode configurar a sua zona:

```

ln -sf /usr/share/zoneinfo/America/Bahia  /etc/localtime

```

Vamos agora configurar o relógio do hardware, apenas no caso de termos uma data errada:

```

hwclock -w -u

```

```

echo -e "NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org``\nFallbackNTP=0.pool.ntp.org 1.pool.ntp.org 0.fr.pool.ntp.org" >> /etc/systemd/timesyncd.conf

```

<h2>Configurar o repositório</h2>


Com este comando habilitamos o repositório multlib:

```

sed -i  '/multilib\]/,+1  s/^#//'  /etc/pacman.conf

```
```

pacman -Sy

```

<h2>Defina seu nome de host</h2>


```

echo ArchLinux > /etc/hostname

```
<h2>Configurando a Conexão </h2>


```

ip link ou ls /sys/class/net

```

```

systemctl enable dhcpcd (rede cabeada)

```


Wifi ( Instalar componentes wifi )

```

pacman -S wpa_supplicant wpa_actiond dialog iw networkmanager 

```


```

systemctl enable NetworkManager

```

<h2>Criar Usuário (s)</h2>
useradd -m -g [initial_group] -G [additional_groups] -s [login_shell] [username] 

```

useradd -m -G audio,dbus,lp,network,optical,power,storage,users,video,wheel -s /bin/bash nomedousuario 

```
```

passwd nomedousuario

```

Instale o bash-completion  para que o Arch complete os comandos dos nomes dos pacotes.

```

pacman -S bash-completion

```

Permitir que os usuários no grupo wheel, sejam capazes de executar tarefas administrativas com o sudo:

```

sed -i '/%wheel ALL=(ALL) ALL/s/^#//' /etc/sudoers

```

<h2>Instalar Boot-loader (grub) ( UEFI )</h2>


```

mkinitcpio -p linux

```
```

pacman -S grub efibootmgr 

```
```

grub-mkconfig -o /boot/grub/grub.cfg

```

```

grub-install /dev/sda

```
```

ls -l /boot/efi/EFI/arch

```



<h2>Desmontar as partições e reiniciar:</h2>

```

exit

```
```

umount -R /mnt

```
```

reboot

```

