+++
title = "DNS records against email spoofing"
description = "How to set up appropriate TXT DNS records to prevent a domain from being used in email spoofing"
tags = ["dns"]
+++

A domain that doesn't receive or send emails can be used in [email spoofing][1] attacks.
Domain owners should set a couple of **TXT records** to configure email authentication:

| Name                       | Content                                          |
| -------------------------- | ------------------------------------------------ |
| `example.com`              | `v=spf1 -all`                                    |
| `*._domainkey.example.com` | `v=DKIM1; p=`                                    |
| `_dmarc.example.com`       | `v=DMARC1; p=reject; sp=reject; adkim=s; aspf=s` |

Spotted on ["How to protect domains that do not send email"][2] by Cloudflare.

[1]: https://www.cloudflare.com/learning/email-security/what-is-email-spoofing/
[2]: https://www.cloudflare.com/learning/dns/dns-records/protect-domains-without-email/
