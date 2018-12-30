+++
title = "How I'm failing literate config in Emacs"
tags = ['editors', 'emacs']
date = '2018-12-30T22:43:00+0300'
description = "Read about how I've tried to rewrite my Emacs config into a literate style and why I didn't like it"
keywords = ['editors', 'emacs', 'elisp', 'emacs-lisp', 'blog', 'init.el', 'emacs.d', 'org-mode']
images = ['posts/emacs-org-config/emacs-org-config.png']
+++
My personal experience with the literate config style in Emacs as of the
end of 2018, and why I think I don't like it.

<!--more-->
{{< resource-figure "emacs-org-config.png" >}}
## TLDR
To skip all the lyrics and get right to the points click [here](#things-i-didn-t-like).

## Lyrical digression and necessary disclaimer
The concept of literate config was attractive to me because when you
read it, it feels like the actual documentation for some library or
framework. I mean, pieces of code with exhaustive description around
it, not just a couple line of comments. Epic examples which inspired
me to try it out were [Sacha
Chua's](https://github.com/sachac/.emacs.d/blob/gh-pages/Sacha.org)
and
[wasamasa's](https://github.com/wasamasa/dotemacs/blob/master/init.org).

## The concept
It's simple. You create a file named `config.org` in your `.emacs.d`
directory.  You organize it as you organize any org file, but with a
bits of elisp like this:

```org
* Org-mode heading
Text describing the code below, your intention etc
#+BEGIN_SRC emacs-lisp
  (defun some-elisp-code (&optional params) (...))
#+END_SRC
```

Then, in your `init.el` you call

```elisp
(org-babel-load-file (concat user-emacs-directory "config.org"))
```

That's it, `org-babel-load-file` grabs all elisp pieces from your org
file and put them together into a file called `config.el`. You can
find it near the original file.

You put your org config file under the VCS, put it on GitHub which
renders it properly, you're happy, people give you stars and life
feels complete. Not even mention that you can export the config to
HTML and put it on your site
([Sacha's](http://pages.sachachua.com/.emacs.d/Sacha.html) one, for
example).

## Things I didn't like
First, my unfinished org config is
[here](https://github.com/valignatev/dotfiles/blob/literate-config/.emacs.d/config.org). Now, to the list:

1. I can't easily navigate elisp from org file. E.g. I can do <kbd>C-h
   f</kbd> and <kbd>C-h v</kbd> over functions and variables, but I
   can't directly go to their definitions with <kbd>M-.</kbd>. Maybe
   `xref` can be configured the right way, but I haven't got to it.

2. I can't edit the code directly in the config file, I have to open
   dedicated buffer with <kbd>C-c '</kbd>. It didn't look like a big
   deal when I've seen somebody else doing that, but it's quite
   cumbersome for me.

3. Text, surrounding the code, creates too much noise. It makes config
   harder to navigate, and the code itself doesn't look like the first
   class citizen. I'm not even mentioning `#+BEGIN_SRC`/`#+END_SRC`
   stuff (I've just mentioned it actually).

4. Surrounding text itself looks redundant to me. I don't do anything
   significantly complex, and when I do, I tend to write comments and
   descriptive commit headers and messages.

5. I don't see any benefit for me personally. I see how it's more
   convenient for other people to read and study such config, but I
   don't seem to gain any benefit myself.

## Closing words
With all that being said, I won't trash this experiment. I'll save it
under the [dedicated
branch](https://github.com/valignatev/dotfiles/blob/literate-config/.emacs.d/config.org)
and will back up to my
[main](https://github.com/valignatev/dotfiles/blob/5d9d152bf27c300fc21d673dc5290a0073165425/.emacs.d/init.el)
config which I'll separate into multiple files, as I would do with
a regular software project.

Meanwhile, I'd gladly hear what possible advantages of an org style
config am I missing. Maybe there is something that would outweigh cons
which are deal-breakers for me now.
