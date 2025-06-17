---
title: "Contributing to Debian Brasil"
date: 2025-06-12 17:00:00 +/-0300
categories: [Open Source Software Development, Debian contributions]
description: "Check out our first time contributing to Debian Brasil and how the experience went"
tags: [linux, debian, contribution, floss, open-source, mac0470]
---

This was the last project developed the Open Source Software Development class, where our professor introduced us to Charles and Lucas Kanashiro, two Debian Brasil Maintainers and contributors. For this one, we were presented to the Debian project and made a small contribution, that should be merged.

### The Debian Project

Debian is a 100% open source project, first started in 1997, that follows an inspiring philosophy of retributing to the open source community, in a way that contributors from all over the world may join and help build this Kernel Distribution.

As a Kernel distribution, it follows some of the distributions principles, such as having the fundamental components of the OS and dealing with its interaction. The way a distribution works, it is a concrete implementation of the theoretical components, packed and deployed for final users. Debian does have a lot of failures, as any other distribution, so the main goal of the project is, not only improving its performance and adding functionalities, but also making sure the final users get as little bugs as possible.

You may think of the main components of the OS that Debian implements as the Kernel, responsible for memory, processes management, software/hardware communication, etc; system libraries, such as libc; user tools; package management system, in order to maintain the OS; applications and graphic environment.

Nowadays, many other distributions are based on Debian, so contributing to this distribution affects users from all the other based distributions as well.

### Packaging

For this contribution, we will be working with packaging, that is, the process of preparing a software for production, deploying the system so users can actually enjoy the product we are developing.

That may seem simple, but the whole process is a lot more complicated. If one has already worked anywhere and put some project in production, they already know this is not a straight forward task, and the whole deploy process take a long time, hours of configuration and an intensive dependencies management. We have been introduced to the main components on Debian, so we could actually understand the contribution.

For the small role we played in this contribution, we have worked on the `debian/control` and `debian/changelog`. The prior one is the file that handles the package metadata, and, alongside lots of metadata, some of the most important ones: the dependencies. The latter works as a log file, where it registers all the changes made on the code, specially packaging changes.

### The contribution

This was meant to be a first contribution to the Debian project and adjust us to the Debian development workflow. The only change we made was on the debian/control, updating it to the current version:

![debian change]({{ '/assets/img/2025-06-12-contributing-to-debian/contribution.png' | relative_url }})
_Debian Control Standards-Version change_

After that, we just generated the changelog file and commited it to the project. After a couple of days, we got the message back from the maintainers, approving our commit.

![debian MR]({{ '/assets/img/2025-06-12-contributing-to-debian/debian-accept.jpeg' | relative_url }})
_Debian merge request accepted_

> This post is used as a checkpoint on Debian contributions, first started as an assignment on the [Open Source development class](https://uspdigital.usp.br/jupiterweb/obterDisciplina?sgldis=MAC0470&codcur=3122&codhab=5000), at the Institute of Mathematics and Statistics of the University of São Paulo, advised by professor [Paulo Meirelles](https://www.ime.usp.br/~paulormm/).
{: .prompt-info }