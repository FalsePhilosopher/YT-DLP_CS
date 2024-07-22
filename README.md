# YT-DLG Cheat Sheet

### Video+Audio
Removes sponsored segments, grabs the highest quality 720p 30FPS video/audio streams and merges them into a .mp4 file and grabs all the urls from a list called `urls.txt`.  
```
yt-dlp --sponsorblock-remove default -f 'bestvideo[height<=720][fps<=30]+bestaudio/best[height<=720][fps<=30]' --merge-output-format mp4 -o %(title)s.%(ext)s -a urls.txt
```
### Audio
Downloads highest quality m4a audio track
```
yt-dlp --sponsorblock-remove default -f 'ba[ext=m4a]' -o %(title)s.%(ext)s -a urls.txt
```
## Notes
`-o %(title)s.%(ext)s` can be removed from the variables by creating `/etc/yt-dlp.conf` with that config added to it.

# Grab latest release
If `yt-dlp -U` for updating doesn't work or installing fresh.
```
cd /tmp && wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp_linux && mv yt-dlp_linux yt-dlp && chmod +x yt-dlp && sudo cp yt-dlp /bin && rm yt-dlp
```
