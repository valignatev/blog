+++
title = 'Open external terminals from Emacs'
tags = ['editors', 'emacs']
date = '2018-12-24T01:45:00+0300'
description = 'Learn how to open your default terminal from within emacs in the project root or in the current directory'
keywords = ['editors', 'emacs', 'elisp', 'emacs-lisp', 'terminal', 'blog', 'init.el', 'emacs.d', 'linux', 'environment variables']
+++
How to integrate your terminal with Emacs in an quick way and open it based on
the current project or current working directory.

<!--more-->
{{< resource-figure "emacs-external-terminal.png" >}}
## Quick note about non-Linux OSes
The trick I'm using applies to Linux only (go, sue me for this). For other
operating systems check out cool package
[terminal-here](https://github.com/davidshepherd7/terminal-here) (it
supports Linux as well).

## The trick
```elisp
(defun open-terminal ()
"Open default terminal emulator in the current directory."
  (interactive)
  (start-process "terminal" nil (getenv "TERMINAL")))

(defun open-terminal-in-project-root ()
"Open default terminal in the project root."
  (interactive)
  (let ((default-directory (projectile-project-root)))
    (open-terminal))
```

These two functions are basically it. They'll open the terminal in the
current working directory (`default-directory` in Emacs terms) and in
the project root[^1] respectively. If you close Emacs (why would you do
this?), all terminals will be closed as well. But how does it work?

## Emacs can start other programs
Amazing, right? `start-process` allows us to run other programs and
bind them to Emacs. It accepts some arguments:

1. Process name which we can see in the process list (`M-x list-processes`).
2. Buffer associated with the process (`nil` means no buffer for the process).
3. The actual program to run.
4. Arguments for the process we're about to start. We're not using it.

Ok, so in Linux, you can get the default terminal by reading from
`$TERMINAL` environment variable (check `echo $TERMINAL` in the console). Emacs way of reading environment variable is
`(getenv "TERMINAL")`. It gets us the name of the default
terminal emulator which Emacs will gladly run for us.

## Changing default terminal directory
Often you're editing a file somewhere deep in the project hierarchy,
but you need the terminal in the project root. Of course, you're using
[projectile](https://github.com/bbatsov/projectile), right[^2]? Projectile
has `projectile-run-<X>` where `<X>` is any terminal built
into Emacs. But we need external one! Luckily,
`(projectile-project-root)` gives us the root directory of the
project. We assign it to the `default-directory` using `let` which
will ensure that our change will stay only during the function
lifetime, and we don't mess with the whole buffer default directory
which other things might be using.

## What else can we do
As an exercise, you can consider adding more functionality. Like
support for opening remote directories (check `make-process`) or you
can add support for passing custom environment variables. This way
your terminal can, for example, start with custom Python virtualenv
activated (check `process-environment`). Or you can start some other
program in the newly opened terminal.

## Why bother
Many people use embedded Emacs terminals successfully (sometimes their
workflow looks like
[magic](https://www.youtube.com/watch?v=RhYNu6i_uY4)). Also, integration
between embedded terminals and the rest of Emacs is better than
anything you could get with an external one. However, there are rough
edges and quirks here and there. External terminals have better TUI
support, they are faster, often they have better colors and even
leverage GPU for rendering. It also makes perfect sense if you're
using a tiling window manager and rely heavily on having multiple
terminals laying around. And people
[asking](https://emacs.stackexchange.com/q/7650/13740) for it.

[^1]: Project root is, for example, the directory where your `.git` resides.
[^2]: There's also `vc-root-dir` available since Emacs 25.1 out of the box. `terminal-here` [falls back](https://github.com/davidshepherd7/terminal-here/blob/master/terminal-here.el#L151) to it
