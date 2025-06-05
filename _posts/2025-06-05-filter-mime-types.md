---
title: "[GNOME] Fix audio/video filters for mime types in Nautilus"
date: 2025-06-05 10:00:00 +/-0300
categories: [Open Source Software Development, GNOME contributions]
description: "In this post, we will cover how we 'solved' the issue reporting that some audio/video filters were not working in Nautilus."
tags: [linux, gnome, setup, contribution, floss, open-source, mac0470]
---

Issue 2452 reported a bug on Nautilus where, given a directory with a bunch of audio/video files, when applying those filters, the files would not show up. This issue was reported 2 years prior to this contribution and was actually solved at the time.

### The issue

On the `nautilus-mime-actions.c` file, the mime types are actually defined inside Nautilus. Those types actually reflect the types of files that can be filtered using the filter tool.

![Filter on Nautilus]({{ '/assets/img/2025-06-05-filter-mime-types/filter.png' | relative_url }})
_The filter tool on Nautilus_

The solution was actually proposed by the issue reporter, and then the code was put there by one of the mantainers, but the merge request got lost and never finished. To that's when I saw that it would be an easy issue to fix and not much was actually necessary.

### How the interaction went

Given the situation on this issue, I decided to leave a comment asking if this was never merged, as I suspected, but there were some weird climax by the guy who reported the issue towards the mantainers. I will probably remove this later on from the post, but he resented the mantainers for not merging and fixing his solution, and came with many negative comments. On the opposite side, one of the mantainers cleared me to just fix the conflicts and pull his commit to the main branch.

![Comment rant 1]({{ '/assets/img/2025-06-05-filter-mime-types/comment1.png' | relative_url }})
_First comment screenshot_

![Comment rant 2]({{ '/assets/img/2025-06-05-filter-mime-types/comment2.png' | relative_url }})
_Second comment screenshot_

![Comment rant 3]({{ '/assets/img/2025-06-05-filter-mime-types/comment3.png' | relative_url }})
_Third comment screenshot_

### What we did

Since this was already done, we had no trouble adjusting it for merge purposes. We knew what file we had to deal with, so we just cherry-picked the commit that proposed the changes and applied it to the main branch. The only work left to do was dealing with the conflicts and then it was all set to a new commit.

With all that done, we sent the merge request and, after one of the mantainers adding a new file type to the filters, it got accepted and we now have our first merge request ever on GNOME. Even though it was a nobrainer, it felt really good contributing for the first time to a project that affects millions of people all over the world :)

![First MR accepted]({{ '/assets/img/2025-06-05-filter-mime-types/mr.png' | relative_url }})
_Our MR on the top of the list_

> This post is used as a checkpoint on GNOME contributions, first started as an assignment on the [Open Source development class](https://uspdigital.usp.br/jupiterweb/obterDisciplina?sgldis=MAC0470&codcur=3122&codhab=5000), at the Institute of Mathematics and Statistics of the University of São Paulo, advised by professor [Paulo Meirelles](https://www.ime.usp.br/~paulormm/).
{: .prompt-info }