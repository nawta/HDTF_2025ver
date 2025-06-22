

# HDTF
Flow-guided One-shot Talking Face Generation with a High-resolution Audio-visual Dataset 
<a href="https://openaccess.thecvf.com/content/CVPR2021/papers/Zhang_Flow-Guided_One-Shot_Talking_Face_Generation_With_a_High-Resolution_Audio-Visual_Dataset_CVPR_2021_paper.pdf" target="_blank">paper</a>    <a href="https://github.com/MRzzm/HDTF/blob/main/Supplementary%20Materials.pdf" target="_blank">supplementary</a>   [demo video](https://www.youtube.com/watch?v=uJdBgWYBTww)

## Overview

The HDTF (High-resolution Talking Face) dataset provides high-quality talking face videos for research in audio-visual speech processing, lip synchronization, and talking face generation. This repository contains an improved version of the dataset downloader that automates the process of downloading, cutting, and cropping videos from YouTube.

## Dataset Structure

The HDTF dataset is organized into three subsets:
- **RD (Radio)**: Radio/podcast style videos
- **WDA**: Videos featuring various speakers (including political figures)
- **WRA**: Additional speaker videos

## Details of HDTF dataset
**./HDTF_dataset** consists of *youtube video url*, *video resolution* (in our method, may not be the best resolution), *time stamps of talking face*, *facial region* (in the our method) and *the zoom scale* of the cropped window.

### Metadata Files

**xx_video_url.txt:**
```
format:     video name | video youtube url
```
**xx_resolution.txt:**
```
format:    video name | resolution(in our method)
```

**xx_annotion_time.txt:**
```
format:    video name | time stamps of clip1 | time stamps of clip2 | time stamps of clip3....
```
**xx_crop_wh.txt:**
```
format:    video name+clip index | min_width | width |  min_height | height (in our method)
```
**xx_crop_ratio.txt:**
```
format:    video name+clip index | window zoom scale
```

## Processing of HDTF dataset
When using HDTF dataset,

 - We provide video and url in  **xx_video_url.txt**. (the highest definition of videos are 1080P or 720P).  Transform video into **.mp4** format and transform interlaced video to progressive video as well.

 - We split long original video into talking head clips with time stamps in **xx_annotion_time.txt**.  Name the splitted clip as **video name_clip index.mp4**. For example, split the video  *Radio11.mp4 00:30-01:00 01:30-02:30*  into *Radio11_0.mp4* and *Radio11_1.mp4* .

 - Our work does not always download videos with the best resolution, so we provide two cropping methods. Thanks @universome and @Feii Yin for pointing out this problem!

	1. Download the video with reference resulotion in **xx_resolution.txt** and crop the facial region with fixed window size in **xx_crop_wh.txt**. (This method is as same as ours, but the downloaded video may not be the best resolution).
	2. First, download the video with best resulotion. Then, detect the facial landmark in the splitted talking head clips and count the square window of the face, specifically, count the facial region in each frame and merge all regions into one square range. Next,  enlarge the window size with **xx_crop_ratio.txt**. Finally, crop the facial region.

- We resize all cropped videos into **512 x 512** resolution.


The HDTF dataset is available to download under a <a href="https://creativecommons.org/licenses/by/4.0/" target="_blank"> Creative Commons Attribution 4.0 International License</a>. **Thanks @universome for provding the the script of data processing, pls visit [here](https://github.com/universome/HDTF) for more details.** If you face any problems when processing HDTF, pls contact me.

## Inference code
#### code of audio-to-animation
coming soon......

#### code of constructing approximate dense flow
The code is in **./code_constructing_Fapp**, pls visit [here](https://github.com/MRzzm/HDTF/tree/main/code_constructing_Fapp) for more details. 

#### code of animation-to-video module
The code is in **./code_animation2video**, pls visit [here](https://github.com/MRzzm/HDTF/tree/main/code_animation2video) for more details. 

#### code of reproducing other works
coming soon......

## Installation

```bash
# Install required dependencies
pip install tqdm yt-dlp

# Ensure ffmpeg is installed
# Ubuntu/Debian:
sudo apt-get install ffmpeg
# macOS:
brew install ffmpeg
```

## Downloading
For convenience, we provide the `download.py` script which downloads, crops and resizes the dataset. You can use it via the following command:
```
python download.py --output_dir /path/to/output/dir --num_workers 8
```

### Command Line Arguments
- `--source_dir` or `-s`: Path to metadata directory (default: `HDTF_dataset`)
- `--output_dir` or `-o`: Where to save processed videos (required)
- `--num_workers` or `-w`: Number of parallel download workers (default: 8)

Note: some videos might become unavailable if the authors will remove them or make them private.

## Output Structure

After running `download.py`, the output directory will contain:

```
output_dir/
├── _videos_raw/                              # Temporary directory (can be deleted after processing)
│   ├── {subset}_{videoname}.mp4             # Raw downloaded videos
│   └── {subset}_{videoname}_download_log.txt # Download logs
├── {subset}_{videoname}_000.mp4             # Processed clip 1
├── {subset}_{videoname}_001.mp4             # Processed clip 2
└── ...                                       # More clips
```

### Output Characteristics
- **Video only**: No audio track included
- **Square format**: All clips are square (width = height)
- **High resolution**: Maintains original resolution (720p or 1080p)
- **Face-centered**: Cropped to center on the speaker's face
- **Systematic naming**: `{subset}_{videoname}_{clipindex:03d}.mp4`

## Processing Pipeline

The `download.py` script performs the following steps:

1. **Download**: Fetches videos from YouTube at specified resolution using yt-dlp
2. **Cut**: Extracts time intervals containing talking faces based on annotation files
3. **Crop**: Applies facial region cropping to create square clips using FFmpeg
4. **Save**: Outputs individual clips with systematic naming

## Improvements Over Original Code

### 1. **Migration to yt-dlp**
- **Original**: Used deprecated `youtube-dl` library
- **Improved**: Updated to actively maintained `yt-dlp` with better reliability
- **Benefits**: 
  - Better handling of YouTube API changes
  - Improved download success rates
  - More robust error handling

### 2. **Enhanced Download Robustness**
Added several flags to improve download reliability:
```python
"--retries", "3",              # Retry failed downloads
"--fragment-retries", "3",     # Retry failed fragments
"--no-part",                   # Avoid partial file issues
"--no-check-certificate",      # Handle SSL certificate issues
"--merge-output-format", mp4   # Ensure consistent output format
```

### 3. **Improved Format Selection**
- **Original**: Basic format string that could fail
- **Improved**: Sophisticated format selection with fallbacks
```python
# Example for 720p:
"bestvideo[height=720][ext=mp4]+bestaudio[ext=m4a]/best[height=720][ext=mp4]"
```

### 4. **Better Error Handling**
- Added graceful handling of missing metadata files
- Skip subsets if files are not found instead of crashing
- More informative error messages for debugging
- Individual download logs for each video

### 5. **Code Quality Improvements**
- Added comprehensive documentation and docstrings
- Better variable naming for clarity
- Type hints in function signatures
- Improved code organization and readability

### 6. **Robustness Features**
- File existence checks before processing
- Validation of downloaded video resolution
- Proper handling of videos with multiple clips
- Clear progress indication with tqdm

## Troubleshooting

### Common Issues

1. **Download Failures**
   - Check internet connection
   - Verify YouTube URLs are still valid
   - Review individual download logs in `_videos_raw/`
   - Some videos may have been removed or made private

2. **Resolution Mismatches**
   - The requested resolution may not be available
   - Script will skip videos if downloaded resolution doesn't match metadata
   - Consider updating metadata files if needed

3. **Missing Videos**
   - Videos without proper cropping information are skipped
   - Check console output for specific reasons
   - Some videos may be discarded due to quality issues

### Manual Recovery

For failed downloads:
1. Check the download log: `_videos_raw/{video_name}_download_log.txt`
2. Try downloading manually with yt-dlp
3. Update metadata files if video formats have changed

## Storage Notes

- The `_videos_raw/` directory contains full-length downloaded videos
- This directory can be safely deleted after processing to save space
- Final processed clips are much smaller than raw videos

## Reference
if you use HDTF, pls reference

```
@inproceedings{zhang2021flow,
  title={Flow-Guided One-Shot Talking Face Generation With a High-Resolution Audio-Visual Dataset},
  author={Zhang, Zhimeng and Li, Lincheng and Ding, Yu and Fan, Changjie},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  pages={3661--3670},
  year={2021}
}
```
