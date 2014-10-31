---
layout: post
title: "My Home Network Setup"
modified:
categories: RaspberryPi
tags: [Raspberry Pi, networks]
image:
  feature: raspi.jpg
date: 2014-10-30T11:54:50+00:00
---

I recently purchased a second Raspberry Pi. After adding it to our LAN, I decided to take the time to document it.

# Overview



# The first Raspberry Pi

I use the first Raspberry Pi, a standard model B (which means 2 USB ports) as a fileserver. Attached is a 1TB HDD. It runs the following services:

* [Samba](https://wiki.debian.org/SambaServerSimple), for sharing files across the LAN,
* [Deluge](http://www.howtogeek.com/142044/how-to-turn-a-raspberry-pi-into-an-always-on-bittorrent-box/), for downloading torrents,
* [CUPS](https://wiki.debian.org/SystemPrinting), for printing. Our printer comes with "wireless printing" that never works, so I disabled that and hooked up the RasPi,
* [minidlna](http://bbrks.me/rpi-minidlna-media-server/) (name is being changed to ReadyMedia), for making media available over DLNA/UPnP.

This is a headless box, there's no screen connected. 

I also set up zsh and [my dotfiles](https://github.com/thomastoye/dotfiles), which makes for more pleasant SSH sessions - working with zsh, I've grown to find bash unfriendly.

# The second Raspberry Pi

It's not fully set up, but I want to make this one a media center. I'm still waiting on more SD cards (they're on backorder) to get started.

# Samba

## Linux machines

Linux machines speak SMB, but will usually have to install an extra package to have it supported in the file manager (for example [thunar-shares-plugin](https://aur.archlinux.org/packages/thunar-shares-plugin/) for thunar).

<figure>
  <a href="../../images/network_setup/xfcesamba.png"><img src="../../images/network_setup/xfcesamba.png" /></a>
  <figcaption>Listing shares with Thunar</figcaption>
</figure>

## Windows configuration

Since SMB came from the Windows world, support is pretty good. It's even possible to map drives, but that won't work outside of the LAN, of course.

<figure class="half">
  <a href="../../images/network_setup/windowssamba1.png"><img src="../../images/network_setup/windowssamba1.png" /></a> 
  <a href="../../images/network_setup/windowssamba2.png"><img src="../../images/network_setup/windowssamba2.png" /></a>
  <figcaption>Listing shares on Windows</figcaption>
</figure>

## OS X configuration

This is what I had most trouble with. Since all of the shares are either read-only or restricted, I made them public access, so no usernames and passwords. Macs seem to have a problem with that: logging in as a guest <strike>is</strike> used to be nothing but trouble. Seems like it's all solved now, I followed [this](https://support.apple.com/kb/HT5884) and all my problems disappeared


<figure>
  <a href="../../images/network_setup/macsamba.png"><img src="../../images/network_setup/macsamba.png" /></a>
  <figcaption>Listing shares on OS X</figcaption>
</figure>


# Deluge

Deluge is able to run on a headless box, like a lot of torrent programs, but what makes it really awesome is it's client-server architecture. I can run deluge on my laptop and connect to my RapPi. Not only can I start, pause and add torrents, all preferences can edited remotely, as if it was running locally.

## Notifications

I like to know when something happens, so I set up deluge to email me when torrents complete.

## Android

I also set up [Transdroid](http://www.transdroid.org/) to see progress from my phone.

# Authentication and management

I manage most of the devices through SSH. I disable password login on most boxen to thwart brute-force attacks.

After copying your public key to the server (either manually or using `ssh-copy-id`), set the following to no in `/etc/ssh/sshd_config`:

{% highlight bash %}
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
{% endhighlight %}

I use a smart card for SSH authentication, but that's something for a different post.

<figure>
  <a href="../../images/network_setup/pinentry.png"><img src="../../images/network_setup/pinentry.png" /></a>
  <figcaption>SSH authentication with a smart card</figcaption>
</figure>

# DLNA/UPnP

For me, this has been a god-send. If you're not aware, DLNA/UPnP allows you to define several actors, the important ones are:

* Servers: store multimedia content, such as music, video and photos,
* Players: a device with the ability to play media, for example, a smart TV or radio,
* Controller: a device that allows you to specify what player should play what from what source.

Sounds rather dull, but consider that this all works together flawlessy (yay standards!). I can just use an iPad to start a movie on my phone and pause the music on my computer, adjust the volume../... And it all just works!

## On the Raspberry Pi

I use the first RasPi as a server, using `minidlna`. Configuration is easy, but remember to `minidlna -R` regurarly, this rebuilds the index. I use a cronjob that will do this every hour.

## On Linux

I use [upmpdcli](http://www.lesbonscomptes.com/upmpdcli/) (what a name) as a DLNA/UPnP audio client. Basically, it presents itself as a DLNA/UPnP client, and lets [mpd](http://www.musicpd.org/) play it.

## On Android

On Android, I use [BubbleUPnP](https://play.google.com/store/apps/details?id=com.bubblesoft.android.bubbleupnp&hl=en). It basically does everything, it's a DNLA/UPnP client, server and controller.

## On Windows

I don't use Windows a lot, but I believe [Foobar2000](http://www.foobar2000.org/) has UPnP support.

## On iOS

I use [media:connect](https://itunes.apple.com/be/app/media-connect-stream-music/id335036887?mt=8) on iOS. It's comparable to BubbeUPnP, but the interface is just a little snappier.

<figure class="half">
  <a href="../../images/network_setup/ipaddlna1.png"><img src="../../images/network_setup/ipaddlna1.png" /></a> 
  <a href="../../images/network_setup/ipaddlna2.png"><img src="../../images/network_setup/ipaddlna2.png" /></a>
  <figcaption>Listing shares on Windows</figcaption>
</figure>


## Security implications

Enabling UPnP has been considered a security risk. Read up before you rely on it. For myself, I decided to enable it (or rather, leave it enabled, since it was enabled by default on the router).

