---
layout: post
title:  "LXC Part 1: Getting started"
description: "In the post we will learn the basics of LXC. We will install library and create basic containers."
date:   2016-05-10
---

<link rel="stylesheet" href="/assets/css/highlight/styles/obsidian.css">
<script src="/assets/js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
<style>pre{padding:0}</style>


<h3>Installation</h3>

<p>Because I work on MacoS I have to use some sort of virtualization software. LXC runs only on Linux machines so I choose to virtualize Ubuntu server through Virtualbox+Vagrant.
So let's create Vagrant file in our project folder.

{% highlight ruby %}
Vagrant.configure("2") do |config|
   config.vm.box = "ubuntu/xenial64"
   config.vm.provider "virtualbox" do |vb|
     vb.name = "ubuntu"
     vb.memory = "1024"
   end
end
{% endhighlight %}
Now we can ssh in newly installed Ubuntu and install LXC

{% highlight bash %}
vagrant ssh

sudo apt install lxc
{% endhighlight %}

<h3>Running a container</h3>
Now we have LXC installed let's run our first container

{% highlight bash %}
lxc-create -t ubuntu -n c1
lxc-ls -f
{% endhighlight %}

We can see our created container in the list

Isn't it simple. Now I want to review some basic operations on containers

Start container

{% highlight bash %}
lxc-start -n c1
{% endhighlight %}

Containers info
{% highlight bash %}
lxc-info -n c1
{% endhighlight %}
<h3>Stoppping and removing container</h3>

{% highlight bash %}
lxc-stop -n c1
lxc-destroy -n c1
{% endhighlight %}

That's it for today. Next time we will look deeper and learn advanced container's managmenssssaae.


Useful link:
https://help.ubuntu.com/lts/serverguide/lxc.html
