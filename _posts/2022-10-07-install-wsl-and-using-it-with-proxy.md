---
layout: post
title: WSL install and using it with proxy
date: 2022-10-07 00:00:00
categories: [programming]
tags: [development environment, beginner]
last_modified_at: 2023-02-15
---

  WSL install instruction from MS.  
1. `dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`  
  List all distribution online  
2. `wsl --list --online`  *// list out available distribution*  
3. `wsl --set-default-version 1` *// using WSL1 because I have some issue on virtual network with WSL2*  
4. `wsl --install -d Ubuntu` *// install Ubuntu WSL*  
5. `wsl -d Ubuntu` *// open WSL*  

  If your PC is behind firewall, when you try to install/update package, following error shall occur.  
{% highlight bash %}
user@usergrp:~$ sudo apt update
Err:1 http://archive.ubuntu.com/ubuntu focal InRelease
  Connection failed [IP: 185.125.190.39 80]
Err:2 http://archive.ubuntu.com/ubuntu focal-updates InRelease
  Connection failed [IP: 91.189.91.39 80]
Err:3 http://security.ubuntu.com/ubuntu focal-security InRelease
  Connection failed [IP: 91.189.91.39 80]
Err:4 http://archive.ubuntu.com/ubuntu focal-backports InRelease
  Connection failed [IP: 185.125.190.39 80]
Reading package lists... Done
Building dependency tree
Reading state information... Done
277 packages can be upgraded. Run 'apt list --upgradable' to see them.
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/focal/InRelease  Connection failed [IP: 185.125.190.39 80]
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/focal-updates/InRelease  Connection failed [IP: 91.189.91.39 80]
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/focal-backports/InRelease  Connection failed [IP: 185.125.190.39 80]
W: Failed to fetch http://security.ubuntu.com/ubuntu/dists/focal-security/InRelease  Connection failed [IP: 91.189.91.39 80]
W: Some index files failed to download. They have been ignored, or old ones used instead.
{% endhighlight %}

to fix above issues, following below step.  

1. `sudo vim ~/.bashrc`  
  Add following at the end of file:  
3. `export http_proxy=http://<address>:<port>`  
4. `sudo visudo`  
  Add following line after line "Defaults env_reset"  
5. `Defaults        env_keep = "http_proxy"`  
  Things should be able to run smoothly now.  
{% highlight bash %}
user@usergrp:~$ sudo apt update
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Hit:2 http://archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://archive.ubuntu.com/ubuntu focal-updates InRelease
Get:4 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Fetched 222 kB in 10s (21.2 kB/s)  
Reading package lists... Done
Building dependency tree       
Reading state information... Done
275 packages can be upgraded. Run 'apt list --upgradable' to see them.
{% endhighlight %}
