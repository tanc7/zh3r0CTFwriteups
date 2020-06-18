# ctf.zh3r0.ml FreeFlag Challenge


__Chang Tan__
AWS Certified Cloud Practitioner and Solutions Architect Associate
ctanlister@gmail.com

## Note: This is a binary exploitation box, but it is NOT a reverse shell box

The author has made the mistake of wasting his time attempting to spawn a reverse shell (which worked locally), until he realized he only called to call the function pointer to print the flag after he identified the pointer location in GHIDRA.

## However: I did manage to spawn a reverse shell with a limited buffer space of 32 bytes WITHOUT control of the Return Instruction Pointer, see bottom of write-up

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

![](ghidrasplash)

Playing around with GHIDRA, you might notice three functions.
1. main()
2. here()
3. winwin()

main() directly calls here(), and here() merely accepts your buffer to overflow. Winwin() however cannot be called, but what is interesting is if you search for the function in GHIDRA, the address is 0x400707.

![](winwin_function)

In other words, the objective is to overflow the RBP (Return Base Pointer), with that address for winwin() AKA 0x400707, to make the exploited app run the command `cat flag.txt`, thereby printing your flag.

Install gdb-PEDA if you have not already.

First let's create a payload to see if we can crash the program. `python -c 'print "A"*100' > payload.txt`
Now have gdb run a debugger session, first make the app executable, `chmod 700 freeflag`, and then run gdb with the peda plugin installed, `gdb freeflag`

In the prompt, crash the program with 100 letter A's, `run < payload.txt`, and let's analyze the crash.

![](firstcrash)

As you can see, there are a few interesting things going on here. Note that the Return Instruction Pointer remains unharmed and not overwritten. This is bad news. But also note that we completely overwrote the Return Base Pointer with eight capital A's. That is promising, because the base pointer controls when the program returns back to the Return Stack Pointer when execution hits it.

Also, please count the length of the buffer that landed between the registers. As you'll find out later, the buffer of A's began in the RSI register, but it is NOT 100 bytes long. Each row is 16 bytes, with the fourth row being occupied by eight bytes of A's. So 3x16+8 = 56 bytes.

Take note of this. We only have 56 bytes to play with.

### 2. Assessing Exploit Opportunities

Now let's create a pattern to map the exact point where RBP is about to be overwritten.

```
gdb freeflag
pattern create 100
```

Then copy and paste the pattern after typing `run`, right into the console and watch the crash.

![](secondcrash)

In gdb-PEDA, you can specifically search for the offset, that is the distance from a desired section of the buffer and the amount of "junk" letter A's needed to reach it, by specifically typing `pattern search $register`. However, a more convenient way is to dump all the offsets that we need, just run __`pattern search`__ and copy and paste the output into a file called exploit.py

![](Register offsets)

As you can see from the register offsets, it takes a buffer of 32 A's to reach the RBP register. By which point, the following eight bytes of our cherry-picked winwin() memory address will overflow the base pointer, making the total exploit length 40 bytes before winwin() is called.

Before we craft a exploit, we need to eliminate bad characters.

### 3. Eliminating Bad Characters
### 4. Building a Exploit
### 5. Successful Exploitation

# Bonus: My big mistake. Accidentally creating a reverse shell instead without controlling the instruction pointer.
