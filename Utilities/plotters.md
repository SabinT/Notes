# Cleaning up SVGs for plots
https://github.com/abey79/vpype#installation

Example commands:

## Optimize, fit to letter size with margin

```
vpype \
  read input.svg \
  linemerge --tolerance 0.1mm \
  layout --fit-to-margins 2cm letter \
  linesort \
  reloop \
  linesimplify \
  write output.svg
```

## Optimize only
```
vpype \
  read input.svg \
  linemerge --tolerance 0.1mm \
  linesort \
  reloop \
  linesimplify \
  write output.svg
```

## Convert STL to SVG
https://plotter.vision/

Seems to also convert triangle pairs to quads when possible.