# Building a workshop sound classifier with PRTools.
## Step 1: Get YouTube audios.
To download the audios using youtube-dl, follow these steps:
1. Copy each set of links to its own *batchfile.txt*, in separated directories.
2. Run `youtube-dl -c -f 140 --batch-file batchfile.txt`
3. Run ` scp -r danicardonaibz@192.168.1.100:/home/danicardonaibz/Videos .` to copy the files to the local machine.

<br/>   

### CDM Aluminio
```
https://www.youtube.com/watch?v=osqX7iQEnuI&t=344s
https://www.youtube.com/watch?v=f7MBVAG7QY4
https://www.youtube.com/watch?v=10jtlnwB0mw
https://www.youtube.com/watch?v=txCMvRF4Bm8
https://www.youtube.com/watch?v=3YzAl29Ag78
https://www.youtube.com/watch?v=Rs0ewf4Ag5E
https://www.youtube.com/watch?v=3UsKrDwd37k
https://www.youtube.com/watch?v=fFFn59KnSp8
https://www.youtube.com/watch?v=XrC9X6XnNN0
https://www.youtube.com/watch?v=jT1I0_jgvJ0
https://www.youtube.com/watch?v=zefnrPRvc9Q
https://www.youtube.com/watch?v=n8U71VR7D0g
https://www.youtube.com/watch?v=0sx9eQ6nRWI
https://www.youtube.com/watch?v=EI416WMoeZI
https://www.youtube.com/watch?v=pbF_cRqW1uU
https://www.youtube.com/watch?v=83Jdc2h3GrI
https://www.youtube.com/watch?v=30ljL6RGgiw
https://www.youtube.com/watch?v=02flL5izS0U
https://www.youtube.com/watch?v=5Xo-eZYpqqs
https://www.youtube.com/watch?v=3pwRKQXwKHA
https://www.youtube.com/watch?v=d-nmglPp2O8
https://www.youtube.com/watch?v=MknuEkM0_JM
https://www.youtube.com/watch?v=k-2kr4tGEIQ
```

<br/>

### Sierra aluminio
```
https://www.youtube.com/watch?v=tzVUfKslacY
https://www.youtube.com/watch?v=PYRAjKxbSS8
https://www.youtube.com/watch?v=MPdbEHeUH4E
https://www.youtube.com/watch?v=yWItPwQaCEc
https://www.youtube.com/watch?v=ZC6d3k2OM9U
https://www.youtube.com/watch?v=2TQG6xtnCwc
https://www.youtube.com/watch?v=Nvsgd9mbWZw
https://www.youtube.com/watch?v=tvzcQ4C6r9o
https://www.youtube.com/watch?v=VnU5f7CFLrk
https://www.youtube.com/watch?v=RQgAesZUiQ0
https://www.youtube.com/watch?v=R4rfGaXJb4w
https://www.youtube.com/watch?v=QeOSgMB54HA
https://www.youtube.com/watch?v=VEnI_d58Azw
https://www.youtube.com/watch?v=ICThAEnvLA0
https://www.youtube.com/watch?v=pQsZyt_dFWA
https://www.youtube.com/watch?v=aDs3eWPe_hM
```

<br/>

## Step 2: Getting the most important chunks with Audacity
1. [Download and install Audacity](https://www.audacityteam.org/download/windows/)
2. [Donwload FFMPeg library (Zip version) to open m4a files with Audacity](https://lame.buanzo.org/ffmpeg64audacity.php)
3. Extract the folder and copy its contents to C:\Program Files\Audacity\FFMpeg (create FFMpeg folder)
4. Open Audacity and go to Edit -> Preferences -> Libraries. Click "Locate FFMpeg library" and point it to the upper path.
5. Open each audio file, and mark as labels all chunks that contain sound willing to recognize with Ctrl + B.
6. Export the labels as .wav, with Ctrl + Shift + L.

## Step 3: Normalize audio files
1. Normalize name schema: `ls | cat -n | while read n f; do mv "$f" "environment_$n.wav"; done`
2. Normalize audio channels (i.e. convert them into mono):
```bash
mkdir mono
for i in *.wav; do
  ffmpeg -i $i -ac 1 mono/$i;
done
```
3. Split .wav files into chunks of 0.5 s duration
```bash
mkdir split
for i in *.wav; do
  ffmpeg -i $i -f segment -segment_time 0.5 split/${i%.*}_%03d.wav
done
```
4. Remove all clips with less than 0.5 s duration
```bash
for i in *.wav; do
  DURATION=$(ffprobe $i -show_format 2>&1 | sed -n 's/duration=//p')
  echo "$i: $DURATION s."
  st=$((`echo "$DURATION < 0.5"| bc`))
  if [ $st -eq 1 ]
  then
    echo "File $i lasts less than 0.5 s. Removing."
    rm $i
  fi
done
```

