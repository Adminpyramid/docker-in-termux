# Installing Docker in Termux
This repository contains instructions on how to install Docker in [Termux](https://termux.com/), a powerful terminal emulator for Android.

# Prerequisites
Before proceeding with the installation, make sure you have the following prerequisites:
- An Android device with Termux installed. You can download Termux from the [F-Droid](https://f-droid.org/packages/com.termux/) app store.
- Stable internet connection.

# Installation Steps
Follow the steps below to install Docker in Termux:
1. Open Termux on your Android device.

2. Update and upgrade the packages by running the following command:
```bash
pkg update -y && pkg upgrade -y
```

3. Install the necessary dependencies by running the following command:
```bash
pkg install qemu-utils qemu-common qemu-system-x86_64-headless wget -y
```

4. Create a seperate directory:
```bash
mkdir alpine && cd alpine
```

5. Download Alpine Linux 3.12 (virt optimized) ISO:
```bash
wget http://dl-cdn.alpinelinux.org/alpine/v3.12/releases/x86_64/alpine-virt-3.12.3-x86_64.iso
```

6. Create disk (note it won't actually take 5GB of space, more like 500-600MB):
```bash
qemu-img create -f qcow2 alpine.img 5G
```

7. Boot it up:
```bash
qemu-system-x86_64 -machine q35 -m 1024 -smp cpus=2 -cpu qemu64 \
  -drive if=pflash,format=raw,read-only,file=$PREFIX/share/qemu/edk2-x86_64-code.fd \
  -netdev user,id=n1,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 \
  -cdrom alpine-virt-3.12.3-x86_64.iso \
  -nographic alpine.img
```

8. Login with username ``root`` (no password)

9. Setup network (press Enter to use
```
```bash
localhost:~# ifup eth0
```
if that not working 
```bash
localhost:~#cd /etc/network
```
write the interfaces file
and put this content
```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet dhcp

```
This can be done using nano, but here is not installed so 
just use #echo command as
copy the content, type echo "...pasteall content here..." > interfaces

The test by enable the interface as 
```bash
ifup eth0
```
10. Create an answerfile to speed up installation:
```bash
wget https://gist.githubusercontent.com/oofnikj/e79aef095cd08756f7f26ed244355d62/raw/answerfile
```
> **NOTE:** You you see any error like this: ``wget: bad address 'gist.githubusercontent.com'``. Then run this command
> ```bash
> echo -e "nameserver 192.168.1.1\nnameserver 1.1.1.1" > /etc/resolv.conf
> ```

11. Patch ``setup-disk`` to enable serial console output on boot:
```bash
sed -i -E 's/(local kernel_opts)=.*/\1="console=ttyS0"/' /sbin/setup-disk
```

12. Run setup to install to disk
```bash
setup-alpine -f answerfile
```

13. Once installation is complete, power off the VM (command ``poweroff``)

14. Boot again without cdrom:
```bash
qemu-system-x86_64 -machine q35 -m 1024 -smp cpus=2 -cpu qemu64 \
  -drive if=pflash,format=raw,read-only,file=$PREFIX/share/qemu/edk2-x86_64-code.fd \
  -netdev user,id=n1,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 \
  -nographic alpine.img
```

15. Update system and install docker:
```bash
apk update && apk add docker
```

16. Start docker:
```bash
service docker start
```

17. Enable docker on boot:
```bash
rc-update add docker
```

18. Check docker install successfully or not:
```bash
docker run hello-world
```

# Some useful keys
- ``Ctrl+a x``: quit emulation
- ``Ctrl+a h``: toggle QEMU console

# Usage
Now that Docker is installed in Termux, you can start using it to manage and run containers on your Android device. Refer to the official [Docker documentation](https://docs.docker.com/) for more information on how to use Docker.

# Contributing
If you encounter any issues during the installation process or have suggestions for improvements, please feel free to open an issue or submit a pull request.

# Acknowledgment
- This article inspired from: https://gist.github.com/oofnikj/e79aef095cd08756f7f26ed244355d62

# License
This project is licensed under the [MIT License](LICENSE).
