# Basics #

The topics covered in this section should be at least familiar to you.  
Therefore, only brief summaries will be given and links to external resources if you need to learn more.

## Anatomy of a PE image ##

| ![viewing the NT header](guide_images/viewing-NT-header.png) |
|:--:|
| <sub>Image: Viewing the NT Header</sub> | 

PE images are executable modules (.exe, .dll, .sys) that contain important header metadata for the OS image loader, contain program code and data, and are often organized into sections. Although the sections are purely for organization and aren't required for the program to function, I'll list the common ones you'll most likely see.<br>
The .idata section contains the import table, which is what the OS image loader reads to map dependencies (and then fills out the import address table (IAT)). The .edata section contains the export table. Some compilers will put both of these in the .rdata section. Either way, you can find them by parsing the NT header's data directories.<br>
Code is generally placed into the .text section, initialized non-const data is generally placed into the .data section, initialized const data can be found in the .rdata section, while uninitialized data can be found in the .bss section.<br>
Again, this is all dependent on the compiler as sections are merely for organizational purposes: they don't effect the operation of the program.<br>

The following resources should be helpful if you want to learn more:
http://www.csn.ul.ie/~caolan/pub/winresdump/winresdump/doc/pefile2.html  
https://docs.microsoft.com/en-us/windows/win32/debug/pe-format

## Windows Processes ##

Processes are containers for programs and their associated data. Processes also provide a natural memory boundary.<br>
Each process is given its own virtual address space and the OS is responsible for translating virtual addresses to physical addresses.<br>
Two programs may share the same base virtual address (for example, running two instances of the 32-bit Assault Cube game you'll find the base address of each one to be `0x400000`) but won't conflict with each other because the final translation to a physical address will be different.<br>
Everything related to a process is stored in an EPROCESS struct which is created by the kernel during process creation. This includes the page tables (for address translation), the VAD root node, the handle table, the thread list, and the usermode-exposed PEB which contains the list of loaded modules. You don't need to be intimately familiar with the EPROCESS struct until you're ready for kernel driver development, but you should be aware that the PEB contains information on every loaded module in the process.<br>
One final note: when a process exits all of its resources are automatically freed by the kernel. All those people on programming forums telling you to free your memory or release your handles before exit are ultimate pedants (but it **is** a good practice to do those things ;)<br>

| ![ntoskrnl!PspRundownSingleProcess](guide_images/processes-cant-leak.png) |
|:--:|
| <sub>Image: Processes never leak their own memory or handles</sub> | 

As for how to learn more, the latest edition of the Windows Internals book is probably going to be the best resource.<br>

## Windows Image Loader ##
Files are just bytes on disks, and so it is the image loader that must parse those bytes, figure out what and where the files it depends on are (and load and parse those...), and then finally hand over execution to the main entry point.<br>
The image loader is able to do this by having a standardized file format for executable files. For modern Windows operating systems the most common standard is the PE file format which was discussed in an earlier section.<br>
Through this file format, the image loader knows that important metadata will be contained at the very start of the file (and that metadata will point to other important metadata at other positions within the file). Such metadata includes various flags indicating whether the image is x86 or x64, whether it's large-address aware, whether it's a GUI app or console app, as well as the Import Table which defines every module and function within each module that the program depends upon.<br>
For a basic level of understanding the only things you need to know are that files are bytes on disk, executable files use a standard file format, and the image loader (which runs in usermode) parses those files to recursively load dependencies and do other initial setup work before handing execution over to the main entry point. It should be noted that some equally important work is done in kernelmode to create the process, which is part of the general 'image loading' process, but in this guide I refer specifically to the usermode part as the 'image loader'.<br>

For additional resources, again, the Windows Internals book is great. But thankfully, you'll also find some great posts online. So look for stackoverflow questions, forum posts, github repos of manual mappers (more on those later).

## Memory ##
You should be familiar with the concept of virtual memory. As mentioned in the section on processes, you should also be aware that each process is given its own virtual address space.  
Basically, modern consumer operating systems use the virtual memory management model. This is a whole thing. There are books about just this. I don't know everything about how or why virtual memory works, but basically, under this model, memory is divided into chunks called pages. Regular pages are usually 4KB but there are special 'large' pages which are 2MB (and there are huge pages which are 1GB, but I don't think many OS's support those). The CPU provides and enforces page protection levels.  
Accessing data is a long and complicated process, but it all boils down to the CPU checking if the virtual page is already backed by (or 'is resident in') physical memory, and if not it will generate a page fault which is handled by the OS kernel. The kernel, then, fetches the page from the pagefile if it exists and makes it resident in physical memory or passes an exception to the executing process.  
The tracking of virtual memory, physical memory, and paged memory is very important and very time-sensitive, so special look-up structures are defined by the CPU manufacturer (which the OS's must conform to). These structures are the page table entries, the page tables, the page directories, the page directory pointers tables, and the page map level 4s. The CPU maintains a cache of these structures in what's called the translation lookaside buffer (TLB). If a mapping can be found from the TLB then the whole process is much quicker. If not, however, that's when the page fault is sent to the OS. The OS, if it's able to fetch the page from secondary storage, then updates the page table entry and the TLB and asks the CPU to try translating again.  

Each element in the PML4 points to a PDPT. Each element in the PDPT points to a PD containing PDEs. Each PDE points to a PT. A PT is a simple array of PTEs. So is a PDPT and PML4. Every PT has 512 PTEs and each PTE maps a single P (4kb).

Good resources for learning more: Windows Internals book, Intel Software Development Manual, Wikipedia, various OSDev wikis or forums.  
(This is something I'm still learning too, so don't feel deterred if you feel lost right now).


## Pointers ##
This guide assumes you have prior programming experience dealing with pointers. Don't sweat it if you mess them up sometimes. Everyone does. I mess up pointer arithmetic at least once per project. You should be comfortable with this exercise: given an address (eg; 0x410000) read the first 4 bytes. Bonus points if you can read the first 4 bits.

## x86 Assembly ##
CPUs read bytes. Bytes are the words of their language. But just like our words have meaning, so too must their bytes. That is where 'instruction sets' come in.  
The x86 instruction set is the most common instruction set used by consumer CPUs.  
The x86 assembly *language* is a set of mnemonics that translate to x86-encoded instruction bytes.  
The human-readable `mov eax, 1` will compile to bytes `B801000000` and that's what will be stored on disk and eventually parsed by the CPU if executed.  
Disassemblers, then, take those bytes and try to figure out which mnemonic it came from.  
Refer to the Intel Software Development Manual (intel SDM) for details on specific encodings on an as-needed basis. There are also websites that try to condense everything into one page which you can usually find by googling something like 'x86 mov instruction encoding'.  
This guide will assume a basic knowledge of what each common mnemonic does (mov, jmp, call, push, pop, etc) and also how indirection works in the assembly language.  
It's worth noting, before moving on, that many disassemblers will provide a decompiler which attempts to figure out the higher-level language (usually looks like C/C++ but it's really just pseudo) code which might've produced the instruction bytes.  
Ghidra is a free, open-source, and popular disassembler that includes a really high quality decompiler too.