---
layout: post
title: Decrypting CCMP using C#
categories: [C#,Packet Gremlin]
excerpt: The final step in the WiFi decryption saga is to deal with Counter Mode Cipher Block Chaining Message Authentication Code Protocol, commonly abbreviated to CCMP. It is the encryption standard used by WPA2, and requires everything done up to this point to generate the temporal key used as input.
---

The final step in the WiFi decryption saga is to deal with Counter Mode Cipher Block Chaining Message Authentication Code Protocol, commonly abbreviated to CCMP. It is the encryption standard used by WPA2, and requires everything done up to this point to generate the temporal key used as input. Here is the code.

```cs
public static bool TryDecryptCCMP(IEEE_802_11 encryptedPacket, byte[] temporalKey, bool strip80211Header, out IPacket decrypted)
        {
            if (temporalKey.Length != 16) throw new ArgumentException("temporalKey must be 16 bytes");
            var decryptedBytes = encryptedPacket.ToArray();
            int z, data_len, blocks, last, offset;
            bool is_a4, is_qos;

            var B0 = new byte[16];
            var B = new byte[16];
            var MIC = new byte[16];
            var PacketNumber = new byte[6];
            var AdditionalAuthData = new byte[32];

            is_a4 = (decryptedBytes[1] & 3) == 3;
            is_qos = (decryptedBytes[0] & 0x8C) == 0x88;
            z = 24 + 6 * (is_a4 ? 1 : 0);
            z += 2 * (is_qos ? 1 : 0);

            PacketNumber[0] = decryptedBytes[z + 7];
            PacketNumber[1] = decryptedBytes[z + 6];
            PacketNumber[2] = decryptedBytes[z + 5];
            PacketNumber[3] = decryptedBytes[z + 4];
            PacketNumber[4] = decryptedBytes[z + 1];
            PacketNumber[5] = decryptedBytes[z + 0];

            data_len = decryptedBytes.Length - z - 8 - 8;

            B0[0] = 0x59;
            B0[1] = 0;
            Array.Copy(decryptedBytes, 10, B0, 2, 6);
            Array.Copy(PacketNumber, 0, B0, 8, 6);
            B0[14] = (byte)((data_len >> 8) & 0xFF);
            B0[15] = (byte)(data_len & 0xFF);

            AdditionalAuthData[2] = (byte)(decryptedBytes[0] & 0x8F);
            AdditionalAuthData[3] = (byte)(decryptedBytes[1] & 0xC7);
            Array.Copy(decryptedBytes, 4, AdditionalAuthData, 4, 3 * 6);
            AdditionalAuthData[22] = (byte)(decryptedBytes[22] & 0x0F);

            if (is_a4)
            {
                Array.Copy(decryptedBytes, 24, AdditionalAuthData, 24, 6);

                if (is_qos)
                {
                    AdditionalAuthData[30] = (byte)(decryptedBytes[z - 2] & 0x0F);
                    AdditionalAuthData[31] = 0;
                    B0[1] = AdditionalAuthData[30];
                    AdditionalAuthData[1] = 22 + 2 + 6;
                }
                else
                {
                    AdditionalAuthData[30] = 0;
                    AdditionalAuthData[31] = 0;

                    B0[1] = 0;
                    AdditionalAuthData[1] = 22 + 6;
                }
            }
            else
            {
                if (is_qos)
                {
                    AdditionalAuthData[24] = (byte)(decryptedBytes[z - 2] & 0x0F);
                    AdditionalAuthData[25] = 0;
                    B0[1] = AdditionalAuthData[24];
                    AdditionalAuthData[1] = 22 + 2;
                }
                else
                {
                    AdditionalAuthData[24] = 0;
                    AdditionalAuthData[25] = 0;

                    B0[1] = 0;
                    AdditionalAuthData[1] = 22;
                }
            }

            using (RijndaelManaged aesFactory = new RijndaelManaged())
            {
                aesFactory.Mode = CipherMode.ECB;
                aesFactory.Key = temporalKey;
                ICryptoTransform aes = aesFactory.CreateEncryptor();

                aes.TransformBlock(B0, 0, B0.Length, MIC, 0);
                XOR(MIC, 0, AdditionalAuthData, 16);
                aes.TransformBlock(MIC, 0, MIC.Length, MIC, 0);
                XOR(MIC, 0, AdditionalAuthData.Skip(16).ToArray(), 16);
                aes.TransformBlock(MIC, 0, MIC.Length, MIC, 0);

                B0[0] &= 0x07;
                B0[14] = 0;
                B0[15] = 0;
                aes.TransformBlock(B0, 0, B0.Length, B, 0);
                XOR(decryptedBytes, decryptedBytes.Length - 8, B, 8);

                blocks = (data_len + 16 - 1) / 16;
                last = data_len % 16;
                offset = z + 8;

                for (int i = 1; i <= blocks; i++)
                {
                    var n = (last > 0 && i == blocks) ? last : 16;

                    B0[14] = (byte)((i >> 8) & 0xFF);
                    B0[15] = (byte)(i & 0xFF);

                    aes.TransformBlock(B0, 0, B0.Length, B, 0);
                    XOR(decryptedBytes, offset, B, n);
                    XOR(MIC, 0, decryptedBytes.Skip(offset).ToArray(), n);
                    aes.TransformBlock(MIC, 0, MIC.Length, MIC, 0);

                    offset += n;
                }
            }
            if (strip80211Header)
            {
                decryptedBytes[1] &= 191; // Remove the protected bit
                var decryptedBytesList = decryptedBytes.ToList();
                decryptedBytesList.RemoveRange(34 - 8, 8); // Remove CCMP Parameters (otherwise, without the protected bit it will break the parsing)
                decrypted = Generic.ParseAsNew(decryptedBytesList, IEEE_802_11.ParseAsNew).Layers().ElementAt(1); // Remove the altered 802.11 header
            }
            else
            {
                decrypted = Generic.ParseAsNew(decryptedBytes, IEEE_802_11.ParseAsNew);
            }

            return memcmp(decryptedBytes.Skip(offset).Take(8).ToArray(), MIC.Take(8).ToArray()) == 0;

        }
```

This is nearly a direct port from airdecap-ng, except for the actual cryptography which of course uses the managed equivalents. First let's get the most obvious parameters out of the way: We need an 802.11 packet to decrypt; the "encryptedPacket" parameter. As before, this isn't the entirety of the frame, which might have radiotap or other things attached, it's just the 802.11 layer and above. We also need a place to put the decrypted result, that's the "decrypted" parameter.

The "strip80211Header" option is one that I'm not too happy with. When the decryption is complete, you're left with the 802.11 header essentially untouched, which indicates that the contents is encrypted. I need to be able to return the full packet that was inputted for verification purposes, but beyond that the more useful output is the actual decrypted contents. So this parameter allows the consumer of the function to choose whether the output will be at the 802.11 layer, or at the decrypted protocol layer.

The temporalKey parameter is of course the most interesting. The Temporal Key is 16 bytes long, 32 bytes into the Pairwise Transit Key (PTK). I discussed how to get the PTK in a previous post.

I won't go into detail about the implementation of this method, as it's largely just data manipulation, but I'll discuss it at a high level. We start by moving bytes around to come up with the B0 and AdditionalAuthData arrays. Next, we prepare an AES encryptor in ECB mode. This mode is important, because normally you'd need to provide an Initialization Vector as well as a key to AES. We use the encryptor to come up with the MIC which we can use to validate our results, and also to perform the actual decryption of the data (which is done not all at once, but in blocks). As a final processing step, remove the 802.11 header if requested, and parse the resulting decrypted packet data. Lastly, we use the MIC we calculated to verify if the decryption was successful.

I was able to put together the PMK, PTK, and CCMP functions that I've gone over in this series to successfully decrypt my WPA2 test capture. Before I can call Packet Gremlin's WiFi decryption support complete, I'll need to also handle TKIP and WEP. There's a lot of overlap between these two; they both use the RC4 encryption algorithm, so it shouldn't take much time to implement. Later, I will post the test captures I've used, as well as links to many of the resources that were helpful in coming up with these implementations.