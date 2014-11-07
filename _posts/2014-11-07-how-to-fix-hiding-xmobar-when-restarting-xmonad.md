---
layout: post
title: "How to Fix Hiding Xmobar When Restarting Xmonad"
modified:
categories: Arch
tags: [xmonad, xmobar]
image:
  feature: xmobar/xmobar1.png
date: 2014-11-07T15:00:52+00:00
---

This is something I've struggled with for a long time. I was never able to restart xmonad gracefully, it would crash or do strange stuff. After sorting that out, it would restart, but xmobar would be gone. I found 2 problems with my set-up, documented here.

# The problem

When restarting xmonad (either through Mod-q or `xmonad --restart`), xmobar is gone. There's still some space at the top where it should be.

# The solution

## Check that you're loading xmobar correctly

Check out your `xmonad.hs`: are you loading xmobar correctly? I have the following in my `xmonad.hs` (it's also [on GitHub](https://github.com/thomastoye/dotfiles/blob/master/dotfiles/xmonad/xmonad.hs)):

{% highlight haskell linenos %}
...

main = do
  xmproc <- spawnPipe "/usr/bin/xmobar ~/.xmobarrc"
  xmonad $ defaults
      { manageHook = manageDocks <+> manageHook defaultConfig
      , layoutHook = avoidStruts $ myLayouts
      , logHook = dynamicLogWithPP xmobarPP
        { ppOutput = hPutStrLn xmproc
        , ppTitle = xmobarColor "#657b83" "" . shorten 100
        , ppCurrent = xmobarColor "#c0c0c0" "" . wrap "" ""
        , ppSep = xmobarColor "#c0c0c0" "" " | "
        , ppUrgent = xmobarColor "#ff69b4" ""
        , ppLayout = const "" -- to disable the layout info on xmobar
        }
    }
  
...
{% endhighlight %}

## Check your .xmobarrc

There's an option called `lowerOnStart` in `.xmobarrc`, which seems to be important. In my `.xmobarrc`, it was set to `False`, and setting it to true fixed my problems. I don't know why it was set to false, it seems to have something to do with desktop enviroments.

Here's an example:

{% highlight haskell linenos %}
Config {
font         = "xft:Inconsolata:pixelsize=14,-*-*-*-r-*-*-16-*-*-*-*-*-*-*"
, position = TopSize C 100 26
, lowerOnStart = False
, commands = [ Run Weather "UUEE" ["-t"," <tempC>C","-L","64","-H","77","--normal","#657b83","--high","#657b83","--low","#657b83"] 36000

...
{% endhighlight %}

