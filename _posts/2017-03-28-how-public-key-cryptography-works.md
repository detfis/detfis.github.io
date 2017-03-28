---
layout: dsp_post
title: How public key cryptography works
categories: dajsiepoznac2017 cryptography
description: Simple explanation how public key cryptography works
---

# Public key cryptography is simple #

In this blog post I want to achieve one thing -  I want to explain in plain English how public key cryptography works.

Couple years ago when asymmetric cryptography was introduced to me I was a little discouraged because I thought that this is very complex and hard to grasp. It turned out that general idea behind it is dead simple and would like to bring it closer to you. 

## Traditional cryptography ##

When we think about how to encrypt some message the first solution that comes to our mind something called *symmetric cryptography*. 

Symmetric cryptography is the simplest and most naive way of encrypting messages. Let's say we want to encrypt a word "HELLO".
To do so we need a *secret key* that will transform our message to something that looks like a random string, let's say "QWERT". To decrypt the message we have to use the same secret key. The process looks like this:

```
HELLO -key-> QWERT -key-> HELLO
```

An example of this symmetric cryptography is [link]Cesear Code.

This way of encrypting data entails some problems:
- both parties need to have access to the same secret key
- how to share this key to one another? 
- we need a key to establish a secure connection to share a key, but we can't do it without a key... vicious circle...

## Asymmetric cryptography to the rescue! ##

Asymmetric cryptography addresses this problem by requiring not one, but two encryption keys, know as key-pair.

They are generated in that way that you can't guess one key from the other and they're linked in such way that anything encrypted with keyA can be decrypted with keyB (and the other way around is also true - the message encrypted with keyB can be decrypted with keyA).

```
HELLO -keyA-> QWERT -keyB-> HELLO
```

Having a key-pair, you pick one to be a public key. You can then publish it anywhere - at the end of your emails, on your web page, upload it to the special key management servers etc, the idea is it's now publicly available with your name on it.

Second key is your private key and this one you have to keep secret.

## How to use public key encryption? ##

Now imagine you want to send me an encrypted message - we both have a key-pairs and we both have another's public keys. Now you don't have to share anything with me. You know my public key, so you encrypt the message with my public key, you send the encrypted message to me and you know I can encrypt it because I have a private key. 

There is also the second use case for this: I can encrypt something with my private key and send it. You may start thinking why would anyone do that since my public key is out there and anyone can decrypt it? But the fact that it can be decrypted with my public key indicates that it has been encrypted with my private key, which means that I'm the author of the message because only I have the private key. So you can be certain that this is an authentic message. 

The best thing is to do both: I encrypt something with my private key and then your public key and then I send it to you. Now we both know that nobody else can read the message, you know that the message comes from me and that the message has not been modified by any of the third parties because otherwise, you won't be able to decrypt the message. This makes a great secure system. 

## Summary ##

I realize that this is a huge oversimplification and there are a lot of details I omitted, but I wanted to show a general idea how it works and that the core concept is quite simple. Hoped you liked it.