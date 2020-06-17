

# ctf.zh3r0.ml Command_1 Challenge


__Chang Tan__
AWS Certified Cloud Practitioner and Solutions Architect Associate
ctanlister@gmail.com


This was actually the easiest of the pwn/binary exploitation boxes. All it took was a 10 minute video on how to use GHIDRA or, if you prefer, IDA Pro to understand everything.

Basically the entire function of the program is to provide a text-GUI prompt for you to run shell commands on the system.

## Binary analysis

![](https://zherowriteups.s3.amazonaws.com/3_command_1.png)

I am going to keep this brief and will provide a copy of the binary to download to play around with. But basically, there is one critical function that stood out like a sore thumb, the addcommand() function.

According to the decompiled code, which in almost all major cases will be in C, there are only five commands allowed to be saved.

<ol>
<li>/bin/sh</li>
<li>/bin/echo</li>
<li>/bin/cat</li>
<li>/bin/shutdown</li>
<li>init 0</li>
</ol>

If there is anything else you entered as a command, the app will rudely mock you and log you out.

That is why it is important to statically analyze a binary before messing with it, either in a CTF or real-time scenario. Instead of just blindly netcatting to the server and getting frustrated. It is also important to do this before any amount of fuzzing will be considered, since you'll be sinking down a ticking rabbit hole.

## Took me 30 minutes after locating this function to pwn the box

Once I realized what was going on, I stopped to immediately...

1. Connect to the server
2. Save a command, in my case it's `sh`
3. Select the command
4. Execute the command

It promptly dropped me into a shell prompt on the remote server without authentication. It was that easy. All I had to do was run `ls` and then `cat flag.txt` and I doubled the points of my team on the first day.



