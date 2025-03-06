---
#layout: posts
tags: 
  - tutorial
  - guide
  - security
  - linux
  - fail2ban
  - ssh
  - samba
title: "Further secure your server with Fail2ban"
#excerpt_separator: <!--more-->
excerpt: "This is my approarch to how to try and increase the security of your servers."
header:
  teaser: "/assets/images/fail2ban.png"
  og_image: "/assets/images/fail2ban.png"
toc: true # Table of contents
toc_sticky: True
#author_profile: false
---

<center>

<img src="{{ site.baseurl }}/assets/images/fail2ban.png" alt="Fail2ban in Github graph" />
 <center><small><em>Fail2ban graph made by Github</em></small></center>
 </center>

 ---

Last February I got somehow succesfully attacked by a Ransomware known as `WantToCry` (not to be confused with `WannaCry` as they appear to be different) in my **homelab server** running a **Samba share**. In that share I stored my collection of shows and films and some of my backup files that I had to delete once they got encrypted.

The way I found this was happening is because I was going to bed and before sleeping I wanted to watch an episode of a tv show with my laptop using my self-hosted `Jellyfin server` 