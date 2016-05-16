---
layout: post
title: TU CTF 2016 - Neverending Crypto
---
This series of challenges is broken out into different levels that must be
completed consecutively.

## Level 1 - Morse

We provide a string, and the server will "encrypt" that string and send it back.
In this challenge, the encryption is just converting the string to morse code.
We are then prompted to decrypt a message of its choosing. We do this with 50
such strings to get to the next level.

{% highlight python %}
MORSE = {
    '.-': 'A',
    '-...': 'B',
    '-.-.': 'C',
    '-..': 'D',
    # ...
    '----.': '9',
    '': ' ',
}

def morse_decode(s, resp=None):
    s = s.decode('utf-8')
    decoded = ''.join(map(MORSE.__getitem__, s.split(' ')))
    return decoded.replace('  ', ' ').strip().encode('utf-8')
{% endhighlight %}

## Level 2 - ROT13

The encryption for this level is [ROT13](https://en.wikipedia.org/wiki/ROT13),
wrapping around the printable ASCII range (0x20 - 0x7e).

{% highlight python %}
def rot(enc, offset=13):
    resp = b''
    for c in enc:
        resp += chr(((c - 32) - offset) % (127 - 32) + 32).encode('ascii')
    return resp
{% endhighlight %}

# Level 3 - Substitution Cipher

This level is some sort of substitution cipher, but I honestly don't know how the
mapping is chosen. It may be random.

So for each challenge string, we send over a list of letters, and the response
will tell us the substitution for each letter. We can't send over the entire
alphabet, we're limited to 23 characters.

Since we can't fully decrypt every letter (we'd need to be able to send 26
characters for the full alphabet), we can maintain a list of all previous
messages sent in the first two levels and see if our partially decrypted text
matches any of those.

{% highlight python %}
def substitution(enc, resp):
    plain = b'abcdefghijklmnopqrstuvw'
    mapping = {32: 32}
    for i in range(len(plain)):
        mapping[resp[i]] = plain[i]
    resp = b''
    for c in enc:
        if c in mapping:
            resp += chr(mapping[c]).encode('ascii')
        else:
            resp += b'\0'
    return resp

def match(s, messages):
    for m in messages:
        if len(m) != len(s):
            continue
        for i in range(len(m)):
            if s[i] != 0 and m[i] != s[i]:
                break
        else:
            return m
    raise Exception('Couldn\'t find a match for %s' % repr(s))
{% endhighlight %}

## Level 4 - Variable-shift ROT

This level is similar to Level 2, but each letter isn't necessarily shifted by
13 places, the shift is chosen randomly. So we send `a`, and check the response
to see what the shift is. For example, if `c` is sent back, we know the shift
is 2.

{% highlight python %}
def variable_rot(enc, resp):
    return rot(enc, ord(resp) - ord(b'a'))
{% endhighlight %}

## Level 5 - Backwards

Starting at the space character (`0x20`), the substitution for each letter
works backwards. So `0x20` encodes to `0x20, 0x21` encodes to `0xfe`,
`0x22` to `0xfd`, and so on.

{% highlight python %}
def backwards(enc):
    resp = b''
    for c in enc:
        resp += chr(-(c - 32) % (127 - 32) + 32).encode('ascii')
    return resp
{% endhighlight %}

## Level 6 -

This is another substitution cipher. We send over `0x20 0x21`, which are the
first two printable ASCII characters. The mapping for these characters are
similar to Level 5. Except we don't assign letters to adjacent letters. `a`
might map to `z`, `b` to `x`, `c` to `v`, `d` to `t`, and so on. Notice that the
mapped characters have two letters between them. The starting character,
direction, and step size are random each time, so we'll have to get those from
the response to our initial message.

{% highlight python %}
def level6(enc, resp):
    offset = ord(resp[1:2]) - ord(resp[0:1])
    start = ord(resp[0:1])
    mapping = {}
    for c in range(32, 127):
        mapping[start] = c
        start = ((start + offset) - 32) % (127 - 32) + 32
    result = b''
    for c in enc:
        result += chr(mapping[c]).encode('ascii')
    return result
{% endhighlight %}

## Level 7 - Weird ROT

In this level, every character is ROT shifted again. However, the shift factors
are different depending on the position of the letter. Every third letter has
the same shift. So the first letter may shift by 3 characters, the second by 25,
the third by -7, the fourth by 3, the fifth by 25, and so on. We'll send `aaa`
to see what each of these three shift values are.

{% highlight python %}
def weird_rot(enc, resp):
    result = b''
    for i in range(len(enc)):
        result += rot(enc[i:i+1], None, resp[i % 3] - ord('a'))
    return result
{% endhighlight %}

## Level 8 - Transposition

In this level, `I really love apples` would encode to
`Ilee y sr a.elp.aop.lvl.`, where `.` represents a null byte. The letters are
transposed by arranging the letters into rows of length 6, and reading down the
columns.

{% highlight text %}
I real
ly lov
e appl
es
{% endhighlight %}

{% highlight python %}
def transposition(enc, _):
    l = len(enc)
    num = l // 6
    result = b''
    for i in range(l):
        n = (i * num) % l + (i * num) // l
        if enc[n] == 0:
            break
        result += enc[n:n+1]
    return result
{% endhighlight %}

## Level 9 - ???

I have no idea how Level 9's encryption works.
