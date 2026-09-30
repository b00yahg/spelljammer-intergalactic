---
publish: true
title: "Part 1: Error Message 3000"
description: A stranger breaks into the crew's chat,
created: 2026-09-30T03:29:35.888Z
modified: 2026-09-30T03:29:35.888Z
published: 2026-09-30T03:29:35.888Z
---

It started with misterious channel in the discord. Anything the crew posted to the channel bounced back as invalid data, stamped with the same name every time. Then a stranger logged on and started talking in code.

Want to crack it yourself? The answers are folded up below the log.

> [!terminal] MS-DOS Prompt - JULIUS.LOG
> `--- Wednesday, October 2, 2024 ---`
>
> `[19:30]` **\<Faisal>** gotta take this one for a spin on hinge
> `[19:31]` ==\[ɪɴᴠᴀʟɪᴅ ᴅᴀᴛᴀ] ERROR MESSAGE 3000: JULIUS==
> `[21:04:17]` _-!- Error Message 3000: Julius has been reviewed_
> `[21:04:32]` _-!- \<UNKNOWN\_USER> has entered the chat_
> `[21:04:45]` _**\<UNKNOWN\_USER> Khoor.**_
> `[21:05:01]` _**\<UNKNOWN\_USER> Wr hqwhu wkh odbhu, vdb brx zlvk wr mrlq.**_
> `[21:05:18]` _**\<UNKNOWN\_USER> Vhh brx vrrq.**_
> `[21:05:30]` _-!- \<UNKNOWN\_USER> has left the chat_
> `[23:44]` **\<Faisal>** Insane that boo has better spelling when sending alien gibberish
> `[23:45]` ==\[ɪɴᴠᴀʟɪᴅ ᴅᴀᴛᴀ] ERROR MESSAGE 3000: JULIUS==
> `[23:45]` ==\[ɪɴᴠᴀʟɪᴅ ᴅᴀᴛᴀ] ERROR MESSAGE 3000: JULIUS==
>
> `--- Thursday, October 3, 2024 ---`
>
> `[01:50]` **\<Shmamy>** ... will this be on the test?
> `[01:50]` **\<Shmamy>** Are we in the ARG right now? It's in the room with us?
> `[01:51]` **\<Clio>** This is the ARG, yes
> `[01:51]` **\<Shmamy>** Holy shit I am so excited
> `[01:52]` **\<Clio>** (Well, I assume it is—but Boo called the last ARG t̴̅̈ḣ̸͛e̷̟͆-̸̂̚d̵̾͘á̷̈r̶͒k-web too, so :P)
> `[01:52]` **\<Clio>** Completely different puzzle tho, so I'm stumped
> `[10:12]` ==\[ɪɴᴠᴀʟɪᴅ ᴅᴀᴛᴀ] ERROR MESSAGE 3000: JULIUS==
> `[10:12]` ==\[ɪɴᴠᴀʟɪᴅ ᴅᴀᴛᴀ] ERROR MESSAGE 3000: JULIUS==
> `[10:12]` ==\[ɪɴᴠᴀʟɪᴅ ᴅᴀᴛᴀ] ERROR MESSAGE 3000: JULIUS (But still epic)==
> `[10:15:03]` _-!- \<UNKNOWN\_USER> has entered the chat_
> `[10:15:17]` _**\<UNKNOWN\_USER> Hyl fvb svvrpun av lualy aol slaaly aolu?**_
> `[10:15:32]` _**\<UNKNOWN\_USER> Flho, aol pucpahapvu…**_
> `[10:15:48]` _**\<UNKNOWN\_USER> Fvb qbza ohcl av hzr mvy luayf...**_
> `[10:16:02]` _**\<UNKNOWN\_USER> pu tf shunbhnl vm jvbyzl.**_
> `[10:16:15]` _-!- \<UNKNOWN\_USER> has left the chat_
> `[10:31]` **\<Shmamy>** This reeks of some deep speech to me
> `[10:17:42]` _-!- \<UNKNOWN\_USER> has entered the chat_
> `[10:17:55]` _**\<UNKNOWN\_USER> ... uv**_
> `[10:18:03]` _-!- \<UNKNOWN\_USER> has left the chat_
>
> `C:\DARKWEB>`

> [!decode]- Decoded messages
> `Khoor.` → Hello.
> `Wr hqwhu wkh odbhu, vdb brx zlvk wr mrlq.` → To enter the layer, say you wish to join.
> `Vhh brx vrrq.` → See you soon.
> `Hyl fvb svvrpun av lualy aol slaaly aolu?` → Are you looking to enter the letter then?
> `Flho, aol pucpahapvu…` → Yeah, the invitation…
> `Fvb qbza ohcl av hzr mvy luayf...` → You just have to ask for entry...
> `pu tf shunbhnl vm jvbyzl.` → in my language of course.
> `... uv` → ... no

> [!answer]- The crew's reply
> The key was in the error the whole time. JULIUS points to Julius Caesar and his cipher, which slides every letter along the alphabet. The stranger's first messages shift each letter by three, and the later ones shift by seven.
>
> To get in, the crew had to ask in the stranger's own language. They sent back `Zh zlvk wr mrlq`, which is "We wish to join" shifted by three.
