<h1 align="center">my-operating-system</h1>

<h2 align="left">project setup</h2>

```console
user@user: ~$ git init
user@user: ~$ git remote add origin https://[TOKEN]@github.com/[USER]/[REPO]
user@user: ~$ sudo apt-get install -y g++ binutils libc6-dev-i386
```
<h2 align="left">concept</h2>
what happens when u start your computer:
motherboard-bios-cpu   harddrive attached to it

cpu has several registers (we'll be using):
ax
bx
stack pointer
instruction pointer

when u start your computer, motherboard takes data from bios and copies it to the ram
it then tells processor to place instruction pointer at the beginning of copied content
cpu will read and execute instructions (firmware)
firmware will then tell cpu to check harddrive (it will check ~2mb of a harddrive(aka bootsector, master boot record))
this will then be loaded into ram (bootloader like grub)
then cpu's instruction pointer jumps into the beginning of bootloader
it will check partition /boot/grub/grub.cfg
it will read this file (with menu entries to select the system)
the bootloader will print this list of systems
grub will know the place on thisk of the selected system (like /boot/kernel.bin)
it will copy this kernel to ram as well and tell the cpu to jump into the kernel location
we have a big problem at this point
the bootloader will not set the stack pointer (but cpp programs expect it to be set before running)
so we have to write two different files: loader.s (to set stack pointer) and kernel.cpp (my os)
loader will be compiled with gnu assembler into loader.o
kernel will be compiled creating kernel.o
now we have two different files in two different programming languages so we have to connect them somehow
we need to use linker.ld which will combine these two into kernel.bin (which well later put into boot directory with an entry in grub.cfg)
even though the 64 bit system is a standard, when your computer starts the cpu will start in 32 bit compatibility mode
so at the start of the kernel we are in 32 bit mode so well make the os in 32 bit mode