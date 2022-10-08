<h1 align="center">my-operating-system</h1>

<h2 align="left">project setup</h2>

```console
user@user: ~$ git init
user@user: ~$ git remote add origin https://[TOKEN]@github.com/[USER]/[REPO]
user@user: ~$ sudo apt-get install -y g++ binutils libc6-dev-i386
user@user: ~$ sudo apt-get install -y virtualbox xorriso mtools
user@user: ~$ mkdir system && cd system && git pull origin master
user@user: ~$ make run
```
<h2 align="left">theory</h2>

cpu registers in use:
<ul>
	<li>ax</li>
	<li>bx</li>
	<li>stack pointer</li>
	<li>instruction pointer</li>
</ul>

timeline of boot:

<ol>
	<li>motherboard takes data from bios and copies it to the ram</li>
	<li>motherboard tells processor to place instruction pointer at the beginning of copied content</li>
	<li>cpu will read and execute instructions (firmware)</li>
	<li>firmware will then tell cpu to check harddrive</li>
	<li>cpu will check ~2mb of a harddrive(aka bootsector, master boot record)</li>
	<li>this will then be loaded into ram (bootloader like grub)</li>
	<li>then cpu's instruction pointer jumps into the beginning of bootloader</li>
	<li>it will check partition /boot/grub/grub.cfg</li>
	<li>it will read this file (with menu entries to select the system)</li>
	<li>the bootloader will print this list of systems</li>
	<li>grub will know the place on thisk of the selected system (like /boot/kernel.bin)</li>
	<li>it will copy this kernel to ram as well and tell the cpu to jump into the kernel location</li>
	<li>your computer starts the cpu will start in 32 bit compatibility mode</li>
</ol>

<h2 align="left">programming</h2>

we have a big problem at the very beginning. the bootloader didnt set the stack pointer (but cpp programs expect it to be set before running). because of that we have to write two different files and connect them with a third one:

<ul>
	<li>loader.s (to set stack pointer) (compiled with gnu assembler into loader.o)</li>
	<li>kernel.cpp (my os) (compiled creating kernel.o)</li>
	<li>linker.ld (combine two above into kernel.bin)</li>
</ul>

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
we can use this info by pushing it with laoder, so then we need to accept it in our kernel
so now the funny thing
when kernel cpp is loaded we cant include libs like standard io or other usual files why is that?
usually when u write a program u have your operating system around it and u use our program in it
this way printf works because system sees that uu are using stuff from glib.c library
when u start this program, system will dynamically link to load glib.c and connect your program to it
we are outside of an os so we have no glib.c .. and we have no os at all.. we have nothing
so we have to write printf simplified on our own
there is a pointer in ram (0xb8000), where everything behind it is just printed on the screen by the graphics card
| |a| |b| -> this will print out ab
but what about this empty bytes?
they are for color information (background,text)
the good thing is that on a computer startup these empty bytes are already set to white text on a black background
so we dont have to set them ourselves
we just have to make sure that we do not override that
we also need to tell compiler that there is nothing that handles error exceptions, no dynamic memory management etc
no memory management flag: -fno-use-cxa-atexit
no glib.c flag: -nostdlib
other flags: -fno-builtin -fno-rtti -fno-exceptions -fno-leading-underscore
linker cant find kernelMain function because gpp has different naming conventions so we need to add extern "C"
all external function pointers are stored between start and end ctors so we need to call these constructors
if we want to communicate with the hardware we need to be byte perfect
we are using assembler file so we cant let cpp compile int as an 8bit type if for ex assembler uses 4bit int type
so we need to create our own type
