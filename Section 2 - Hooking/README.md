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