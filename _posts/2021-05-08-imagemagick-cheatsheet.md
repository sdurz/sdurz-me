---
title: Quick options for scaling images with imagemagick (for email use, previews...)
category: blog
tags: imagemagick batch image processing linux console scale
---

```bash
$ mogrify -resize 2048x2048 -quality 85 *.JPG
```