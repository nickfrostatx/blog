---
layout: post
title: ASIS CTF Quals 2016 - Odrrere
---
In this challenge we get a PNG with invalid data.

![Image](/assets/odrrere-start.png)

Looking at it in an image viewer, only a portion of the file appears to be
viewable. Interestingly, if we remove every IDAT block except block 0, the
image looks identical. Maybe the blocks are out of order.

Let's try all the other IDAT blocks in this file, and see if any of them make
sense after this block. Block 12 works!

![Image](/assets/odrrere-progress.png)

One-by-one, we can try each of the remaining blocks until we find one that looks
correct.

After some trial and error, the correct order is 0,12,8,4,9,10,6,7,3,5,2,11,1.

Here's the final image:

![Image](/assets/odrrere-final.png)

{% highlight python %}
import binascii
import struct
import sys


def read_chunk(f):
    length_bytes = f.read(4)
    if len(length_bytes) == 0:
        return None
    length = struct.unpack('>I', length_bytes)[0]
    blkname = f.read(4)
    data = f.read(length)
    crc = f.read(4)
    assert struct.pack('>I', binascii.crc32(blkname + data)) == crc
    return blkname, data, length_bytes + blkname + data + crc


with open('odrrere.png', 'rb') as f, open('odrrere1.png', 'wb') as out:
    out.write(f.read(8))
    blocks = []
    end = None
    while True:
        chunk = read_chunk(f)
        if chunk is None:
            break
        blk, data, raw = chunk
        if blk == b'IDAT':
            blocks.append(raw)
        elif blk == b'IEND':
            end = raw
        else:
            out.write(raw)
    for c in sys.argv[1]:
        print(c)
        out.write(blocks['0123456789abc'.index(c)])
    out.write(end)
{% endhighlight %}

{% highlight bash %}
$ python odrre.python 0c849a67352b1
{% endhighlight %}
