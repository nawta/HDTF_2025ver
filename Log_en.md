# HDTF Download Script Update Log

## 2025-06-22

### Migration from youtube-dl to yt-dlp

#### Background
- Multiple errors occurred in the `download.py` script that was using `youtube-dl`
- Due to changes in YouTube's specifications, `youtube-dl` was no longer functioning correctly

#### Modifications

1. **Updated Download Function** (`download_video` function)
   - Changed from `youtube-dl` to `yt-dlp`
   - Improved format selection:
     - Old: `bestvideo[ext=mp4]` (video only)
     - New: `bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]` (video+audio)
   - Additional options:
     - `--no-check-certificate`: Avoid SSL certificate errors
     - `--retries 3`: Retry on download failure
     - `--fragment-retries 3`: Retry on fragment download failure
     - `--no-part`: Do not use .part files
     - `--merge-output-format`: Ensure specified output format

2. **Improved Error Handling** (`construct_download_queue` function)
   - Added functionality to skip when subset files don't exist
   - Improved to avoid processing unnecessary subsets during test runs

3. **Bug Fixes**
   - Fixed logic error in resolution comparison:
     - Old: `if not video_resolution != video_data['resolution']:` (double negative)
     - New: `if video_resolution != int(video_data['resolution']):`
   - Also fixed string vs integer comparison error

4. **Documentation Updates**
   - Updated help text and comments from `youtube-dl` to `yt-dlp`

#### Test Results
- Successfully tested with a single video
- Confirmed successful video download, cropping, and resizing

#### Usage
```bash
# Verify yt-dlp is installed
which yt-dlp

# Download HDTF dataset
python download.py --output_dir /data/nishida/HDTF --num_workers 8
```

#### Notes
- Some videos may not be available for download as they might have been removed from YouTube
- The `_videos_raw` directory can be deleted after processing (to save disk space)
