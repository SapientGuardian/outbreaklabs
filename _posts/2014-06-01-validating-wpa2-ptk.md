---
layout: post
title: Validating the WPA2 PTK using C#
categories: [C#,Packet Gremlin]
excerpt: Continuing on the quest to decrypt WiFi traffic, I've written code to validate the WPA PTK. This wasn't strictly necessary, and I tried to avoid doing so if only to get to the end goal faster, but when I started implementing the CCMP decryption and it wasn't working, I needed more visibility into what was going on. As before, I arrived at this code by studying various online resources and the Aircrack-ng source code.
---

Continuing on the quest to decrypt WiFi traffic, I've written code to validate the WPA PTK. This wasn't strictly necessary, and I tried to avoid doing so if only to get to the end goal faster, but when I started implementing the CCMP decryption and it wasn't working, I needed more visibility into what was going on. As before, I arrived at this code by studying various online resources and the Aircrack-ng source code.

```cs
public static bool PtkIsValid(byte[] ptk, IEEE_802_1X<IEEE_802_1X.Key> eapolKey)
        {
            bool tkip = (eapolKey.Payload.KeyInformation & 7) == 1;
            bool isValid;
            var MIC = eapolKey.Payload.MIC;
            eapolKey.Payload.MIC = new byte[MIC.Length];
            if (tkip)
            {
                var hmacmd5 = new HMACMD5(ptk.Take(16).ToArray());
                hmacmd5.ComputeHash(eapolKey.ToArray());
                isValid = hmacmd5.Hash.Take(16).SequenceEqual(MIC.Take(16));
            }
            else
            {
                var hmacsha1 = new HMACSHA1(ptk.Take(16).ToArray());
                hmacsha1.ComputeHash(eapolKey.ToArray());
                isValid = hmacsha1.Hash.Take(16).SequenceEqual(MIC.Take(16));
            }
            eapolKey.Payload.MIC = MIC;
            return isValid;
        }
```

We take the PTK (of course) and an EAPOL Key packet as parameters. This EAPOL packet is part of the four way authentication handshake. Of the four messages in the handshake, any of the latter three will do, though the fourth would probably be quickest to process because it contains no WPA key data. The first message in the handshake is insufficient, as it has no MIC with which we can verify. It's important to note that this isn't the entirety of the frame, which would include the 802.11 header as well as any cruft from the wireless driver (such as the radiotap header); it's just the 802.1x and above. This is one of the nice things about the Packet Gremlin packet structure - instead of accepting a large array and getting offsets into it, I can more clearly express what data I need.

We start by determining if the handshake was done with AES or TKIP. This is done by checking the KeyInformation field of the EAPOL key packet. We need to know this to decide whether to use the MD5 or SHA1 algorithm.

Next, we take the MIC out of the EAPOL packet and zero it out. Figuring out that this needed to be done was tricky; the Aircrack-ng unit test has the frame data with this already zeroed out, and the MIC stored separately.

If the handshake was done with TKIP, we initialize the HMACMD5 algorithm with the first 16 bytes of the PTK, and use it to compute the hash over the EAPOL frame with the MIC zeroed out. We compare the first 16 bytes of the resulting hash with the first 16 bytes of the MIC we copied out of the EAPOL packet. If they are equal, then the PTK is valid.

Similarly, if the handshake was done with AES, we initialize the HMACSHA1 algorithm with the first 16 bytes of the PTK, and use it to compute the hash over the EAPOL frame with the MIC zeroed out. We compare the first 16 bytes of the resulting hash with the first 16 bytes of the MIC we copied out of the EAPOL packet. If they are equal, then the PTK is valid.

Before we're done, we put the MIC back in the EAPOL packet. I don't like having to do this temporary corruption of the packet. I could make a copy first, but I think that would be more of a performance hit than it's worth.

And there you have it! In the next post, I'll go over CCMP decryption, the final step in the process.