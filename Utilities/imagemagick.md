# Batch convert PNG to JPG

```
mogrify -format jpg *.png
```

# Batch resize (best-fit) while maintaining aspect ratio

> WARNING: `mogrify` modifies images inline. Add `-path newFolder` to make results not overwrite originals.

```
mogrify -resize 1080x1080 -path output -format jpg -quality 75 *.jpg
```

# Create multiresolution .ICO file
```
convert -background transparent "icon256.png" -define icon:auto-resize=16,24,32,48,64,72,96,128,256 "icon.ico"
```