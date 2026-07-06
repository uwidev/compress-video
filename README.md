# archive-video

A media compression script optimized for animated content, using hardware-accelerated HEVC encoding with smart fallback and dynamic load balancing.

## Features

- Hardware encoding with Intel QSV and NVIDIA NVENC
- Smart encoder fallback (NVIDIA -> Intel -> Software)
- Perceptual transparency at quality 15 (default)
- Lossless mode for archival-quality encoding
- Audio handling: copies lossless audio, re-encodes lossy audio to Opus
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

## Usage

### Basic Examples

Process all media in a directory:

```
archive-videos input_folder
```

Process specific files:

```
archive-videos video1.mov video2.gif animation.mp4
```

Process only with Intel GPU:

```
archive-videos input_folder --no-nvidia
```

Process only with NVIDIA GPU:

```
archive-videos input_folder --no-intel
```

Use CPU-only (software encoding):

```
archive-videos input_folder --no-intel --no-nvidia
```

### Quality Control

Adjust quality level (1 = highest quality, 51 = smallest files, default = 15):

```
archive-videos input_folder --quality 20
```

Force lossless encoding:

```
archive-videos input_folder --strategy lossless
```

### Resume Options

Skip the confirmation prompt:

```
archive-videos input_folder -y
```

Fast resume (skip duration verification, only check if output file exists):

```
archive-videos input_folder --fast-resume
```

Force re-encode all files (disable resume):

```
archive-videos input_folder --force
```

### Worker Tuning

Adjust parallel encoding threads:

```
archive-videos input_folder --nvidia-workers 6 --intel-workers 4
```

### Dry Run

Preview what would be encoded without actually doing it:

```
archive-videos input_folder --dry-run
```

## Example Output

```
Source: input_folder
Output: input_folder-compressed

GPU Detection:
  Intel HEVC (QSV): ✓ Available
  NVIDIA HEVC (NVENC): ✓ Available

✅ Selected encoders: NVIDIA HEVC, INTEL HEVC

⚙️ Settings:
  Strategy: auto-detect
  Quality: 15 (1=best, 15=transparent, 30=smaller)
  Audio: auto
  Force re-encode: NO
  Workers: NVIDIA=4, Intel=6, CPU=1

Scanning directory...
  Done scanning 10886 files

Pre-scanning for resume...
  Probing 10550 files with parallel workers...
  Scanned 500/10550 files... (489 found complete)
  Scanned 1000/10550 files... (998 found complete)
  Scanned 2000/10550 files... (1995 found complete)

  Pre-scan complete:
  • 10 files need encoding (0.41 GB)
  • 10540 files already complete (will be skipped)

  Trying NVIDIA NVENC...
  [NVIDIA HEVC] ✓ file1.mov -> 8.45MB -> 1.23MB (85.4% reduction)
  [INTEL HEVC] ✓ file2.gif -> 15.67MB -> 2.34MB (85.1% reduction)
  [INTEL HEVC] ✓ file3.mov -> 22.10MB -> 3.45MB (84.4% reduction)

==================================================
SUMMARY
==================================================
Converted: 10550
Skipped (already complete): 0
Skipped GIFs (video exists): 236
Non-media copied: 68

==================================================
COMPRESSION STATISTICS
==================================================
Total input size:  45.23 GB
Total output size: 12.89 GB
Space saved:       32.34 GB (71.5% reduction)
Files encoded:     10550 files

Total time: 2490.09s (41.5 minutes)
```

## How It Works

### Encoder Selection

The script tries encoders in this order:

1. NVIDIA NVENC (if available and not disabled)
2. Intel QSV (if available and not disabled)
3. Software HEVC (fallback, if needed)

If an encoder produces a file larger than the source, it is discarded and the next encoder is tried. If all encoders fail to produce a smaller file, the original file is copied as-is.

### Quality Levels

- Quality 15: Perceptually transparent (visually lossless)
- Quality 1-10: Near-lossless (larger files, higher quality)
- Quality 20-30: Smaller files, slightly lower quality
- Quality 30-51: Aggressive compression, visible artifacts possible

### Audio Handling

- Lossy audio (AAC, MP3) -> re-encoded to Opus (128 kbps)
- Lossless audio (FLAC, ALAC, PCM) -> copied without re-encoding
- Audio is preserved when copying original files

### Resume Behavior

- Normal resume: verifies output duration matches input
- Fast resume (`--fast-resume`): only checks if output file exists
- `--force`: re-encodes all files, ignoring any existing output

## Performance Notes

The worker counts determine how many parallel encoding jobs run on each GPU. Optimal values depend on your hardware:

- NVIDIA RTX 5070 Ti: 4-6 workers
- Intel Core Ultra 265K QSV: 6-8 workers
- CPU: 1-2 workers

## Limitations

- Software encoding is significantly slower than hardware encoding
- Lossless software HEVC produces very large files
- Some codecs may not support all input formats (e.g., HEVC may fail on odd resolutions)
- Hardware encoders may have specific resolution limitations

## AI Disclosure

This tool was developed with assistance from artificial intelligence. While every effort has been made to ensure correctness and reliability, the code is provided "as is" without warranty of any kind. Use at your own risk. Always verify the output quality and file integrity before deleting original files, especially for archival purposes.
