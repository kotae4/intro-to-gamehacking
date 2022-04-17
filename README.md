# Intro to Game Hacking #

Videogames provide a great medium for programmers to improve their skills through designing and implementing many complex systems that must be performant and act cohesively.<br>
In turn, videogames also serve as a great medium for fledgling reverse engineers to learn the ropes and gain practical experience picking apart those complex systems and exploiting them.<br>

Navigate this guide by clicking the links in the table of contents below or by visiting each folder in this git repository.<br>
Example projects may be included in each section, but they are also linked to in the `All Example Projects` folder for convenience.


## Topics to cover (loosely ordered) ##

1. <a href="Section 1 - Basics/README.md">Basics</a>
	<ol type="a">
	<li><a href="Section 1 - Basics/README.md#anatomy-of-a-pe-image">anatomy of a Portable Executable (PE) image</a></li>
	<li><a href="Section 1 - Basics/README.md#windows-processes">windows processes are containers</a></li>
	<li><a href="Section 1 - Basics/README.md#windows-image-loader">windows image loader</a></li>
	<li><a href="Section 1 - Basics/README.md#memory">physical memory, virtual memory</a></li>
		<ol type="i">
		<li>why addresses change</li>
		</ol>
	<li><a href="Section 1 - Basics/README.md#pointers">pointers and multi-level pointers</a></li>
	<li><a href="Section 1 - Basics/README.md#x86-assembly">x86-encoded instruction bytes and the x86 assembly language</a></li>
	</ol>
2. <a href="Section 2 - Hooking/README.md">Hooking</a>
	<ol type="a">
	<li><a href="Section 2 - Hooking/README.md#inline-hooking">Inline detour</a></li>
		<ol type="i">
		<li>e9 jmp and ff25 jmp</li>
		</ol>
	<li><a href="Section 2 - Hooking/README.md#trampolines">Trampolines</a></li>
	<li><a href="Section 2 - Hooking/README.md#virtual-method-table-hooking">Virtual Method Table (VMT / VTable) hooking</a></li>
		<ol type="i">
		<li>swap vtable ptr or overwrite element in vtable</li>
		</ol>
	<li><a href="Section 2 - Hooking/README.md#vectored-exception-handler-hooking">Vectored Exception Handler (VEH) hooking</a></li>
	<li><a href="Section 2 - Hooking/README.md#hardware-breakpoint-hooking">Hardware Breakpoint (HWBP) hooking</a></li>
	<li><a href="Section 2 - Hooking/README.md#summary">Summary</a></li>
	</ol>
3. <a href="Section 3 - Injection/README.md">Injection</a>
	<ol type="a">
	<li><a href="Section 3 - Injection/README.md#making-the-target-call-loadlibrary-for-you">Making the target call LoadLibrary for you</a></li>
	<li><a href="Section 3 - Injection/README.md#manual-mapping">Manual Mapping</a></li>
		<ol type="i">
		<li><a href="Section 3 - Injection/README.md#shortcut">Shortcut</a></li>
		</ol>
	<li><a href="Section 3 - Injection/README.md#thread-hijacking">Thread Hijacking</a></li>
	<li><a href="Section 3 - Injection/README.md#unlinking-from-peb">Process Environment Block (PEB) unlinking for all cases</a></li>
		<ol type="i">
		<li>TO-DO: move the above two into a new top-level 'Stealth' section</li>
		</ol>
	<li><a href="Section 3 - Injection/README.md#uwp-apps">Considerations for UWP apps</a></li>
	<li><a href="Section 3 - Injection/README.md#summary">Summary</a></li>
	</ol>
4. <a href="Section 4 - Information Gathering/README.md">Information Gathering</a>
	<ol type="a">
	<li><a href="Section 4 - Information Gathering/README.md#dynamic-analysis">Dynamic Analysis</a></li>
	<li><a href="Section 4 - Information Gathering/README.md#static-analysis">Static Analysis</a></li>
	<li><a href="Section 4 - Information Gathering/README.md#open-source">Open Source</a></li>
	<li><a href="Section 4 - Information Gathering/README.md#all-together-now">All Together Now</a></li>
	<li><a href="Section 4 - Information Gathering/README.md#other-useful-tools">Other Useful Tools</a></li>
	<li><a href="Section 4 - Information Gathering/README.md#summary-and-a-tip">Summary And A Tip</a></li>
	</ol>
5. <a href="Section 5 - About ESP/README.md">"how 2maek esp???"</a>
	<ol type="a">
	<li><a href="Section 5 - About ESP/README.md#different-ways-to-draw">Different Ways to Draw</a></li>
		<ol type="i">
		<li>Directx/OpenGL hook</li>
		<li>Engine drawing</li>
		</ol>
	<li><a href="Section 5 - About ESP/README.md#requirements-for-an-esp-internal">Requirements for an ESP</a></li>
	<li><a href="Section 5 - About ESP/README.md#world-to-screen">World to Screen</a></li>
		<ol type="i">
		<li>math is hard</li>
		</ol>
	<li><a href="Section 5 - About ESP/README.md#externals">Externals</a></li>
	<li><a href="Section 5 - About ESP/README.md#chams">Chams</a></li>
		<ol type="i">
		<li>stride, numverts, index, and depth</li>
		<li>d3d11+ equivalents</li>
		</ol>
	<li><a href="Section 5 - About ESP/README.md#summary-and-tips">Summary and Tips</a></li>
	</ol>
6. <a href="Section 6 - About Aimbot/README.md">Aimbot</a>
	<ol type="a">
	<li><a href="Section 6 - About Aimbot/README.md#requirements-for-aimbot-internal">Requirements for Aimbot</a></li>
	<li><a href="Section 6 - About Aimbot/README.md#calculating-the-aim-angle">Calculating the Aim Angle</a></li>
		<ol type="i">
		<li>math is still hard</li>
		</ol>
	<li><a href="Section 6 - About Aimbot/README.md#facing-the-target">Facing the Target</a></li>
	<li><a href="Section 6 - About Aimbot/README.md#stop-aiming-through-walls">(Stop) Aiming Through Walls</a></li>
	<li><a href="Section 6 - About Aimbot/README.md#pixel-scanners">Pixel Scanners</a></li>
	<li><a href="Section 6 - About Aimbot/README.md#summary">Summary</a></li>
	</ol>
7. <a href="Section 7 - Networking/README.md">Networking</a>
	<ol type="a">
	<li><a href="Section 7 - Networking/README.md#videogame-networking">Videogame Networking</a></li>
	<li><a href="Section 7 - Networking/README.md#network-authority">Network Authority</a></li>
	<li><a href="Section 7 - Networking/README.md#packet-inspection">Packet Inspection</a></li>
	<li><a href="Section 7 - Networking/README.md#packet-editing">Packet Editing</a></li>
	<li><a href="Section 7 - Networking/README.md#complications">Complications</a></li>
	<li><a href="Section 7 - Networking/README.md#summary">Summary</a></li>
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

1. Viewing data structures in memory and in a disassembler
2. Basic manual mapper
3. Aimbot + ESP quick rundown for assault cube
	<ol type="a">
	<li>internal and maybe external?</li>
	</ol>
4. Aimbot + ESP quick rundown for an old quake engine game
5. Aimbot + ESP quick rundown for a ue4 game
	<ol type="a">
	<li>GObjects + GNames = GG</li>
	</ol>
6. Aimbot + ESP quick rundown for a unity game
	<ol type="a">
	<li>mono backend vs il2cpp backend</li>
	</ol>
7. Aimbot + ESP quick rundown for a cryengine (5?) game
8. Converting assault cube hack to kernel-mode