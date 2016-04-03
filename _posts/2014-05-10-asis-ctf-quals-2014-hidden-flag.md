---
layout: post
title: ASIS CTF Quals 2014 - Hidden Flag
---
This challenge has no description, so the solution must be on the site itself.

Clicking on a problem makes a request for some HTML to embed in the modal that
pops up. If we take a look at this request, this header comes along with it:

{% highlight yaml %}
x-flag: ASIS_b6b?244608c2?c2e869cb56?67b64?b1
{% endhighlight %}

Interesting.

This isn't the final flag, because four of the hex characters are missing.
However, taking another look at the code reveals something useful.

{% highlight javascript %}
var shaObj = new jsSHA(document.forms["flag_submission"]["id_flag"].value,
                       "TEXT");
var hash = shaObj.getHash("SHA-256", "HEX");
var shaObj2 = new jsSHA(hash, "TEXT");
var hash2 = shaObj2.getHash("SHA-256", "HEX");
if (document.forms["flag_submission"]["check"].value !== hash2) {
{% endhighlight %}

Before submitting an answer, it checks your submission against a hash of the
correct answer to see if you're right. The hash itself also comes with the
request.

{% highlight html %}
<input id="id_check" name="check" type="hidden"
    value="2b127c77074e44b6e74074b1eb8d32dfe27fe78e6a05e302baed68e2cc643ca1" />
{% endhighlight %}

So to find the hash, we have to loop through all possible replacements of those
four missing hex characters in the flag (16^4, or 65536 combinations), and test
them against the hash until we find the correct flag. A quick Python script will
do the trick.

{% highlight python %}
#!/usr/bin/env python

from hashlib import sha256
from itertools import product

FLAG = 'ASIS_b6b?244608c2?c2e869cb56?67b64?b1'
HASH = '2b127c77074e44b6e74074b1eb8d32dfe27fe78e6a05e302baed68e2cc643ca1'

pieces = FLAG.split('?')

for chars in product('0123456789abcdef', repeat=len(pieces) - 1):
    s = pieces[0]
    for i, piece in enumerate(pieces[1:]):
        s += chars[i] + piece
    if sha256(sha256(s).hexdigest()).hexdigest() == HASH:
        print(s)
        break

{% endhighlight %}
