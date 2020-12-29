---
layout: post
title: Decrypting WEP using C#
categories: [C#,Packet Gremlin]
excerpt: The WPA2 decryption from the earlier posts was challenging to implement. I knew that WEP used a simple RC4 decryption, and didn't think its implementation would warrant a post. And while it's true that it was trivial to implement the decryption, less so was performing the CRC validation.
---

The WPA2 decryption from the earlier posts was challenging to implement. I knew that WEP used a simple RC4 decryption, and didn't think its implementation would warrant a post. And while it's true that it was trivial to implement the decryption, less so was performing the CRC validation. The display of the WEP ICV in Wireshark was very misleading here, because it is *not* the expected CRC. As it turns out, that value is also encrypted, so it must be decrypted along with the data. Here's the implementation:

```cs
public public static bool TryDecryptWEP(IEEE_802_11<Generic> encryptedPacket, byte[] key, out IPacket decrypted)
        {
            
            if (key.Length != 5 && key.Length != 13 && key.Length != 16 && key.Length != 29 && key.Length != 61)
            {                
                decrypted = null;
                return false;
            }

            if (encryptedPacket.FrameType != (int)FrameTypes.Data || !encryptedPacket.IsWep)
            {
                decrypted = null;
                return false;
            }

            var keyWithIV = new byte[key.Length + 3];
            Array.Copy(encryptedPacket.CCMP_WEP_Data, keyWithIV, 3);
            Array.Copy(key, 0, keyWithIV, 3, key.Length);

            var rc4 = new RC4(keyWithIV);
            var encryptedPayload = encryptedPacket.Payload.Concat(BitConverter.GetBytes(ByteOrder.NetworkToHostOrder(encryptedPacket.WEP_ICV))).ToArray();
            var dec = new byte[encryptedPayload.Length];
            rc4.decrypt(encryptedPayload, encryptedPayload.Length, dec);

            var expectedCRC = BitConverter.ToUInt32(dec, dec.Length - 4);
            dec = dec.Take(dec.Length - 4).ToArray();


            if (Crc32.Crc32Algorithm.Compute(dec) == expectedCRC)
            {
                var decryptedBytes = encryptedPacket.Take(26).Concat(dec).ToArray();
                decryptedBytes[1] &= 191; //Everything except protected
                decrypted = Generic.ParseAsNew(decryptedBytes, IEEE_802_11.ParseAsNew);
                return true;
            }
            else
            {
                decrypted = null;
                return false;
            }
        }
```

First, some simple sanity checks - a valid key length, and a data packet encrypted under WEP. Next, we append the key to the IV from the packet. The WEP_ICV is then appended to the data, and decrypted using the IV+Key. The result is our data + expected CRC. We slice off the expected CRC, then compute a standard CRC32 checksum over the allegedly decrypted data. If the calculated CRC matches the expected CRC, then the decryption was successful. As with the WPA2 decryption implementation, we have to fudge the packet header a bit, flipping the protected bit to 0 and excluding the encryption information. PacketGremlin can now handle WEP and WPA2/AES, only WPA/TKIP remains.