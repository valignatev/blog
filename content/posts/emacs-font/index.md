+++
title = 'How to change Emacs font with minibuffer together'
tags = ['editors', 'emacs', 'elisp']
date = '2018-12-16T15:35:00+0300'
description = "We'll learn how to change Emacs font for the whole frame, not only current window, and touch a little bit how Emacs customization system works."
keywords = ['editors', 'emacs', 'elisp', 'emacs-lisp', 'font size', 'blog', 'init.el', 'emacs.d']
+++
I see how people struggle with Emacs font size all the time in
their live demos. Well, enough is enough!

<!--more-->

## The disclaimer

GUI only. If you're using the terminal version of Emacs, use font changing
features it provides. To skip to the end version click [TLDR](#whole-code-together).

## The problem

{{< resource-figure "emacs-font.png" >}}

So you're doing a live demo. Or it's a middle of the night, and your
eyes are a bit tired. In any case, you want to make the font size a bit bigger.
You hit <kbd>C-x</kbd> <kbd>C-+</kbd> couple of times, your text
increases, but the minibuffer stays
small. And other windows stay small as well. Just one last [example](https://youtu.be/6INMXmsCCC8?t=1298
"The pain is real") I've encountered recently.

## What we can do about it

What should you know to set the font in Emacs? First, you should know
that the default font name is, well, `default`. Second, there are multiple
ways to express the name of the
font. Emacs [manual](https://www.gnu.org/software/emacs/manual/html_node/emacs/Fonts.html)
shows four ways. It also says us to add this line to our `init.el`:

```elisp
;; "DejaVu Sans Mono-10" is the font name,
;; expressed in whatever way Emacs understands
(add-to-list 'default-frame-alist
             '(font . "DejaVu Sans Mono-10"))
```

Really? It isn't friendly at all, even for an Emacs user. Also, the
font size is a part of the string, not a number, which isn't helpful at
all. However, if you do add this line and restart your Emacs (or if
you create a new frame [^1] with `M-x new-frame`), you'll notice that it
indeed displays everything with the font you've specified in this
line. Including the modeline and minibuffer parts which is precisely
what we need.

Next step is to figure out how to set default font
interactively. [EmacsWiki](https://www.emacswiki.org/emacs/SetFonts#toc2)
suggest us to use `set-frame-font` which accepts the font name in the
same bizarre string format. Let's customize it a bit so we can
increase, decrease and reset the font size. We'll see if we can
reduce the pain.

## The solution

First, let's define font name and size separately:

```elisp
(setq my-font-name "monospaced")
(defcustom my-font-size 12 "My font size")
```

We'll be abusing Emacs [custom
variables](https://www.gnu.org/software/emacs/manual/html_node/elisp/Variable-Definitions.html)
so we can get both current and default font sizes. Without `defcustom`
we'd have to introduce another variable, something like
`my-current-font-size`. Which is possible but less convenient.

Now we can call `set-frame-font` by formatting variables together:

```elisp
(set-frame-font
  (format "%s %d" my-font-name my-font-size) nil t)
```

Let's turn it into a function which accepts the font size and sets it:

```elisp
(defun set-frame-font-size (&optional font-size)
  (let ((font-size
     (or font-size
         (car (get 'my-font-size 'standard-value)))))
    (customize-set-variable 'my-font-size font-size)
    (set-frame-font
     (format "%s %d" my-font-name font-size) nil t)))
```

Notice that `font-size` is optional. If it's not provided, we drop the
font size to the default value of `my-font-size`. Here Emacs
customization feature comes in handy because it remembers original
value. `(car (get 'my-font-size 'standard-value))` does exactly this -
it gets the original value of the `my-font-size`.

If `font-size` argument is provided, we assign it to the
`my-font-size` with the help of `customize-set-variable` and then call
`set-frame-font` with this number. Now we can call the function as
both `(set-frame-font-size NUMBER)` and `(set-frame-font-size)`.

Now we'll use this function as a foundation for an increase, decrease and
reset the font size functions:

```elisp
(defun increase-frame-font ()
  (interactive)
  (set-frame-font-size (+ my-font-size 1)))

(defun decrease-frame-font ()
  (interactive)
  (set-frame-font-size (- my-font-size 1)))

(defun reset-frame-font ()
  (interactive)
  (set-frame-font-size))
```

Note the `(interactive)`, so we can call these functions with `M-x`
like `M-x increase-frame-font` and bind them to keyboard shortcuts.
We also can remap builtins `text-scale-increase`, `text-scale-decrease`
and `text-scale-adjust` with our new functions. I'll leave this as an
exercise for you.

## Whole code together

Here what we have in the end, with bits of documentation added:

```elisp
;; Any available font name
(setq my-font-name "DejaVu Sans Mono")
(defcustom my-font-size 12 "My font size")

(defun set-frame-font-size (&optional font-size)
  "Change frame font size to FONT-SIZE.
If no FONT-SIZE provided, reset the size to its default variable."
  (let ((font-size
     (or font-size
       (car (get 'my-font-size 'standard-value)))))
    (customize-set-variable 'my-font-size font-size)
    (set-frame-font
     (format "%s %d" my-font-name font-size) nil t)))

(defun increase-frame-font ()
  "Increase frame font by one."
  (interactive)
  (set-frame-font-size (+ my-font-size 1)))

(defun decrease-frame-font ()
  "Decrease frame font by one."
  (interactive)
  (set-frame-font-size (- my-font-size 1)))

(defun reset-frame-font ()
  "Reset frame font to its default value."
  (interactive)
  (set-frame-font-size))
```

I hope you've got the idea, and maybe you've even learned something
new about Emacs and elisp (I did). This is not the only way
of achieving the purpose of this post (it is definitely not the most
universal one). Emacs is indefinitely flexible (sometimes at the cost
of a more verbose API), but it gives us an ability to bend it to our
will as much as we need and truly changes the way we think about the
tools we use every day.

[^1]: `frame` is a fancy word in Emacs to describe a separate independent window. Like, for example, a separate browser window. Emacs is 200 years old, remember?
