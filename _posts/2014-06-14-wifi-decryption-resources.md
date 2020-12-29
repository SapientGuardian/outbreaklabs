---
layout: post
title: WiFi Decryption resources and test data
categories: [Packet Gremlin]
excerpt: I thought I'd share some of the resources I used in the recent WiFi decryption series of posts.
---

I thought I'd share some of the resources I used in the recent WiFi decryption series of posts.

First, much time was saved by my friend [Daniel Smullen](http://www.daniel-smullen.com/){:target="_blank"}, who put together a set of clean captures of WEP, WPA, and WPA2 for me.

The [Aircrack-ng project](http://www.aircrack-ng.org/){:target="_blank"} was invaluable; it likely would have taken many months to accomplish what I did, were it not for the source code of this tool suite being available.

Finally, here are some miscellaneous resources that were useful in understanding the various protocols and encryption schemes involved:

[http://www.willhackforsushi.com/papers/80211_Pocket_Reference_Guide.pdf](http://www.willhackforsushi.com/papers/80211_Pocket_Reference_Guide.pdf){:target="_blank"}  
[http://svn.fonosfera.org/fon-ng/trunk/openwrt/package/broadcom-wl/src/driver/proto/eapol.h](http://svn.fonosfera.org/fon-ng/trunk/openwrt/package/broadcom-wl/src/driver/proto/eapol.h){:target="_blank"}  
[http://www.xirrus.com/cdn/pdf/wifi-demystified/documents_posters_encryption_plotter](http://www.xirrus.com/cdn/pdf/wifi-demystified/documents_posters_encryption_plotter){:target="_blank"}  
[http://my.safaribooksonline.com/book/networking/wireless/0596001835/802dot11-framing-in-detail/wireless802dot11-chp-4-sect-3](http://my.safaribooksonline.com/book/networking/wireless/0596001835/802dot11-framing-in-detail/wireless802dot11-chp-4-sect-3){:target="_blank"}  
[https://chromium.googlesource.com/chromiumos/third_party/hostap/+/0.12.369.B/wlantest/ccmp.c](https://chromium.googlesource.com/chromiumos/third_party/hostap/+/0.12.369.B/wlantest/ccmp.c){:target="_blank"}  
[http://security.stackexchange.com/questions/46670/does-using-wpa2-enterprise-just-change-the-attack-model-vs-wpa2-psk](http://security.stackexchange.com/questions/46670/does-using-wpa2-enterprise-just-change-the-attack-model-vs-wpa2-psk){:target="_blank"}  
[http://www.seas.gwu.edu/~cheng/388/LecNotes/CCMP.pdf](http://www.seas.gwu.edu/~cheng/388/LecNotes/CCMP.pdf){:target="_blank"}  
[http://stackoverflow.com/questions/12018920/wpa-handshake-with-python-hashing-difficulties](http://stackoverflow.com/questions/12018920/wpa-handshake-with-python-hashing-difficulties){:target="_blank"}  
[http://stackoverflow.com/questions/19144775/4-way-handshake-simulation-in-c-sharp?rq=1](http://stackoverflow.com/questions/19144775/4-way-handshake-simulation-in-c-sharp?rq=1){:target="_blank"}  
[http://stackoverflow.com/questions/2465690/pbkdf2-hmac-sha1](http://stackoverflow.com/questions/2465690/pbkdf2-hmac-sha1){:target="_blank"}  
[http://hashcat.net/forum/thread-1745.html](http://hashcat.net/forum/thread-1745.html){:target="_blank"}