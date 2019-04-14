---
title: "Telegram and Secure Messaging"
date: 2017-07-05T12:00:00+00:00
draft: false
---

Telegram, as it states on its home page, is a secure messenger. Over the years it became a simple, powerful multi-device messenger which is selected as the main messaging platform by many people (the official Android app has more than [100 million installs](https://play.google.com/store/apps/details?id=org.telegram.messenger), for example).

Many people say Telegram really is a secure messenger. They trust Pavel Durov, its creator, they get overwhelmed with its promotion, they get obsessed. I will try to dissolve Telegram as a secure messenger suitable for private messaging from the end user's point of view.

So, let's start with the main promotional Telegram's feature which was heavily advertised—the secret chats.

## Secret chats

Secret chats is the way Telegram does end-to-end encryption on your messages. They are said to leave no trace on Telegram's servers and to be not available to them. This is the recommended way to message securely on Telegram.

However, they are pretty unconvenient, as you have to start them by hand. They also have a few problems that make their usage painful.

_Secret chats don't work on multiple devices_. According to [Telegram FAQ](https://telegram.org/faq#q-how-are-secret-chats-different "How are Secret Chats Different?"), the secret chats are device-specific and can be opened only from device they were created from. We now can already conclude the secret chats are not scalable (or why would someone limit their usage to only one device?), so are not suitable for convenient messaging.

> All secret chats in Telegram are device-specific and are not part of the Telegram cloud. This means you can only access messages in a secret chat from their device of origin.

Messengers really caring about end-to-end encryption of its clients should provide the way to somehow synchronize encrypted chats, or such feature would not be used by masses. For example, Signal Protocol, the cryptographic protocol used by Signal, does scale, so as its port to XMPP, [OMEMO](https://xmpp.org/extensions/xep-0384.html "XEP-0384: OMEMO Encryption"), and [Proteus](https://github.com/wireapp/proteus), the port made for Wire messenger. They achieve multi-device encryption by encrypting the message separately for every recipient device.

_Secret chats don't allow you to start chats at any time_. You'll have to wait for the recipient to become online to send him encrypted messages on Telegram, when you don't need this if using protocols designed to provide offline messages delivery, like [Signal Protocol](https://en.wikipedia.org/wiki/Signal_Protocol), which was already mentioned above.

Telegram's secret chats are like OTR in XMPP—an old encryption protocol that doesn't allow you to neither synchronize messages nor send messages to offline devices.

> OTR has significant usability drawbacks for inter-client mobility. As OTR sessions exist between exactly two clients, the chat history will not be synchronized across other clients of the involved parties. Furthermore, OTR chats are only possible if both participants are currently online, due to how the rolling key agreement scheme of OTR works.

_Introduction to XEP-0384, the motivation to replace OTR_

_Secret chats don't work on Telegram Desktop_. Here comes another problem of the secret chats — they are only available on the mobile phones (and on the Telegram client from App Store). Telegram Desktop, the official Telegram app for Windows, macOS and Linux, doesn't support secret chats, while this feature was first requested at [June 1, 2014](https://github.com/telegramdesktop/tdesktop/issues/5). There were many requests since then ([1](https://github.com/telegramdesktop/tdesktop/issues/118), [2](https://github.com/telegramdesktop/tdesktop/issues/619), [3](https://github.com/telegramdesktop/tdesktop/issues/871), etc), but secret chats are still unimplemented as of Jule 5 in the official desktop app of the messenger which claims itself secure.

_Secret chats don't work in group chats_. You can't end-to-end encrypt Telegram group chats at all, that's it. If you want to chat with many people at once securely... you can't, forget it. The E2EE protocol of Telegram doesn't allow you to group chat securely, when modern protocols, like Signal Protocol, actually allow you to, by the technique already stated above—encrypting messages to every recipient separately.

_Secret chats just aren't usable_. The whole secret chats thing just does it the wrong way. As Telegram's secret chats are not really suitable for multi-device environment and do not allow you to use them in group chats, and you have to force them by hand, you probably won't use them at all, therefore your messages will be easily available to Telegram.

## MTProto security

Any security protocol should be independently audited to be trusted on being secure. MTProto, the homegrown encryption protocol used by Telegram to encrypt messages, wasn't audited, but instead a [challenge was posted](https://telegram.org/blog/cryptocontest "$300000 for Cracking Telegram Encryption") by Pavel Durov to crack the secret chat he started with his brother.

The lack of winners noted by Pavel Durov is the thing he mostly admires when claiming the protocol security. Why is a marketing promotional better than an independent expert audit which would state things better? We only have to guess and hope the protocol is really secure.
![image](https://69.media.tumblr.com/f7b8134db0d7e4c3c24b0d2abe18b248/tumblr_inline_p1m4e1wF0q1vumr7z_540.png)

_[xkcd #153](https://xkcd.com/153/)_

## Spreading fear and doubt

Pavel Durov recently [stated](https://twitter.com/durov/status/872891017418113024 "The encryption of Signal (=WhatsApp, FB) was funded by the US Government. I predict a backdoor will be found there within 5 years from now.") the encryption of Signal was funded by US government, so the backdoor is predicted to be found in 5 years.
> The encryption of Signal (=WhatsApp, FB) was funded by the US Government. I predict a backdoor will be found there within 5 years from now.

The development of a new crypto is a hard task to accomplish. The cryptoexperts funded by the governments know there shouldn't be any backdoors, as any security tamper used by "good guys" can eventually be used by "bad guys".

If the protocol is sound from the bad guys, it is safe and sound from anyone, so predicting the protocol to be found vulnerable in just 5 years sounds more like spreading [FUD](https://en.wikipedia.org/wiki/Fear,_uncertainty_and_doubt "Fear, uncertainty and doubt") among people to help promoting Telegram. Not only it doesn't look like a fair play, it's a very dangerous thing to do as people trusting Durov probably will just abandon on things after such statements.
![image](https://69.media.tumblr.com/06f1a00577ef9417f633f8d9540c011f/tumblr_inline_p1m4g3kaD11vumr7z_540.png)

[_xkcd #1820_](https://xkcd.com/1820/)

Undermining the competitors isn't a good thing to do. Undermining them with spreading FUD among people is even worse.

## Source Code

Telegram is a program with sources published under the [GNU General Public License v2](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)&nbsp;[1]. Yet the only apps whose recent code is always available are Telegram Desktop and Telegram for Web while the mobile apps’ code is updated irregularly ([Android](https://github.com/DrKLO/Telegram/commits/master), [iOS](https://github.com/peter-iakovlev/Telegram/commits/public)).

The repos linked above face occasional updates with the interval close to 6 months or something about that. This is a rare case in the open source world, where the development almost always goes social, so you can see any interaction happened with the app’s code. But Telegram’s mobile apps development goes behind closed curtains: no pull requests are accepted, no issues are closed, commits do not link to individual changes, but instead consist of huge changesets pushed at once.

What prevents the messenger’s team from going fully open source as others already do? No idea. Still the reason to watch out.

## Summary

Let's summarize the things we've learned:

*   Telegram uses its own cryptographic protocol, which hasn't been audited by third-parties, but instead was starred in an advertisement promotional;
*   Telegram's secret chats are not scalable, do not allow to sync messages and do not apply to group chats;
*   there isn't an implementation of Telegram's end-to-end encryption in the official desktop client;
*   there isn't any proof the Telegram servers do not spoof on messages except for its creator's statements;
*   Telegram's creator makes FUD statements to promote his own service and undermine the competitors;
*   the code for the mobile apps is updated irregularly, no one can tell why.

If you're going to use Telegram for secure messaging, you'll have to trust it on being secure, and there are many signs you shouldn't. So, why don't you make a change then? There are already a few secure messengers on the market that allow you to chat securely while handling things like multiple devices much better than Telegram does, like [Wire](https://wireapp.com) or [Signal](https://whispersystems.org).

* * *

1: The apps licensed under the GNU GPLv2 are [Telegram’s mobile apps](https://telegram.org/apps). Telegram for Web and Telegram Desktop are licensed under the GNU GPLv3.

