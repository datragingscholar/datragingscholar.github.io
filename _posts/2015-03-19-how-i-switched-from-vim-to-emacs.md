---
title        : "How I Switched From Vim to Emacs"
date         : "2015-03-19"
tags         : [Emacs]
---

I've been using vim as my preferred choice of editor for a little more than
five years by the time I decided to leave for emacs for good. During these five
years, I've tried other text editors and IDEs, sometimes in search of a better
editing tool, sometimes to fit the development evironment of the projects I
worked on, but vim was trumphant as I kept installing vim emulators even when I
was using other editors, and I always went back to it eventually.

One may have many different kinds of reasons for switching to another text
editor, and many feel obliged to justify their reasonings as editors mean more
than just tools to program in, they've become faith, even religions.

Jokingly or seriously, switching from vim to emacs can already be considered
**sacriligous**, let alone the fact that I actually use evil-mode to emulate vim
key mappings to ease the learning curve. Though I have personnally gone through
lots of thinkings and justifications to make the final decision, I figure it
would mean little to those who doesn't care about editor wars, and even less
to editor apologists, my reasonings may just offend them more.

So to completely avoid the fruitless effort of blogging my justifications, and
as the title of this post intended, this is about **how** I made the switch, not
**why** I did it. If you are new to emacs, and have used vim before, my config and
comments below can be a good boilerplate to get you started.


# Summary

This setup was tailored specifically to meet my requirements and use cases,
I'll however be more than happy if you find it useful out-of-the-box, though I
deem this better as a starter config to get things going and to showcase what
emacs and some popular plugins can do.

This config is heavily inspired (and copied from) [Harry R. Schwartz's](https://github.com/hrs/dotfiles/blob/master/emacs/.emacs.d/configuration.org),
it's his talk that got me switching to emacs in the first place.


## General Structure

Save [`init.el`](https://github.com/datragingscholar/dotfiles/blob/master/.emacs.d/init.el) as `~/.emacs.d/init.el`, it installs [*use-package*](https://github.com/jwiegley/use-package) and
configures it with some popular repos.

Save [`conf.org`](https://github.com/datragingscholar/dotfiles/blob/master/.emacs.d/conf.org) as `~/.emacs.d/conf.org`, `init.el` would compile this org
document into `conf.el` and then load it with *org-babel-load-file*.

This allows *literature programming*, which helps keeping this configuration
readable and maintainable.


# Package Configuration

Make sure [*use-package*](https://github.com/jwiegley/use-package) auto compiles and always uses the newest version.

This is the package or plugin control system if you will for emacs.

``` emacs-lisp
    (setq use-package-always-ensure t)
    (setq load-prefer-newer t)
    (use-package auto-compile
      :config (auto-compile-on-load-mode))
```

# Default Configuration

Use [*sensible-defaults.el*](https://github.com/hrs/dotfiles/blob/master/emacs/.emacs.d/configuration.org) for some recommended default setups.

Unfortunately this can't be installed as a package. Though the repo hasn't
been updated for two years, makes sense to just copy-paste it into emacs.d folder.

You need to manually checkout the file into your emacs config folder for this
to work properly.

``` emacs-lisp
    (load-file "~/.emacs.d/sensible-defaults.el")
    (sensible-defaults/use-all-settings)
    (sensible-defaults/use-all-keybindings)
    (sensible-defaults/backup-to-temp-directory)
```

Bind the M-o key to switch between windows. This does not work on my Macbook
though, need to find out why. The good old *C-x o* works perfectly still.

``` emacs-lisp
    (global-set-key (kbd "M-o") 'other-window)
```

# Evil Mode

[*evil-mode*](https://github.com/Somelauw/evil-org-mode) enables vim keymappings for emacs, it makes my life much easier.

``` emacs-lisp
    (use-package evil
      :config
      (evil-mode 1))
```

Enable org-mode agenda keymaps with [*evil-org-mode*](https://github.com/Somelauw/evil-org-mode). Now you can use vim key bindings in org-mode.

``` emacs-lisp
    (use-package evil-org
      :diminish evil-org-mode
      :after org
      :config
      (add-hook 'org-mode-hook 'evil-org-mode)
      (add-hook 'evil-org-mode-hook
                (lambda () (evil-org-set-key-theme)))
      (require 'evil-org-agenda)
      (evil-org-agenda-set-keys))
```

Here comes [*evil-surround*](https://github.com/emacs-evil/evil-surround), a ported version of [*surround.vim*](https://github.com/tpope/vim-surround), good stuff.

``` emacs-lisp
    (use-package evil-surround
      :ensure t
      :config
      (global-evil-surround-mode 1))
```

[*Powerline*](https://github.com/milkypostman/powerline) for emacs, to make you feel right at home.

``` emacs-lisp
    (use-package powerline-evil
      :ensure t
      :config
      (powerline-evil-center-color-theme))
```

# UI Preferences


## Tweak window chrome

Disables tool-bar and menu-bar, they take spaces but I rarely use them.

``` emacs-lisp
    (tool-bar-mode 0)
    (menu-bar-mode 0)
```

There's a tiny scroll bar that appears in the minibuffer window. This
disables that:

``` emacs-lisp
    (set-window-scroll-bars (minibuffer-window) nil nil)
```

The default frame title isn't useful. This binds it to the name of the
current projectile project:

``` emacs-lisp
    (setq frame-title-format '((:eval (projectile-project-name))))
```

## Use Fancy Lambdas

Yea, why not.

``` emacs-lisp
    (global-prettify-symbols-mode t)
```

## Theme

I like `wilson theme` from [*sublime themes*](https://github.com/owainlewis/emacs-color-themes).

``` emacs-lisp
    (use-package sublime-themes)

    (load-theme 'wilson t)
```

## Scroll conservatively

This prevents the screen from jumping when your cursor moves out of the window.

I actually find the default *center at the cursor location* fun and useful. Uncomment this line and try if it works for you.

``` emacs-lisp
    ;; (setq scroll-conservatively 100)
```

## Highlight the current line

*global-hl-line-mode* softly highlights the background color of the line containing point.
It makes it a bit easier to find point, and it's useful when pairing or presenting code.

``` emacs-lisp
    (global-hl-line-mode)
```

## Diminish unnecessary modes

Use [*diminish*](https://github.com/myrjola/diminish.el) to hide or abbreviates minor modes from the mode line. They continue to work, though.

``` emacs-lisp
    (use-package diminish)
```

# Project management


## dired

Use [*dired-hide-dotfiles*](https://github.com/mattiasb/dired-hide-dotfiles) to hide dot files and toggle visibility with `.`.

``` emacs-lisp
    (use-package dired-hide-dotfiles)

    (defun my-dired-mode-hook ()
      "My `dired' mode hook."
      ;; To hide dot-files by default
      (dired-hide-dotfiles-mode)

       ;; To toggle hiding
       (define-key dired-mode-map "." #'dired-hide-dotfiles-mode))

    (add-hook 'dired-mode-hook #'my-dired-mode-hook)
```

These are the switches that get passed to `ls` when `dired` gets a list of files. We're using:

-   `l`: Use the long listing format.
-   `h`: Use human-readable sizes.
-   `v`: Sort numbers naturally.
-   `a`: Include all files.

Change this if you want your folder listing in a different flavor.

``` emacs-lisp
    (setq-default dired-listing-switches "-lhva")
```

Kill buffers of files/directories that are deleted in `dired`.

``` emacs-lisp
    (setq dired-clean-up-buffers-too t)
```

Always copy directories recursively instead of asking every time.

``` emacs-lisp
    (setq dired-recursive-copies 'always)
```

Ask before recursively *deleting* a directory, though.

``` emacs-lisp
    (setq dired-recursive-deletes 'top)
```

Open a file with an external program (I use a Mac, so it's `open`) by hitting
`C-c C-o`.

``` emacs-lisp
    (defun dired-xdg-open ()
      "In dired, open the file named on this line."
      (interactive)
      (let* ((file (dired-get-filename nil t)))
        (call-process "open" nil 0 nil file)))

    (define-key dired-mode-map (kbd "C-c C-o") 'dired-xdg-open)
```

## ag

Set up [*ag*](https://agel.readthedocs.io/en/latest/installation.html) for displaying search results. You need to install `ag` binary for this to work properly.

Run *brew install ag* manually if you are a Mac user.

Hit `M-x`, then type `ag` or `ag-project` and press enter to search
recursively.

``` emacs-lisp
    (use-package ag)
```

## company

Use [*company-mode*](http://company-mode.github.io) everywhere.

``` emacs-lisp
    (use-package company
      :diminish company-mode)
    (add-hook 'after-init-hook 'global-company-mode)
```

I use *ac-php* for auto completion. Still useful to bind a *company-mode* completion key.

You can hit `M-/` for auto completion no matter what language you use.

``` emacs-lisp
    (global-set-key (kbd "M-/") 'company-complete-common)
```

## dumb-jump

[*dumb-jump*](https://github.com/jacktasia/dumb-jump) is the "jump to definition" package for emacs.

I bind `M-v` and `M-w` to 'go' and 'back' respectively since I use Dvorak
keyboard.

They are equivalent to `M->` and `M-<` if you use QWERT keyboard, so change
the following keybindings if you do.

``` emacs-lisp
    (use-package dumb-jump
      :config
      (define-key evil-normal-state-map (kbd "M-v") 'dumb-jump-go)
      (define-key evil-normal-state-map (kbd "M-w") 'dumb-jump-back)
      (setq dumb-jump-selector 'ivy))
```

## flycheck

[*flycheck*](https://www.flycheck.org/en/latest/) is a on the fly syntax checking extension. It supports many
programming languages out of the box.

``` emacs-lisp
    (use-package flycheck
      :ensure t
      :init (global-flycheck-mode))
```

## magit

[*magit*](https://magit.vc) is a fantastic version control extension for emacs.

There are some tweaks here:

-   Bind magit status menu to `C-x g`.
    -   After that, bring up help menu with `h` and select actions you want to perform.

    -   It'll tell you the key binding combination for that action, you can memerize it for next time.

-   Use [evil-magit](https://github.com/emacs-evil/evil-magit) for evil key bindings.

-   Per [tpope's suggestions](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html), highlight commit text in the summary line that goes beyond 50 characters.

-   I'd like to start in the insert state when writing a commit message.

``` emacs-lisp
    (use-package magit
      :bind
      ("C-x g" . magit-status)

      :config
      (use-package evil-magit)
      (use-package with-editor)
      (setq git-commit-summary-max-length 50)

      (add-hook 'with-editor-mode-hook 'evil-insert-state))
```

## Projectile

[*projectile*](https://github.com/bbatsov/projectile) is a project interaction library for emacs which enables
functionalities like switching between projects, search for file in a
project, replace in project and so on.

The following enables projectile globally, customizes some key chords and
sets the default directory to look for projects. You can find a more detailed
customization guide in its documentation.

``` emacs-lisp
    (use-package projectile
      :bind
      ("C-c v" . 'projectile-ag)

      :config
      (define-key projectile-mode-map (kbd "C-c p") 'projectile-command-map)
      (define-key evil-normal-state-map (kbd "C-p") 'projectile-find-file)
      (evil-define-key 'motion ag-mode-map (kbd "C-p") 'projectile-find-file)
      (evil-define-key 'motion rspec-mode-map (kbd "C-p") 'projectile-find-file)

      (setq projectile-completion-system 'ivy)
      (setq projectile-switch-project-action 'projectile-dired)
      (setq projectile-require-project-root nil))
      (setq projectile-project-search-path '("~/Projects/"))

    (projectile-global-mode)
```

# Programming Language Specifics


## General Programming

Some adjustments and tweaks for general text editing.


### Tabs

Change tab width to 2.

``` emacs-lisp
    (setq-default tab-width 2)
```

### superword

I'd like to treat camelCasedWord as a whole and don't often have to navigate inside them.
If treating camelCasedWord as three different words is what you want, uncomment the following code.

``` emacs-lisp
    ;; (use-package subword-mode
    ;;  :diminish subword-mode
    ;;  :config (global-subword-mode 1))
```

I enabled *superword-mode* as a hook for /php-mode/(see PHP section) since that's what I desire mostly when working on PHP projects.


### Ya-snippet

[*yasnippet*](https://github.com/joaotavora/yasnippet) is a templating system for emacs. I'm not very crazy about
snippets, but it's good to know it's available.

``` emacs-lisp
    (use-package yasnippet
      :diminish yasnippet-mode
      :config
      (yas-global-mode 1))
```

I keep my non-existent snippets in `~/.emacs/snippets/text-mode`

``` emacs-lisp
    (setq yas-snippet-dirs '("~/.emacs.d/snippets/text-mode"))
```

I *don’t* want `yas` to automatically indent the snippets it inserts.
Sometimes this looks pretty bad (when indenting org-mode, for example, or
trying to guess at the correct indentation for Python).

``` emacs-lisp
    (setq yas/indent-line nil)
```

### Spell-checking

Endable [*flyspell*](https://www.emacswiki.org/emacs/FlySpell) when editing text, markdown, org-mode and git commit message.

Hit `Ctrl-xs` to auto correct previous word from cursor.

``` emacs-lisp
    (use-package flyspell
      :diminish flyspell-mode

      :config
      (add-hook 'text-mode-hook 'turn-on-auto-fill)
      (add-hook 'gfm-mode-hook 'flyspell-mode)
      (add-hook 'org-mode-hook 'flyspell-mode)

      (add-hook 'git-commit-mode-hook 'flyspell-mode))

    (global-set-key (kbd "\C-xs") 'flyspell-auto-correct-previous-word)
```

### Wrap paragraphs automatically

Automatically trigger [*autofillmode*](https://www.emacswiki.org/emacs/AutoFillMode) when edition text, markdown or in org-mode.
Same as hitting `M-q`

``` emacs-lisp
    (add-hook 'text-mode-hook 'auto-fill-mode)
    (add-hook 'gfm-mode-hook 'auto-fill-mode)
    (add-hook 'org-mode-hook 'auto-fill-mode)
    (diminish 'auto-fill-function)
```

### Set up helpful

[*helpful*](https://github.com/Wilfred/helpful) is an alternative emacs help extension that provides much more
contextual information.

``` emacs-lisp
    (use-package helpful)

    (global-set-key (kbd "C-h f") #'helpful-callable)
    (global-set-key (kbd "C-h v") #'helpful-variable)
    (global-set-key (kbd "C-h k") #'helpful-key)
    (evil-define-key 'normal helpful-mode-map (kbd "q") 'quit-window)

    ;; Lookup the current symbol at point. C-c C-d is a common keybinding
    ;; for this in lisp modes.
    (global-set-key (kbd "C-c C-d") #'helpful-at-point)

    ;; Look up *F*unctions (excludes macros).
    ;;
    ;; By default, C-h F is bound to `Info-goto-emacs-command-node'. Helpful
    ;; already links to the manual, if a function is referenced there.
    (global-set-key (kbd "C-h F") #'helpful-function)

    ;; Look up *C*ommands.
    ;;
    ;; By default, C-h C is bound to describe `describe-coding-system'. I
    ;; don't find this very useful, but it's frequently useful to only
    ;; look at interactive functions.
    (global-set-key (kbd "C-h C") #'helpful-command)
```

### Save my location within a file

Using `save-place-mode` saves the location of point for every file I visit.
If I close the file or close the editor, then later re-open it, point will
be at the last place I visited.

``` emacs-lisp
    (save-place-mode t)
```

### Always indent with spaces

Never use tabs. Tabs are the devil’s whitespace.

``` emacs-lisp
    (setq-default indent-tabs-mode nil)
```

### Install and configure `which-key`

[*which-key*](https://github.com/justbur/emacs-which-key) displays the possible completions for a long keybinding. Which
is really helpful for some modes (like `projectile`, for example).

Handy when you forget a keybinding, try hitting something like `Ctrl-x` and
wait for the magic to happen.

``` emacs-lisp
    (use-package which-key
      :diminish
      :config
      (which-key-mode))
```

### Configure Swiper

[*swiper*](https://github.com/abo-abo/swiper) repo contains *ivy* for completion, *counsel* for a collection of
emacs commands and *swiper* as an alternative to isearch.

This configuration:

-   Uses `counsel-M-x` for command completion,
-   Replaces *isearch* with *swiper*,
-   Uses [*smex*](https://github.com/nonsequitur/smex/) to maintain `M-x` history,
-   Enables fuzzy matching everywhere except swiper (where it's thoroughly unhelpful), and
-   Includes recent files in the switch buffer.

``` emacs-lisp
    (use-package counsel
      :diminish ivy-mode
      :bind
      ("M-x" . 'counsel-M-x)
      ("C-s" . 'swiper)

      :config
      (use-package flx)
      (use-package smex)

      (ivy-mode 1)
      (setq ivy-use-virtual-buffers t)
      (setq ivy-count-format "(%d/%d) ")
      (setq ivy-initial-inputs-alist nil)
      (setq ivy-re-builders-alist
        '((swiper . ivy--regex-plus)
          (t . ivy--regex-fuzzy))))
```

### Switch and rebalance windows when splitting

When splitting a window, I invariably want to switch to the new window. This
makes that automatic.

``` emacs-lisp
    (defun hrs/split-window-below-and-switch ()
      "Split the window horizontally, then switch to the new pane."
      (interactive)
      (split-window-below)
      (balance-windows)
      (other-window 1))

    (defun hrs/split-window-right-and-switch ()
      "Split the window vertically, then switch to the new pane."
      (interactive)
      (split-window-right)
      (balance-windows)
      (other-window 1))

    (global-set-key (kbd "C-x 2") 'hrs/split-window-below-and-switch)
    (global-set-key (kbd "C-x 3") 'hrs/split-window-right-and-switch)
```

## Markdown

Use [*markdown-mode*](https://github.com/jrblevin/markdown-mode) to handle .md files and [*pandoc*](http://pandoc.org) for generating result.

-   Associate `.md` files with GitHub-flavored Markdown.
-   Use `pandoc` to render the results.
-   Leave the code block font unchanged.

``` emacs-lisp
    (use-package markdown-mode
      :commands gfm-mode

      :mode (("\\.md$" . gfm-mode))

      :config
      (setq markdown-command "pandoc --standalone --mathjax --from=markdown")
      (custom-set-faces
       '(markdown-code-face ((t nil)))))
```

## PHP

There are other PHP extensions such as *phpunit* and *composer*,
but I feel they don't contribute too much to my emacs experience and I still prefer to run these manually.


### ac-php and php-mode

[*php-mode*](https://github.com/emacs-php/php-mode) for PHP support and [*ac-php*](https://github.com/xcwen/ac-php) for auto completion.

Things to note:

-   Enabled *superword-mode* when editing PHP files.
-   Common lisp library *cl* as suggested by *ac-php*'s documentation was
    replaced by *cl-lib* as the latter is newer and recommended.
-   You can choose to use *auto-complet-mode* or *company-mode*.
-   I believe if you want *jump-to-definition* to work, [*phpctags*](https://packagist.org/packages/techlivezheng/phpctags) is required
    to generate ctags, though I haven't been able to make it work properly.
-   I'm using *company-mode*, `M-v` and `M-w` from *dumb-jump* work good enough for now.
-   Need to `touch .ac-php-conf-json` in root directory of the project to have *ac-php* work properly.
-   Need to manually run `ac-php-remake-tags-all` to regenerate tags when source code changes.

``` emacs-lisp
    (use-package cl-lib)
    (use-package php-mode)
    (use-package ac-php)
    (use-package company-php)

    (add-hook 'php-mode-hook
      '(lambda ()
        (superword-mode 1)
        (ac-php-core-eldoc-setup) ;; enable eldoc
        (make-local-variable 'company-backends)
        (add-to-list 'company-backends 'company-ac-php-backend)))
```

## WEB

I don't work much on front-end development, I only have some minimal configuration here.


### web-mode

As suggested by [*web-mode*](http://web-mode.org) documentation, both *add-to-list* and *web-mode-engines-alist* are recommened.
This enables web-mode for most web templates as well as blade template for Laravel framework.

``` emacs-lisp
    (use-package web-mode)
    (add-to-list 'auto-mode-alist '("\\.phtml\\'" . web-mode))
    (add-to-list 'auto-mode-alist '("\\.tpl\\.php\\'" . web-mode))
    (add-to-list 'auto-mode-alist '("\\.[agj]sp\\'" . web-mode))
    (add-to-list 'auto-mode-alist '("\\.as[cp]x\\'" . web-mode))
    (add-to-list 'auto-mode-alist '("\\.erb\\'" . web-mode))
    (add-to-list 'auto-mode-alist '("\\.mustache\\'" . web-mode))
    (add-to-list 'auto-mode-alist '("\\.djhtml\\'" . web-mode))

    (setq web-mode-engines-alist
          '(("php"    . "\\.phtml\\'")
            ("blade"  . "\\.blade\\."))
    )
```

### CSS, Sass, and Less

Indent by 2 spaces when working with css files.

``` emacs-lisp
    (use-package css-mode
      :config
      (setq css-indent-offset 2))
```

Don’t compile the current SCSS file every time I save.

``` emacs-lisp
    (use-package scss-mode
      :config
      (setq scss-compile-at-save nil))
```

Install Less

``` emacs-lisp
    (use-package less-css-mode)
```

### Javascript

Indent javascripts by 2 spaces

``` emacs-lisp
    (setq js-indent-level 2)
```

# Terminal

Use [*multi-term*](https://www.emacswiki.org/emacs/MultiTerm) for terminals and bind it to `C-c t`.

``` emacs-lisp
    (use-package multi-term)
    (global-set-key (kbd "C-c t") 'multi-term)
```

Set the default shell, I use zsh.

``` emacs-lisp
    (setq multi-term-program "/bin/zsh")
```

Don't use evil mode in terminal buffers.

``` emacs-lisp
    (evil-set-initial-state 'term-mode 'emacs)
```

# Org Mode

[*org-mode*](https://orgmode.org) is luuuuuuuuuv. If you haven't made your mind switching to emacs
yet, click on the link and read about org-mode.

``` emacs-lisp
    (use-package org
      :diminish org-indent-mode)
```

## Display preferences

I like to see an outline of pretty bullets instead of a list of asterisks.

``` emacs-lisp
    (use-package org-bullets
      :init
      (add-hook 'org-mode-hook 'org-bullets-mode))
```

I like seeing a little downward-pointing arrow instead of the usual ellipsis
(`...`) that org displays when there's stuff under a header.

``` emacs-lisp
    (setq org-ellipsis "⤵")
```

Use syntax highlighting in source blocks while editing.

``` emacs-lisp
    (setq org-src-fontify-natively t)
```

Make TAB act as if it were issued in a buffer of the language's major mode.

``` emacs-lisp
    (setq org-src-tab-acts-natively t)
```

When editing a code snippet, use the current window rather than popping open a
new one (which shows the same information).

``` emacs-lisp
    (setq org-src-window-setup 'current-window)
```

## Task and org-capture management

Store my org files in `~/Projects/org`, the location of an index file (my main todo list), and archive finished tasks in `~/Projects/org/archive.org`.

``` emacs-lisp
    (setq org-directory "~/Projects/org")

    (defun org-file-path (filename)
      "Return the absolute address of an org file, given its relative name."
      (concat (file-name-as-directory org-directory) filename))

    (setq org-index-file (org-file-path "index.org"))
    (setq org-archive-location
          (concat (org-file-path "archive.org") "::* From %s"))
```

Set the master todo file `~/Projects/org/index.org` as the agenda file

``` emacs-lisp
    (setq org-agenda-files (list org-index-file))
```

Hitting `C-c C-x C-s` will mark a todo as done and move it to an appropriate
place in the archive.

``` emacs-lisp
    (defun hrs/mark-done-and-archive ()
      "Mark the state of an org-mode item as DONE and archive it."
      (interactive)
      (org-todo 'done)
      (org-archive-subtree))

    (define-key org-mode-map (kbd "C-c C-x C-s") 'hrs/mark-done-and-archive)
```

Record the time that a todo was archived.

``` emacs-lisp
    (setq org-log-done 'time)
```

-   Capturing tasks

    Define a few common tasks as capture templates.

``` emacs-lisp
        (setq org-capture-templates
          '(("n" "Notes"
            entry (file "~/Projects/org/notes.org")
            "* %?\n")

            ("t" "Todo"
             entry
             (file+headline org-index-file "New Tasks")
             "* TODO %?\n")))
```

   When I'm starting an Org capture template I'd like to begin in insert mode.
   I'm opening it up in order to start typing something, so this skips a step.

``` emacs-lisp
        (add-hook 'org-capture-mode-hook 'evil-insert-state)
```

   Refiling according to the document's hierarchy.

``` emacs-lisp
        (setq org-refile-use-outline-path t)
        (setq org-outline-path-complete-in-steps nil)
```

-   Keybindings

   Bind a few handy keys.

``` emacs-lisp
        (define-key global-map "\C-cl" 'org-store-link)
        (define-key global-map "\C-ca" 'org-agenda)
        (define-key global-map "\C-cs" 'org-agenda-show)
        (define-key global-map "\C-cc" 'org-capture)
```

   Hit `C-c i` to quickly open up my todo list.

``` emacs-lisp
        (defun open-index-file ()
          "Open the master org TODO list."
          (interactive)
          (find-file org-index-file)
          (flycheck-mode -1)
          (end-of-buffer))

        (global-set-key (kbd "C-c i") 'open-index-file)
```

   Bind `C-c n` to quickly open up notes.

``` emacs-lisp
        (defun open-note-file ()
          "Open the notes."
          (interactive)
          (find-file "~/Projects/org/notes.org")
          (flycheck-mode -1)
          (end-of-buffer))

        (global-set-key (kbd "C-c n") 'open-note-file)
```

   Hit `M-n` to quickly open up a capture template for a new todo.

``` emacs-lisp
        (defun org-capture-todo ()
          (interactive)
          (org-capture :keys "t"))

        (global-set-key (kbd "M-n") 'org-capture-todo)
```

## Export

Use [*htmlize*](https://github.com/hniksic/emacs-htmlize) for exporting html file. Disable footer.

``` emacs-lisp
    (use-package htmlize)
    (setq org-html-postamble nil)
```

# Blogging

Copied these from hrs's config as I also blog with jekyll. You may need to
modify some paths to have it working correctly for you.

I don't yet want to introduce more key chords for blogging functionalities,
use *M-x* to invoke these functions or bind your customized key chords as you
like.

``` emacs-lisp
    (defvar hrs/jekyll-posts-directory "~/Projects/ragingscholar/_posts/")
    (defvar hrs/jekyll-post-extension ".md")

    (defun hrs/replace-whitespace-with-hyphens (s)
      (replace-regexp-in-string " " "-" s))

    (defun hrs/replace-nonalphanumeric-with-whitespace (s)
      (replace-regexp-in-string "[^A-Za-z0-9 ]" " " s))

    (defun hrs/remove-quotes (s)
      (replace-regexp-in-string "[\'\"]" "" s))

    (defun hrs/replace-unusual-characters (title)
      "Remove quotes, downcase everything, and replace characters
    that aren't alphanumeric with hyphens."
      (hrs/replace-whitespace-with-hyphens
       (s-trim
        (downcase
         (hrs/replace-nonalphanumeric-with-whitespace
          (hrs/remove-quotes title))))))

    (defun hrs/slug-for (title)
      "Given a blog post title, return a convenient URL slug.
       Downcase letters and remove special characters."
      (let ((slug (hrs/replace-unusual-characters title)))
        (while (string-match "--" slug)
          (setq slug (replace-regexp-in-string "--" "-" slug)))
        slug))

    (defun hrs/timestamped-slug-for (title)
      "Turn a string into a slug with a timestamp and title."
      (concat (format-time-string "%Y-%m-%d")
              "-"
              (hrs/slug-for title)))

    (defun hrs/jekyll-yaml-template (title)
      "Return the YAML header information appropriate for a blog
       post. Include the title, the current date, the post layout,
       and an empty list of tags."
      (concat
       "---\n"
       "title: " title "\n"
       "date: " (format-time-string "%Y-%m-%d %H:%M:%S") "\n"
       "tags: []\n"
       "---\n\n"))

    (defun hrs/new-blog-post (title)
      "Create a new blog post in Jekyll."
      (interactive "sPost title: ")
      (let ((post (concat hrs/jekyll-posts-directory
                          (hrs/timestamped-slug-for title)
                          hrs/jekyll-post-extension)))
        (if (file-exists-p post)
            (find-file post)
          (find-file post)
          (insert (hrs/jekyll-yaml-template title)))))
```

# Email

I use *mu4e* to send and receive emails. To have the following configuration
work correctly, you have to install [*mu*](https://www.djcbsoftware.nl/code/mu/mu4e/index.html#SEC_Contents) with emacs support, then
[*offlineimap*](http://www.offlineimap.org) as a service to receive and send emails. You can click on their
names to refer to their documentation respectively, or you can checkout [A
COMPLETE GUIDE TO EMAIL IN EMACS USING MU AND MU4E](http://cachestocaches.com/2017/3/complete-guide-email-emacs-using-mu-and-/) for a specific
walkthrough.

I use [*evil-mu4e*](https://github.com/JorisE/evil-mu4e) to have a consistent vim keymapping experience when working
with emails. You should change the path to the mail directory you set up with mu.

``` emacs-lisp
    (use-package evil-mu4e)
    (require 'evil-mu4e)

    (setq mu4e-maildir "~/Mail")
    (setq mu4e-confirm-quit nil)
```

Setting mu4e contexts, this is good when you have more than one email
addresses.

``` emacs-lisp
    (setq mu4e-contexts
    `( ,(make-mu4e-context
        :name "Yandex"
        :match-func (lambda (msg) (when msg
          (string-prefix-p "/Yandex" (mu4e-message-field msg :maildir))))
        :vars '(
          (user-mail-address . "send@hailang.email")
          (mu4e-trash-folder . "/Yandex/Trash")
          (mu4e-refile-folder . "/Yandex/Archive")
          (mu4e-sent-folder . "/Yandex/Sent")
          (mu4e-drafts-folder . "/Yandex/Drafts")
          ))
    ))
```

## Reading Emails

Display sender's email address along with their names.

``` emacs-lisp
    (setq mu4e-view-show-addresses t)
```

Save attachments in my ~/Downloads directory, not my home directory. This
works better on a Mac.

``` emacs-lisp
    (setq mu4e-attachment-dir "~/Downloads")
```

Some html emails are hideous to read in emacs, this binds *a h* to open the
email in your default browser.

``` emacs-lisp
    (add-to-list 'mu4e-view-actions
      '("html in browser" . mu4e-action-view-in-browser)
      t)
```

## Composing Emails

Enable Org-style tables and list manipulation.

``` emacs-lisp
    (add-hook 'message-mode-hook 'turn-on-orgtbl)
    (add-hook 'message-mode-hook 'turn-on-orgstruct++)
```

Once I’ve sent an email, kill the associated buffer instead of just burying it.

``` emacs-lisp
    (setq message-kill-buffer-on-exit t)
```

## Sending Emails

Send email with sendmail.

``` emacs-lisp
    (setq message-send-mail-function 'message-send-mail-with-sendmail)
    (setq message-sendmail-f-is-evil 't)

    (setq mu4e-sent-folder "~/Mail/Yandex/Sent"
      mu4e-drafts-folder "~/Mail/Yandex/Drafts"
      user-mail-address "send@hailang.email"
      smtpmail-default-smtp-server "smtp.yandex.com"
      smtpmail-smtp-server "smtp.yandex.com"
      smtpmail-smtp-service 587)

    (defvar my-mu4e-account-alist
      '(("Yandex"
        (mu4e-sent-folder "/Yandex/Sent")
        (user-mail-address "send@hailang.email")
        (smtpmail-smtp-user "send@hailang.email")
        (smtpmail-local-domain "yandex.com")
        (smtpmail-default-smtp-server "smtp.yandex.com")
        (smtpmail-smtp-server "smtp.yandex.com")
        (smtpmail-smtp-service 587)
        )
      ;; Include any other accounts here ...
      ))

    (defun my-mu4e-set-account ()
      "Set the account for composing a message.
       This function is taken from:
         https://www.djcbsoftware.nl/code/mu/mu4e/Multiple-accounts.html"
      (let* ((account
        (if mu4e-compose-parent-message
            (let ((maildir (mu4e-message-field mu4e-compose-parent-message :maildir)))
        (string-match "/\\(.*?\\)/" maildir)
        (match-string 1 maildir))
          (completing-read (format "Compose with account: (%s) "
                 (mapconcat #'(lambda (var) (car var))
                my-mu4e-account-alist "/"))
               (mapcar #'(lambda (var) (car var)) my-mu4e-account-alist)
               nil t nil nil (caar my-mu4e-account-alist))))
       (account-vars (cdr (assoc account my-mu4e-account-alist))))
        (if account-vars
      (mapc #'(lambda (var)
          (set (car var) (cadr var)))
            account-vars)
          (error "No email account found"))))
    (add-hook 'mu4e-compose-pre-hook 'my-mu4e-set-account)
```

## Org Integration

I use *org-mu4e* to store emails or search queries as links (org-store-link),
insert links in any org document such as my todo list (org-store-link) and
follow the link when I need to (evil-org-open-links).

Refer to [*this link*](https://www.djcbsoftware.nl/code/mu/mu4e/Org_002dmode-links.html) for more customizations.

``` emacs-lisp
    (require 'org-mu4e)
    (setq org-mu4e-link-query-in-headers-mode nil)
```

# Further On

I'll keep tweaking this configuration as I continue to use this wonderful
text editing system. Many of the functions and configs that I copied from the
Internet need more experimenting before I can feel comfortable hacking, and
I'll need much more experience to have my own customizations and hacks.

If you are thinking about switching to emacs from vim yourself and like what
you read, consider giving it a try for a few days to see if it works for you.

You can certainly try my configuration out even if you've never used vim
before, though that would make *evil-mode* and its related keybinding settings
less sensible. I would highly recommend learning the *original* emacs
keybindings in this case.

Check back to this repository for updates if you liked it, have fun hacking
around!
