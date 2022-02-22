# Intro to Game Hacking #

Videogames provide a great medium for programmers to improve their skills through designing and implementing many complex systems that must be performant and act cohesively.  
In turn, videogames also serve as a great medium for fledgling reverse engineers to learn the ropes and gain practical experience picking apart those complex systems and exploiting them.


## Topics to cover (loosely ordered) ##

1. Basics
	<ol type="a">
	<li>anatomy of a Portable Executable (PE) image</li>
	<li>windows processes are containers</li>
	<li>windows image loader</li>
	<li>physical memory, virtual memory</li>
		<ol type="i">
		<li>why addresses change</li></li>
		</ol>
	<li>pointers and multi-level pointers</li>
	<li>x86-encoded instruction bytes and the x86 assembly language</li>
	</ol>
2. Hooking
	<ol type="a">
	<li>Inline detour</li>
		<ol type="i">
		<li>e9 jmp and ff25 jmp</li>
		</ol>
	<li>Trampolines</li>
	<li>Virtual Method Table (VMT / VTable) hooking</li>
		<ol type="i">
		<li>swap vtable ptr or overwrite element in vtable</li>
		</ol>
	<li>Vectored Exception Handler (VEH) hooking</li>
	<li>Hardware Breakpoint (HWBP) hooking</li>
	<li>Summary</li>
	</ol>
3. Injection
	<ol type="a">
	<li>VirtualAllocEx, WriteProcessMemory (WPM), CreateRemoteThread (CRT) @ LoadLibrary</li>
	<li>manual mapping</li>
		<ol type="i">
		<li>the wrong way</li>
		</ol>
	<li>thread hijacking</li>
	<li>Process Environment Block (PEB) unlinking for all cases</li>
	<li>considerations for UWP apps</li>
	<li>Summary</li>
	</ol>
4. Information gathering
	<ol type="a">
	<li>live analysis</li>
		<ol type="i">
		<li>write program containing varied structs and examine them</li>
		</ol>
	<li>static analysis</li>
		<ol type="i">
		<li>press F5</li>
		</ol>
	<li>open source</li>
	</ol>
5. "how 2maek esp???"
	<ol type="a">
	<li>rendering</li>
		<ol type="i">
		<li>directx/opengl hook</li>
		<li>engine</li>
		</ol>
	<li>world to screen transformation</li>
		<ol type="i">
		<li>math is hard</li>
		</ol>
	<li>chams</li>
		<ol type="i">
		<li>stride, numverts, index, and depth</li>
		<li>d3d11+ equivalents</li>
		</ol>
	<li>considerations for externals</li>
	<li>tips on finding player iteration</li>
	</ol>
6. "i have this small, tiny problem with my aimbot *not working at all* can you show me yours?"
	<ol type="a">
	<li>calculating aim angle</li>
		<ol type="i">
		<li>math is still hard</li>
		</ol>
	<li>facing the target</li>
		<ol type="i">
		<li>engine funcs</li>
		<li>modifying model / view matrices</li>
		<li>winapi mouse funcs</li>
		</ol>
	<li>(stop) aiming through walls</li>
		<ol type="i">
		<li>tip(s) on finding raycast</li>
		<li>pixel scanner and why you should find engine raycast func...</li>
		</ol>
	</ol>
7. Kernel mode
	<ol type="a">
	<li>mapping your driver & executing it</li>
	<li>reading/writing usermode memory</li>
	<li>kernel<->user communication</li>
	<li>stealth... and why you'll get caught</li>
	</ol>
8. Networking
	<ol type="a">
	<li>packet inspection</li>
	<li>anatomy of a packet</li>
	<li>techniques for packet editing</li>
		<ol type="i">
		<li>libpcap or other drivers</li>
		<li>hosts file + forwarder server</li>
		<li>hardware</li>
		</ol>
	<li>authoritative servers</li>
		<ol type="i">
		<li>you can't be god</li>
		</ol>
	<li>compression and encryption complicate things</li>
	</ol>
9. Hypervisors
	<ol type="a">
	<li>type 1 & type 2</li>
	<li>hijacking hyper-v</i>
	<li>(i still have to learn this myself)</li>
	</ol>
10. UEFI boot drivers
	<ol type="a">
	<li>(i still have to learn this myself)</li>
	</ol>
11. Future complications
	<ol type="a">
	<li>TPM requirement on win11</li>
	</ol>
	
### Bonus Topics ###

1. Aimbot + ESP quick rundown for assault cube
	<ol type="a">
	<li>internal and maybe external?</li>
	</ol>
2. Aimbot + ESP quick rundown for an old quake engine game
3. Aimbot + ESP quick rundown for a ue4 game
	<ol type="a">
	<li>GObjects + GNames = GG</li>
	</ol>
4. Aimbot + ESP quick rundown for a unity game
	<ol type="a">
	<li>mono backend vs il2cpp backend</li>
	</ol>
5. Aimbot + ESP quick rundown for a cryengine (5?) game
6. Converting assault cube hack to kernel-mode

# Basics #

The topics covered in this section should be at least familiar to you.  
Therefore, only brief summaries will be given and links to external resources if you need to learn more.

## Anatomy of a PE image ##

| ![viewing the NT header](guide_images/viewing-NT-header.png) |
|:--:|
| <sub>Image: Viewing the NT Header</sub> | 

PE images are executable modules (.exe, .dll, .sys) that contain important header metadata for the OS image loader, contain program code and data, and are often organized into sections. Although the sections are purely for organization and aren't required for the program to function, I'll list the common ones you'll most likely see.  
The .idata section contains the import table, which is what the OS image loader reads to map dependencies (and then fills out the import address table (IAT)). The .edata section contains the export table. Some compilers will put both of these in the .rdata section. Either way, you can find them by parsing the NT header's data directories.  
Code is generally placed into the .text section, initialized non-const data is generally placed into the .data section, initialized const data can be found in the .rdata section, while uninitialized data can be found in the .bss section.  
Again, this is all dependent on the compiler as sections are merely for organizational purposes: they don't effect the operation of the program.  

The following resources should be helpful if you want to learn more:
http://www.csn.ul.ie/~caolan/pub/winresdump/winresdump/doc/pefile2.html  
https://docs.microsoft.com/en-us/windows/win32/debug/pe-format

## Windows Processes ##

Processes are containers for programs and their associated data. Processes also provide a natural memory boundary.  
Each process is given its own virtual address space and the OS is responsible for translating virtual addresses to physical addresses.  
Two programs may share the same base virtual address (for example, running two instances of the 32-bit Assault Cube game you'll find the base address of each one to be `0x400000`) but won't conflict with each other because the final translation to a physical address will be different.  
Everything related to a process is stored in an EPROCESS struct which is created by the kernel during process creation. This includes the page tables (for address translation), the VAD root node, the handle table, the thread list, and the usermode-exposed PEB which contains the list of loaded modules. You don't need to be intimately familiar with the EPROCESS struct until you're ready for kernel driver development, but you should be aware that the PEB contains information on every loaded module in the process.  
One final note: when a process exits all of its resources are automatically freed by the kernel. All those people on programming forums telling you to free your memory or release your handles before exit are ultimate pedants (but it **is** a good practice to do those things ;).

| ![ntoskrnl!PspRundownSingleProcess](guide_images/processes-cant-leak.png) |
|:--:|
| <sub>Image: Processes never leak their own memory or handles</sub> | 

As for how to learn more, the latest edition of the Windows Internals book is probably going to be the best resource.

## Windows Image Loader ##
Files are just bytes on disks, and so it is the image loader that must parse those bytes, figure out what and where the files it depends on are (and load and parse those...), and then finally hand over execution to the main entry point.  
The image loader is able to do this by having a standardized file format for executable files. For modern Windows operating systems the most common standard is the PE file format which was discussed in an earlier section.  
Through this file format, the image loader knows that important metadata will be contained at the very start of the file (and that metadata will point to other important metadata at other positions within the file). Such metadata includes various flags indicating whether the image is x86 or x64, whether it's large-address aware, whether it's a GUI app or console app, as well as the Import Table which defines every module and function within each module that the program depends upon.  
For a basic level of understanding the only things you need to know are that files are bytes on disk, executable files use a standard file format, and the image loader (which runs in usermode) parses those files to recursively load dependencies and do other initial setup work before handing execution over to the main entry point. It should be noted that some equally important work is done in kernelmode to create the process, which is part of the general 'image loading' process, but in this guide I refer specifically to the usermode part as the 'image loader'.

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
CPUs read bytes. Bytes are the words of their language. But just like our words have meaning, so to must their bytes. That is where 'instruction sets' come in.  
The x86 instruction set is the most common instruction set used by consumer CPUs.  
The x86 assembly *language* is a set of mnemonics that translate to x86-encoded instruction bytes.  
The human-readable `mov eax, 1` will compile to bytes `B801000000` and that's what will be stored on disk and eventually parsed by the CPU if executed.  
Disassemblers, then, take those bytes and try to figure out which mnemonic it came from.  
Refer to the Intel Software Development Manual (intel SDM) for details on specific encodings on an as-needed basis. There are also websites that try to condense everything into one page which you can usually find by googling something like 'x86 mov instruction encoding'.  
This guide will assume a basic knowledge of what each common mnemonic does (mov, jmp, call, push, pop, etc) and also how indirection works in the assembly language.  
It's worth noting, before moving on, that many disassemblers will provide a decompiler which attempts to figure out the higher-level language (usually looks like C/C++ but it's really just pseudo) code which might've produced the instruction bytes.  
Ghidra is a free, open-source, and popular disassembler that includes a really high quality decompiler too.

# Hooking #

The art of making someone else's code execute your code instead.

## Inline Hooking ##

An inline hook is one in which a jmp to your hook function has been written over some instruction bytes in the target function (usually at the very top of the target function).  
Because you're overwriting instruction bytes that may be executed at any time you'll need to first suspend all threads other than your own. Furthermore, because CPUs like to cache things, it's also a good idea to flush the instruction cache after you've written your jmp instruction. Lastly, remember that bytes in the .text section usually have only RX page protection, so you'll need to modify the page protection to RWX, write your jmp, then set the page protection back to its original value.
Furthermore, you should be aware that not all jmp instructions are the same. There are near and far jmps which have the further modifier of relative, or absolute indirect, or absolute. If you're working with a 32 bit address space, a near relative jmp should be fine, but if not then you should make sure that the distance between your target function and hook function isn't greater than the max offset of a near relative jmp. This is one case where you'll definitely want to refer to the intel SDM to find the specifics of each jmp encoding.

### TO-DO ###
Image of before and after control flow

## Trampolines ##

Often you'll want to call the original function at some point in your hook function, but you'll find if you do that you'll eventually hit a stack overflow exception. Your hook calls your target which is hooked so execution winds back up in your hook which calls your target which... eventually, all those calls will exhaust the stack and generate that exception.  
So you have two options: either unhook your target function, call it, then re-hook it, or simply write a 'trampoline' and call your trampoline.  
A trampoline is simply a copy of the instructions that your hook overwrote in the target function followed by a jmp to the position in the target function just past your hook. This allows the full target function to execute without going into the infinite loop of jmp'ing to your hook.  
Of course, because instructions are just encoded bytes, how can you know which bytes comprise a full instruction? Or rather, where does one instruction end and the next begin? This is where you'll need to employ a disassembly engine. Popular ones include `hde` and `capstone`. You'll feed the disassembly engine the bytes and it'll feed you the instructions. With that, you should have all you need to write your trampoline. Again, though, make sure you're using the write jmp encoding in your trampoline.  

Ah, this was written with inline hooks in mind but trampolines can be used for other types of hooks too. For example, if you have a hardware breakpoint set on the first instruction of the target function, your trampoline would need to have a copy of that instruction and then jmp to the second instruction of the target function. This will, again, allow you to call the original target function without it executing your hook function.

### TO-DO ###
Image of before and after control flow of an inline hook, including the trampoline

## Virtual Method Table Hooking ##

If your target program is a C++ program, you may find that your target function is a virtual class member function. If you're familiar with the low-level details of C++ you'll know that any class instance that has virtual member functions will have a pointer to something called a Virtual Method Table (VMT) containing pointers to each virtual member function. This pointer to the VMT is always the first field of the class instance in memory.  
With that knowledge in hand, it should be obvious that if you could obtain a pointer to that class instance you could then copy their virtual method table, modify the entry pointing to your target function to point to your hook function, then replace the class instance's VMT pointer to point to your modified VMT.  
However, there is an even easier way. If you don't want to make a copy of the entire VMT you could simply modify the target function's specific entry within the original VMT in-place to point to your hook function.  
As with all hooks, you should always suspend all threads besides your own before doing this. However, because this isn't overwriting any *instructions* you don't need to flush the instruction cache. This is why this is often referred to as a 'data hook' - you're only modifying data, not instructions.  

### TO-DO ###
Image of VMT, maybe image of how a virtual function is often called in assembly

## Vectored Exception Handler Hooking ##

Windows debugging API allows you to register your own exception handler. You can register an exception handler and then intentionally trigger an exception using a variety of methods.  
One such method would be to mark the page containing your function as a guard page or no-access page. In your exception handler you then step through until you reach your target function then call your hook function. Not every exception triggered will lead to your target function - pages will almost always contain a multitude of other functions. Furthermore, exceptions can and do occur normally as part of program operation, so your exception handler must handle all cases gracefully.  
Another way of triggering an exception is to overwrite the first byte of your target function with `0xCC`. This is the special INT 3 instruction used by debuggers to trigger breakpoints at specific instructions. The benefit of this method is that your exception handler can be simplified: since you know *your* exception will only occur at the address you wrote the INT 3 instruction, you can pass the exception to the next handler whenever the XIP doesn't match. The drawback of this method, of course, is that you're still overwriting instructions like in the inline hook.  
No matter which method you use to trigger an exception, you should keep in mind that exceptions are expensive so these hooks tend to slow down the program if it's used on a function that's called very often.

## Hardware Breakpoint Hooking ##

This is very similar to the Vectored Exception Handler Hooking. Except instead of purposefully generating an exception, you are using the debug registers associated with each thread. Unfortunately, there are only 4 debug registers available. The configuration of each breakpoint must be carefully set in dr7 using bitwise operators.  
Bits 0, 2, 4, and 6 should be set corresponding to which debug register you're using for your hook.  
Next, there are 2-bit flags to define when you want your breakpoint to trigger. For a hook you want these flags to be `00` indicating a trigger on execution. These bits are 16&17, 20&21, 24&25, and 28&29 again corresponding to which debug register you're using for your hook.  
Finally, there is another 2-bit flag to define how large of an area should be watched for your breakpoint to trigger. For a hook this can just be `00` again indicating one byte. These bits are 18&19, 22&23, 26&27, and 30&31 again corresponding to which debug register you're using for your hook.  
The benefit to this is you aren't modifying anything other than the thread context. No data, no instructions, no page protections. The drawback is that it's still an exception and so it's still more expensive than an inline or VMT hook.

## Summary ##

We've only covered the most popular hooking methods here. There are probably countless ways of altering control flow to execute your code. I encourage you to think about the other possibilities. In the end though, you shouldn't feel any pressure to come up with something totally unique. These methods covered are popular and get the job done 99.99% of the time :).