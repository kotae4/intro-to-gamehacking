# Intro to Game Hacking #
Videogames provide a great medium for programmers to improve their skills through designing and implementing many complex systems that must be performant and act cohesively.  
In turn, videogames also serve as a great medium for fledgling reverse engineers to learn the ropes and gain practical experience picking apart those complex systems and exploiting them.


## Topics to cover (loosely ordered) ##
1. Basics
	<ol type="a">
	<li>anatomy of a PE image</li>
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
	<li>detours</li>
		<ol type="i">
		<li>e9 jmp and ff25 jmp</li>
		<li>trampoline</li>
		</ol>
	<li>VEH hooking</li>
	<li>HWBP hooking</li>
	<li>vtable hooking</li>
		<ol type="i">
		<li>swap vtable ptr or overwrite element in vtable</li>
		</ol>
	<li>many more, value of creativity</li>
	</ol>
3. Injection
	<ol type="a">
	<li>VirtualAllocEx, WPM, CRT @ LoadLibrary</li>
	<li>manual mapping</li>
		<ol type="i">
		<li>the wrong way</li>
		</ol>
	<li>thread hijacking</li>
	<li>PEB unlinking for all cases</li>
	<li>considerations for UWP apps</li>
	<li>many more, value of creativity</li>
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
