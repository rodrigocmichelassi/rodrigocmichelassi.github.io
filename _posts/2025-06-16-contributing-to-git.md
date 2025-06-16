---
title: "First contribution to Git (GSoC Microproject)"
date: 2025-06-16 10:00:00 +/-0300
categories: [Open Source Software Development, Git contributions]
description: "This post covers our first time contributing to Git"
tags: [linux, git, contribution, floss, open-source, mac0470]
---

This is probably our last contribution during Open Source Development class, where me and my colleague Isabella decided to contribute to Git, on an issue reported for GSoC applicants. Since this was our first ever contribution to the Git project, we took a easier issue, but it sure helps a lot learning about the Git contribution community and context.

### The issue

For GSoC applicants, which is not really our case, the idea is to [work on microprojects](https://git.github.io/SoC-2025-Microprojects/), which are small contributions for starters, before running deep on some actual code. For a first time contribution, we decide to work on the `t2400` test file, which implements a lot of tests with `test -[efd]`, a bash command for testing if a executable, file, or directory exists.

That implementation is outdated, and one of the microprojects is helping removing that from the code, not because it does not work, but because git contributors have created built-in functions, such as `test_path_is_missing`, `test_path_is_dir`, `test_path_is_file` etc., which applies the same testing functions, but emit useful diagnostics when applying the function fails.

Some of our changes are:

```bash
-       test -f .git/worktrees/here-with-lock-reason/locked &&
+       test_path_is_file .git/worktrees/here-with-lock-reason/locked &&
```

```bash
-       ! test -e swamp/init.t &&
+       ! test_path_is_executable swamp/init.t &&
```

### Feedback

Notice that, for the second one change showed above, the maintainers came back to us, explaining why it is not desirable to solve it this way, even though it is going to work, but may generate comments we are not interested in when the test fails.

![contributor comment]({{ '/assets/img/2025-06-16-contributing-to-git/feedback.png' | relative_url }})
_Contributor comments to our patch_

Alongside those comments on the places where this mistake happens, the contributors also gave us feedback about the commit structure and commit message, which should be formatted a different way. 

The Git community was also very welcoming, with 3 different maintainers reviewing our contribution and talking to each other and discussing what was done, and how to improve it. Thankfully, they were all agreeing on the changes that should be made, so we refactored the code, in order to fix what was detailed.

To fix the mistakes made, we changed some of the path checks made:

```bash
-       ! test -d zere &&
-       ! test -d .git/worktrees/zere
+       test_path_is_missing zere &&
+       test_path_is_missing .git/worktrees/zere
```

That way, we can still make the same path check, but in case it fails, we get no error message, as wished for test files and request by the maintainers. For now, we are waiting for new comments on V2 :)

> This post is used as a checkpoint on Debian contributions, first started as an assignment on the [Open Source development class](https://uspdigital.usp.br/jupiterweb/obterDisciplina?sgldis=MAC0470&codcur=3122&codhab=5000), at the Institute of Mathematics and Statistics of the University of São Paulo, advised by professor [Paulo Meirelles](https://www.ime.usp.br/~paulormm/).
{: .prompt-info }