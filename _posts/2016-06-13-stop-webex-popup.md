---
layout: post
title:  Live Patching of Cisco Webex to Stop the Popup
categories: [C#,Reverse Engineering]
excerpt: The company I work for recently started using Cisco WebEx, an online collaboration application. It's been configured with a rather annoying behavior - upon completion of every meeting, it launches our company web page in my browser. This is, of course, the last thing anyone would want to do after the conclusion of their meeting.
---

The company I work for recently started using Cisco WebEx, an online collaboration application. It's been configured with a rather annoying behavior - upon completion of every meeting, it launches our company web page in my browser. This is, of course, the last thing anyone would want to do after the conclusion of their meeting.

After diplomacy with the IT department failed, my first attempt at stopping this was to patch the WebEx binaries to remove the offending routine. This was successful, but the module was overwritten by some sort of auto-update process after a few days of use.

To avoid having the patch inadvertently removed, I developed a small Windows service that waits for WebEx to be launched and patches it in memory to remove the code that spawns the popup.

I have uploaded the source code to the service at [https://github.com/SapientGuardian/WebExPopupKiller](https://github.com/SapientGuardian/WebExPopupKiller)){:target="_blank"} for anyone curious as to how to write such a thing.