---
# vim: tw=80
title: Python Guide 
blurb: How to use the official python library, linode-api
layout: guide
---

<style>
table {
    table-layout: fixed;
}
</style>

With the release of API 4, Linode has also released an official python library.  This guide is a simple introduction to working with
the python library - if you're looking for comprehensive documentation, please refer to the [Python Library Reference](/libraries/python-reference).

## Getting Started

The official Linode python library is open-source on [github](http://github.com/Linode/python-linode-api), and can be installed through
pypi with:

{% highlight shell %}
pip install linode-api
{% endhighlight %}

In order to make requests to the API, you're going to need an OAuth Token.  If you haven't already, sign up for the [API V4 Alpha](https://alpha.linode.com),
then login to the [Alpha Login Service](https://login.alpha.linode.com), click "Manage personal access tokens" and "Generate a
new token".  This will generate and display a valid OAuth Token with access to your entire (alpha) account, which we will use for this
tutorial.

## Connecting to the API

The Python Library connects to the Linode API V4 using the `LinodeClient` class, which expects an OAuth Token in his constructor.  In this
example we'll also provide a `base_url` keyword argument, since we're talking to the alpha environment.

_All example code in this guide is executed in a python shell._

{% highlight python %}
>>> import linode
>>> client = linode.LinodeClient('my-oauth-token', base_url='https://api.alpha.linode.com/v4')
{% endhighlight %}

## Requirements for Creating a Linode

In order to create a Linode, we need a Datacenter (which defines where it will live) and a Service (which defines the size of the Linode).
Since we don't know the IDs of any objects in the API, we'll list them to see what we want:

{% highlight python %}
>>> for d in client.get_datacenters():
...   print(d.label)
...
Newark, NJ
{% endhighlight %}

Looks like we only have one option in the alpha environment - so we'll just grab it and move on.

{% highlight python %}
>>> dc = client.get_datacenters()[0]
{% endhighlight %}

For a Service, we know that we want "Linode 1024".  Since we know the service name, we don't need to list all services, we
can just ask for it by filtering the service list:

{% highlight python %}
>>> serv = client.get_services(linode.Service.label == "Linode 1024")[0]
{% endhighlight %}

The Python Library uses SQLAlchemy-like filtering syntax - any field [marked filterable](/libraries/python-reference/#api-objects) can be searched by in this manner.  Since we
searched a listing endpoint, we need to take the first result (which in this case is the only one) of the returned list.

## Making a Linode

Since we have everything we need to make a Linode, we can create one with the LinodeClient:

{% highlight python %}
>>> l = client.create_linode(serv, dc)
{% endhighlight %}

And that's it!  Now we've got a fresh new Linode.  Let's check it out:

{% highlight python %}
>>> l.label
'linode263'
>>> l.ip_addresses.public.ipv4
['97.107.143.33']
>>> l.state
'offline'
{% endhighlight %}

That's great, but this Linode is empty, and booting it wouldn't make sense, since we created it without disks or a distribution.  Let's make another Linode,
this time with Debian on it, and boot it so we can ssh in.

## Getting a Distribution

We already know how to retrieve objects from the API, but this time we want something more vague - a Debian template.  Which version?  What are our options?  Let's
have a look at all of the recommended Debian templates:

{% highlight python %}
>>> for d in client.get_distributions(linode.Distribution.vendor == 'Debian', linode.Distribution.recommended == True):
...   print("{}: {}".format(d.label, d.id))
...
Debian 7: distro_130
Debian 8.1: distro_140
{% endhighlight %}

Great, we have some options.  We can chain filters together to run more complex searches.  Since Debian 8.1 is the newest Debian template available, let's use it.
We already have the ID, so this time we'll create the Distribution object without querying for it:

{% highlight python %}
>>> distro = linode.Distribution(client, 'distro_140')
{% endhighlight %}

We need to give the Distribution object a reference to the LinodeClient so it knows how to talk to the API to populate itself.

## Creating a Linode (with a Distribution)

Now that we have a distribution, let's make a new Linode running it!  But first, let's clean up that first Linode that we don't really need:

{% highlight python %}
>>> l.delete()
True
{% endhighlight %}

Now that that's gone, we can create a new Linode running Debian 1.8:

{% highlight python %}
>>> l, pw = client.create_linode(serv, dc, source=distro)
{% endhighlight %}

This time, we called `create_linode` with a "source" keywork argument.  The source tells the API what to deploy, and it will do 
"[the right thing](/reference/#ep-linodes-POST)" to give you a working Linode.  Since a Distribution needs a root password and we didn't
provide one, the client helpfully generaeted one for us and returned it as well.  Let's boot it and wait for it to come online:

{% highlight python %}
>>> l.boot()
True
>>> while not l.state == 'running':
...   pass
...
>>> l.ip_addresses.public.ipv4
['97.107.143.34']
>>> pw
'663Iso_f1y4Zb5xeClY13fBGdeu5l&f3'
{% endhighlight %}

`boot()` does just what you'd expect.  Once we've requested a boot, we wait for the Linode's state to be "running".  Since state is a "volatile" attribute of
a Linode, we can poll it.  At fixed intervals, the value will be updated via a background API request.  Once that loop exists, we can ssh in.


{% highlight shell %}
>>> exit()
$ ssh root@97.107.143.34
The authenticity of host '97.107.143.34 (97.107.143.34)' can't be established.
ECDSA key fingerprint is SHA256:+/zkZlskou45MscmID8Upp5egrBBEssvL0PEx24C5Zw.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '97.107.143.34' (ECDSA) to the list of known hosts.
root@97.107.143.34's password:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@localhost:~#
{% endhighlight %}

## Further Reading

Now that you've had an overview of the features and concepts in the Python Library, check out the [Python Library Reference](/libraries/python-reference) for in-depth documentation,
or look at the [Install on Linode sample project](https://github.com/linode/python-linode-api/tree/master/examples/install-on-linode) for an example multi-user
application using the Library.
