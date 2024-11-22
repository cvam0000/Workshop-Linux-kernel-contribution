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
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/ && cd linux
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
