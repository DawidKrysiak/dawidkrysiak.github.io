---
layout: post
title: Apple Mac keyboard on Linux
date: 2021-12-13T19:42:00Z
tags:
  - macos
  - linux
draft: false
share: "true"
---

## Preface
Das Keyboard for Apple systems contain an 'Eject' Key (so the chasis of a computer is nice and slick :)). Problem is, it comes at the cost of 'Insert' key.

Solution should work on all Linux distros.

* (if doesn't exist) create ~/.Xmodmap
```
touch ~/.Xmodmap
```
* Add a keycode to the end of the file:
```
echo "keycode 169 = Insert" >> ~/.Xmodmap
```
* Reload the config
```
xmodmap ~/.Xmodmap
```
'Eject' key should stop ejecting the optical drive tray and act as Insert. Win :)


## Known issues!!!
For some reason, certain Linux distros (in my case pop!OS) struggle with the above a bit, FREEZING the whole WindowsManager for a few seconds ecery time a terminal window is opened and the above command invoked.
So there is another solution.

```echo "xmodmap -e 'keycode 169 = Insert'" >> ~/bash_rc```

That will make the screen freeze for a second once - upon login to the desktop, but I see no issues afterwards.


(c) Dawid Krysiak https://itisoktoask.me/