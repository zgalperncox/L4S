# L4S Linux Kernel Setup

Contact zack.galpern@cox.com or Zack Galpern on Teams for questions/comments. These instructions are based off Neal Cardwell's 'L4S Linux Testing Setup: A Public Recipe for How to Set Up a Linux Machine for L4S Testing of TCP Prague and TCP BBRv2'.


## Build
On a PC with Linux installed, install tools:

```
sudo apt install git libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm
```

Clone Linux L4S repo and checkout latest branch:
```
git clone https://github.com/google/bbr.git
cd bbr
git checkout l4s-testing-2023-02-23-v3
```

If you receive an error related to 'clang' not existing:

```
sudo apt update && sudo apt upgrade
sudo apt install clang
```
Then build the .deb kernel:
```./build.sh```

Verify the kernel was built:
```ls -l ../*deb```

There should be three .deb files generated:

- linux-headers-5.15.72+l4s+testing...
- linux-image-5.15.72+l4s+testing...
	- This one contains the kernel
- linux-libc-deb-5.15.72+l4s+testing

## Install Kernel
The kernel should be installed on either the same or different Linux machine. Physical access to the machine is necessary in order to boot the kernel (I've had to do it each time).

On the original build machine, secure copy the files over to the test machine (I would recommend **NOT** placing these in /tmp/ in case files are deleted and need to restart):

```
scp ../linux-headers-(autocomplete) ../linux-image-(autocomplete) ../linux-libc-(autocomplete) user@destination:location
```

On the test machine, cd into the location where the files were placed and install the .deb files:

```
cd location
sudo dpkg --install linux-image(autocomplete) linux-h(autocomplete) linux-libc(autocomplete)
```

Now you must reboot the machine and select the new kernel. (It's extremely important to **DISABLE** secure boot):

- Reboot the machine
	- On startup, click F10 to open the boot options menu and navigate to the Boot options. Disable secure boot. Save and exit.
	- Spam the esc key while the Intel NUC logo shows up, and when it disappears immediately stop pressing the key. This will open the boot menu. 
		- If you are entered into the grub command line, quickly type 'normal', hit enter, then **IMMEDIATELY** spam escape again. This needs to be quick so that the machine doesn't skip over the grub menu.
	- Select 'Advanced options for Ubuntu' and then notice the kernel name that includes l4s-testing and (recovery mode). This will boot the machine with the correct kernel. Before selecting the kernel, read the information directly below and jot down the position number of the kernel. Once the number is written down, select the kernel and click enter.

- Automatically boot machine with L4S kernel:
	- The usual location for Advanced options is position 1 (as seen below - lists start at position 0).
![enter image description here](https://media.licdn.com/dms/image/D4D12AQErfgoiNV7_vw/article-inline_image-shrink_400_744/0/1685965207575?e=2147483647&v=beta&t=4cZ6uLYlX1j1xCWkRta9HYZf-jP-m4QSsdkMQ1KsAdY)
	- The location for the kernel will depend on your machine. Keep in mind the position of the l4s-testing (recovery mode) kernel in this list on this menu. You will need it.
		- For example, the kernel surrounded by the red box below is in position 2.
![enter image description here](https://media.licdn.com/dms/image/D4D12AQHv1r978b400Q/article-inline_image-shrink_400_744/0/1685965256052?e=2147483647&v=beta&t=drWrRTpJ0X-NnZ2-bcdtuYJkJxcn1DXvSPVIoeRFJP8)
- Once booted up:
	```sudo nano /etc/default/grub```
	- Inside the editor, find GRUB_DEFAULT= (should be the first uncommented line)
	- Replace the current variable (most likely 0) with the list position numbers recorded above.
		- For example, if Advanced options was in position 1 and the kernel was in position 5, set GRUB_DEFAULT='1>5'
		- Save and exit the file
		- Run ```sudo update-grub```

From now on, each time the machine is rebooted it will automatically boot with the l4s-testing kernel.

## Verify Kernel
Please make sure to always verify you are on the correct kernel by running ```uname -a```. If incorrect, then redo these steps.

## Configure Kernel
Enable Accurate ECN (this may need to be done after each reboot of the machine).
- ```sysctl net.ipv4.tcp_ecn=3```
