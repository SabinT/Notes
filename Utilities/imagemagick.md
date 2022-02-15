# Batch convert PNG to JPG

```
mogrify -format jpg *.png
```

# Batch resize (best-fit) while maintaining aspect ratio

> WARNING: `mogrify` modifies images inline. Add `-path newFolder` to make results not overwrite originals.

```
mogrify -resize 1080x1080 -format jpg -quality 75 *.jpg
```