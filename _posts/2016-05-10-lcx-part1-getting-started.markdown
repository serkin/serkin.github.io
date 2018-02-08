---
layout: post
title:  "LXC Part 1: Getting started"
description: "In the post we will learn the basics of LXC. We will install library and create basic containers."
date:   2016-05-10
---

<h3>Installation</h3>

<p>Because I work on MacoS I have to use some sort of virtualization software. LXC runs only on Linux machines so I choose to virtualize Ubuntu server through Virtualbox+Vagrant.
So let's create Vagrant file in our project folder.


Vagrant.configure("2") do |config|
   config.vm.box = "ubuntu/xenial64"
   config.vm.provider "virtualbox" do |vb|
     vb.name = "ubuntu"
     vb.memory = "1024"
   end
end

Now we can ssh in newly installed Ubuntu and install LXC

vagrant ssh

sudo apt install lxc


<h3>Running a container</h3>
Now we have LXC installed let's run our first container

lxc-create -t ubuntu -n c1
lxc-ls -f

We can see our created container in the list

Isn't it simple. Now I want to review some basic operations on containers

Start container

lxc-start -n c1


Containers info
lxc-info -n c1

<h3>Stoppping and removing container</h3>


lxc-stop -n c1
lxc-destroy -n c1


That's it for today. Next time we will look deeper and learn advanced container's managmenssssaae.


Useful link:
https://help.ubuntu.com/lts/serverguide/lxc.html
