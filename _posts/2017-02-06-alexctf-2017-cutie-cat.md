---
layout: post
title: AlexCTF 2017 - Cutie Cat
---
This challenge is a PNG image. Apparently, it contains a message hidden with
steganography.

![Image](/assets/cat_with_secrets.png)

If we look it up with Google reverse image search, exactly one result is the
same size:

[http://ndl.mgccw.com/mu3/app/20140301/08/1393643661844/sss/2_small.png](http://ndl.mgccw.com/mu3/app/20140301/08/1393643661844/sss/2_small.png)

![Image](/assets/cat_with_secrets_original.jpg)

These images are *almost* identical. There are a few pixels that don't match up.
Here are where they show up:

![Image](/assets/cat_with_secrets_diff.png)

Some of that is noise. The pixels that look interesting are the 3 rows, where
pixels seem to differ at somewhat regular intervals:

![Image](/assets/cat_with_secrets_diff_top.png)

If we print out the x values of these pixels, we get:

``4 17 36 60 68 85 101 120 132 147 165 180 196 214 231 251 260 275 292 305 325
340 357 371 389 415 420 440 452 473 484 500 4 21 37 63 69 83 100 117 132 147 165
178 196 213 229 244 261 275 293 319 324 340 356 383 388 414 421 436 453 479 485
500 4 24 36 53 69 89 103 125``

Or in hex:

``04 11 24 3c 44 55 65 78 84 93 a5 b4 c4 d6 e7 fb 104 113 124 131 145 154 165
173 185 19f 1a4 1b8 1c4 1d9 1e4 1f4 04 15 25 3f 45 53 64 75 84 93 a5 b2 c4 d5 e5
f4 105 113 125 13f 144 154 164 17f 184 19e 1a5 1b4 1c5 1df 1e5 1f4 04 18 24 35
45 59 67 7d``

Note that the leading hex character always increments by 1. However, what
happens when we look at just the lower 4 bits of each position in hex?

``414c45584354467b434154535f484944455f534543524554535f444f4e545f544845597d``

That looks like ASCII! It decodes to ``ALEXCTF{CATS_HIDE_SECRETS_DONT_THEY}``.

Here's my script (it uses [Pillow](https://python-pillow.org/) to read the
images):

{% highlight python %}
import binascii
from PIL import Image

new = Image.open('cat_with_secrets.png').load()
old = Image.open('cat_with_secrets_original.jpg').load()

width = 512

in_hex = ''

for y in range(3):
    for x in range(width):
        if (y, x) <= (2, 125) and new[x, y] != old[x, y]:
            in_hex += '%x' % (x % 16)

print(binascii.unhexlify(in_hex))
{% endhighlight %}
