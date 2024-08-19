# YT-DLP Cheat Sheet

## Video Only
Grabs all the URLs from a list called `urls.txt`, removes sponsored segments, grabs the highest quality 720p/30FPS or below video with it's metadata and embeds the subtitles.  
**Note** If using the `--sponsorblock-remove` arg you will need to use the `--force-keyframes-at-cuts` arg or you will get artifacts at the cuts if you don't use that option. By default ffmpeg will use the CPU, to enable HWaccel encoding for ffmpeg during the modifychapters part of the process if using the `--sponsorblock-remove` option [RTFM](https://trac.ffmpeg.org/wiki/HWAccelIntro) on what is supported for your hardware/installing configs, then run `ffmpeg -encoders >> ffmpeg-encode.txt` and `ffmpeg -hwaccels >> ffmpeg-encode.txt` to see what HWaccels/encoders are available on your system. Then add `--ppa "ModifyChapters+ffmpeg_i:-hwaccel vaapi -hwaccel_output_format vaapi" --ppa "ModifyChapters+ffmpeg_o:-c:v hevc_vaapi -x265-params lossless=1"` to your args. Change `vaapi` to your supported HWaccel/encoder to the command or conf file. Embeding metadata and subtitles are seperate encode calls, so be sure to change the `sample+ffmpeg_i/o:` values for hwaccel for those steps as well.  
```
yt-dlp --sponsorblock-remove default --sub-langs 'en' --write-auto-sub --embed-metadata --embed-subs --force-keyframes-at-cuts -f 'mp4[height<=720][fps<=30]' -o '%(title)s.%(ext)s' -a urls.txt
```
### Video+Audio streams merged
The most commonly run command, the one you probably want.
```
yt-dlp --sponsorblock-remove all --sub-langs 'en' --write-auto-sub --embed-metadata --embed-subs --force-keyframes-at-cuts -f 'bv[height<=720][fps<=30]+ba/b[height<=720][fps<=30]' --merge-output-format mp4 -o %(title)s.%(ext)s -a urls.txt
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

---

## Notes
Variables like `-o '%(title)s.%(ext)s'` can be removed from the command by creating `/etc/yt-dlp.conf` with that variable added to it.  
Too long of file names `-o "%(title).200s.%(ext)s"` for 200 max char, or `--trim-filenames 200`.  
Creator subtitles are picked over auto generated subtitles by default when using `--write-auto-sub`  
To ensure you get all english subtitles you can use `--sub-langs 'en.*'` to grab both the creator/auto subs.  
You will get faster download speeds using aria2c instead of the default `--downloader aria2c`  
On a mobile device with a TPU it's 3X more efficent to use opencl as your HWaccel and mediacodec/GPU for your encode.

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
Create a shortcut script that pastes your clipboard into your urls.txt file and executes yt-dlp against your urls.txt file.

1. Install Termux from [F-Droid](https://f-droid.org/en/packages/com.termux/) first(the playstore version is old and cursed, the devs themselves say stay away from it.), then Termux Widget/API.
2. Add the termux shortcut widget to your homescreen.
3. In Termux install termux-api `pkg install termux-api`
4. cd to `.shortcuts`
5. Create `ytdl.sh` with `touch ytdl.sh` containing the contents `cd SAMPLE-PATH/ && termux-clipboard-get > urls.txt && yt-dlp -a urls.txt` with `echo "content" > ytdl.sh`.
6. You can now copy a video url from your web browser and select `ytdl.sh` in your termux shortcut widget on the homescreen to auto rip that video.

---

## RTFM  
https://man.archlinux.org/man/extra/yt-dlp/yt-dlp.1.en  
https://github.com/yt-dlp/yt-dlp/wiki  
https://trac.ffmpeg.org/wiki/HWAccelIntro  
### Some useful commands  
https://gist.github.com/DiegoFleitas/c940d4b94d6b92b55a7084afe84bf571
