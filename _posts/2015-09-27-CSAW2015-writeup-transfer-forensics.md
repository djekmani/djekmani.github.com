---
layout: post
title: "CSAW2015 Writeup -  Transfer"
---

challenge description:

>I was sniffing some web traffic for a while, I think i finally got something interesting. Help me find flag through all these packets.
 [net_756d631588cb0a400cc16d1848a5f0fb.pcap](https://ctf.isis.poly.edu/static/uploads/9816b472715fa536ab95bf43edc10540/net_756d631588cb0a400cc16d1848a5f0fb.pcap) 


I began with analyzing the packets for some fifteen minutes before finding something interesting, a HTTP traffic containing a [Python script](https://gist.github.com/efd480035c9cf5b98e44) and it's [output](https://gist.github.com/58ecf8683b223da7df4e) as shown in the picture below :








![Alt text](/public/images/csaw-transfer1.png "packets analysis")

It seemed interesting, it takes the string: 

`FLAG  = 'flag{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}'` 



and encrypts using the cryptosystem described by the `encode()` function :

{% gist djekmani/ab437586723c14e6b563 %}

After a profound analysis I understood that the function took for each iteration a random encryption (rot13, base64 , ceasar) using `random.choice(enc_cipthers)`.

`enc_ciphers = ['rot13', 'b64e', 'caesar']`  <<< encryption index (0 , 1 , 2)

Then it concatenates the encrypted message with its encryption index +1

This theory is verified by the [output](https://gist.github.com/58ecf8683b223da7df4e) I could extract along with the script :

`2Mk16Sk5iakYxV.....`

It begins with **2**, and hence the rest is *base64* encoded message. By confirming it with an online decoder, we obtain another message which begins with :


`2MzJNbjF1TVhKTlZvWlVZb1ZkZkZad05VTk5Sb281WkVnMFZuMXVWbVJaTUZKWFYydmFWMWd1ZkZrV2VGc1lXblpu...`

Another base64 begin with 2 therefore, my first analysis seems correct. So we only have to write a reverse script for that cryptogram :

{% gist djekmani/86b40c2d887a8506e171 %}

Executing the script and reading the output are the only things left to do.

![Alt text](/public/images/csaw-transfer2.png "getting the flag")


oooops, Mr Flag is right before your eyes.


NB: This is one of the challenges I have found pretty cool, even if I could only solve it the challenge was finished.



