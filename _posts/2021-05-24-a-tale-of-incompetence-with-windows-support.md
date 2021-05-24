---
layout: post
title: A Tale of Incompetence with Windows Support
---

Today, a strange thing happened. My partner and I returned from lunch when she resumed her Surface Studio running Windows 10 19042.985. Upon doing so, two Windows Defender alerts popped up, indicating that features were blocked for two processes, `taskhostw` and `sihost`, both in `\Windows\System32`.

Unsure whether to allow or cancel, I joined in on the investigation. I did some web searches to see if I could understand if these alerts were spurious or legitimate, but advice on the Internet is all over the place.

For example, here's [someone with the exact same problem](https://answers.microsoft.com/en-us/windows/forum/all/windows-defender-firewall-has-blocked-some/fc986caa-2c2d-4cba-99d8-4b3e59a25aac) and the Microsoft Agent answer there doesn't provide any meaningful explanation nor do they answer the question the OP asked, but they do suggest to scan the system for viruses, suggesting that the alert is not spurious but is in fact a legitimate exploit.

Unable to find any good information online, I recommended to call Microsoft Support. I remember years ago Microsoft was making a point to provide top-tier support for security issues.

After some fussing around the Internet and trial and error, we found our way to Windows support and scheduled a call back. After being assigned a case number 1023094695 and being transferred to technical support, we were connected with Rohit S., who verified our identity and then connected to the computer through Quick Assist. "Great!," I thought, an experienced support agent will take a peek and make a quick assessment and maybe I can ask some more questions about why this should happen at all.

Instead, Rohit rushes to do two things. First, he creates a restore point and explains this is to make it easy to recover if any changes cause a problem. Next, he rushes over to Bitlocker Encryption and *begins the process to decrypt the hard drive*!!!

WTF?!? I ask why he did that, and he asks me if I meant to have Bitlocker enabled and if I have the recovery key. I'm unsure if or where the recovery key is stored, but I explain that the drive is encrypted to ensure confidentiality if the device is stolen. Of course, by now it's too late. The computer is already decrypting the drive, a process that will take several hours. And the restore point is no help in this situation.

Rohit then moves on from his destruction spree to focus on the question we posed when we called, but his answers instill little confidence. He explains (a couple of times) that some times installing a printer driver can cause these alerts. We explain a few times that we haven't done anything with the computer but resume it from sleep. He chooses to "allow" these features on these processes, but doesn't choose to allow them for Public networks, with no explanation given. He clearly doesn't know the cause of these alerts or if they're legitimate or not, but he's happy to say there's no risk and to accept the risk on our behalf.

"Can I help with anything else?"

"No, you've done enough."

We decide to disconnect and to deal with the Bitlocker fallout on our own.

After disconnecting, we feel angry and violated. We're now stuck waiting hours for the drive to decrypt and then hours again to wait for the drive to encrypt. So we decide to call back and speak to a supervisor to complain about the experience.

In the first attempt to call back, we reach Jannat, but shortly into the call explaining the issue, we get feedback, silence, and then the call disconnects.

So we call again. This time, Joseph listens to our concern and then transfers us into technical support to speak to a supervisor, but we get Jannat again. This time, she assigns a new case number 1023096227. She listens to our concern and we request to speak to a supervisor. She offers to schedule a callback from a supervisor and asks for 2-3 minutes to schedule a call. Several minutes pass and she returns saying there is no schedule for a supervisor. She advises to send a complaint through the Feedback Hub, but I'm about 99% confident that any complaint I send through there would go into a black hole. Although I plea, she refuses to connect me with anyone else or to help me connect to a supervisor. She says because it's a Work From Home situation, "no supervisor schedule is available".

Obviously, she's caught in a lie. I call her out on her lie, but it makes no difference. My only option is to call again tomorrow and try again. It seems she's paid to be an impediment to accountability and nothing more. After being on the phone for almost an hour (and well over 90 minutes total), I ask Jannat to send me an e-mail stating there's nothing they can do to help and she agrees, but nothing ever arrived.

I honestly don't expect to receive anything of substance from Micrsoft on this matter. The damage is done. The machine will spend the next 2+ hours decrypting the drive and we'll spend some time re-enabling Bitlocker. Our productivity for the afternoon is shot. Microsoft's "security" has created a problem where there is none and in a way that's incomprehensible even to trained experts.

If Windows Defender was released last month, this kind of experience might be excusable. Instead, it's been around for over a decade, so spurious alerts about Windows' own processes should not be presented to the user. Also, support agents like Rohit S. should be trained not to rapidly enact irreversible operations during a remote assistance session without consulting with the user and should be fired if they do. Finally, agents like Jannat should be empowered to help customers address complaints of damage and abuse and should be fired if they refuse to do so.

I'm not sure it's worth all this grief just to be on a platform that has a touch screen. If I'd had any idea how badly this all could go, I would have recorded the whole ordeal for entertainment value.
