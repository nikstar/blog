# blog

Source code for [nikstar.me](https://nikstar.me).

### Develop

```bash
hugo server -D & ; sleep 0.5 && open http://localhost:1313/ ; fg
```

### Generate

```bash
hugo
```

### Publish

```bash
rm -r public && hugo && rsync -aO public/ nikstar.me:/var/www/blog
```
