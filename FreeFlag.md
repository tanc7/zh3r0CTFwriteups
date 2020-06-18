# ctf.zh3r0.ml FreeFlag Challenge


__Chang Tan__
AWS Certified Cloud Practitioner and Solutions Architect Associate
ctanlister@gmail.com

## Note: This is a binary exploitation box, but it is NOT a reverse shell box

The author has made the mistake of wasting his time attempting to spawn a reverse shell (which worked locally), until he realized he only called to call the function pointer to print the flag after he identified the pointer location in GHIDRA.

## However: I did manage to spawn a reverse shell with a limited buffer space of 32 bytes WITHOUT control of the Return Instruction Pointer

I called it the __inertia-less trampoline technique__.

After a half hour of fiddling with it, I realized that I cannot overwrite the Return Instruction Pointer, the Instruction Pointer being the holy grail of exploit developers as it enables them to perform, but not limited to...

1. Redirection/Hijacking of execution into attacker controlled buffer space
2. ROP-chaining to shut down NX (Non-Executable) Data bits (in Windows, known as DEP or Data Execution Prevention)
3. Stack-pivoting by feeding malicious instructions (a fake stack) to execute shellcode or avoid stack canaries/cookies
4. Performing short and long jumps in the stack
5. Bypassing ASLR (Address Space Layout Randomization)
6. Dropping egghunters into uncertain buffer space ranges to execute hard to find shellcode or dealing with extremely small writable buffers inadequate to hold primary-stage shellcode

However I had control of the other registers, and managed to overwrite the Return Base Pointer. That gave me a idea.

![](https://zherowriteups.s3.amazonaws.com/1_freeflag_overwrote_rbp_with_rsp.png)

Instead of overwriting the RBP register with what I was supposed to do (later in this write-up), I ended up overwriting the Return Base Pointer with the address of the the Return Stack Pointer, which points to the current position at the top of the executable stack. Thereby, throwing the application into a infinite loop.

![](https://zherowriteups.s3.amazonaws.com/Screenshot+from+2020-06-17+17-57-30.png)

Additional fuzzing and dumping of register virtual address spaces revealed that I had a 32-byte buffer to play with, to drop a reverse netcat payload. 

## Ultimately, I behaved. And located the address I was supposed to call in my return instruction

Within Evan's Debugger (edb-debugger), I located the memory address to properly overwrite the base pointer.

![](https://zherowriteups.s3.amazonaws.com/1_freeflag_winwin.png)

## Let's take a big step back and go through the basics of fuzzing, stack based buffer overflows, and pointer overwrites

We're going to break it down to the following steps...

1. Static Analysis, Fuzzing & Crash Analysis
2. Assessing Exploit Opportunities
3. Eliminating Bad Characters
4. Building a Exploit
5. Successful Exploitation

### 1. Static Analysis, Fuzzing & Crash Analysis

In a CTF like environment, it's a good idea to gain as much intel as you can about your intended target application. And that means, obtaining a copy of the binary for static analysis so that we can focus on what prints our flag or pops a shell instead of mindlessly throwing strings until the app blows up with errors to sift through.

Know at least what we are attacking.

Today, we are using GHIDRA, the NSA's infamous open-sourced competitor to IDA-Pro. You may download it at ghidra-sre.org

First unzip GHIDRA into your installation directory of choice, for me, I am running it on my persistent bootable Kali Linux USB Drive because my KVM Kali VM has been screwing up lately.

__```sudo unzip ghidra_9.1.2_PUBLIC_20200212.zip -d /run/live/persistence/sdc3/```__

Now make a symbolic link so you can just run __`ghidra`__ on the command line to run it. First navigate to `/run/live/persistence/sdc3/ghidra_9.1.2_PUBLIC` and then type __`sudo ln -s $PWD/ghidraRun /usr/local/bin/ghidra`__

__I strongly advise watching this short eleven-and-a-half minute video of how to run GHIDRA and perform a simple exploit by viewing the decompilation of functions in a binary.__

`https://www.youtube.com/watch?v=fTGTnrgjuGA`

When you are done. Run `ghidra` on the command line

### 2. Assessing Exploit Opportunities
### 3. Eliminating Bad Characters
### 4. Building a Exploit
### 5. Successful Exploitation

# Bonus: My big mistake. Accidentally creating a reverse shell instead without controlling the instruction pointer.
