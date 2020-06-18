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

Instead of overwriting the RBP register with what I was supposed to do (later in this write-up), I ended up overwriting the Return Base Pointer with the address of the the Return Stack Pointer, which points to the current position at the top of the executable stack. Thereby, throwing the application into a infinite loop.

Additional fuzzing and dumping of register virtual address spaces revealed that I had a 32-byte buffer to play with, to drop a reverse netcat payload. 