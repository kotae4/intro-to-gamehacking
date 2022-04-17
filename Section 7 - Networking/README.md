# Networking #

Network-based cheats are possible as well, though they are certainly seen less frequently. Instead of reading and/or writing the game's memory, you'll be reading and/or writing the data the game sends to the game server.

## Videogame Networking ##

Videogames are real-time applications and user actions often occur dozens of times per minute, so naturally there will be a lot of incoming and outgoing packets in most multiplayer games. Furthermore, players may be located very far from the game server or very close, so latency per-player is variable. Games have to design their networking system with this in mind.<br>
Consider how you'd implement hit detection when player 1 has a latency of 300ms and player 2 has a latency of 100ms. If you send player 2's 'real' position to player 1, player 1 will still be looking at player 2's 'old' position for at least 300ms, so what happens if player 1 shoots at player 2's 'old position'? If you say 'player 1 will miss' then consider the consequences and scale this out to 10 players or even 100 players. Also keep in mind that the server is performing logic too, so there's a 'hidden' latency in how long it takes the server to perform its tasks.<br>
All of this is to say that videogame networking must solve some non-trivial problems and the solutions will never be perfect.<br>
So some of the things that some networking systems may do:
1. Compress data. Player positions and rotations are usually represented by Vector3f's which are 12 bytes each. Compressing the floats into a short can half that size while still retaining enough precision to be useful. Integers can be compressed to bytes if their value is under 255 or shorts if their value is under 2^15.
2. Pack multiple 'game packets' into a single network packet. For example, instead of sending a 'movement' packet and a 'shoot' packet, if they occur at the same time then you could combine both the 'movement' and 'shoot' game packet into one network packet (if size limits aren't exceeded).
3. Build TCP-like features on top of UDP (connections, sequencing, re-sending of missed packets - though most missed packets will just be ignored).
4. Use delta values instead of absolute values. The game knows that the player was at (68319, 200, 59402) last update so we don't need to tell them they're at (68328, 180, 59392) now, we can just say they moved by (-9, 20, 10).
5. Synchronize inputs rather than gamestate. If we know pressing the 'W' key always modifies the gamestate in the same way (the player moves forward at X velocity), then we can just tell the server we pressed the 'W' key instead of telling them 'we moved by (X units, Y units, Z units)' (and the server will send 'player1 pressed W' to all other clients too, and the other clients will modify their game state accordingly).
6. Send only relevant data to each player rather than all data to all players. If player2 is outside the render distance of player1 then neither player needs to receive updates about the other. Another example might be if they **are** close but there's no line of sight and no way to interact with each other without a line of sight.

Nearly every multiplayer game will interpolate between network-updated positions. Corrections must occur as the latest 'real' data comes in. Sometimes, games will send delta sync packets a dozen times a second accompanied by one 'full' sync packet every second. The full sync packet would contain all the data necessary to re-create the gamestate (or at least the part of the gamestate that the packet relates to).

## Network Authority ##

Some games may use a client-server networking paradigm where the gameserver is a dedicated and authoritative server. Other games may use a relay-server networking paradigm where one client (often the first client to connect to the relay-server) is the designated 'host' and is given authority.<br>
Authority here refers to authority over the gamestate. It's the authority to determine whether an attack hits or misses, whether someone's health is 100 or 30, and even whether someone can move forward or not (collision detection and other physics-related tasks are performed by the authority too, which requires knowledge of the physical game world and a beefy CPU).<br>
If the game isn't relay-based or you're not the authoritative host, then some kinds of cheats, like god-mode or teleportation, are simply impossible to perform. Of course, there may be bugs within the game's code that you might be able to abuse to perform these tasks, but those are often quickly patched.<br>
In my experience, games that use relay-based networking have far more bugs, but this may be some confirmation bias and selection bias on my part.

## Packet Inspection ##

The first step in reverse engineering any network communication is to familiarize yourself with a packet sniffer. Wireshark is probably the most popular and has been around for ages. With a packet sniffer you'll be able to see packets as they come and go in real time. Wireshark will also break down the packets for you and show you the various headers and data.

## Packet Editing ##

Often, packet sniffers do not offer the ability to edit packets on the fly. Some may allow you to edit a packet and 're-send' it, but this isn't good enough for our usage. We need to be able to either stop a specific packet from being sent or edit it before it's sent.<br>
I can only think of a few ways of doing this:
1. Inject a DLL into the target process and hook the necessary winsock functions
2. Install a kernel driver in the network interface card (NIC) driver stack
3. Utilize additional hardware (virtual or physical) in between your PC and your modem. You'd write a simple forwarder server, except you'd add rules to filter your game packets and perform whatever modifications before forwarding those.

#1 is probably the easiest and #3 second easiest if you have a spare PC or a powerful enough main PC to run a VM while gaming.<br>
I haven't tested any of these techniques myself, but I have used tools that performed technique #1.

## Complications ##

As mentioned earlier, compression and other techniques utilized in the game's networking system may make identifying patterns in packets difficult.<br>
Some newer games are even encrypting their packets so you'd need to figure out how to decrypt them in order to glean anything useful.<br>
Whether packets are encrypted or not, you'll most likely need to reverse engineer the game to discern packet structures. The good news is that since all packets are sent via winsock winapi calls you can start from there and work backwards until you find the game function(s) responsible for assembling these packets.<br>
This, coupled with the limited methods for editing packets on the fly, ultimately makes network-based cheats no less challenging than memory-based cheats.

## Summary ##

Network-based cheats are appealing because it allows bypassing any memory-based protection the game may be using, but in practice you may find network-based cheats are full of their own complications that make it no less challenging than memory-based cheats.<br>
With that said, however, I believe there may be better re-usability of network-based cheats, and the reverse engineering necessary for each game has a static starting point, so this may be something worth pursuing and specializing in.