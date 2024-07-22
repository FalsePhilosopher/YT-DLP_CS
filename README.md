# YT-DLG Cheat Sheet

Removes sponsored segments, grabs the highest quality 720p video/audio streams and merges them into a .mp4 file and grabs all the urls from a list called `urls.txt`.  
```yt-dlp --sponsorblock-remove default -f 'bestvideo[height<=720][fps=30]+bestaudio/best[height<=720][fps=30]' --merge-output-format mp4 -a urls.txt```
