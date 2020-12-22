---
layout: post
title:  Calculating the WPA2 PMK using C#
categories: [C#,Packet Gremlin]
excerpt: One of the features I've wanted to implement for [Packet Gremlin](https://github.com/SapientGuardian/LibPacketGremlin){:target="_blank"} is the ability to decrypt encrypted WiFi traffic. I only know of two tools capable of this: [Airdecap-ng](http://www.aircrack-ng.org/doku.php?id=airdecap-ng){:target="_blank"}, which can't do it on a live capture, and [Wireshark](http://www.wireshark.org){:target="_blank"}, which can't capture wireless traffic on Windows due to a limitation of WinPCap. One of the things needed for implementing this feature is to calculate the PMK (Pairwise Master Key). By studying the source code of airdecap-ng and various online resources, I found that this is actually trivial.
---

One of the features I've wanted to implement for [Packet Gremlin](https://github.com/SapientGuardian/LibPacketGremlin){:target="_blank"} is the ability to decrypt encrypted WiFi traffic. I only know of two tools capable of this: [Airdecap-ng](http://www.aircrack-ng.org/doku.php?id=airdecap-ng){:target="_blank"}, which can't do it on a live capture, and [Wireshark](http://www.wireshark.org){:target="_blank"}, which can't capture wireless traffic on Windows due to a limitation of WinPCap. One of the things needed for implementing this feature is to calculate the PMK (Pairwise Master Key). By studying the source code of airdecap-ng and various online resources, I found that this is actually trivial:

```
public static byte[] CalculatePMK(byte[] psk, byte[] ssid, int pmkLength = 32)
        {
            Rfc2898DeriveBytes pbkdf2 = new Rfc2898DeriveBytes(psk, ssid, 4096);
            return pbkdf2.GetBytes(pmkLength);
        }
```

I was pleased to find a small set of unit tests accompany the aircrack suite, including one for PMK calculation. I ported it to C# and... it crashed. It turns out that, for reasons unknown to me, the Rfc2898DeriveBytes class requires that the salt (in this case, the ssid) be at least 8 bytes in length. Since ssids (including the one in this unit test) can be fewer than 8 bytes in length, I needed to work around this artificial limitation. I used [Reflector](https://www.red-gate.com/products/dotnet-development/reflector/){:target="_blank"} to examine the source of this class, and found that the exception is thrown in the setter for the Salt property, which is invoked in the constructor. The setter does three things:

1. Validates the length of the salt
2. Copies the salt to the private byte[] m_salt
3. Calls the private method Initialize()

I found that the public method Reset calls the private method Initialize, but upon further inspection of the Initialize method itself, I realized that in this particular case, I don't need to do it since this is a fresh instance of the class. All I really needed to do was fill in m_salt.

I used an empty byte[8] as a placeholder for the salt in the constructor, then used reflection to get a reference to the private m_salt field, and set it with the ssid:

```
public static byte[] CalculatePMK(byte[] psk, byte[] ssid, int pmkLength = 32)
        {
            Rfc2898DeriveBytes pbkdf2 = new Rfc2898DeriveBytes(psk, /*ssid*/ new byte[8], 4096);
            //This reflection is required because there's an arbitrary restriction that the salt must be at least 8 bytes
            var saltProp = pbkdf2.GetType().GetField("m_salt", System.Reflection.BindingFlags.NonPublic | System.Reflection.BindingFlags.Instance);
            saltProp.SetValue(pbkdf2, ssid);

            //pbkdf2.Reset(); 
            //To officially complete the reflection trick, the private method Initialize() should be called. That's all Reset() does. 
            //But I don't think it's needed because we haven't hashed anything yet.

            return pbkdf2.GetBytes(pmkLength);
        }
```

With that change, the unit test passed, and the PMK calculation was complete. In a subsequent post I will discuss the less trivial implementation of the PTK calculation.