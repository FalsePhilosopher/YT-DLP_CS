# YT-DLG Cheat Sheet

## Video
Grabs all the urls from a list called `urls.txt`, removes sponsored segments, grabs the highest quality 720p and 30(or 24FPS)FPS video with it's metadata and embeds the subtitles. 
```
yt-dlp --sponsorblock-remove all --embed-metadata --embed-subs --sub-langs 'en-orig' --force-keyframes-at-cuts -f 'bv[height<=720][fps<=30][ext=mp4]' -o '%(title)s.%(ext)s' -a urls.txt
```
### Seperate Video+Audio streams merged
For videos with multiple language audio tracks to pick from.
```
yt-dlp --sponsorblock-remove all --embed-metadata --embed-subs --sub-langs 'en-orig' --force-keyframes-at-cuts -f 'bv[height<=720][fps<=30]+ba/b[height<=720][fps<=30]' --merge-output-format mp4 -o %(title)s.%(ext)s -a urls.txt
```
## Audio
Downloads highest quality m4a audio track
```
yt-dlp --embed-metadata -f 'ba[ext=m4a]' -o %(title)s.%(ext)s -a urls.txt
```
### Rip all songs in a mix individually  
```
yt-dlp --embed-metadata --split-chapters -f 'ba[ext=m4a]' --output '%(title)s.%(ext)s' -a urls.txt
```
## Grab latest release
If `yt-dlp -U` for updating doesn't work or installing fresh.
```
cd /tmp && wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp_linux && mv yt-dlp_linux yt-dlp && chmod +x yt-dlp && sudo cp yt-dlp /bin && rm yt-dlp
```
## Notes
Variables like `-o '%(title)s.%(ext)s'` can be removed from the command by creating `/etc/yt-dlp.conf` with that variable added to it.  
Too long of file names `-o "%(title).200s.%(ext)s"` for 200 max char, or `--trim-filenames 200`.

## RTFM  
https://man.archlinux.org/man/extra/yt-dlp/yt-dlp.1.en  
https://github.com/yt-dlp/yt-dlp/wiki

### Some useful commands  
https://gist.github.com/DiegoFleitas/c940d4b94d6b92b55a7084afe84bf571
