# scratch-profiling

Scripts for profiling scratch memory usage etc.

Makes use of python + psutil to obtain memory usage recursively for a particular spawned process.

# Usage

To examine memory usage:

```
sudo apt install python3-matplotlib
./memory.py --cmd scratch3 --out graph.png
```

To examine files from the ASAR:

```
npx asar extract app.asar unpacked
./asar_investigate.py --in unpacked/1.bundle.js
```

# Examples

I believe now that USS gives a more accurate measure of memory utilised when summed for child processes than RSS, as 
I ran 'free -m' while Scratch was running and after, and the difference was around 350MB, which is much closer to the USS value.

## Scratch3 Raspbian Version

![scratch3 memory](images/memory-scratch3.png)


## Scratch3 New Version

![scratch3 memory](images/memory-scratch3new.png)

# Base64

I found the running Scratch processes doing:

```
ps ax | grep "[s]cratch" | awk '{ print $1 }' | tr '\n' ' '
```

Using the PIDs obtained through the previous step, I then did the following to create core dumps for each process, and output strings from them:

```
gcore -o dump.bin 4899 4902 4904 4930 4935 4937 4939 4970 5021
strings dump.bin.*
```

I noticed in the dumps there appeared to be some Base64, which existed in the app.asar file, confirmed by:
```
grep "<BASE64 String>" app.asar
```

I need to determine if this Base64 data was actually in the RAM itself, or was a memory mapped file.

I am currently looking at the section "Controlling which mappings are written to the core dump" in https://man7.org/linux/man-pages/man5/core.5.html

It says "By default, the following bits are set: 0, 1, 4", which would mean file backed storage wouldn't be written with the core dump.

It seems using 'gcore -a' ignores these filter bits and captures all data, so I removed using the '-a' argument.

# To Do

* Investigate ASAR contents
* Profile memory usage
