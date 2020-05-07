---
layout: post
title: How HSBC has become deranged by security
---

Today, I finally received my instructions from [TfL](https://tfl.gov.uk/) to refund my Oyster card balance facilitated by HSBC. The message from TfL was clear and fine, giving a reference number and advising me to watch for a separate e-mail from HSBC.

When the [e-mail from HSBC](/images/HSBC instructions.pdf) came, however, it had many hallmarks of a scam. In particular:

- The "From:" line has just the e-mail address with no legal entity name.
- The syntax was awkward, with the reference number appearing on the same line as "Dear &lt;recipient>,"
- The wording of the reference number was awkward, stating "ending in", yet presenting a full number masked by asterixes.
- The spelling of the service, "client benef service", is reminiscent of techniques a phishing attack will use to get a spelling similar to something legitimate.
- The link is a URL with backslashes (instead of forward slashes as the URL spec demands).
  - In my nearly 30 years on the Internet, I've never seen illegitimate characters used in a URL.
  - This techinque relies on obscure features (and web site features) to correct the slashes.
  - Again this technique smells like a phish, a hack that a scammer would use to bypass security features for URL scanning and checking.
  - This hack requires special instructions for iOS users, and I confirmed - on Safari and Firefox on iOS, the backslashes must be manually replaced with forward slashes. Honestly, I wouldn't expect _anybody_ to go through the trouble of substituting all six of those backslashes on an iOS device.

In any case, the last three digits of the reference number matched the digits from TfL and I was expecting the e-mail and I checked the SSL certificate and it was issued to HSBC by Digicert, so it seemed safe to proceed. Yet, when I got to the website, I ran into many more issues that encumbered the process and reduced my confidence that this was a legitimate endeavor.

- SSL cert was not for `*.hsbc.com`, but for `clientbenefservice.hsbc.com`. That's not a huge deal, because the proper HSBC name was still in the domain. Yet, if I wasn't familiar with the name HSBC, this domain would be as suspicious as any, and by having a cert just for this specific, strangely-spelled name is the kind of thing that might fly under the radar of a certificate issuer.
- To be as accurate as possible, I copied the reference number and attempted to paste it into the form, but the form disallows pasting during registration. So I instead worked around it by pasting the number into the e-mail field, hand-copying it into the registration number field, then completing my e-mail correctly (again).
- Once logging in, while entering my contact information (mobile number), I was met by [glitchy controls](https://www.dropbox.com/s/ulbyx7qz7qemt4y/2020-05-07%20HSBC%20glitch.mov?dl=0). It was impossible to select a country code from the list and the dialog was repeatedly flashing.
- In an attempt to work around the glitchy form, I refreshed the page, which reset the process back to the login, and required me to use the manual-copy-paste workaround again.
- While completing the form the second time, I found I could type "UU" (or "US") to get the relevant country code, but tabbing between fields did not work. Only some of the form fields were reachable by tabs.
- I completed the form, but after submitting, I was returned to the login page again with an obscure "Authentication Failure" error message. Had I mistyped the reference number? There's no way to know because they mask the number.
- Like an insane person, I try the same thing a third time. This time, the glitches in the webiste subside and I'm able to complete the form.
- I'm prompted to do "mobile number verification", which they send through e-mail as well as mobile.
- The button to proceed awkwardly reads "Proceed Payment".
- The button to "validate" the bank details requires completing the section _after_ bank details form (and the validate button), requiring the user to continue down the page, then return back to the validate button in an earlier section.
- During the (what should have been two-minute) process, I was required to validate twice, when once would surely have been sufficient.
- Even though they required a mobile phone number, I never had to validate through it. Presumably any phone number would have sufficed.
- Each time a mobile validation was sent (and a confirmation later sent), it came from a different number.

So ultimately, it sounds as if this process is legitimate, but it's no surprise to me why if TfL was using this same system to issue refunds, they struggled so much to issue me a refund for months. This process was horrible and sketchy, and I'm glad it's over.
