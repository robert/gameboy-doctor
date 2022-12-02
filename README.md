# Gameboy Doctor

Are you working on building a Gameboy emulator? Are you failing [Blargg's test ROMs](TODO), or can't even run them? Are you having trouble finding your bugs?

Gameboy Doctor can help!

## What is Gameboy Doctor?

Gameboy Doctor is a tool that compares your emulator's CPU to an example emulator that passes Blargg's test ROMs. It finds the exact tick when your CPU's state diverges from the example CPU, which helps you isolate and fix your bugs.

## How do I use Gameboy Doctor?

1. Choose a test ROM (if you're having trouble running any of them then [`cpu_instrs`](TODO) numbers `06` and `07` are good ones to start with)
1. Make 2 tweaks to your emulator:

1. Initialize the CPU's state to the state it should have immediately after executing the boot ROM:

| Register | Value |
| ----------- | ----------- |
|A|0x01|
|F|0xB0 (or `CH-Z` if managing flags individually)|
|B|0x00|
|C|0x13|
|D|0x13|
|E|0xD8|
|H|0x01|
|L|0x4D|
|SP|0xFFFE|
|PC|0x0100|

1. Hardcode your LCD (or memory map if you haven't implemented an LCD yet) to return `0x90` when the `LY` register is read (memory location `0xFF44`). This is what I did when generating these logs, in order to prevent spurious log divergences.

1. Update your emulator to log to a file the state of the CPU after each operation to a, using the following format:

```
A:00 F:11 B:22 C:33 D:44 E:55 H:66 L:77 SP:8888 PC:9999 PCMEM:AA,BB,CC,DD
```

All of the values between `A` and `PC` are the hex-encoded values of the corresponding registers. The final value (`PCMEM`) is the 4 bytes stored in the memory locations near `PC` (ie. the values at `pc,pc+1,pc+2,pc+3`).

1. Run your emulator and create a log file. You can kill the program at any point - Gameboy Doctor will tell you if your log file is correct but ends before the test ROM has finished. If you pass the test then your emulator will display the word "Passed" on the LCD, and write the bytes for the word "Passed" to [the serial output](TODO)

1. Once you have your logfile, feed it into Gameboy Doctor like so:

```
./gameboy-doctor /path/to/your/logfile $ROM_TYPE $ROM_NUMBER
```

For example, to check the 3rd cpu_instrs ROM:

```
./gameboy-doctor /path/to/your/logfile cpu_instrs 3
```

On windows you may need to invoke the Python interpreter directly:

```
python3 gameboy-doctor /path/to/your/logfile cpu_instrs 3
```

1. Gameboy Doctor will tell you how you've done! For example:

```
$ ./gameboy-doctor ../my-emulator/logs/3.log cpu_instrs 3
============== ERROR ==============

Mismatch in CPU state at line 9997:

MINE:	  A:3E F:C--- B:01 C:07 D:C9 E:BA H:49 L:BB SP:FFFE PC:0208 PCMEM:1C,20,FB,14
YOURS:	A:3D F:C--- B:01 C:07 D:C9 E:BA H:49 L:BB SP:FFFE PC:0208 PCMEM:1C,20,FB,14

The CPU state before this (at line 9996) was:

	      A:3E F:10 B:01 C:07 D:C9 E:BA H:49 L:BB SP:FFFE PC:0207 PCMEM:12,1C,20,FB

The last operation executed (in between lines 9996 and 9997) was:

	      0x12 LD (DE) A

Perhaps the problem is with this opcode, or with your interrupt handling?
```

Or eventually:

```
$ ./gameboy-doctor ../my-emulator/logs/3.log cpu_instrs 3
============== SUCCESS ==============

Your log file matched mine for all 1066160 lines - you passed the test ROM!
```
