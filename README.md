# Instalacao_ArchLinux
Guia de instalação PT-BR para instalacao do Arch linux criptografado

##### ARCH LINUX GUIA DE INSTALAÇÃO COM LUKS CRIPTOGRAFIA (PT-BR) #####
	
	Este guia abordará o processo para instalação do sistema arch linux usando Luks para criptografia. Todo o sistema será criptografado
exceto o /boot. Concluindo que o pendrive de boot já está configurado e o sistema para instalação foi bootado, vamos começar:

1 - Formatar o hd com " dd if=/dev/zero of=/dev/sdX count=1 "

2 - Criar duas partições primarias utilizando o cfdisk ou fdisk, uma para o /boot de no minimo 100mb do tipo ext4 (Marque a tag bootable), e outra para o 
volume de criptografia do tipo Linux LVM.

3 - Carregar a correta configuração de teclas para o seu teclado. (Por padrão o arch linux inicia com o teclado americano - US). A localização dos
mapas de teclado pode ser localizada no caminho: "/usr/share/kbd/keymaps/". Caso seu teclado seja PT-BR-ABNT2 utilize o comando abaixo:
"loadkeys /usr/share/kbd/keymaps/i386/qwerty/br-abnt2" Caso ocorra um erro acesse o caminho com "cd" para verificar o nome correto do mapa de teclas.

4 - Verificar o nome da sua placa, pode ser verificado com o comando "ip addr".

4 - Conectar-se na internet, caso esteje usando uma conexão Wi-fi, utilize o comando "wifi-menu". Caso esteje usando uma conexão Ethernet terá de
configurar um novo perfil netctl. Os arquivos de perfil netctl são armazenados em /etc/netctl/ e exemplo arquivos de configuração estão 
disponíveis em /etc/netctl/examples/. Para uma conexão ethernet simples com dhcp copie o perfil ethernet-dhcp em /etc/netctl/examples/ para /etc/netctl/
edite com seu editor de preferência para configurar o nome correto de sua placa.

5 - Checar se o perfil foi reconhecido com o comando "netctl list" e depois inicie o perfil com o comando " netctl start nome_do_perfil ".

6 - Carregar o modulo de criptografia com o comando "modprobe dm-crypt"

7 - Criptografar a grande partição (sda2) com o comando " cryptsetup -c serpent -y -s 256 luksFormat /dev/sda2 "

8 - Desbloquear a partição criptografada com o comando " cryptsetup luksOpen /dev/sda2 lvm "

9 - Criar um volume físico, grupo de volume, os volumes lógicos (exemplo abaixo):
		pvcreate /dev/mapper/lvm
		vgcreate vg00 /dev/mapper/lvm
		lvcreate -L 65GB -n root main
		lvcreate -L 8GB -n swap main
		lvcreate -l 100%FREE -n home main
	
10 - Formatar os volumes de com os comandos abaixo:
		mkswap /dev/mapper/vg00-swap
		mkfs.ext4 /dev/mapper/vg00-root
		mkfs.ext4 /dev/mapper/vg00-home
	
11 - Montar os volumes:
		mount /dev/mapper/vg00-root /mnt
		mkdir /mnt/boot
		mount /dev/sda1 /mnt/boot
		mkdir /mnt/home
		mount /dev/mapper/vg00-home /mnt/home

12 - Instalar sistema basico " pacstrap /mnt base base-devel ".

13 - Instalar o GRUB 2 para o /mnt " pacstrap /mnt grub-bios ".

14 - Ativar SWAP para ser reconhecida no fstab " swapon /dev/mapper/vg00-swap ".

15 - Gerar o arquivo fstab " genfstab -p -U /mnt > /mnt/etc/fstab ".

16 - Alterar o bash para o root do sistema no HD " arch-chroot /mnt ".

17 - Excluir o # na frente do idioma de sua escolha (por exemplo de_DE.UTF-8 UTF-8) em locale.gen e gerar o locale:
		vi /etc/locale.gen
		locale-gen
		echo LANG=de_DE.UTF-8 > /etc/locale.conf
		export LANG=de_DE.UTF-8

18 - Gerar /etc/vconsole.conf com as 3 linhas a seguir para vincular suas chaves corretamente: (Keymap é o nome do mapa do seu teclado localizado em
/usr/share/kbd/keymaps)
		KEYMAP="de-latin1-nodeadkeys"
		FONT=Lat2-Terminus16
		FONT_MAP=

19 - Crie um link simbólico /etc /localtime para o seu arquivo de zona /usr/share/zoneinfo/ <zona> / <subzona>:
		" ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime "
		
20 - Definir seu hostname: " echo archserv > /etc/hostname "

21 - Editar o arquivo /etc/mkinitcpio.conf: Coloque "keymap", "encrypt" e "lvm2" (nessa ordem!) Antes de "filesystems" no vetor HOOKS. Muito importante
esse passo!!!

22 - Regerar o ramdisk: " mkinitcpio -p linux ".

23 - Instalar o GRUB 2 em um dispositivo e não em uma partição ou volume: " grub-install /dev/sda ".

24 - Em /etc/default/grub editar a linha GRUB_CMDLINE_LINUX = "" para GRUB_CMDLINE_LINUX = 
"cryptdevice = /dev/sda2:main", em seguida, execute: " grub-mkconfig -o /boot/grub/grub.cfg ". Pode aparecer vários warnings na tela, mas se
não aparecer nenhum ERROR está tudo OK. Para ter certeza se tudo ocorreu bem verifique se o arquivo /boot/grub/grub.cfg foi gerado.

25 - Setar a senha para o root: " passwd ".

26 - Sair do chroot " exit ".

27 - Desmontar tudo:
		umount /mnt/boot
		umount /mnt/home
		umount/mnt

28 - Se algum dispositivo aparecer ocupado pode mandar um reboot que não da nada. Pronto! o sistema deverá estar instalado, só fazer a configuração
de acordo com sua preferência.
	
