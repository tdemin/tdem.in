---
title: "Recoverable 2FA Token HOWTO"
date: 2017-10-27T12:00:00+00:00
draft: false
---

![](https://69.media.tumblr.com/2dfce6565e1af513ef972256171edd2b/tumblr_inline_pd7rqjIC7d1vumr7z_540.png)<p style="font-size:8pt;text-align:center;">_Image credit: [Auth0](https://auth0.com/learn/two-factor-authentication/)_</p>

So you want to enable two-factor authentication for your online accounts? That’s fairly easy, made relatively painless by Google with its [Authenticator](https://support.google.com/accounts/answer/1066447?co=GENIE.Platform%3DAndroid&amp;amp;hl=en). Well, enabling 2FA isn’t the problem. There are other caveats to watch out for.

First of all, _you can’t back up your token_. There isn’t an option for moving your keys to another device, your only option is to use so-called recovery codes that are meant to be entered if the token is lost. Google suggests to write those codes somewhere in a safe place, and most of the other services who offer 2FA do. But well, this means storing those codes in an insecure way.

Another problem is directly pointed out from the first one: _you can’t generate codes with multiple devices_. Changing device in most 2FA systems means your previous phone will no longer generate codes that work.

And last, but not the least: _you can’t get your private keys back_ after they’re saved on your device. Neither you can edit them. Yes, this is a security measure, but what a headache this actually is!

Some 2FA apps realizing these constraints make 2FA much harder to enforce tried to address those issues. At least [Authy](https://authy.com/) allows for multiple code generators, and it also makes backups of the keys in the cloud, so you can easily restore those on your new device. The thing is, storing keys in cloud negates security, as not only someone has your codes, but this also introduces another attack vector. What if that cloud is hacked?

This is exactly what I’ve tried to overcome.

***

So, let’s proceed to the practical part. We want all of those:

*   multiple devices that generate codes

*   private key backups

*   easy restoration of the codes on new devices

*   adding the keys to the device with scanning the code

And we also don’t want to harm our accounts’ security.

The only sustainable way of doing this is storing the private keys in a secure place. With your PC, this means creating an encrypted container that would hold our private key material. I shall not explain the whole process of creating the container, I’ll just point you to [VeraCrypt](https://www.veracrypt.fr/en/Home.html), an excellent program for doing exactly that. Probably the only recommendation to consider: your container doesn’t need to be that big, 2MB should be enough so far as we’ll just store a plain text file with keys in it.

Next, we’ll enable two-factor authentication, just in a little bit different way. Don’t scan the QR code, select “I can’t scan the code” or whatever and then copy the code to the text file placed in your container.
Now we’ll do some magic on the code. The QR codes made to scan with your 2FA app are not generated with the super cow powers, they are just modified a little so your device instantly recognizes the name of your 2FA provider and the options to use. We’ll just form the codes we can scan using the same [algorithm](https://github.com/google/google-authenticator/wiki/Key-Uri-Format). Just make the strings containing your keys look like this:

`otpauth://totp/SiteName:AccountName?secret=PRIVATEKEY`

And ensure that the string contains no spaces. The `:AccountName` part is optional and can be omitted. Do this with all your codes.

Now we have the codes, time to generate the QR codes that can be scanned! Get any free app that generates QR codes (I use [this one](https://code.google.com/archive/p/qrencode-win32/)) and generate the codes from the strings you’ve resulted with in the previous step. Scan them one-by-one with Google Authenticator.

You’re done! You now have your private keys that are backed up in a secure manner. You can also add those to another app or device if you want. And, of course, you can add them without hassling with entering the keys by hand. All of these without having your accounts compromised.
