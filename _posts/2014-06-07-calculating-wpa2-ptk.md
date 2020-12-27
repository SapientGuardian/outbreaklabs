---
layout: post
title: Calculating the WPA2 PTK using C#
categories: [C#,Packet Gremlin]
excerpt: Continuing with the quest to decrypt wifi traffic, I needed to be able to calculate the PTK (Pairwise Transient Key).
---

Continuing with the quest to decrypt wifi traffic, I needed to be able to calculate the PTK (Pairwise Transient Key). Continuing with the quest to decrypt wifi traffic, I needed to be able to calculate the PTK (Pairwise Transient Key). As before, I studied the airdecap-ng source code as well as various online resources, to come up with the following code:

```cs
public static byte[] CalculatePTK(byte[] pmk, byte[] stmac, byte[] bssid, byte[] snonce, byte[] anonce)
        {           
            var pke = new byte[100];
            var ptk = new byte[80];

            using (var ms = new System.IO.MemoryStream(pke))
            {
                using (var bw = new System.IO.BinaryWriter(ms))
                {
                    bw.Write(new byte[] { 0x50, 0x61, 0x69, 0x72, 0x77, 0x69, 0x73, 0x65, 0x20, 0x6b, 0x65, 0x79, 0x20, 0x65, 0x78, 0x70, 0x61, 0x6e, 0x73, 0x69, 0x6f, 0x6e, 0 });/* Literally the string Pairwise key expansion, with a trailing 0*/

                    if (memcmp(stmac, bssid) &amp;lt; 0)
                    {
                        bw.Write(stmac);
                        bw.Write(bssid);
                    }
                    else
                    {
                        bw.Write(bssid);
                        bw.Write(stmac);
                    }

                    if (memcmp(snonce, anonce) &amp;lt; 0)
                    {
                        bw.Write(snonce);
                        bw.Write(anonce);
                    }
                    else
                    {
                        bw.Write(anonce);
                        bw.Write(snonce);
                    }

                    bw.Write((byte)0); // Will be swapped out on each round in the loop below
                }
            }
                        
            for (byte i = 0; i &amp;lt; 4; i++ )
            {
                pke[99] = i;
                var hmacsha1 = new HMACSHA1(pmk);                
                hmacsha1.ComputeHash(pke);
                hmacsha1.Hash.CopyTo(ptk, i * 20);                
            }
            return ptk;
        }
```

Perhaps the most unexpected part of this algorithm was that the string "Pairwise key expansion" is actually part of it. The airdecap-ng implementation is more efficient in a number of ways, but performance was not my goal here. It takes advantage of the fast memcmp c function in order to compare some of the variables not just for equality, but also "order". This behavior of memcmp is often overlooked; most "ports" of the function to other languages only determine equality. Here is the implementation I used, I'm afraid I've lost the original source:

```cs
for (int i = 0; i &amp;lt; b1.Length; i++)
            {
                if (b1[i] != b2[i])
                {
                    if ((b1[i] &amp;gt;= 0 &amp;amp;&amp;amp; b2[i] &amp;gt;= 0) || (b1[i] &amp;lt; 0 &amp;amp;&amp;amp; b2[i] &amp;lt; 0))
                        return b1[i] - b2[i];
                    if (b1[i] &amp;lt; 0 &amp;amp;&amp;amp; b2[i] &amp;gt;= 0)
                        return 1;
                    if (b2[i] &amp;lt; 0 &amp;amp;&amp;amp; b1[i] &amp;gt;= 0)
                        return -1;
                }
            }
            return 0;
          }
```

Part 1:
1. Construct a 100 element array beginning with the null terminated ascii string "Pairwise key expansion".
2. Next, write the following pairs, with the smaller of each element first:
(stmac, bssid)
(snonce, anonce)
3. Finally, write an additional 0 byte, which will be swapped out in the next part of the algorithm.

Part 2:
1. Using the HMACSHA1 algorithm initialized with the PMK as data (see the previous post for calculating the PMK), compute the hash of the array. This hash is this first 20 bytes of the PTK.
2. Swap out the last byte of the array you constructed (the 0 byte) for a 1.
3. Repeat the hash in step 1. This is the second 20 bytes of the PTK.
4. Swap out the last byte of the array you constructed (now a 1) for a 2.
5. Repeat the hash in step 1. This is the third 20 bytes of the PTK.
6. Swap out the last byte of the array you constructed (now a 2) for a 3.
7. Repeat the hash in step 1. This is the final 20 bytes of the PTK.

The parameters for the PTK calculation consist of the the PMK, which we already know how to calculate, the station MAC and bssid, which are obtained easily enough, and the snonce and the anonce. How do we get those nonces? From the EAPOL four-way handshake. This is one of the aspects of WPA2 that is more secure than WEP: Traffic cannot be decrypted, even if you know the PSK, without also capturing the authentication handshake to obtain the nonces.

As part of this adventure, I've had to add support for Radiotap, 802.1X, and EAPOL to PacketGremlin. I don't think any of that is deserving of a post, but I just wanted to mention it. I've got nearly all the pieces required for decryption in place!
