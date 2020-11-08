# 			[Emacs #5: Documents and Presentations with org-mode](https://changelog.complete.org/archives/9900-emacs-5-documents-and-presentations-with-org-mode) 	

​			[April 4, 2018](https://changelog.complete.org/archives/9900-emacs-5-documents-and-presentations-with-org-mode)[Software](https://changelog.complete.org/archives/category/technology/software)[emacs](https://changelog.complete.org/archives/tag/emacs-s9y), [emacs2018](https://changelog.complete.org/archives/tag/emacs2018)[John Goerzen](http://changelog.complete.org)					

### The Emacs series

This is fifth in a [series on Emacs and org-mode](https://changelog.complete.org/archives/tag/emacs2018).

This blog post was generated from [an org-mode source](https://github.com/jgoerzen/public-snippets/blob/master/emacs/emacs-org-beamer/emacs-org-beamer.org) and is available as: [a blog page](http://changelog.complete.org/archives/9900-emacs-5-documents-and-presentations-with-org-mode), [slides (PDF format)](https://github.com/jgoerzen/public-snippets/raw/master/emacs/emacs-org-beamer/emacs-org-beamer.pdf), and a [PDF document](https://github.com/jgoerzen/public-snippets/raw/master/emacs/emacs-org-beamer/emacs-org-beamer-document.pdf).

## 1 About org-mode exporting



### 1.1 Background

org-mode isn't just an agenda-making program.  It can also export to lots of formats: LaTeX, PDF, Beamer, iCalendar (agendas), HTML, Markdown, ODT, plain text, man pages, and more complicated formats such as a set of web pages.

This isn't just some afterthought either; it's a core part of the system and integrates very well.

One file can be source code, automatically-generated output, task list, documentation, and presentation, all at once.

Some use org-mode as their preferred markup format, even for things like LaTeX documents.  The org-mode manual has an extensive [section on exporting](https://orgmode.org/manual/Exporting.html#Exporting).

### 1.2 Getting started

From any org-mode document, just hit `C-c C-e`.  From there will come up a menu, letting you choose various export formats and options. These are generally single-key options so it's easy to set and execute.  For instance, to export a document to a PDF, use  `C-c C-e l p` or for HTML export, `C-c C-e h h`.

There are lots of settings available for all of these export options; see the manual.  It is, in fact, quite possible to use LaTeX-format equations in both LaTeX and HTML modes, to insert arbitrary preambles and settings for different modes, etc.

### 1.3 Add-on packages

ELPA containts many addition exporters for org-mode as well.  Check there for details.

## 2 Beamer slides with org-mode



### 2.1 About Beamer

[Beamer](https://en.wikipedia.org/wiki/Beamer_(LaTeX)) is a LaTeX environment for making presentations.  Its features include:

- Automated generating of structural elements in the presentation (see, for example, [the Marburg theme](https://hartwork.org/beamer-theme-matrix/all/beamer-albatross-Marburg-1.png)).  This provides a visual reference for the audience of where they are in the presentation.
- Strong help for structuring the presentation
- Themes
- Full LaTeX available

### 2.2 Benefits of Beamer in org-mode

org-mode has a lot of benefits for working with Beamer.  Among them:

- org-mode's very easy and strong support for visualizing and changing the structure makes it very quick to reorganize your material.
- Combined with org-babel, live source code (with syntax highlighting) and results can be embedded.
- The syntax is often easier to work with.

I have completely replaced my usage of LibreOffice/Powerpoint/GoogleDocs with org-mode and beamer.  It is, in fact, rather frustrating when I have to use one of those tools, as they are nowhere near as strong as org-mode for visualizing a presentation structure.

### 2.3 Headline Levels

org-mode's Beamer export will convert sections of your document (defined by headings) into slides.  The question, of course, is: *which* sections?  This is governed by the H [export setting](https://orgmode.org/manual/Export-settings.html#Export-settings) (`org-export-headline-levels`).

There are many ways to go, which suit people.  I like to have my presentation like this:

```
#+OPTIONS: H:2
#+BEAMER_HEADER: \AtBeginSection{\frame{\sectionpage}}
```

This gives a standalone section slide for each major topic, to highlight major transitions, and then takes the level 2 (two asterisks) headings to set the slide.  Many Beamer themes expect a third level of indirection, so you would set `H:3` for them.

### 2.4 Themes and settings

You can configure many Beamer and LaTeX settings in your document by inserting lines at the top of your org file.  This document, for instance, defines:

```
#+TITLE:  Documents and presentations with org-mode
#+AUTHOR: John Goerzen
#+BEAMER_HEADER: \institute{The Changelog}
#+PROPERTY: comments yes
#+PROPERTY: header-args :exports both :eval never-export
#+OPTIONS: H:2
#+BEAMER_THEME: CambridgeUS
#+BEAMER_COLOR_THEME: default
```

### 2.5 Advanced settings

I like to change some colors, bullet formatting, and the like.  I round out my document with:

```
# We can't just +BEAMER_INNER_THEME: default because that picks the theme default.
# Override per https://tex.stackexchange.com/questions/11168/change-bullet-style-formatting-in-beamer
#+BEAMER_INNER_THEME: default
#+LaTeX_CLASS_OPTIONS: [aspectratio=169]
#+BEAMER_HEADER: \definecolor{links}{HTML}{0000A0}
#+BEAMER_HEADER: \hypersetup{colorlinks=,linkcolor=,urlcolor=links}
#+BEAMER_HEADER: \setbeamertemplate{itemize items}[default]
#+BEAMER_HEADER: \setbeamertemplate{enumerate items}[default]
#+BEAMER_HEADER: \setbeamertemplate{items}[default]
#+BEAMER_HEADER: \setbeamercolor*{local structure}{fg=darkred}
#+BEAMER_HEADER: \setbeamercolor{section in toc}{fg=darkred}
#+BEAMER_HEADER: \setlength{\parskip}{\smallskipamount}
```

Here, `aspectratio=169` sets a 16:9 aspect ratio, and the remaining are standard LaTeX/Beamer configuration bits.

### 2.6 Shrink (to fit)

Sometimes you've got some really large code examples and you might prefer to just shrink the slide to fit.  

Just type `C-c C-x p`, set the `BEAMER_opt` property to `shrink=15`.  

(Or a larger value of shrink).  The previous slide uses this here.

### 2.7 Result

Here's the end result:

[![screenshot1](https://farm1.staticflickr.com/889/26366340577_fbde8ff266_o.png)](https://www.flickr.com/photos/jgoerzen/26366340577/in/dateposted/)

## 3 Interactive Slides



### 3.1 Interactive Emacs Slideshows

With the [org-tree-slide package](https://orgmode.org/worg/org-tutorials/non-beamer-presentations.html#org-tree-slide), you can display your slideshow from right within Emacs.  Just run `M-x org-tree-slide-mode`.  Then, use `C->` and `C-<` to move between slides.

You might find `C-c C-x C-v` (which is `org-toggle-inline-images`) helpful to cause the system to display embedded images.

### 3.2 HTML Slideshows

There are a lot of ways to export org-mode presentations to HTML, with various levels of JavaScript integration.  See the [non-beamer presentations section](https://orgmode.org/worg/org-tutorials/non-beamer-presentations.html) of the org-mode wiki for details.

## 4 Miscellaneous



### 4.1 Additional resources to accompany this post

- [orgmode.org beamer tutorial](https://orgmode.org/worg/exporters/beamer/tutorial.html)

- [LaTeX wiki](https://en.wikibooks.org/wiki/LaTeX/Presentations)

- [Generating section title slides](https://tex.stackexchange.com/questions/117658/automatically-generate-section-title-slides-in-beamer/117661)

- [Shrinking content to fit on slide](https://tex.stackexchange.com/questions/78514/content-doesnt-fit-in-one-slide)

- A 

  great

   resource: refcard-org-beamer

  - See its [Github repo](https://github.com/fniessen/refcard-org-beamer)
  - Make sure to check out both the PDF and the .org file

- A nice [Theme matrix](https://hartwork.org/beamer-theme-matrix/)

### 4.2 Up next in my Emacs series…

mu4e for email!

### Share this:

- [Facebook](https://changelog.complete.org/archives/9900-emacs-5-documents-and-presentations-with-org-mode?share=facebook&nb=1)
- [Twitter](https://changelog.complete.org/archives/9900-emacs-5-documents-and-presentations-with-org-mode?share=twitter&nb=1)
- [Reddit](https://changelog.complete.org/archives/9900-emacs-5-documents-and-presentations-with-org-mode?share=reddit&nb=1)
- 

​				[One comment so far](https://changelog.complete.org/archives/9900-emacs-5-documents-and-presentations-with-org-mode#comments)			

# 			[Emacs #4: Automated emails to org-mode and org-mode syncing](https://changelog.complete.org/archives/9898-emacs-4-automated-emails-to-org-mode-and-org-mode-syncing) 	

​			[March 31, 2018](https://changelog.complete.org/archives/9898-emacs-4-automated-emails-to-org-mode-and-org-mode-syncing)[Linux](https://changelog.complete.org/archives/category/technology/software/linux), [Software](https://changelog.complete.org/archives/category/technology/software)[emacs](https://changelog.complete.org/archives/tag/emacs-s9y), [emacs2018](https://changelog.complete.org/archives/tag/emacs2018), [org-mode](https://changelog.complete.org/archives/tag/org-mode)[John Goerzen](http://changelog.complete.org)					

This is fourth in [a series on Emacs and org-mode](https://changelog.complete.org/archives/tag/emacs2018).

Hopefully by now you’ve started to see how powerful and useful org-mode is.  If you’re like me, you’re thinking:

“I’d really like to have this in sync across all my devices.”

and, perhaps:

“Can I forward emails into org-mode?”

This being Emacs, the answers, of course, are “Yes.”

**Syncing**

Since org-mode just uses text files, syncing is pretty easily  accomplished using any number of tools.  I use git with  git-remote-gcrypt.  Due to some limitations of git-remote-gcrypt, each  machine  tends to push to its own branch, and to master on command.   Each machine merges from all the other branches and pushes the result to master after a merge.  A cron job causes pushes to the machine’s branch to happen, and a bit of elisp coordinates it all — making sure to save  buffers before a sync, refresh them from disk after, etc.

The code for this post is somewhat more extended, so I will be linking to it on github rather than posting inline.

I have a directory $HOME/org where all my org-stuff lives.  In ~/org lives [a Makefile](https://github.com/jgoerzen/public-snippets/blob/master/emacs/org-tools/Makefile) that handles the syncing.  It defines these targets:

- push: adds, commits, and pushes to a branch named after the machine’s hostname
- fetch: does a simple git fetch
- sync: adds, commits, pulls remote changes, merges, and (assuming the merge was successful) pushes to the branch named after the machine’s  hostname plus master

Now, in my user’s crontab, I have this:

```
*/15   *   *  *   *      make -C $HOME/org push fetch 2>&1 | logger --tag 'orgsync'
```

The [accompanying elisp code](https://github.com/jgoerzen/public-snippets/blob/master/emacs/org-tools/emacs-config.org) defines a shortcut (C-c s) to cause a sync to occur.  Thanks to the  cronjob, as long as files were saved — even if I didn’t explicitly sync  on the other boxen — they’ll be pulled in.

I have found this setup to work really well.

**Emailing to org-mode**

Before going down this path, one should ask the question: do you  really need it?  I use org-mode with mu4e, and the integration is  excellent; any org task can link to an email by message-id, and this is  ideal — it lets a person do things like make a reminder to reply to a  message in a week.

However, org is not just about reminders.  It’s also a knowledge  base, authoring system, etc.  And, not all of my mail clients use mu4e.  (Note: things like MobileOrg exist for mobile devices).  I don’t  actually use this as much as I thought I would, but it has its uses and I thought I’d document it here too.

Now I didn’t want to just be able to accept plain text email.  I  wanted to be able to handle attachments, HTML mail, etc.  This quickly  starts to sound problematic — but with tools like ripmime and pandoc,  it’s not too bad.

The first step is to set up some way to get mail into a specific  folder.  A plus-extension, special user, whatever.  I then use a [fetchmail configuration](https://github.com/jgoerzen/public-snippets/blob/master/emacs/org-tools/fetchmailrc.orgmail) to pull it down and run my [insorgmail](https://github.com/jgoerzen/public-snippets/blob/master/emacs/org-tools/insorgmail) script.

This script is where all the interesting bits happen.  It starts with ripmime to process the message.  HTML bits are converted from HTML to  org format using pandoc.  an org hierarchy is made to represent the  structure of the email as best as possible.  emails can get pretty  complicated, with HTML and the rest, but I have found this does an  acceptable job with my use cases.

**Up next…**

My last post on org-mode will talk about using it to write documents  and prepare slides — a use for which I found myself surprisingly pleased with it, but which needed a bit of tweaking.

### Share this:

- [Facebook](https://changelog.complete.org/archives/9898-emacs-4-automated-emails-to-org-mode-and-org-mode-syncing?share=facebook&nb=1)
- [Twitter](https://changelog.complete.org/archives/9898-emacs-4-automated-emails-to-org-mode-and-org-mode-syncing?share=twitter&nb=1)
- [Reddit](https://changelog.complete.org/archives/9898-emacs-4-automated-emails-to-org-mode-and-org-mode-syncing?share=reddit&nb=1)
- 

​				[Leave a comment](https://changelog.complete.org/archives/9898-emacs-4-automated-emails-to-org-mode-and-org-mode-syncing#respond)			

# 			[Emacs #3: More on org-mode](https://changelog.complete.org/archives/9877-emacs-3-more-on-org-mode) 	

​			[March 2, 2018](https://changelog.complete.org/archives/9877-emacs-3-more-on-org-mode)[Software](https://changelog.complete.org/archives/category/technology/software)[emacs](https://changelog.complete.org/archives/tag/emacs-s9y), [emacs2018](https://changelog.complete.org/archives/tag/emacs2018), [org-mode](https://changelog.complete.org/archives/tag/org-mode)[John Goerzen](http://changelog.complete.org)					

This is third in [a series on Emacs and org-mode](https://changelog.complete.org/archives/tag/emacs2018).

**Todo tracking and keywords**

When using org-mode to [track your TODOs](https://orgmode.org/guide/TODO-Items.html#TODO-Items), it can have multiple states.  You can press `C-c C-t` for a quick shift between states.  I have set this:

```
(setq org-todo-keywords '(
  (sequence "TODO(t!)" "NEXT(n!)" "STARTED(a!)" "WAIT(w@/!)" "OTHERS(o!)" "|" "DONE(d)" "CANCELLED(c)")
))
```

Here, I set up 5 states that are for a task that is not yet done:  TODO, NEXT, STARTED, WAIT, and OTHERS.  Each has a single-character  shortcut (t, n, a, etc).  The states after the pipe symbol are ones that are considered “done”.  I have two: DONE (for things that I have done)  and CANCELED (for things that I haven’t done, but for whatever reason,  won’t).

The exclamation mark means to log the time when an item was changed  to a state.  I don’t add this to the done states because those are  already logged anyhow.  The @ sign means to prompt for a reason; so when switching to WAIT, org-mode will ask me why and add this to the note.

Here’s an example of an entry that has had some state changes:

```
** DONE This is a test
   CLOSED: [2018-03-02 Fri 03:05]
  
   - State "DONE"       from "WAIT"       [2018-03-02 Fri 03:05]
   - State "WAIT"       from "TODO"       [2018-03-02 Fri 03:05] \\
     waiting for pigs to fly
   - State "TODO"       from "NEXT"       [2018-03-02 Fri 03:05]
   - State "NEXT"       from "TODO"       [2018-03-02 Fri 03:05]
```

Here, the most recent items are on top.

**Agenda mode, schedules, and deadlines**

When you’re in a todo item, C-c C-s or C-c C-d can set a schedule or a deadline for it, respectively.  These show up in agenda mode.  The  difference is in intent and presentation.  A schedule is something that  you expect to work on at around a time, while a deadline is something  that is due at a specific time.  By default, the agenda view will start  warning you about deadline items in advance.

And while we’re at it, the [agenda view](https://orgmode.org/guide/Agenda-Views.html#Agenda-Views) will show you the items that you have coming up, offers a nice way to  search for items based on plain text or tags, and handles bulk  manipulation of items even across multiple files.  I covered setting the files for agenda mode in [part 2](https://changelog.complete.org/archives/9865-emacs-2-introducing-org-mode) of this series.

**Tags**

Of course org-mode has [tags](https://orgmode.org/guide/Tags.html#Tags).  You can quickly set them with C-c C-q.

You can set shortcuts for tags you might like to use often.  Perhaps something like this:

```
  (setq org-tag-persistent-alist 
        '(("@phone" . ?p) 
          ("@computer" . ?c) 
          ("@websurfing" . ?w)
          ("@errands" . ?e)
          ("@outdoors" . ?o)
          ("MIT" . ?m)
          ("BIGROCK" . ?b)
          ("CONTACTS" . ?C)
          ("INBOX" . ?i)
          ))
```

You can also add tags to this list on a per-file basis, and also set  tags for something on a per-file basis.  I use that for my inbox.org and email.org files to set an INBOX tag.  I can then review all items  tagged INBOX from the agenda view each day, and the simple act of  refiling them into other files will cause them to lost the INBOX tag.

**Refiling**

“Refiling” is moving things around, either within a file or elsewhere.  It has completion using your headlines.  `C-c C-w` does this.  I like these settings:

```
(setq org-outline-path-complete-in-steps nil)         ; Refile in a single go
(setq org-refile-use-outline-path 'file)
```

**Archiving**

After awhile, you’ll get your files all cluttered with things that are done.  org-mode has an [archive](https://orgmode.org/guide/Archiving.html#Archiving) feature to move things out of your main .org files and into some other  files for future reference.  If you have your org files in git or  something, you may wish to delete these other files since you’d have  things in history anyhow, but I find them handy for grepping and  searching.

I periodically want to go through and archive everything in my files.  Based on a [stackoverflow discussion](https://stackoverflow.com/questions/6997387/how-to-archive-all-the-done-tasks-using-a-single-command), I have this code:

```
(defun org-archive-done-tasks ()
  (interactive)
  (org-map-entries
   (lambda ()
     (org-archive-subtree)
     (setq org-map-continue-from (outline-previous-heading)))
   "/DONE" 'file)
  (org-map-entries
   (lambda ()
     (org-archive-subtree)
     (setq org-map-continue-from (outline-previous-heading)))
   "/CANCELLED" 'file)
)
```

This is based on [a particular answer](https://stackoverflow.com/a/27043756) — see the comments there for some additional hints.  Now you can run `M-x org-archive-done-tasks` and everything in the current file marked DONE or CANCELED will be pulled out into a different file.

**Up next**

I’ll wrap up org-mode with a discussion of automatically receiving emails into org, and syncing org between machines.

**Resources to accompany this article**

- [Printable org-mode reference card](https://orgmode.org/orgcard.pdf)

### Share this:

- [Facebook](https://changelog.complete.org/archives/9877-emacs-3-more-on-org-mode?share=facebook&nb=1)
- [Twitter](https://changelog.complete.org/archives/9877-emacs-3-more-on-org-mode?share=twitter&nb=1)
- [Reddit](https://changelog.complete.org/archives/9877-emacs-3-more-on-org-mode?share=reddit&nb=1)
- 

​				[View all 2 comments](https://changelog.complete.org/archives/9877-emacs-3-more-on-org-mode#comments)			

# 			[Emacs #2: Introducing org-mode](https://changelog.complete.org/archives/9865-emacs-2-introducing-org-mode) 	

​			[February 28, 2018](https://changelog.complete.org/archives/9865-emacs-2-introducing-org-mode)[Software](https://changelog.complete.org/archives/category/technology/software)[emacs](https://changelog.complete.org/archives/tag/emacs-s9y), [emacs2018](https://changelog.complete.org/archives/tag/emacs2018), [org-mode](https://changelog.complete.org/archives/tag/org-mode)[John Goerzen](http://changelog.complete.org)					

In [my first post in my series on Emacs](https://changelog.complete.org/archives/9861-emacs-1-ditching-a-bunch-of-stuff-and-moving-to-emacs-and-org-mode), I described returning to Emacs after over a decade of vim, and org-mode being the reason why.

I really am astounded at the usefulness, and simplicity, of org-mode.  It is really a killer app.

**So what exactly is org-mode?**

I wrote yesterday:

> It’s an information organization platform. Its website  says “Your life in plain text: Org mode is for keeping notes,  maintaining TODO lists, planning projects, and authoring documents with a fast and effective plain-text system.”

That’s true, but doesn’t quite capture it.  org-mode is a toolkit for you to organize things.  It has reasonable out-of-the-box defaults, but it’s designed throughout for you to customize.

To highlight a few things:

- **Maintaining TODO lists**: items can be scattered across  org-mode files, contain attachments, have tags, deadlines, schedules.   There is a convenient “agenda” view to show you what needs to be done.   Items can repeat.
- **Authoring documents**: org-mode has special features for  generating HTML, LaTeX, slides (with LaTeX beamer), and all sorts of  other formats.  It also supports direct evaluation of code in-buffer and literate programming in virtually any Emacs-supported language.  If you want to bend your mind on this stuff, read [this article on literate devops](http://www.howardism.org/Technical/Emacs/literate-devops.html).  The [entire Worg website](https://orgmode.org/worg/)
   is made with org-mode.
- **Keeping notes**: yep, it can do that too.  With full-text  search, cross-referencing by file (as a wiki), by UUID, and even into  other systems (into mu4e by Message-ID, into ERC logs, etc, etc.)

**Getting started**

I highly recommend watching [Carsten Dominik’s excellent Google Talk on org-mode](https://www.youtube.com/watch?v=oJTwQvgfgMM).  It is an excellent introduction.

org-mode is included with Emacs, but you’ll often want a more recent version.  Debian users can `apt-get install org-mode`, or it comes with the Emacs packaging system; `M-x package-install RET org-mode RET` may do it for you.

Now, you’ll probably want to start with the org-mode compact guide’s [introduction section](https://orgmode.org/guide/Introduction.html#Introduction), noting in particular to set the keybindings mentioned in the [activation section](https://orgmode.org/guide/Activation.html#Activation).

**A good tutorial…**

I’ve linked to a number of excellent tutorials and introductory  items; this post is not going to serve as a tutorial.  There are two  good videos linked at the end of this post, in particular.

**Some of my configuration**

I’ll document some of my configuration here, and go into a bit of  what it does.  This isn’t necessarily because you’ll want to copy all of this verbatim — but just to give you a bit of an idea of some of what  can be configured, an idea of what to look up in the manual, and maybe a reference for “now how do I do that?”  

First, I set up Emacs to work in UTF-8 by default.
 ` (prefer-coding-system 'utf-8) (set-language-environment "UTF-8") `

org-mode can follow URLs.  By default, it opens in Firefox, but I use Chromium.
 ` (setq browse-url-browser-function 'browse-url-chromium) `

I set the basic key bindings as documented in the Guide, plus configure the M-RET behavior.

```
(global-set-key "\C-cl" 'org-store-link) (global-set-key "\C-ca" 'org-agenda) (global-set-key "\C-cc" 'org-capture) (global-set-key "\C-cb" 'org-iswitchb)
 
(setq org-M-RET-may-split-line nil) 
```

**Configuration: Capturing**

I can press `C-c c` from anywhere in Emacs.  It will [capture something for me](https://orgmode.org/guide/Capture.html#Capture), and include a link back to whatever I was working on.

You can define [capture templates](https://orgmode.org/guide/Capture-templates.html#Capture-templates) to set how this will work.  I am going to keep two journal files for  general notes about meetings, phone calls, etc.  One for personal, one  for work items.  If I press `C-c c j`, then it will capture a personal item.  The %a in all of these includes the link to where I was (or a link I had stored with `C-c l`).

```
(setq org-default-notes-file "~/org/tasks.org")
(setq org-capture-templates
      '(
        ("t" "Todo" entry (file+headline "inbox.org" "Tasks")
         "* TODO %?\n  %i\n  %u\n  %a")
        ("n" "Note/Data" entry (file+headline "inbox.org" "Notes/Data")
         "* %?   \n  %i\n  %u\n  %a")
        ("j" "Journal" entry (file+datetree "~/org/journal.org")
         "* %?\nEntered on %U\n %i\n %a")
        ("J" "Work-Journal" entry (file+datetree "~/org/wjournal.org")
         "* %?\nEntered on %U\n %i\n %a")
        ))
(setq org-irc-link-to-logs t)
```

I like to link by UUIDs, which lets me move things between files  without breaking locations.  This helps generate UUIDs when I ask Org to store a link target for future insertion.

```
 (require 'org-id) (setq org-id-link-to-org-use-id 'create-if-interactive) 
```

**Configuration: agenda views**

I like my week to start on a Sunday, and for org to note the time when I mark something as done.

```
 (setq org-log-done 'time) (setq org-agenda-start-on-weekday 0) 
```

**Configuration: files and refiling**

Here I tell it what files to use in the agenda, and to add a few more to the plain text search.  I like to keep a general inbox (from which I can move, or “refile”, content), and then separate tasks, journal, and  knowledge base for personal and work items.

```
  (setq org-agenda-files (list "~/org/inbox.org"
                               "~/org/email.org"
                               "~/org/tasks.org"
                               "~/org/wtasks.org"
                               "~/org/journal.org"
                               "~/org/wjournal.org"
                               "~/org/kb.org"
                               "~/org/wkb.org"
  ))
  (setq org-agenda-text-search-extra-files
        (list "~/org/someday.org"
              "~/org/config.org"
  ))

  (setq org-refile-targets '((nil :maxlevel . 2)
                             (org-agenda-files :maxlevel . 2)
                             ("~/org/someday.org" :maxlevel . 2)
                             ("~/org/templates.org" :maxlevel . 2)
                             )
        )
(setq org-outline-path-complete-in-steps nil)         ; Refile in a single go
(setq org-refile-use-outline-path 'file)
```

**Configuration: Appearance**

I like a pretty screen.  After you’ve gotten used to org a bit, you might try this.

```
(require 'org-bullets)
(add-hook 'org-mode-hook
          (lambda ()
            (org-bullets-mode t)))
(setq org-ellipsis "")
```

**Coming up next…**

This hopefully showed a few things that org-mode can do.  Coming up  next, I’ll cover how to customize TODO keywords and tags, archiving old  tasks, forwarding emails to org-mode, and using git to synchronize  between machines.

You can also see a [list of all articles in this series](https://changelog.complete.org/archives/tag/emacs2018).

**Resources to accompany this article**

- [org-mode compact guide](https://orgmode.org/guide/)
- [Full org manual](https://orgmode.org/org.html)
- [List of tutorials](https://orgmode.org/worg/org-tutorials/)
- [Carsten Dominik’s excellent Google Talk on org-mode](https://www.youtube.com/watch?v=oJTwQvgfgMM)
- A great intro: [“My workflow with org-agenda”](http://cachestocaches.com/2016/9/my-workflow-org-agenda/)
- [A more detailed intro](http://doc.norang.ca/org-mode.html)
- [David O’Toole’s org-mode tutorial](https://orgmode.org/worg/org-tutorials/orgtutorial_dto.html)
- [Getting Started with org-mode video by Harry Schwartz](https://www.youtube.com/watch?v=SzA2YODtgK4) (another excellent watch).  He also has his [example config files](https://github.com/hrs/dotfiles/blob/master/emacs/.emacs.d/configuration.org) available.

### Share this:

- [Facebook](https://changelog.complete.org/archives/9865-emacs-2-introducing-org-mode?share=facebook&nb=1)
- [Twitter](https://changelog.complete.org/archives/9865-emacs-2-introducing-org-mode?share=twitter&nb=1)
- [Reddit](https://changelog.complete.org/archives/9865-emacs-2-introducing-org-mode?share=reddit&nb=1)
- 

​				[View all 12 comments](https://changelog.complete.org/archives/9865-emacs-2-introducing-org-mode#comments)			

# 			[Emacs #1: Ditching a bunch of stuff and moving to Emacs and org-mode](https://changelog.complete.org/archives/9861-emacs-1-ditching-a-bunch-of-stuff-and-moving-to-emacs-and-org-mode) 	

​			[February 27, 2018](https://changelog.complete.org/archives/9861-emacs-1-ditching-a-bunch-of-stuff-and-moving-to-emacs-and-org-mode)[Software](https://changelog.complete.org/archives/category/technology/software)[emacs](https://changelog.complete.org/archives/tag/emacs-s9y), [emacs2018](https://changelog.complete.org/archives/tag/emacs2018), [org-mode](https://changelog.complete.org/archives/tag/org-mode)[John Goerzen](http://changelog.complete.org)					

I’ll admit it.  After over a decade of vim, I’m hooked on [Emacs](https://www.gnu.org/software/emacs/).

I’ve long had this frustration over how to organize things.  I’ve followed approaches like [GTD](https://gettingthingsdone.com/) and [ZTD](https://zenhabits.net/zen-to-done-the-simple-productivity-e-book/), but things like email or large files are really hard to organize.

I had been using Asana for tasks, Evernote for notes, Thunderbird for email, a combination of ikiwiki and some other items for a personal  knowledge base, and various files in an archive directory on my PC.   When my new job added Slack to the mix, that was finally the last straw.

A lot of todo-management tools integrate with email — poorly.  When  you want to do something like “remind me to reply to this in a week”, a  lot of times that’s impossible because the tool doesn’t store the email  in a fashion you can easily reply to.  And that problem is even worse  with Slack.

It was right around then that I stumbled onto [Carsten Dominik’s Google Talk on org-mode](https://www.youtube.com/watch?v=oJTwQvgfgMM).  Carsten was the author of org-mode, and although the talk is 10 years old, it is still highly relevant.

I’d stumbled across [org-mode](https://orgmode.org/)  before, but each time I didn’t really dig in because I had the reaction  of “an outliner?  But I need a todo list.”  Turns out I was missing out.  org-mode is all that.

**Just what IS Emacs?  And org-mode?**

Emacs grew up as a text editor.  It still is, and that heritage is  definitely present throughout.  But to say Emacs is an editor would be  rather unfair.

Emacs is something more like a platform or a toolkit.  Not only do  you have source code to it, but the very configuration is a program, and there are hooks all over the place.  It’s as if it was super easy to  write a Firefox plugin.  A couple lines, and boom, behavior changed.

org-mode is very similar.  Yes, it’s an outliner, but that’s not  really what it is.  It’s an information organization platform.  Its  website says “Your life in plain text: Org mode is for keeping notes,  maintaining TODO lists, planning projects, and authoring documents with a fast and effective plain-text system.”

**Capturing**

If you’ve ever read productivity guides based on GTD, one of the  things they stress is effortless capture of items.  The idea is that  when something pops into your head, get it down into a trusted system  quickly so you can get on with what you were doing.  org-mode has a  capture system for just this.  I can press `C-c c` from anywhere  in Emacs, and up pops a spot to type my note.  But, critically,  automatically embedded in that note is a link back to what I was doing  when I pressed `C-c c`.  If I was editing a file, it’ll have a  link back to that file and the line I was on.  If I was viewing an  email, it’ll link back to that email (by Message-Id, no less, so it  finds it in any folder).  Same for participating in a chat, or even  viewing another org-mode entry.

So I can make a note that will remind me in a week to reply to a  certain email, and when I click the link in that note, it’ll bring up  the email in my mail reader — even if I subsequently archived it out of  my inbox.

YES, this is what I was looking for!

**The tool suite**

Once you’re using org-mode, pretty soon you want to integrate  everything with it.  There are browser plugins for capturing things from the web.  Multiple Emacs mail or news readers integrate with it.  ERC  (IRC client) does as well.  So I found myself switching from Thunderbird and mairix+mutt (for the mail archives) to mu4e, and from xchat+slack  to ERC.

And wouldn’t you know it, I liked each of those Emacs-based tools **better** than the standalone they replaced.

A small side tidbit: I’m using OfflineIMAP again!  I even used it with GNUS way back when.

**One Emacs process to rule them**

I used to use Emacs extensively, way back.  Back then, Emacs was a  “large” program.  (Now my battery status applet literally uses more RAM  than Emacs).  There was this problem of startup time back then, so there was a way to connect to a running Emacs process.

I like to spawn programs with Mod-p (an xmonad shortcut to a dzen  menubar, but Alt-F2 in more traditional DEs would do the trick).  It’s  convenient to not run several emacsen with this setup, so you don’t run  into issues with trying to capture to a file that’s open in another one.  The solution is very simple: I created a script, named it `em`, and put it on my path.  All it does is this:

```
 #!/bin/bash exec emacsclient -c -a "" "$@" 
```

It creates a new emacs process if one doesn’t already exist;  otherwise, it uses what you’ve got.  A bonus here: parameters such as `-nw` work just fine, so it really acts just as if you’d typed `emacs` at the shell prompt.  It’s a suitable setting for `EDITOR`.

**Up next…**

I’ll be talking about my use of, and showing off configurations for:

- org-mode, including syncing between computers, capturing, agenda and todos, files, linking, keywords and tags, various exporting  (slideshows), etc.
- mu4e for email, including multiple accounts, bbdb integration
- ERC for IRC and IM

You can also see a [list of all articles in this series](https://changelog.complete.org/archives/tag/emacs2018).