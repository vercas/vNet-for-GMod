vNet for GMod
=============

A high-level networking library for Garry's Mod

## Features

- Package-oriented communication, while also allowing sending of raw data (as strings)
- Automatic compression within a size treshold
- Fragmentation of packages and multiplexing of fragments
- Fair picking of which package to send next
- Automatic throttling based on client convars
- Will make best use of the network whilst not exhausting the engine's resources

### Packages
- Use files as buffers for binary data
- Practically unlimited size
- Timeouts and arbitrary cancellation
- May contain all primitive types (byte, short, int, float, double, bool)
- May contain several Source engine primitives types (Vector, Angle, Entity) and common Garry's Mod types (Color tables)
- Strings will be encoded as a networked ID if one is available
- Possibility to send tables featuring all data types mentioned above
- Recursivity checks in tables for sub-tables, handled by implicit reference counting (no bytes wasted)
- String de-duplication in tables, also by implicit reference counting, besides encoding as networked ID
- Special handling for booleans and entities in tables to save one byte when possible

## Usability, documentation and examples

vNet exposes many more functions than a user is likely to care about. They are not documented yet, but may be soon.  
Usage examples will follow shortly.

Regarding usability, this same version has been in use for over 5 months and no issue has been found what-so-ever.  
Admittedly, a limited number of people had access to the beta version, but the use cases were putting vNet to its
very limits, which it handles flawlessly.

I would go as far as saying it is production-ready.

Some advice: make good use of writing tables to packages, as well as sending raw strings.
Network common strings that would need to be sent to save space in packages.  
Luckily, quite a great deal of strings are pre-networked. Since the checking for networked ID is transparent, this may
save space in surprising situations.

## Description

I started working on vNet initially due to the need to reliably send large payloads from clients to the server and vice-versa,
such as images and code files, which are way out of the capabilities and scope of the standard net library provided in the game.

Whilst working on that, I had to fragment large packages, allow small packages with little overhead and give them a fair share
of the bandwidth. Thus, I added fragmentation of large packages and multiplexing fragments from many packages.
Packages that fit into one fragment will easily go through, and the number of fragments per message is recalculated based
on the available space in a message.

I also looked up and tested several convars for controlling the networking between the server and the client.
I learned which variables to read and use to throttle the connection in such a way that packages get send efficiently
while the game engine also gets to send its data, without dropping snapshots or increasing latency.
Each client tells the server the rate at which it wants to communicate.

Packages have timeouts and may be cancelled by either side at any time.
Another feature which was efficiently implemented is client-to-client communication, with full and flexible support for
fragmentation and multiplexing like any other packet.

The choice to use files as package buffers is because of the (lack of) speed of converting values to binary data in Lua,
accompanied by the horror of continuous concatenation of them. (think of appending one byte to a string, which
requires doubling the memory used)  
Write caching (in the engine and/or the OS) saves performance, luckily.

File buffers can easily be bypassed by sending raw strings, which may contain arbitrary binary data as well as just text.  
This is useful for many purposes where file buffers are unnecessary, mainly sending a single string or a whole file.

## Internals

The code is really messy. Mainly because it has to be fast in a slow language and environment.  
To keep track of its stuff, vNet has three different internal "structures" (2 actually, but one of them is different
depending on whether it is on a client or on a server) with many optional fields, magic values...  
The temporary data is also cleaned up whenever possible (both from memory and file storage).

I fear the maintainability of the code is extremely low, but, luckily, it has few dependencies and a possible change
in Garry's Mod's networking or filesystem interfaces will likely be easy to fit into the code.

## Future

I really do not know what else I should or could add.  

I have started a stub for tracking global packages (for debugging, etc.) but I have not added any means to hook into those
functions externally. If anyone would want this done, find me and ask me, but not before you figure out a way to do so
without making me want to call this abandoned/unsupported.  
Yes, I'm really busy, and yes, I'd like to maintain this little project alive.
