---
layout: post
title: "Unity Configuration"
description: ""
category:
tags: [unity, citrix, compiz, config]
---
If you worked with any computer professionals from exUSSR who started their careers in the eighties and nineties you are no doubt familiar with their(mine) obsession with two-panel or [orthodox file managers](http://en.wikipedia.org/wiki/Orthodox_file_manager#Orthodox_file_managers). Efficiency of their use largely depends on hotkeys that have remained unchanged for the last twenty five years.

Unfortunately Ubuntu uses some of those keys for its own purposes. To regain control you can use CompizConfig Settings Manager. To install it use *sudo apt-get install compizconfig-settings-manager*. Most keys are used by "Ubuntu Unity Plugin" under "Desktop". Critical "Alt+F7" is used by "Move Window" under "Windows Management".

The same CompizConfig Settings Manager helped me to solve a problem with my citrix receiver that stopped working properly in fullscreen mode ever since switching to Unity. To deal with this try enabling "Legacy Fullscreen Support" in "Workarounds" under "Utility" as suggested [here](http://ubuntuforums.org/showpost.php?p=11606441&postcount=6).
