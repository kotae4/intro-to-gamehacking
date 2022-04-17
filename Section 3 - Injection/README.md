# Injection #

Shoving your code some place it doesn't belong.

## Making the target call LoadLibrary for you ##

I don't know the actual name for this particular technique, but it's by far the most common and perhaps oldest.  
Your injector allocates enough memory in the target process to hold your dll's path string, then writes the path string into that allocated memory, then creates a thread starting at the address of LoadLibrary passing it the address of that allocated memory.  
The effect is that the target process will call LoadLibrary on your dll.  
One thing to take care of is that if your injector is 64bit and the target is 32bit then the address of LoadLibrary in your injector won't match the one in the target. Vice versa is true as well. So I recommend walking the target process' module list to find kernel32.dll then parse its EAT to find LoadLibrary.  
As for downsides, this method involves winapi calls in the target process so if there is any kind of security it'd be trivial to detect this (and even block it completely).

## Manual Mapping ##

Remember the OS image loader? Well, this technique involves re-creating it :)<br>
This is one case where you'll definitely want to look up additional resources or do some reversing yourself. The windows image loader has changed quite a bit between OS versions so it seems entirely possible it could continue to change in the future. The core ideas should remain the same, though:
1. Read the image into your injector process
2. Allocate space for image in target process
3. Apply relocations if necessary (according to the .reloc section)
4. Enumerate imported modules and check if they're loaded in the target process already
	<ol type="a">
	<li>If not, then try to resolve its path according to Window's own DLL search order and then load it starting from step 1.</li>
	</ol>
5. For each module mapped, fill in its IAT. This involves looking up each imported function in the imported module's EAT.
6. Make exceptions possible (SEH and VEH)
7. Handle TLS entries
8. Invoke each module's TLS entries
9. Invoke each module's entrypoint

Steps 6, 7, and 8 aren't *strictly* required but they're nice to have. If a module does anything meaningful with TLS data then 6 & 7 would be necessary, but I'm not sure how frequently you'd encounter that.

The big issue to solve, in my opinion, is path resolution. Imported modules are stored in the image's Import Table with only the filename (eg; `KERNEL32.DLL`) so you need to resolve that to `C:\\Windows\\System32\\kernel32.dll` or `C:\\Windows\\SysWOW64\\kernel32.dll`.  
It becomes even more complicated when you encounter an import named `api-ms-win-crt-math-l1-1-0.dll`. This would (at the time of writing) need to resolve to `C:\\Windows\\System32\\ucrtbase.dll` or `C:\\Windows\\SysWOW64\\ucrtbase.dll`. That's one of the things introduced to the image loader in a recent version of the OS: API sets.  
Additionally, SxS images add another complication. SxS images are either stored next to a manifest or often have a manifest file embedded. This manifest file effectively overrides the image search order.

As for detection, since you won't be linking your module to the PEB it won't show up in any module snapshot. However, the memory allocated by VirtualAlloc could be seen, and the thread that executes each module's DllMain (or any that you create later) could be seen. If you do any hooking then of course that could be seen as well. Memory allocations aren't necessarily a huge red flag, so the main concern is the detection of your threads. Generally, manual mapping is an effective way to evade actions from most usermode anticheats.

### Shortcut ###

One common thing you'll see in a lot of open source manual mappers is a call to LoadLibrary for each imported module. This certainly makes things easier as you don't need to do any path resolution, but you're invoking the Windows image loader with your call to LoadLibrary (or LdrLoadDll) and that kind of defeats the purpose of writing a *manual* mapper, doesn't it?<br>

## Thread Hijacking ##

This isn't so much 'injection' as it is just a way of executing your injected code (or any code), but I'm throwing it in the 'injection' section anyway.

The principle of thread hijacking is simple:
1. Enumerate threads associated with the target process
2. Pick one, the best one, the main one and suspend it.
3. Call `GetThreadContext`
4. Modify the instruction pointer to point to whatever code you want to execute
5. Call `SetThreadContext` with the modified context
6. Resume the thread

Of course, that thread is now garbage because it can't return to where it was. So you'd be best off running an infinite loop and taking over the processs.

Unless you save and then restore all registers once your code has finished executing. Then the thread you hijacked can return to where it was.

The benefit of doing this is that you aren't creating any additional threads, so you're minimizing your detection footprint. Another benefit is if you're operating externally but want to hide within a 'trusted' process, you can hijack a thread in that trusted process and take over - this is similar to process hollowing.

## Unlinking from PEB ##

Depending on how you inject, your module may be placed in the PEB's Ldr module list. You can remove it to prevent your module from showing up in any snapshots.

## UWP Apps ##

UWP apps have different security access descriptors. If you're trying something and can't figure out why it's not working then check to see if you're dealing with a UWP app. Then research all the differences between native processes and 'modern' processes to figure out the workarounds.

## Summary ##

This is a relatively small section covering just two injection techniques with some bonus topics thrown in. There are so many more techniques to learn, but I've found the two I covered here to be more than enough. If you're interested in this topic I highly recommend going down a rabbit hole researching all the different techniques though. Doing so would expand your knowledge of the OS too.