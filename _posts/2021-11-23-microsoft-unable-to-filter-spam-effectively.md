---
layout: post
title: Microsoft unable to filter spam effectively
---

I've hosted the jaraco.com email on Microsoft Exchange since 2000 when I ran my own Exchange server on a Comcast internet connection (before they capped upload speeds and blocked ports capriciously). Sometime around 2010, I moved my domain to "Microsoft Online" (hosted Exchange). And for a long time, this service was great.

But in late 2019, something changed. Regular email messages started showing up in the "Junk Email" folder, including commercial messages that I'd received for years, responses to messages I'd initiated, and nearly every message from any sender from whom I'd not previously received a message.

In practice, the false positive rate is 3-5%.

The problem then, is that I'm forced, on a regular basis (since Junk Email is automatically deleted every 30 days), to review _every single spam message_ and manually move the false positives back to the Inbox.

I suspect what happened is they changed their spam filtering engine to some ML-based solution (or tweaked their ML model).

I lived with it for a few months, then contacted Microsoft support. I spent hours trying to explain the problem to the support agent, but ultimately, they were unable to provide a solution. They recommended lots of ineffective tweaks.

Frustrated, I lived with it for another year and in early 2021, contacted Microsoft support again about it, and again, they were ineffective. The agent was unable to provide a verifiable reason for the false positives, recommended tweaks to the sensitivity that would increase the false positives, and ultimately could not provide any help. I tried to direct the support rep through the factors, pointing out that the "spam confidence level" was lower than the configured threshold, but the agent was unable to provide any justification for the undesirable behavior.

I continue to expend the manual effort to regularly scan my Junk Email and move messages. I use a technique where I mark the Junk Email as read after scanning, so I can know where to stop next time. I spend approximately one hour per month managing this issue, an investment I did not have to make before late 2019. I keep diligently "reporting" each of these false positives, but for all the training, the artificial intelligence seems to have learned nothing.

For example, here's a message that was recently filtered as junk:

![message from self to list](/images/Screen Shot 2021-11-23 at 09.59.49.png)

It's a message from myself to a mailing list in a response to a message that I'd previously received, a message that was not filtered. I received 90% of the messages on that chain, but \~10% were filtered as spam.

Here's another example, where Microsoft flagged as spam a tracking update from FedEx:

![tracking update from FedEx](/images/bad-spam-filter-2022-05.png)

This week, Microsoft flagged a message _from myself_, _sent through Exchange_ to a list as spam:

![message from self is spam](/images/Screen Shot 2022-06-23 at 12.16.34)

In this case, Microsoft allowed the two status messages that arrived before and after the indicated spam message but inexplicably marked one of them as spam, even though they came from the same sender about the same subject with almost identical content. Clearly deranged behavior.

I'll continue to update this post with the most egregious failures, but since they've been unsuccessful in improving the system for over two years, I'm not holding out hope.