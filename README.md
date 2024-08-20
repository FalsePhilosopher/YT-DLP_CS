# YT-DLP Cheat Sheet

## Video Only
Grabs all the URLs from a list called `URLs.txt`, removes sponsored segments, grabs the highest quality 720p/30FPS or below video with it's metadata and embeds the subtitles.  
**Note** If using the `--sponsorblock-remove` arg you will need to use the `--force-keyframes-at-cuts` arg or you will get artifacts at the cuts if you don't use that option. By default ffmpeg will use the CPU, to enable HWaccel encoding for ffmpeg during the modify chapters part of the process if using the `--sponsorblock-remove` option [RTFM](https://trac.ffmpeg.org/wiki/HWAccelIntro) on what is supported for your hardware/installing configs, then run `ffmpeg -encoders >> ffmpeg-encode.txt` and `ffmpeg -hwaccels >> ffmpeg-encode.txt` to see what HWaccels/encoders are available on your system. Then add `--ppa "ModifyChapters+ffmpeg_i:-hwaccel vaapi -hwaccel_output_format vaapi" --ppa "ModifyChapters+ffmpeg_o:-c:v hevc_vaapi"` to your args. Change `vaapi` to your supported HWaccel/encoder to the command or conf file. Embedding metadata and subtitles are separate encode calls, so be sure to change the `sample+ffmpeg_i/o:` values for hwaccel for those steps as well. On a mobile device with a TPU it's 3X more efficient to use opencl as your HWaccel and mediacodec/GPU for your encode. Mediacodec does not support lossless encoding, to see what profiles are supported for your supported encoders are run `ffmpeg -h encoder=hevc_vaapi >> ffmpeg-encode.txt`.  
```
yt-dlp --sponsorblock-remove default --sub-langs 'en' --write-auto-sub --embed-metadata --embed-subs --force-keyframes-at-cuts -f 'mp4[height<=720][fps<=30]' -o '%(title)s.%(ext)s' -a urls.txt
```
### Video+Audio streams merged
The most commonly run command, the one you probably want.
```
yt-dlp --sponsorblock-remove all --sub-langs 'en' --write-auto-sub --embed-metadata --embed-subs --force-keyframes-at-cuts -f 'bv[height<=720][fps<=30]+ba/b[height<=720][fps<=30]' --merge-output-format mp4 -o %(title)s.%(ext)s -a urls.txt
```
## Stream to Video Player
Streams the video to VLC or MPV

MPV uses yt-dlp under the hood and can be called with
```
mpv "ytdl://URL"
```
Pipeline to MPV
```
yt-dlp -o - URL | mpv -
```
Pipeline to VLC
```
yt-dlp -o - URL | vlc -
```
## Audio
Downloads highest quality m4a audio track
```
yt-dlp --embed-metadata -f 'ba[ext=m4a]' -o %(title)s.%(ext)s -a urls.txt
```
### Rip all songs in a mix individually  
```
yt-dlp --embed-metadata --split-chapters -f 'ba[ext=m4a]' -o '%(title)s.%(ext)s' -a urls.txt
```
## Grab latest release
If `yt-dlp -U` for updating doesn't work or installing fresh.
```
cd /tmp && wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp_linux && mv yt-dlp_linux yt-dlp && chmod +x yt-dlp && sudo cp yt-dlp /bin && rm yt-dlp
```

---

## Notes
Variables like `-o '%(title)s.%(ext)s'` can be removed from the command by creating `/etc/yt-dlp.conf` with that variable added to it.  
Too long of file names `-o "%(title).200s.%(ext)s"` for 200 max char, or `--trim-filenames 200`.  
Creator subtitles are picked over auto generated subtitles by default when using `--write-auto-sub`  
To ensure you get all english subtitles you can use `--sub-langs 'en.*'` to grab both the creator/auto subs.  
You will get faster download speeds using aria2c instead of the default `--downloader aria2c`  
yt-dlp doesn't need the full youtube URL, if it's `https://www.youtube.com/watch?v=EXAMPLE` you can pull it with `yt-dlp EXAMPLE`  

## SCP to NAS after encode
Create a bash script named `scp.sh` containing  
```
password="your password"
username="username"
Ip="IP"
sshpass -p "$password" scp /SAMPLE-PATH/*.mp4 $username@$Ip:/SAMPLE-PATH
```
and add to your command 
```
--exec after_move:'/SAMPLE-PATH/scp.sh && rm %(filepath,_filename|)q'
```
## Termux Widget Shortcut
The fastest way to rip videos on mobile is to create a termux shortcut script that pastes your clipboard into your urls.txt file and executes yt-dlp against your urls.txt file. It will place the video in the `movies/YT` folder for you to watch with VLC later. Once the video is done it will vibrate your device and push a toast notification that your video is done as a ease of life function. All you have to do is copy the URL from your web browser, hit the home function, tap the shortcut and enjoy at your convenience!

1. Install Termux from [F-Droid](https://f-droid.org/en/packages/com.termux/) or [Github](https://github.com/termux/termux-app/releases/latest)(the playstore version is old and cursed, the devs themselves say stay away from it.), then install Termux Widget and Termux API.
2. Add the termux shortcut widget to your homescreen.
3. Setup storage [(wiki)](https://wiki.termux.com/wiki/Termux-setup-storage) and install yt-dlp [(wiki)](https://github.com/yt-dlp/yt-dlp/wiki/Installation#android)
```
termux-setup-storage                 # Allow termux to download files into your phone's storage
pkg update && pkg upgrade            # Update all packages
pkg install libexpat openssl python  # Install python
pip install -U "yt-dlp[default]"     # Install yt-dlp with default dependencies
pkg install ffmpeg                   # OPTIONAL: Install ffmpeg
```
5. In Termux install termux-api `pkg install termux-api`
6. cd to `.shortcuts`
7. Create `ytdl.sh`(`touch ytdl.sh`) containing the contents below.(`pkg install nano && nano ytdl.sh` copy+paste contents below, `CTRL+x, Press Y, ENTER` to save.)  
```
cd /data/data/com.termux/files/home/storage/movies/YT
&& termux-clipboard-get > urls.txt
&& yt-dlp -a urls.txt -o %(title)s.%(ext)s --exec after_move:'termux-vibrate & termux-toast -g top "Video is Done!"
```
**Termux does not use shebangs, more info [here](https://wiki.termux.com/wiki/Differences_from_Linux)**  
8. cd to movies and create the YT dir/urls.txt
```
cd /data/data/com.termux/files/home/storage/movies/ && mkdir YT && cd YT && touch urls.txt
```
9. You can now copy a video URL from your web browser and select `ytdl.sh` in your termux shortcut widget on the homescreen.

---

## RTFM  
https://man.archlinux.org/man/extra/yt-dlp/yt-dlp.1.en  
https://github.com/yt-dlp/yt-dlp/wiki  
https://trac.ffmpeg.org/wiki/HWAccelIntro  
### Some useful commands  
https://gist.github.com/DiegoFleitas/c940d4b94d6b92b55a7084afe84bf571
