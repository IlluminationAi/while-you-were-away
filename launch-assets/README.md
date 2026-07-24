# Launch assets

These private-data-free Product Hunt graphics are generated from product claims
that are backed by the public tests and authenticated non-root dogfood.

- `01` through `05` are 1270x760 gallery PNGs.
- `thumbnail-240.png` is the square 240x240 thumbnail.
- Regenerate them with `bin/build-launch-assets`.

The builder requires Python 3 and Pillow 10.1 or newer. It uses Pillow's bundled
font, fixed colors, fixed copy, no external input, and no embedded host data.

Product Hunt's official launch guidance, accessed 2026-07-24, recommends
1270x760 gallery images, at least two gallery images, and a 240x240 square
thumbnail under 3 MiB:

https://help.producthunt.com/en/articles/479557-how-to-post-a-product
