# ctf.zh3r0.ml Hardwork Challenge


__Chang Tan__
AWS Certified Cloud Practitioner and Solutions Architect Associate
ctanlister@gmail.com

## I just wanted to say I never completed the challenge but I believe I was very close to it

There were multiple steps involved that required good analytical and forensics skills. Almost nothing in this challenge meets the eye. Almost every file format you see, is hiding something, or encrypted with a password, or hidden via stegnagraphy.

## Two innocous files

First you are going to start off with `unzip OpenThis.zip` and you will get two files

1. A PNG image (supposedly) called ArrayOfCharsThis.jpg
2. A "zip" file called ThisIsAZipFile.zip (it is not)

Attempting to unzip it will give you this

```
(base) ctlister@System76:/1tbdisk/zh3r0ctf/hardwork$ unzip ThisIsAZipFile.zip
Archive:  ThisIsAZipFile.zip
   skipping: ThisIsAPDFfile.pdf      need PK compat. v5.1 (can do v4.6)
   skipping: umbraco.zip             need PK compat. v5.1 (can do v4.6)
(base) ctlister@System76:/1tbdisk/zh3r0ctf/hardwork$
```

## Unzipping the codebase

If you are confused, a quick Google search will yield the result that this is actually a 7zip file. Install p7zip on your Linux box and type `7z x ThisIsAZipFile.zip`

You will be prompted for a password that you do not have. So reinvestigate the PNG image and notice that there is embedded text at the bottom of the file. Reveal it by `strings ArrayOfCharsThis.jpg` and it will reveal the password __`animeandmangarules`__

Unzip the file with the command `7z x -panimeandmangarules ThisIsAZipFile.zip`

Trying to unzip the "zip" file, you'll find out that it throws a error message. The reason why, is that the compressed archive is in 7z format, so `7z x file.zip` to extract it.

You now have two more files

1. A password protected PDF file that you do not know the creds for yet
2. Another zipfile called Umbraco, unzip it into it's own directory

Now take a deep breath, the next one is going to be long.

## Mastering your grep, sed, awk, cut, paste skillz

You will have a lot of files to sift through, but there will be many more to come. You are taking this challenge with the author having the assumption that you know the basic console commands to print, parse, sort, reduce, and organize with efficiency without the use of graphical GUIs.

You will need to learn how to use the following commands, as this exercise is basically a log-analysis task on steroids.

1. `cat`, which concatenates (either prints or combines) multiple file contents together
2. `grep` and it's variants like `egrep` or `grep -e`. In particular, know that `egrep -i` means ignore case, `egrep -irna` means search for characters recursively in your current directory, number the source file, and print binary characters. 
3. `sed`, which stands for stream editor. A example is that you took a previous command printed to the screen by `cat` on the file, and you want to replace all the periods `.` with commas `,`, by using `cat file.txt | sed s/'.'/','/g` you replaced all of the periods printed onto the screen with a comma
4. `awk` is useful for vertical organization. So if you `cat` a file, and then `cat file.txt | awk '{print $1}`, you print only the first column in the text file. Instead of whitespaces as the default delimiter, you can specify `awk -F 'delimiter or string' '{print $column}`. You can also cut down on empty lines by taking the previous command and then `command | awk 'NF'`, to get rid of empty lines on the screen.
5. `cut` has limited usage, because it can only be used with a delimiter of a single character or symbol, so you can `cat` a file, and then `cut -d \: -f2`, to print the second column of the file with `:` as a delimiter.
6. `paste`, is useful for reading files already in a vertical list, like a list of passwords to crack that needs to be accepted in a new format for your app. For example, `autorecon.py` only accepts hosts separated by a space, so you would run `python3 autorecon.py $(paste -sd, targets.txt | sed s/','/'\s'/g)`
7. The pipe operators read `<`, pipe to file `>`, and to read from stdout from a previous command `|` to run the next command on. Specify programs like netcat `nc` will have a use for the pipe operators. 
8. Remember the POSIX terminal operators like `\s` for space, `\t` for tab, `\r` for carriage return, `\n\` for new line. If I learned that credentials start with a capital `KEY=`, then I can `egrep -irna KEY` then separate them using stream editor to create unique lines on the screen like `egrep -irna key | sed s/'KEY='/'\r\nKEY='/g` to make them more distinctive to see, by separating each line that starts with `KEY=` on a brand new line.
9. Remember the `command 2>&1` trick, if you are running some shoddy program that prints to stderr instead of stdout normally (like airodump-ng). By doing that, you can run `airodump-ng wlan0mon 2>&1 | tee logfile.txt` to write everything you see on the screen to a logfile at the same time. Basically, it took stderr and redirected it to stdout along with what was supposed to be printed on stdout (your screen).

Putting all of these together, I successfully recovered the hashes by experimenting with different command combinations and operators to locate the hash. Navigate to your Umbraco folder and then type the command... __next section__

## Locating the hashes
What you are looking for is a SHA-512 hash that has already been cracked. Along the way, you'll find things like PublicTokens and other nice things that may help in completing the challenge. Since I never completed the challenge, I believed the tokens and the clues left behind on the encryption type of the Umbraco config files will help you solve it.

![](https://zherowriteups.s3.amazonaws.com/IMG_20200617_015624881.jpg)

Finally through a insanely command combining what I have taught you in the previous section, you will be able to come across the SHA-512 hash.

![](https://zherowriteups.s3.amazonaws.com/2_hardwork_found_SHA512_hash.png)

When you locate the SHA-512 hash, it is unsalted and is already cracked on crackstation.net


## Monkeying around with QR-codes and barcodes in a PDF

Don't get too excited when you finally unlock the lengthy password for the PDF file. You will not be amused.

There is over 1,400 pages, all containing all kinds of gibberish, QR-codes that reveal insults against you and discourage you from scanning them manually. 

![](https://zherowriteups.s3.amazonaws.com/2_hardwork_barcodes.png)

Entropy analysis of the unlocked PDF file from the `binwalk -E file.pdf` command shows a extreme amount of entropy (randomness) at approximately the first 80% of the file. It briefly goes down in entropy (more alike) before spiking up in entropy again and then plummeting to 0.4 entropy score at the end, meaning that the last 10% of the PDF file are mostly similar

![](https://zherowriteups.s3.amazonaws.com/IMG_20200617_015624881.jpg)


However, by combining several tools together, you can extract each image and barcode as a PNG file, and then have another application scan through them to decode them for clues.


## Automating the scanning of QR-codes

![](https://zherowriteups.s3.amazonaws.com/2_hardwork_automated_taunts.png)

There is a automatic way to parse the barcodes and QR-codes out of the unlocked PDF file, although it took me a hour of Googling and trying different options before I figured out what worked for me.

## More rabbit holes

Auto-parsing the PNG files by the thousands (there is actually 1,410 of them), you will find a message written in hex format.

![](https://zherowriteups.s3.amazonaws.com/2_hardwork_url.png)

Using a online hexidecimal to ASCII converter, you'll get a URL to the hosts of the CTF and their background information, as well as a encouraging mantra to try harder

![](https://zherowriteups.s3.amazonaws.com/2_hardwork_foundurl.png)

## Possible clue

