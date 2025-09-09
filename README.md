# Rootconf-linuxKernelCompilation

#### Guide: Building and Installing a Linux Kernel from Source

##### Install the prerequsite
```
#Debian based users
sudo apt install bc binutils bison dwarves flex gcc git gnupg2 gzip libelf-dev libncurses5-dev libssl-dev make openssl pahole perl-base rsync tar xz-utils
```

```
#Arch based users
sudo pacman -S base-devel bc coreutils cpio gettext initramfs kmod libelf ncurses pahole perl python rsync tar xz
```

##### Step 1: Clone the Linux Kernel Source
```
git clone --depth=1 https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/ && cd linux
```

##### Step 2: Copy the Current Kernel Configuration
```
cp /boot/config-$(uname -r) .config
```

##### Step 3: Modify the Kernel Configuration
```
make menuconfig  # Make your changes and save them.
```

##### Step 4: Compile the Kernel
```
sudo make -j8  # Adjust the -j flag based on the number of CPU cores.
```

##### Step 5: Install the Kernel and Update GRUB This step will be done tomorrow.
## Don't worry your grub will be safe :)

##### Step 5: Install new build.
```
sudo make modules_install -j 4 && sudo make install
```
- Installs the kernel modules (from make modules) into /lib/modules/<kernel-version>.
- Installs the compiled kernel into the boot directory (typically /boot)

##### Step 6: Update GRUB
```
sudo update-grub
```
- Check you bootloader config in /boot/grub/grub.cfg










# Configure git send-email
##### install git send-email
```
apt install git-email
```
##### setup ~/.gitconfig
```
[user]
    name = First Last
    email = my_email@gmail.com

[format]
	signoff = true

[core]
	editor = vim

[sendemail]
    smtpServer = smtp.gmail.com
    smtpServerPort = 587
    smtpEncryption = tls
    smtpUser = my_email@gmail.com
    smtppass = xxxxxxxxxxxxxxxxxx

[credential]
    helper = store
```
##### Setup your mail client if its gmail if not you are on your own.
```
Resources : https://gist.github.com/winksaville/dd69a860d0d05298d1945ceff048ce46
          : https://stackoverflow.com/questions/68238912/how-to-configure-and-use-git-send-email-to-work-with-gmail-to-email-patches-to

```


