---
title: PDF invoice by mail to each subscription and on each renewal?
categories: questions
tags: api-notifications, mu-plugins-hacks
author: jaswsinc
github-issue: https://github.com/websharks/s2member-kb/issues/90
---

>  Question: What solution can we use to send a PDF invoice by mail to each subscription and on each renewal?

I'd suggest the use of s2Member's event-driven API Notifications for this. You can connect an email address or URL (Webhook) to every payment that s2Member processes and/or receives notice about; i.e. for every initial payment, and for every recurring payment too. This would allow you to generate a PDF invoice using a 3rd-party invoicing service, or through a custom script that you develop on your own.

_s2Member itself does not support the automatic generation of PDF files._

Please see: **s2Member ⥱ API / Notifications ⥱ Payment Notification**  
See also: [Building an API Notification Handler (Webhook)](https://github.com/websharks/s2member-kb/issues/155)
