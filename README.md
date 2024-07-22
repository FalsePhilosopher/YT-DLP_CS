# YT-DLG Cheat Sheet

## Video+Audio
Removes sponsored segments, grabs the highest quality 720p and 30(or 24FPS)FPS video/audio streams with it's metadata and merges it into a .mp4 file and grabs all the urls from a list called `urls.txt`.  
```
yt-dlp --sponsorblock-remove all --embed-metadata -f 'bv[height<=720][fps<=30]+ba/b[height<=720][fps<=30]' --merge-output-format mp4 -o %(title)s.%(ext)s -a urls.txt
```
## Audio
Downloads highest quality m4a audio track
```
yt-dlp --embed-metadata -f 'ba[ext=m4a]' -o %(title)s.%(ext)s -a urls.txt
```
Rip all songs in a mix individually  
Note: The -o system variable isn't respected when splitting chapters and the `--output` is necessary. 
```
yt-dlp --embed-metadata --split-chapters -f 'ba[ext=m4a]' --output '%(title)s.%(ext)s' -a urls.txt
```
## Grab latest release
If `yt-dlp -U` for updating doesn't work or installing fresh.
```
cd /tmp && wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp_linux && mv yt-dlp_linux yt-dlp && chmod +x yt-dlp && sudo cp yt-dlp /bin && rm yt-dlp
```
## Notes
`-o %(title)s.%(ext)s` can be removed from the variables by creating `/etc/yt-dlp.conf` with that config added to it.  
If you want subtitles add `--embed-subs`  
Too long of file names `-o "%(title).200s.%(ext)s"` for 200 max char.

## RTFM  
https://man.archlinux.org/man/extra/yt-dlp/yt-dlp.1.en  
https://github.com/yt-dlp/yt-dlp/wiki

### Some useful commands  
https://gist.github.com/DiegoFleitas/c940d4b94d6b92b55a7084afe84bf571
