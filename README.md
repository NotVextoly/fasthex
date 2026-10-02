# fasthex

fasthex — a very fast hex dumper (written in Rust), with all features that other hex dumpers have too.

## Table of Contents

- [Quick Start](#quick-start)
- [Benchmarks](#benchmarks)
- [Uninstall](#Uninstall)
- [Usage](#usage)
- [Features](#features)
- [Example Output](#example-output)
- [How It Works](#how-it-works)
- [Testing Conditions](#testing-conditions)

## Quick Start
- **On Arch**
```bash
paru -S fasthex 
# or fasthex-bin for a prebuilt release
```
- **Non-arch**
```bash
cargo install fasthex
```
> **Note**: Make sure `~/.cargo/bin` is in your `PATH`. It's added automatically by rustup, but if `fasthex` isn't found, add this to your shell config file:
> ```bash
> # If you use Bash:
> export PATH="$HOME/.cargo/bin:$PATH"
>
> # If you use Fish:
> fish_add_path $HOME/.cargo/bin
>
> # If you use Zsh:
> export PATH="$PATH:$HOME/.cargo/bin"
> ```

```bash
# Use it!
fasthex /path/to/file
```

## Benchmarks

All benchmarks were run on the same system and under the methodology described in [Testing Conditions](#testing-conditions). File I/O measurements use Linux page cache (RAM) speed rather than physical disk throughput.

### Benchmark 1: Small Files — CLI Startup Overhead

50 measured runs per test.

| Tool       | 4KB         | 16KB        | 64KB        |
| ---------- | ----------- | ----------- | ----------- |
| **rawhex** | **1.04 ms** | **1.04 ms** | **1.22 ms** |
| fasthex    | 1.67 ms     | 1.61 ms     | 1.95 ms     |
| xxd        | 1.51 ms     | 3.90 ms     | 13.51 ms    |
| hexdump -C | 1.51 ms     | 4.21 ms     | 14.37 ms    |

### Benchmark 2: Default Settings — 50MB

5 measured runs due to the runtime of xxd and hexdump.

| Tool       | Average     | Min      | Max      | Median   | StdDev  |
| ---------- | ----------- | -------- | -------- | -------- | ------- |
| **rawhex** | **9.91 ms** | 8.73 ms  | 11.31 ms | 9.77 ms  | 0.97 ms |
| fasthex    | 44.92 ms    | 42.50 ms | 50.09 ms | 43.74 ms | 3.02 ms |
| xxd        | 9.34 s      | 9.11 s   | 9.47 s   | 9.42 s   | 153 ms  |
| hexdump -C | 9.79 s      | 9.59 s   | 10.05 s  | 9.79 s   | 203 ms  |

### Benchmark 3: Large Files — rawhex vs fasthex

50 measured runs per test.

#### 500MB

| Tool       | Average      | Min       | Max       | Median    | StdDev   | Speedup   |
| ---------- | ------------ | --------- | --------- | --------- | -------- | --------- |
| **rawhex** | **71.71 ms** | 54.99 ms  | 104.13 ms | 67.90 ms  | 15.51 ms | **4.89x** |
| fasthex    | 350.43 ms    | 335.45 ms | 391.69 ms | 348.93 ms | 9.12 ms  | 1x        |

#### 1GB

| Tool       | Average       | Min       | Max       | Median    | StdDev   | Speedup   |
| ---------- | ------------- | --------- | --------- | --------- | -------- | --------- |
| **rawhex** | **124.42 ms** | 108.32 ms | 210.48 ms | 112.59 ms | 23.55 ms | **5.54x** |
| fasthex    | 689.08 ms     | 674.86 ms | 714.26 ms | 687.96 ms | 9.07 ms  | 1x        |

#### 1.5GB

| Tool       | Average       | Min       | Max       | Median    | StdDev   | Speedup   |
| ---------- | ------------- | --------- | --------- | --------- | -------- | --------- |
| **rawhex** | **229.01 ms** | 164.46 ms | 311.75 ms | 230.95 ms | 48.68 ms | **4.52x** |
| fasthex    | 1.04 s        | 1.00 s    | 1.08 s    | 1.03 s    | 18.03 ms | 1x        |

#### 3GB

| Tool       | Average       | Min       | Max       | Median    | StdDev   | Speedup   |
| ---------- | ------------- | --------- | --------- | --------- | -------- | --------- |
| **rawhex** | **464.78 ms** | 334.98 ms | 605.70 ms | 456.42 ms | 81.30 ms | **4.39x** |
| fasthex    | 2.04 s        | 1.97 s    | 2.11 s    | 2.04 s    | 31.47 ms | 1x        |

### Benchmark 4: Zero File — 100MB

5 measured runs.

| Tool       | Average      | Min       | Max       | Median    | StdDev  |
| ---------- | ------------ | --------- | --------- | --------- | ------- |
| **rawhex** | **16.94 ms** | 15.42 ms  | 19.35 ms  | 16.75 ms  | 1.60 ms |
| fasthex    | 80.09 ms     | 78.24 ms  | 82.75 ms  | 80.27 ms  | 1.79 ms |
| xxd        | 22.03 s      | 21.17 s   | 22.86 s   | 22.32 s   | 722 ms  |
| hexdump -C | 198.05 ms    | 194.10 ms | 204.68 ms | 196.69 ms | 4.52 ms |

### Benchmark 5: Column Width — 100MB

50 measured runs.

| Tool    | Width    | Average      | Min       | Max       | Median    | StdDev  |
| ------- | -------- | ------------ | --------- | --------- | --------- | ------- |
| rawhex  | 8 bytes  | 144.40 ms    | 126.32 ms | 153.39 ms | 146.59 ms | 6.41 ms |
| rawhex  | 16 bytes | **17.81 ms** | 12.94 ms  | 25.32 ms  | 16.73 ms  | 3.48 ms |
| rawhex  | 32 bytes | 56.39 ms     | 42.91 ms  | 68.50 ms  | 57.13 ms  | 4.40 ms |
| rawhex  | 64 bytes | 52.89 ms     | 44.08 ms  | 59.77 ms  | 53.18 ms  | 3.59 ms |
| fasthex | 8 bytes  | 96.59 ms     | 90.14 ms  | 107.49 ms | 95.78 ms  | 4.39 ms |
| fasthex | 16 bytes | 79.82 ms     | 74.81 ms  | 88.13 ms  | 79.27 ms  | 3.17 ms |
| fasthex | 32 bytes | 76.36 ms     | 69.90 ms  | 83.68 ms  | 76.03 ms  | 3.36 ms |
| fasthex | 64 bytes | **72.81 ms** | 66.30 ms  | 80.03 ms  | 72.52 ms  | 2.83 ms |

### Benchmark 6: Minimal Output — 100MB

50 measured runs.

| Tool    | Mode            | Average      | Min      | Max      | Median   | StdDev  |
| ------- | --------------- | ------------ | -------- | -------- | -------- | ------- |
| rawhex  | default         | 18.99 ms     | 13.59 ms | 27.68 ms | 18.12 ms | 3.83 ms |
| rawhex  | `-m` (no ASCII) | **16.02 ms** | 11.73 ms | 22.58 ms | 15.60 ms | 2.80 ms |
| fasthex | default         | 80.41 ms     | 71.71 ms | 93.87 ms | 80.19 ms | 4.25 ms |
| fasthex | `--minimal`     | 65.16 ms     | 57.31 ms | 74.30 ms | 64.42 ms | 3.09 ms |

### Benchmark 7: Thread Count Scaling — 100MB

50 measured runs.

| Tool    | Threads            | Average      | Min      | Max      | Median   | StdDev  |
| ------- | ------------------ | ------------ | -------- | -------- | -------- | ------- |
| rawhex  | 1                  | 38.08 ms     | 35.58 ms | 46.93 ms | 37.85 ms | 1.94 ms |
| rawhex  | 6 (physical cores) | **17.40 ms** | 13.02 ms | 25.64 ms | 15.89 ms | 3.67 ms |
| rawhex  | 12 (logical cores) | 22.11 ms     | 15.89 ms | 29.10 ms | 22.70 ms | 3.39 ms |
| fasthex | single-threaded    | 79.09 ms     | 74.99 ms | 85.40 ms | 78.81 ms | 2.29 ms |

### Benchmark 8: Grouping — 100MB

50 measured runs.

#### rawhex

| Group   | Average      | Min       | Max       | Median    | StdDev  |
| ------- | ------------ | --------- | --------- | --------- | ------- |
| 1 byte  | 140.30 ms    | 127.93 ms | 150.40 ms | 140.76 ms | 3.88 ms |
| 2 bytes | **15.77 ms** | 13.08 ms  | 25.13 ms  | 14.64 ms  | 2.83 ms |
| 4 bytes | 134.51 ms    | 117.68 ms | 140.19 ms | 135.51 ms | 4.36 ms |
| 8 bytes | 133.79 ms    | 116.93 ms | 139.35 ms | 134.35 ms | 4.41 ms |

#### fasthex

| Group   | Average      | Min      | Max      | Median   | StdDev  |
| ------- | ------------ | -------- | -------- | -------- | ------- |
| 1 byte  | **77.99 ms** | 73.59 ms | 82.97 ms | 77.76 ms | 2.24 ms |
| 2 bytes | 85.14 ms     | 78.60 ms | 91.55 ms | 84.90 ms | 2.93 ms |
| 4 bytes | 81.67 ms     | 76.67 ms | 91.90 ms | 80.99 ms | 3.19 ms |
| 8 bytes | 78.01 ms     | 73.37 ms | 86.13 ms | 78.24 ms | 2.73 ms |

### Benchmark 9: Uppercase Output — 100MB

50 measured runs.

| Tool    | Mode             | Average      | Min      | Max      | Median   | StdDev  |
| ------- | ---------------- | ------------ | -------- | -------- | -------- | ------- |
| rawhex  | default          | 16.49 ms     | 13.39 ms | 24.94 ms | 15.52 ms | 2.81 ms |
| rawhex  | `-u` (uppercase) | **15.86 ms** | 13.06 ms | 23.93 ms | 14.87 ms | 2.63 ms |
| fasthex | default          | 78.03 ms     | 73.15 ms | 84.02 ms | 77.81 ms | 2.09 ms |
| fasthex | `-u` (uppercase) | 78.50 ms     | 74.14 ms | 89.82 ms | 77.81 ms | 2.90 ms |

### Benchmark 10: Squeeze — 100MB Zero File

50 measured runs.

| Tool    | Mode           | Average      | Min      | Max       | Median   | StdDev  |
| ------- | -------------- | ------------ | -------- | --------- | -------- | ------- |
| rawhex  | default        | **17.38 ms** | 12.63 ms | 26.97 ms  | 15.97 ms | 4.02 ms |
| rawhex  | `-z` (squeeze) | 52.11 ms     | 46.40 ms | 70.77 ms  | 49.95 ms | 5.55 ms |
| fasthex | default        | 77.10 ms     | 71.40 ms | 82.78 ms  | 76.92 ms | 2.19 ms |
| fasthex | `-w` (squeeze) | 88.31 ms     | 80.68 ms | 108.65 ms | 86.47 ms | 6.54 ms |

### Benchmark 11: Skip Offset — 100MB

50 measured runs.

| Tool    | Skip     | Average      | Speedup vs full |
| ------- | -------- | ------------ | --------------- |
| rawhex  | 0 (full) | 16.78 ms     | 1.00x           |
| rawhex  | 1 MB     | 16.44 ms     | 1.02x           |
| rawhex  | 10 MB    | **15.43 ms** | **1.09x**       |
| fasthex | 0 (full) | 79.82 ms     | 1.00x           |
| fasthex | 1 MB     | 78.70 ms     | 1.01x           |
| fasthex | 10 MB    | 71.24 ms     | **1.12x**       |

### Benchmark 12: SIMD vs No-SIMD — 100MB

50 measured runs.

| Tool    | Mode           | Average      | Speedup                 |
| ------- | -------------- | ------------ | ----------------------- |
| rawhex  | SIMD (default) | **16.53 ms** | **1.00x**               |
| rawhex  | `--no-simd`    | 134.86 ms    | **0.12x (8.2x slower)** |
| fasthex | always SIMD    | 81.63 ms     | N/A                     |

## Testing Conditions

### System Configuration

**CPU:** AMD Ryzen 5 3600 6-Core Processor (12 threads)

* 6 physical cores
* 2 threads per core
* L1d cache: 192 KiB (6 instances)
* L1i cache: 192 KiB (6 instances)
* L2 cache: 3 MiB (6 instances)
* L3 cache: 32 MiB (2 instances)
* Max frequency: 4208 MHz

**Memory:** 31.3 GB DDR4

**Disk:** Intenso Internal 2.5 Inch SSD SATA III Top, 512 GB, 520 MB/s, ext4

**OS:** Alpine Linux, Linux 6.18.44-0-lts

### Tools Tested

* rawhex 1.6 (clang 22.1.3, `-O3 -flto`)
* fasthex 0.3.5
* xxd (BusyBox v1.37.0)
* hexdump -C (BusyBox v1.37.0)

### Test Files

| File            | Size   | Source         |
| --------------- | ------ | -------------- |
| `urandom_50MB`  | 50 MB  | `/dev/urandom` |
| `urandom_100MB` | 100 MB | `/dev/urandom` |
| `urandom_500MB` | 500 MB | `/dev/urandom` |
| `urandom_1GB`   | 1 GB   | `/dev/urandom` |
| `urandom_1.5GB` | 1.5 GB | `/dev/urandom` |
| `urandom_3GB`   | 3 GB   | `/dev/urandom` |
| `zero_100MB`    | 100 MB | `/dev/zero`    |

### Benchmark Methodology

* Tool: `rawbench` (C11 benchmarking tool)
* Flags: `-c` (compare), `-w 2` (2 warmup runs), `-a 50` (50 measured runs)
* All times are wall-clock time (real time)
* Results report average, minimum, maximum, median, and standard deviation where applicable
* The 50MB all-tools comparison and zero-file benchmark use 5 runs because of xxd/hexdump runtime
* All other benchmarks use 50 measured runs
* **Important:** File I/O benchmarks measure Linux page-cache (RAM) performance, not physical disk I/O. Test files are read from memory after initial caching.

## Uninstall

```bash
cargo uninstall fasthex
```

## Usage

### Basic usage

```bash
# Display a file
fasthex /path/to/file

# Pipe it (faster with zero-copy output)
fasthex /path/to/file > output.txt

# Skip first 1 KiB and read only 512 bytes
fasthex -s 1KiB -n 512 /path/to/file

# Display with colors
fasthex --color=always /path/to/file

# Binary display
fasthex -b /path/to/file
```

### Common use cases

```bash
# Test on RAM disk (avoids SSD/HDD bottleneck)
sudo mkdir -p /mnt/ramdisk
sudo mount -t tmpfs -o size=2G tmpfs /mnt/ramdisk
cp /path/to/file /mnt/ramdisk
time fasthex /mnt/ramdisk/file > /dev/null

# Measure time with `time`
time fasthex /path/to/file

# Interactive viewing
fasthex /path/to/file | less
```

## Features

### Supported Output Formats

- Octal (`-o`)
- Decimal (`-d`)
- Binary (`-b`)
- Colored output (`--color=always`)
- ... and more!

### Display Options

- ASCII representation (default, disable with `-A`)
- Line squeezing for identical lines (`-w`)
- Arbitrary skip/length with size suffixes (`KiB, MiB, GiB, TiB, PiB, EiB, ZiB, YiB`)

## Example Output

```
❯ fasthex3 /bin/ls | head
00000000: 7f 45 4c 46 02 01 01 00  00 00 00 00 00 00 00 00  |.ELF............|
00000010: 03 00 3e 00 01 00 00 00  f0 54 00 00 00 00 00 00  |..>......T......|
00000020: 40 00 00 00 00 00 00 00  a8 73 02 00 00 00 00 00  |@........s......|
00000030: 00 00 00 00 40 00 38 00  0e 00 40 00 1c 00 1b 00  |....@.8...@.....|
00000040: 06 00 00 00 04 00 00 00  40 00 00 00 00 00 00 00  |........@.......|
00000050: 40 00 00 00 00 00 00 00  40 00 00 00 00 00 00 00  |@.......@.......|
00000060: 10 03 00 00 00 00 00 00  10 03 00 00 00 00 00 00  |................|
00000070: 08 00 00 00 00 00 00 00  03 00 00 00 04 00 00 00  |................|
00000080: 74 03 00 00 00 00 00 00  74 03 00 00 00 00 00 00  |t.......t.......|
00000090: 74 03 00 00 00 00 00 00  1c 00 00 00 00 00 00 00  |t...............|
```

## How It Works:
  1. mmap path: output formatted in parallel with rayon in 64 MiB chunks.
  2. AVX2 path: processes 32 bytes (2 rows) per SIMD call; falls back to
     SSE4.1/SSSE3 (16 bytes / 1 row) or scalar.
  3. Double-buffered I/O: a dedicated writer thread drains completed chunks
     while rayon formats the next one.
  4. MADV_SEQUENTIAL on open + MADV_WILLNEED two chunks ahead to hide
     mmap page-fault latency.
  5. Zero-copy output via an internal pipe pair:
       formatted buffer
         → vmsplice  (userspace pages → kernel pipe, zero-copy)
         → splice    (kernel pipe → stdout fd, zero-copy)
     This path works regardless of what stdout is (/dev/null, file, pipe,
     socket) because we own the intermediate pipe. Falls back to write_all
     if splice rejects the stdout fd (e.g. a tty).
  6. Streaming (stdin) path uses a 4 MiB write buffer.


### Full Help

```
fasthex 0.3.0 - a very fast hex dumper

Usage:
  fasthex [options] [file]...
  fasthex -r [options] [file] [-j <offset>]
  fasthex [options] -          read from stdin explicitly

Multiple files are concatenated and treated as one stream.
If no file is given, reads from stdin.

OUTPUT FORMAT
  Rule: lowercase = one-byte mode, UPPERCASE = two-byte mode.

      (default)               canonical hex + ASCII display
  -x, --hex                   one-byte hexadecimal display
  -X, --hex-wide              two-byte hexadecimal display
  -o, --octal                 one-byte octal display
  -O, --octal-wide            two-byte octal display
  -d, --decimal               one-byte decimal display
  -D, --decimal-wide          two-byte decimal display
  -c, --chars                 one-byte character display
  -b, --binary                binary display (8 bits per byte)
  -p, --plain                 plain hex string, no offset or ASCII
  -i, --include               C include file style output
  -r, --reverse               convert hex dump back to binary

LAYOUT
  -W, --width <N>             bytes per row (default: 16)
  -g, --group <N>             bytes per group: 1, 2, 4, 8
  -E, --endian <MODE>         big | little  (default: big)
  -B, --border <STYLE>        none | ascii | unicode  (default: none)
  -A, --no-ascii              hide the ASCII panel
  -P, --no-position           hide the offset/position column

OFFSET & NAVIGATION
  -s, --skip <N>              skip first N bytes (negative = from end)
  -n, --length <N>            read only N bytes
  -j, --jump <N>              bias added to every displayed offset
  -u, --uppercase             uppercase hex digits (A-F)
      --offset-dec            show offsets in decimal

COLOR
  -L, --color <WHEN>          auto | always | never  (default: auto)
  -S, --scheme <NAME>         default | type | gradient
  -T, --table <MODE>          ascii | default | braille | cp437 | ebcdic

FILTERING & FLOW
  -w, --squeeze               replace identical rows with '*'
  -m, --max-lines <N>         stop after N output lines
  -q, --quiet                 suppress warnings

CUSTOM FORMAT
  -F, --format <FMT>          hexdump -e style format string
  -f, --format-file <FILE>    read format strings from file

MISC
  -h, --help                  show this help
  -v, --version               show version

SIZE SUFFIXES: KiB/K/MiB/M/GiB/G/TiB/T/PiB/P/EiB/E  kB/MB/GB/TB/PB/EB  0x…
```


## Testing Conditions

https://gist.github.com/CallMeAlphabet/4b7022c4b1a8849e6943526de6a23582
