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
yt-dlp --sponsorblock-remove default --sub-langs 'en' --write-auto-sub --embed-metadata --embed-subs --force-keyframes-at-cuts -f 'bv[height<=720][fps<=30]+ba/b[height<=720][fps<=30]' --merge-output-format mp4 -o '%(title)s.%(ext)s' -a urls.txt
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
yt-dlp --embed-metadata -f 'ba[ext=m4a]' -o '%(title)s.%(ext)s' -a urls.txt
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

---

## Termux yt-dlp Shortcut
The fastest way to rip videos on mobile is to share a url from youtube(web browser, clipious, libretube, youtube app) to a termux-url-opener yt-dlp shortcut script. It will place the video in the `Movies` folder for you to watch with VLC later. Once the video is done it will vibrate your device and push a toast/text to speech notification that your video is done as a ease of life. All you have to do is share the URL and tap Termux once this is setup.

1. Install [F-droid](https://f-droid.org/F-Droid.apk)
2. Install [Termux](https://f-droid.org/en/packages/com.termux/) and Termux [API](https://f-droid.org/en/packages/com.termux.api/).
3. Open Termux and setup storage/install script dependancies by copy/pasting the contents below into termux and pressing enter.
```
termux-setup-storage
pkg update && pkg upgrade
pkg install libexpat openssl python termux-api ffmpeg
pip install -U "yt-dlp[default]"
mkdir bin
cd bin
wget https://raw.githubusercontent.com/FalsePhilosopher/YT-DLP_CS/main/termux-url-opener
```
That's it, you can now share a video URL from your youtube app, web browser, clipious, or libretube and select termux for it to auto rip the video.

**Manual Install**  
3. Follow up to step 2 then open Termux to setup storage [(wiki)](https://wiki.termux.com/wiki/Termux-setup-storage), install yt-dlp/script dependancies [(wiki)](https://github.com/yt-dlp/yt-dlp/wiki/Installation#android) by copy/pasting the contents below into termux and pressing enter. **Termux does not use shebangs, more info [here](https://wiki.termux.com/wiki/Differences_from_Linux)**  
```
termux-setup-storage
pkg update && pkg upgrade
pkg install libexpat openssl python termux-api nano ffmpeg
pip install -U "yt-dlp[default]"
```
4. create the bin dir/termux-url-opener and open termux-url-opener for editing
```
mkdir bin && cd bin && touch termux-url-opener && nano termux-url-opener
```
5. Copy+paste the contents below and exit (`CTRL+x, Press Y, ENTER` to save and exit.)  
```
url=$1
cd /sdcard/Movies/
&& yt-dlp $1 -f 'bv[height<=720][fps<=30]+ba/b[height<=720][fps<=30]' --merge-output-format mp4 -o %(title)s.%(ext)s --exec after_move:'termux-vibrate & termux-tts-speak "video is done" & termux-toast -g top "Video is Done!"
```

---

**HWaccel Version for Qualcomm SoC's** *Make sure you follow the optional step for this to work.*
```
url=$1
cd /sdcard/Movies/
&& yt-dlp $1 -f 'bv[height<=720][fps<=30]+ba/b[height<=720][fps<=30]' --merge-output-format mp4 --sponsorblock-remove default --force-keyframes-at-cuts --ppa "ModifyChapters+ffmpeg_i:-hwaccel opencl -hwaccel_output_format opencl" --ppa "ModifyChapters+ffmpeg_o:-c:v hevc_mediacodec -level 6.2" -o %(title)s.%(ext)s --exec after_move:'termux-vibrate & termux-tts-speak "video is done" & termux-toast -g top "Video is Done!"
```
**HWaccel/Aria2c Version** *Make sure you follow the optional step for this to work.*
```
url=$1
cd /sdcard/Movies/
&& yt-dlp url=$1 -f 'bv[height<=720][fps<=30]+ba/b[height<=720][fps<=30]' --merge-output-format mp4 --downloader aria2c --sponsorblock-remove default --force-keyframes-at-cuts --ppa "ModifyChapters+ffmpeg_i:-hwaccel opencl -hwaccel_output_format opencl" --ppa "ModifyChapters+ffmpeg_o:-c:v hevc_mediacodec -level 6.2" -o %(title)s.%(ext)s --exec after_move:'termux-vibrate & termux-tts-speak "video is done" & termux-toast -g top "Video is Done!"
```

---

You can now share a video URL from your web browser, clipious, libretube, or youtube app and select termux for it to auto rip the video.  
## Optional Steps
6. Install OpenCL to enable HWaccel if using the `--sponsorblock-remove` option on Qualcomm SoC's.
```
pkg install opencl-vendor-driver opencl-headers ocl-icd clinfo
```
7. Run `clinfo` to confirm OpenCL works.
8. Install aria2c for faster downloads `pkg install aria2c`
9. For more expanded termux-url-opener options follow [LordH3lmchen's gist](https://gist.github.com/LordH3lmchen/dc35e8df3dc41d126683f18fe44ebe17) or [qwerty12's gist](https://gist.github.com/qwerty12/e236f9c05bfdccffca35338383bdc02e)

---

## Notes
Variables like `-o '%(title)s.%(ext)s'` can be removed from the command by creating `/etc/yt-dlp.conf` with that variable added to it.  
Too long of file names `-o "%(title).200s.%(ext)s"` for 200 max char, or `--trim-filenames 200`.  
Creator subtitles are picked over auto generated subtitles by default when using `--write-auto-sub`  
To ensure you get all english subtitles you can use `--sub-langs 'en.*'` to grab both the creator/auto subs.  
You will get faster download speeds using aria2c instead of the default `--downloader aria2c`  
yt-dlp doesn't need the full youtube URL, if it's `https://www.youtube.com/watch?v=EXAMPLE` you can pull it with `yt-dlp EXAMPLE`  
Proof of origin token info can be found [here](https://github.com/yt-dlp/yt-dlp/pull/10648) and [here](https://github.com/yt-dlp/yt-dlp-wiki/pull/40/files).  
Changing your TTL to 65 might help with ripping on a mobile data connection ;) [Linux info here](https://askubuntu.com/questions/667096/how-to-change-the-default-ttl-of-tcp-ip-packets) [Windows info here](https://gist.github.com/farneser/3acdc050b59235e95c96802747b02113#file-windows-change-ttl-md)

## RTFM  
https://man.archlinux.org/man/extra/yt-dlp/yt-dlp.1.en  
https://github.com/yt-dlp/yt-dlp/wiki  
https://trac.ffmpeg.org/wiki/HWAccelIntro  
### Some useful commands  
https://gist.github.com/DiegoFleitas/c940d4b94d6b92b55a7084afe84bf571
