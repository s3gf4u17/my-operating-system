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
we dont want kernel to stop so we need to apply an infinite loop somewhere
with extern we give assembler info that there will be a process kernelMain and that well want to jump into it
it tells that if we want to call kernelMain just assume it will be there the linker will take care of it
global loader - program entry point
we need to mov stack pointer (esp) to some kernel stack pointer
another problem
if we want to set esp somewhere close to where kernelMain begins then it will override the stuff we have in ram
because stack moves to the left
when we push a few things there well create a lot of chaos
so we need to put some empty space in between and have the kernel stack pointer behind it
so well for ex use 2mb of space between
stop is just an another safety infinite loop in case the one in kerbnel.cpp fails
another problem is that the bootloader will not reckognize it as a kernel because bootloader will look for so called magic number in the file and if its not there then it will not believe that is is a kernel
this magic number is a hex dec number 0x1badb002 we need some flags and a checksum as well
before bootloader jumps into kernel it will store some information somewhere in ram (multiboot structure)
containing info like size of ram and it store the pointer to that structure in a ax register
it also copies the magic number into the bx register