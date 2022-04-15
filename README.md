# slides

```bash
# generate marp html slideshow:
cd cgroups-v2
docker run --rm --init -v $PWD:/home/marp/app/ -e LANG=$LANG -p 37717:37717 marpteam/marp-cli -w slides.md
```
