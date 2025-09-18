## Minimal QEMU boot of a self-built Linux kernel

This guide shows how to boot your freshly compiled Linux kernel in QEMU using a tiny BusyBox-based `initramfs`. It uses variables/placeholders so you don’t hard-code paths or addresses that can change between builds.

### Prerequisites (Ubuntu/Debian)

```bash
sudo apt-get update && sudo apt-get install -y \
  qemu-system-x86 qemu-system-arm qemu-system-misc \
  busybox-static cpio gzip
```

### Define paths and variables (adjust for your environment)

```bash
# REQUIRED: path to your kernel source tree (where you ran make)
export KERNEL_SRC="/path/to/your/linux"

# Work directory for the initramfs artifact
export WORKDIR="$HOME/qemu-min"
mkdir -p "$WORKDIR"

# Target architecture (choose one): x86_64 or arm64
export ARCH=x86_64   # or: arm64

# Kernel image path (depends on ARCH)
if [ "$ARCH" = "x86_64" ]; then
  export KIMG="$KERNEL_SRC/arch/x86/boot/bzImage"
elif [ "$ARCH" = "arm64" ]; then
  export KIMG="$KERNEL_SRC/arch/arm64/boot/Image"
else
  echo "Unsupported ARCH: $ARCH" && exit 1
fi

# Output initramfs archive path
export INITRAMFS="$WORKDIR/initramfs.cpio.gz"
```

### Build a tiny BusyBox `initramfs`

```bash
INITDIR="$WORKDIR/initramfs"
rm -rf "$INITDIR" && mkdir -p "$INITDIR"/{bin,sbin,etc,proc,sys,dev,usr/bin,usr/sbin}

# Place BusyBox and provide a shell
cp "$(command -v busybox)" "$INITDIR/bin/busybox"
ln -sf busybox "$INITDIR/bin/sh"

# Minimal /init (PID 1)
cat > "$INITDIR/init" <<'EOF'
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev 2>/dev/null || /bin/busybox mdev -s
exec /bin/sh
EOF
chmod +x "$INITDIR/init"

# Pack as initramfs (newc) and compress
( cd "$INITDIR" && find . -print0 | cpio --null -ov --format=newc | gzip -9 ) > "$INITRAMFS"
```

### Boot in QEMU

Pick the command that matches your `ARCH`. These use a serial console and no graphics.

```bash
if [ "$ARCH" = "x86_64" ]; then
  qemu-system-x86_64 \
    -m 2048 -smp 2 \
    -kernel "$KIMG" \
    -initrd "$INITRAMFS" \
    -append "console=ttyS0 earlyprintk=ttyS0 nokaslr" \
    -nographic -serial mon:stdio \
    -accel kvm
elif [ "$ARCH" = "arm64" ]; then
  qemu-system-aarch64 \
    -M virt -cpu cortex-a57 -m 2048 -smp 2 \
    -kernel "$KIMG" \
    -initrd "$INITRAMFS" \
    -append "console=ttyAMA0 nokaslr" \
    -nographic -serial mon:stdio \
    -accel kvm
fi
```

Notes:
- If KVM isn’t available on your host, remove `-accel kvm` (QEMU will use TCG and be slower).
- Ensure your kernel config enables `CONFIG_DEVTMPFS` and `CONFIG_DEVTMPFS_MOUNT` (otherwise the init script falls back to BusyBox `mdev`).

### Interacting and exiting

- You will land in a shell provided by BusyBox (`/bin/sh`).
- To exit QEMU in `-nographic` mode: press Ctrl-a then x.
- QEMU monitor: Ctrl-a then c, then `quit`.
- From inside the guest: run `poweroff` (if supported).

- From the host (last resort):
```bash
pkill -f qemu-system
```

### About addresses and paths

- Do not hard-code memory addresses or load addresses from previous boots; they can change between builds and runs (e.g., due to KASLR or different kernel layouts).
- This guide deliberately uses variables (`$KERNEL_SRC`, `$KIMG`, `$INITRAMFS`, `$ARCH`) instead of fixed absolute paths. Adjust these variables rather than editing commands.

### Troubleshooting

- No console output: confirm `console=ttyS0` (x86_64) or `console=ttyAMA0` (arm64) is on the kernel command line.
- Kernel image not found: verify `KIMG` exists for your `ARCH`.
- Stuck at early boot: try removing `-accel kvm`, and/or add `earlyprintk` (x86_64) for more logs.


