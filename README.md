# Building a workshop sound classifier
## Step 1: Building the dataset
### Grabbing audios from YouTube
For downloading the audios using youtube-dl, follow these steps:
1. Copy each set of links to its own *batchfile.txt*, in separated directories
2. Run `youtube-dl -c -f 140 --batch-file batchfile.txt` in each directory
3. Let the magic begin
 

#### Aluminium milling
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


#### Aluminium sawing
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


### Extracting relevant audio chunks with Audacity
1. [Download and install Audacity](https://www.audacityteam.org/download/windows/)
2. [Donwload FFMPeg library (zip version)] to open m4a files with Audacity (https://lame.buanzo.org/ffmpeg64audacity.php)
3. Extract the folder and copy its contents to C:\Program Files\Audacity\FFMpeg (create FFMpeg folder)
4. Open Audacity and go to Edit -> Preferences -> Libraries. Click "Locate FFMpeg library" and point it to the upper path
5. Open each audio file, and mark as labels all chunks that contain usable sound to train the classifier with Ctrl + B
![Spectrogram extraction example with Audacity](/assets/images/spectrogram_audacity.png "Spectrogram extraction example with Audacity")
6. Export the labels as .wav files with Ctrl + Shift + L


### Normalize audio files
1. Normalize audio channels (i.e. convert them into mono):
```bash
mkdir mono
for i in *.wav; do
  ffmpeg -i $i -ac 1 mono/$i;
done
```
2. Make an 80/20 split for train and validation
```bash
ACC_DURATION_DIRECTORY=0
ACC_DURATION=0
mkdir train test

# Get accumulated duration of audio files in directory

for i in *.wav; do
  DURATION=$(ffprobe $i -show_format 2>&1 | sed -n 's/duration=//p')
  ACC_DURATION_DIRECTORY=`echo "$ACC_DURATION_DIRECTORY+$DURATION"|bc`
done

echo "Accumulated file duration is $ACC_DURATION_DIRECTORY s."
echo ""

TRAIN_FRACTION=`echo "$ACC_DURATION_DIRECTORY*0.8"|bc`

# Copy all files whose accumulated duration is below 0.8 to train set, and the other ones
# to the validation set

for i in `find . -maxdepth 1 -iname '*.wav' -type f | shuf`; do
  DURATION=$(ffprobe $i -show_format 2>&1 | sed -n 's/duration=//p')
  ACC_DURATION=`echo "$ACC_DURATION+$DURATION"|bc`
  statement=$((`echo "$ACC_DURATION <= $TRAIN_FRACTION"|bc`))
  if [ $statement -eq 1 ]
  then
    echo "$ACC_DURATION is less than $TRAIN_FRACTION s. Copying $i to ./train"
    cp $i ./train
  else
    echo "$ACC_DURATION is not less than $TRAIN_FRACTION s. Copying $i to ./test"
    cp $i ./test
  fi
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
5. Normalize name schema (example for milling audio dataset): `i=1; for f in *.wav; do mv "$f" "mill_$((i++)).wav"; done`
6. Get number of files to make sure no data loss has occured: `ls | wc -l`

## Step 2: Perform DL analysis over the audio dataset