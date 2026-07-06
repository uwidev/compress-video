# archive-videos

A media compression script optimized for animated content, using hardware-accelerated HEVC encoding with smart fallback and dynamic load balancing.

## Features

- Hardware encoding with Intel QSV and NVIDIA NVENC
- Smart encoder fallback (NVIDIA -> Intel -> Software)
- Perceptual transparency at quality 15 (default)
- Lossless mode for archival-quality encoding
- Smart audio handling (copies lossless audio, re-encodes lossy audio to Opus)
- Resume support with fast mode (skip duration verification)
- Directory or individual file processing
- Dynamic load balancing across multiple GPUs
- ETA and progress reporting
- Compression statistics summary

## Requirements

- ffmpeg with:
  - `hevc_qsv` (Intel QuickSync)
  - `hevc_nvenc` (NVIDIA NVENC)
  - `libx265` (software fallback)
- Python 3.11 or higher

## Installation

Make the script executable and place it in your PATH:

```
chmod +x archive-videos
sudo cp archive-videos /usr/local/bin/
```

## Core Usage

Process all media in a directory:

```
archive-videos input_folder
```

Process specific files:

```
archive-videos video1.mov video2.gif animation.mp4
```

Adjust quality (1 = highest, 51 = smallest, default 15):

```
archive-videos input_folder --quality 20
```

Disable a GPU:

```
archive-videos input_folder --no-nvidia
```

Skip confirmation prompt:

```
archive-videos input_folder -y
```

Preview without encoding:

```
archive-videos input_folder --dry-run
```

Full help:

```
archive-videos --help
```

## Command-Line Options

```
usage: archive-videos [-h] [-y] [--fast-resume] [-o OUTPUT_DIR] [--nvidia-workers N] [--intel-workers N] [--cpu-workers N] [--strategy {auto,lossless,transparent}] [--audio {copy,flac,aac,opus}] [--quality {1..51}] [--force] [--nvidia] [--no-nvidia] [--intel] [--no-intel] [--dry-run] inputs [inputs ...]

Animation-optimized media compression with HEVC

positional arguments:
  inputs                One or more files or directories (if one directory, process all media inside)

optional arguments:
  -h, --help            show this help message and exit
  -y, --yes             Skip confirmation prompt
  --fast-resume         Skip duration verification; skip if output file exists (faster resume)
  -o OUTPUT_DIR, --output-dir OUTPUT_DIR
                        Output directory (only for directory mode)
  --nvidia-workers N    Number of parallel NVIDIA encoding threads (default: 4)
  --intel-workers N     Number of parallel Intel encoding threads (default: 6)
  --cpu-workers N       Number of parallel CPU encoding threads (default: 1)
  --strategy {auto,lossless,transparent}
                        Encoding strategy (default: auto)
  --audio {copy,flac,aac,opus}
                        Audio handling (default: auto – copy if lossless, opus if lossy)
  --quality {1..51}     Quality level 1-51 (default: 15, lower=better quality)
  --force               Force re-encode even if output exists
  --nvidia              Enable NVIDIA GPU (default)
  --no-nvidia           Disable NVIDIA GPU
  --intel               Enable Intel GPU (default)
  --no-intel            Disable Intel GPU
  --dry-run             Preview only

Examples:
  archive-videos Wa17                              # Process all media in directory
  archive-videos file1.mov file2.gif               # Process specific files
  archive-videos Wa17 --no-nvidia                  # Intel only
  archive-videos Wa17 --no-intel                   # NVIDIA only
  archive-videos --no-intel --no-nvidia file.mov   # CPU only
  archive-videos Wa17 --fast-resume                # Skip duration check
  archive-videos Wa17 -y                           # Skip confirmation
```

## AI Disclosure

This tool was developed with assistance from artificial intelligence. While every effort has been made to ensure correctness and reliability, the code is provided "as is" without warranty of any kind. Use at your own risk. Always verify the output quality and file integrity before deleting original files, especially for archival purposes.
