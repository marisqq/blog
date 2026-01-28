---
title: "setup commands - random"
layout: default
parent: CEH v13 Practical Notes
nav_order: 11
---

ubuntu server change network interfaces to enable access to internet:  

sudo networkctl up enp7s0 && sudo networkctl renew enp7s0  

sudo networkctl down enp7s0  

ip route  
