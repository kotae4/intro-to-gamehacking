# Intro to Game Hacking #

Videogames provide a great medium for programmers to improve their skills through designing and implementing many complex systems that must be performant and act cohesively.<br>
In turn, videogames also serve as a great medium for fledgling reverse engineers to learn the ropes and gain practical experience picking apart those complex systems and exploiting them.<br>

Navigate this guide by clicking the links in the table of contents below or by visiting each folder in this git repository.<br>
Example projects may be included in each section, but they are also linked to in the `All Example Projects` folder for convenience.


## Topics to cover (loosely ordered) ##

1. <a href="Section 1 - Basics">Basics</a>
	<ol type="a">
	<li><a href="Section 1 - Basics#anatomy-of-a-pe-image">anatomy of a Portable Executable (PE) image</a></li>
	<li><a href="Section 1 - Basics#windows-processes">windows processes are containers</a></li>
	<li><a href="Section 1 - Basics#windows-image-loader">windows image loader</a></li>
	<li><a href="Section 1 - Basics#memory">physical memory, virtual memory</a></li>
		<ol type="i">
		<li>why addresses change</li>
		</ol>
	<li><a href="Section 1 - Basics#pointers">pointers and multi-level pointers</a></li>
	<li><a href="Section 1 - Basics#x86-assembly">x86-encoded instruction bytes and the x86 assembly language</a></li>
	</ol>
2. <a href="Section 2 - Hooking">Hooking</a>
	<ol type="a">
	<li><a href="Section 2 - Hooking#inline-hooking">Inline detour</a></li>
		<ol type="i">
		<li>e9 jmp and ff25 jmp</li>
		</ol>
	<li><a href="Section 2 - Hooking#trampolines">Trampolines</a></li>
	<li><a href="Section 2 - Hooking#virtual-method-table-hooking">Virtual Method Table (VMT / VTable) hooking</a></li>
		<ol type="i">
		<li>swap vtable ptr or overwrite element in vtable</li>
		</ol>
	<li><a href="Section 2 - Hooking#vectored-exception-handler-hooking">Vectored Exception Handler (VEH) hooking</a></li>
	<li><a href="Section 2 - Hooking#hardware-breakpoint-hooking">Hardware Breakpoint (HWBP) hooking</a></li>
	<li><a href="Section 2 - Hooking#summary">Summary</a></li>
	</ol>
3. <a href="Section 3 - Injection">Injection</a>
	<ol type="a">
	<li><a href="Section 3 - Injection#making-the-target-call-loadlibrary-for-you">Making the target call LoadLibrary for you</a></li>
	<li><a href="Section 3 - Injection#manual-mapping">Manual Mapping</a></li>
		<ol type="i">
		<li><a href="Section 3 - Injection#shortcut">Shortcut</a></li>
		</ol>
	<li><a href="Section 3 - Injection#thread-hijacking">Thread Hijacking</a></li>
	<li><a href="Section 3 - Injection#unlinking-from-peb">Process Environment Block (PEB) unlinking for all cases</a></li>
		<ol type="i">
		<li>TO-DO: move the above two into a new top-level 'Stealth' section</li>
		</ol>
	<li><a href="Section 3 - Injection#uwp-apps">Considerations for UWP apps</a></li>
	<li><a href="Section 3 - Injection#summary">Summary</a></li>
	</ol>
4. <a href="Section 4 - Information Gathering">Information Gathering</a>
	<ol type="a">
	<li><a href="Section 4 - Information Gathering#dynamic-analysis">Dynamic Analysis</a></li>
	<li><a href="Section 4 - Information Gathering#static-analysis">Static Analysis</a></li>
	<li><a href="Section 4 - Information Gathering#open-source">Open Source</a></li>
	<li><a href="Section 4 - Information Gathering#all-together-now">All Together Now</a></li>
	<li><a href="Section 4 - Information Gathering#other-useful-tools">Other Useful Tools</a></li>
	<li><a href="Section 4 - Information Gathering#summary-and-a-tip">Summary And A Tip</a></li>
	</ol>
5. <a href="Section 5 - About ESP">"how 2maek esp???"</a>
	<ol type="a">
	<li><a href="Section 5 - About ESP#different-ways-to-draw">Different Ways to Draw</a></li>
		<ol type="i">
		<li>Directx/OpenGL hook</li>
		<li>Engine drawing</li>
		</ol>
	<li><a href="Section 5 - About ESP#requirements-for-an-esp-internal">Requirements for an ESP</a></li>
	<li><a href="Section 5 - About ESP#world-to-screen">World to Screen</a></li>
		<ol type="i">
		<li>math is hard</li>
		</ol>
	<li><a href="Section 5 - About ESP#externals">Externals</a></li>
	<li><a href="Section 5 - About ESP#chams">Chams</a></li>
		<ol type="i">
		<li>stride, numverts, index, and depth</li>
		<li>d3d11+ equivalents</li>
		</ol>
	<li><a href="Section 5 - About ESP#summary-and-tips">Summary and Tips</a></li>
	</ol>
6. <a href="Section 6 - About Aimbot">Aimbot</a>
	<ol type="a">
	<li><a href="Section 6 - About Aimbot#requirements-for-aimbot-internal">Requirements for Aimbot</a></li>
	<li><a href="Section 6 - About Aimbot#calculating-the-aim-angle">Calculating the Aim Angle</a></li>
		<ol type="i">
		<li>math is still hard</li>
		</ol>
	<li><a href="Section 6 - About Aimbot#facing-the-target">Facing the Target</a></li>
	<li><a href="Section 6 - About Aimbot#stop-aiming-through-walls">(Stop) Aiming Through Walls</a></li>
	<li><a href="Section 6 - About Aimbot#pixel-scanners">Pixel Scanners</a></li>
	<li><a href="Section 6 - About Aimbot#summary">Summary</a></li>
	</ol>
7. <a href="Section 7 - Networking">Networking</a>
	<ol type="a">
	<li><a href="Section 7 - Networking#videogame-networking">Videogame Networking</a></li>
	<li><a href="Section 7 - Networking#network-authority">Network Authority</a></li>
	<li><a href="Section 7 - Networking#packet-inspection">Packet Inspection</a></li>
	<li><a href="Section 7 - Networking#packet-editing">Packet Editing</a></li>
	<li><a href="Section 7 - Networking#complications">Complications</a></li>
	<li><a href="Section 7 - Networking#summary">Summary</a></li>
	</ol>
8. (Work-In-Progress) Kernel mode
	<ol type="a">
	<li>mapping your driver & executing it</li>
	<li>reading/writing usermode memory</li>
	<li>kernel<->user communication</li>
	<li>stealth... and why you'll get caught</li>
	<li>(i'm still learning this myself)</li>
	</ol>
9. (Work-In-Progress) Hypervisors
	<ol type="a">
	<li>type 1 & type 2</li>
	<li>hijacking hyper-v</i>
	<li>(i still have to learn this myself)</li>
	</ol>
10. (Work-In-Progress) UEFI boot drivers
	<ol type="a">
	<li>(i still have to learn this myself)</li>
	</ol>
	
### (Work-In-Progress) Example Projects / Labs ###

- [x] Viewing data structures in memory and in a disassembler
- [x] Basic manual mapper
- [x] Aimbot + ESP quick rundown for assault cube
	<ol type="a">
	<li>internal and maybe external?</li>
	</ol>
- [ ] Aimbot + ESP quick rundown for an old quake engine game
- [ ] Aimbot + ESP quick rundown for a ue4 game
	<ol type="a">
	<li>GObjects + GNames = GG</li>
	</ol>
- [ ] Aimbot + ESP quick rundown for a unity game
	<ol type="a">
	<li>mono backend vs il2cpp backend</li>
	</ol>
- [ ] Aimbot + ESP quick rundown for a cryengine (5?) game
- [ ] Converting assault cube hack to kernel-mode