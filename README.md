# blog

Source code for [nikstar.me](https://nikstar.me).

### Develop

```bash
hugo server -D & ; sleep 0.5 && open http://localhost:1313/ ; fg
```

Encoding video to VP9 and H.264:

```
ffmpeg -i video.mp4 -vf scale=-1:1200 -c:v libvpx-vp9 -b:v 0 -crf 31 -pass 1 -an -f null /dev/null && \
ffmpeg -i video.mp4 -vf scale=-1:1200 -c:v libvpx-vp9 -b:v 0 -crf 31 -pass 2 -an video.webm

ffmpeg -i video-orig.mp4 -vf scale=-2:1200 -c:v libx264 -crf 21 -preset veryslow -an video.mp4
```

### Generate

```bash
hugo
```

### Publish

```bash
rm -r public && hugo && rsync -aO public/ nikstar.me:/var/www/blog
```
