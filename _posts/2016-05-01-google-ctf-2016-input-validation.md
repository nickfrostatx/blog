---
layout: post
title: Google CTF 2016 - Opabina Regalis - Input Validation
---
> Following on from Opabina Regalis - Fetch Token and Opabina Regalis -
> Downgrade Attack - can you find an input validation request that would allow
> you to access otherwise protected resources?
>
> [This](https://en.wikipedia.org/wiki/Digest_access_authentication) may give
> you some inspiration on where the issue lies.
>
> Listening on port 12001 on ssl-added-and-removed-here.ctfcompetition.com

This challenge was a little boring. I used my solution to [Opabina Regalis -
Downgrade Attack]({% post_url 2016-05-01-google-ctf-2016-downgrade-attack %}),
but replaced ``/protected/secret`` with ``/protected/token`` and got the flag.

{% highlight http %}
HTTP/1.1 200 OK
Server: opabina-regalis.go

CTF{-Why-dont-eggs-tell-jokes...Theyd-crack-each-other-up-}"
{% endhighlight %}
