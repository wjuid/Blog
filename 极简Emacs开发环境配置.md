# 极简Emacs开发环境配置

## Table of Contents

- Minimal Emacs
  - Environment
    - [Packages](https://huadeyu.tech/tools/emacs-setup-notes.html#org1b3f247)
    - [Encoding and Envs](https://huadeyu.tech/tools/emacs-setup-notes.html#orgc96fa4b)
    - [Feature Mode](https://huadeyu.tech/tools/emacs-setup-notes.html#orgad3e63e)
    - [File Operation](https://huadeyu.tech/tools/emacs-setup-notes.html#orgccca1fa)
    - [History](https://huadeyu.tech/tools/emacs-setup-notes.html#org5654618)
    - [MacOS System](https://huadeyu.tech/tools/emacs-setup-notes.html#org9c072db)
    - [Performance](https://huadeyu.tech/tools/emacs-setup-notes.html#org2e10bdf)
  - Appearance
    - [Titlebar](https://huadeyu.tech/tools/emacs-setup-notes.html#org393e171)
    - [Doom Theme](https://huadeyu.tech/tools/emacs-setup-notes.html#orge381861)
    - [Rainbow](https://huadeyu.tech/tools/emacs-setup-notes.html#org3043099)
  - Tools
    - [Undo Tree](https://huadeyu.tech/tools/emacs-setup-notes.html#org65efebf)
    - [AutoCompany](https://huadeyu.tech/tools/emacs-setup-notes.html#orgc9d7a19)
    - [Display Keybind](https://huadeyu.tech/tools/emacs-setup-notes.html#org20216c0)
    - [Recent File](https://huadeyu.tech/tools/emacs-setup-notes.html#org44c0d91)
    - [Line Number](https://huadeyu.tech/tools/emacs-setup-notes.html#orgafd68d7)
    - [Auto Pair Bracket](https://huadeyu.tech/tools/emacs-setup-notes.html#orgaba5af3)
    - [Neotree Sidebar](https://huadeyu.tech/tools/emacs-setup-notes.html#org88c3bec)
    - [Git Tool](https://huadeyu.tech/tools/emacs-setup-notes.html#orgcfcee7e)
    - [Sinppet Management](https://huadeyu.tech/tools/emacs-setup-notes.html#org27bee21)
    - [Smart Tab](https://huadeyu.tech/tools/emacs-setup-notes.html#orgba71790)
    - [Helm](https://huadeyu.tech/tools/emacs-setup-notes.html#org262995e)
    - [Fuzzy Searcha](https://huadeyu.tech/tools/emacs-setup-notes.html#org97d1153)
    - [Smart Move](https://huadeyu.tech/tools/emacs-setup-notes.html#org9f777e2)
    - [All The Icons](https://huadeyu.tech/tools/emacs-setup-notes.html#orga388301)
    - [Treemacs](https://huadeyu.tech/tools/emacs-setup-notes.html#orga1adbf9)
  - Programming
    - [Lsp Mode](https://huadeyu.tech/tools/emacs-setup-notes.html#org585d908)
    - [Golang](https://huadeyu.tech/tools/emacs-setup-notes.html#org80be74c)
    - [Python](https://huadeyu.tech/tools/emacs-setup-notes.html#org522f045)
    - [Webdev](https://huadeyu.tech/tools/emacs-setup-notes.html#org8320a5f)
    - [Json](https://huadeyu.tech/tools/emacs-setup-notes.html#org70af1a8)
    - [Yaml](https://huadeyu.tech/tools/emacs-setup-notes.html#orgf05e1c9)
    - [Dockfile](https://huadeyu.tech/tools/emacs-setup-notes.html#org9867449)
    - [Protobuf](https://huadeyu.tech/tools/emacs-setup-notes.html#orgba786ef)
  - OrgMode
    - [Org Publish](https://huadeyu.tech/tools/emacs-setup-notes.html#org1f969c4)
  - [Functions](https://huadeyu.tech/tools/emacs-setup-notes.html#orgadc5a5e)
  - [Keybind](https://huadeyu.tech/tools/emacs-setup-notes.html#orga93575f)

## Minimal Emacs



### Environment



#### Packages

```
(require 'package)
(setq package-enable-at-startup nil)

;; Add useful package repo
(unless (assoc-default "melpa" package-archives)
  (add-to-list 'package-archives '("melpa" . "http://mirrors.tuna.tsinghua.edu.cn/elpa/melpa/") t))
(unless (assoc-default "org" package-archives)
  (add-to-list 'package-archives '("org" . "http://mirrors.tuna.tsinghua.edu.cn/elpa/org/") t))
(unless (assoc-default "marmalade" package-archives)
  (add-to-list 'package-archives '("marmalade" . "http://mirrors.tuna.tsinghua.edu.cn/elpa/marmalade/")))
(unless (assoc-default "gnu" package-archives)
  (add-to-list 'package-archives '("gnu" . "http://mirrors.tuna.tsinghua.edu.cn/elpa/gnu/")))
(package-initialize)

;; Install use-package if not available
(unless (package-installed-p 'use-package)
  (package-refresh-contents)
  (package-install 'use-package))

(eval-when-compile
  (require 'use-package))
(setq use-package-verbose t)
(setq use-package-always-ensure t)
(setq warning-minimum-level :emergency)
```

#### Encoding and Envs

```
(prefer-coding-system 'utf-8)

(setenv "LANG" "en_US.UTF-8")
(setenv "LC_ALL" "en_US.UTF-8")
(setenv "LC_CTYPE" "en_US.UTF-8")
```

#### Feature Mode

```
(display-time-mode 1)
(column-number-mode 1)
(show-paren-mode nil)
(display-battery-mode 1)
(tool-bar-mode -1)
(menu-bar-mode -1)
(toggle-scroll-bar -1)
(global-auto-revert-mode t)
(global-hl-line-mode nil)

(fset 'yes-or-no-p 'y-or-n-p)
(toggle-frame-fullscreen)
```

#### File Operation

```
(setq tab-width 4
      inhibit-splash-screen t
      initial-scratch-message nil
      sentence-end-double-space nil
      make-backup-files nil
      indent-tabs-mode nil
      make-backup-files nil
      auto-save-default nil)
(setq create-lockfiles nil)
```

#### History

```
(savehist-mode 1)
(setq savehist-file "~/.emacs.d/.savehist")
(setq history-length t)
(setq history-delete-duplicates t)
(setq savehist-save-minibuffer-history 1)
(setq savehist-additional-variables
      '(kill-ring
        search-ring
        regexp-search-ring))
```

#### MacOS System

```
(cond ((string-equal system-type "darwin")
       (progn
         ;; modify option and command key
         (setq mac-command-modifier 'control)
         (setq mac-option-modifier 'meta)

         ;; batter copy and paste support for mac os x
         (defun copy-from-osx ()
           (shell-command-to-string "pbpaste"))
         (defun paste-to-osx (text &optional push)
           (let ((process-connection-type nil))
             (let ((proc (start-process "pbcopy" "*Messages*" "pbcopy")))
               (process-send-string proc text)
               (process-send-eof proc))))
         (setq interprogram-cut-function 'paste-to-osx)
         (setq interprogram-paste-function 'copy-from-osx)

         (use-package exec-path-from-shell)
         (when (memq window-system '(mac ns x))
           (exec-path-from-shell-initialize))

         (message "Wellcome To Mac OS X, Have A Nice Day!!!"))))
```

#### Performance

```
(if (not (display-graphic-p))
    (progn
      ;; 增大垃圾回收的阈值，提高整体性能（内存换效率）
      (setq gc-cons-threshold (* 8192 8192))
      ;; 增大同LSP服务器交互时的读取文件的大小
      (setq read-process-output-max (* 1024 1024 128)) ;; 128MB
      ))
```

### Appearance



#### Titlebar

```
(add-to-list 'default-frame-alist '(ns-transparent-titlebar . t))
(add-to-list 'default-frame-alist '(ns-appearance . dark))

(let ((display-table (or standard-display-table (make-display-table))))
  (set-display-table-slot display-table 'vertical-border (make-glyph-code ?│)) ; or ┃ │
  (setq standard-display-table display-table))
(set-face-background 'vertical-border (face-background 'default))
(set-face-foreground 'vertical-border "grey")
```

#### Doom Theme

```
 (setq custom-safe-themes t)

;; doom theme enable
 (use-package doom-themes
   :config
   ;; Global settings (defaults)
   (setq doom-themes-enable-bold t    ; if nil, bold is universally disabled
         doom-themes-enable-italic t) ; if nil, italics is universally disabled
   ;; Enable flashing mode-line on errors
   (doom-themes-visual-bell-config)

   (if (display-graphic-p)
       (progn
         ;; Enable custom neotree theme (all-the-icons must be installed!)
         (doom-themes-neotree-config)
         ;; or for treemacs users
         (setq doom-themes-treemacs-theme "doom-colors") ; use the colorful treemacs theme
         (doom-themes-treemacs-config)
         ))

   ;; Corrects (and improves) org-mode's native fontification.
   (doom-themes-org-config))

(if (string-equal system-type "darwin")
    (use-package darkokai-theme
      :ensure t
      :config (load-theme 'darkokai t))
  (use-package doom-themes
    :ensure t
    :config (load-theme 'doom-nord))
  )


;; nerd-icons
;;(add-to-list 'load-path (expand-file-name "~/.emacs.d/site-lisp/nerd-icons"))
;;(require 'nerd-icons)

;; modeline
 (use-package doom-modeline
   :ensure t
   :hook (after-init . doom-modeline-mode))

 (set-face-background 'mode-line nil)
```

#### Rainbow

```
(use-package rainbow-mode
  :config
  (progn
    (defun @-enable-rainbow ()
      (rainbow-mode t))
    (add-hook 'prog-mode-hook '@-enable-rainbow)
))
(use-package rainbow-delimiters
  :config
  (progn
    (defun @-enable-rainbow-delimiters ()
      (rainbow-delimiters-mode t))
    (add-hook 'prog-mode-hook '@-enable-rainbow-delimiters)))
(if (display-graphic-p)
    (progn
      (set-face-attribute 'default nil
                          :family "LigaSauceCodeProMedium Nerd Font"
                          :height 130
                          :weight 'Normal)
      (dolist (charset '(kana han symbol cjk-misc bopomofo))
        (set-fontset-font (frame-parameter nil 'font)
                          charset (font-spec :family "Microsoft Yahei"
                                             :size 13)))
      (let ((alist '((33 . ".\\(?:\\(?:==\\|!!\\)\\|[!=]\\)")
                     (35 . ".\\(?:###\\|##\\|_(\\|[#(?[_{]\\)")
                     (36 . ".\\(?:>\\)")
                     (37 . ".\\(?:\\(?:%%\\)\\|%\\)")
                     (38 . ".\\(?:\\(?:&&\\)\\|&\\)")
                     (42 . ".\\(?:\\(?:\\*\\*/\\)\\|\\(?:\\*[*/]\\)\\|[*/>]\\)")
                     (43 . ".\\(?:\\(?:\\+\\+\\)\\|[+>]\\)")
                     (45 . ".\\(?:\\(?:-[>-]\\|<<\\|>>\\)\\|[<>}~-]\\)")
                     (46 . ".\\(?:\\(?:\\.[.<]\\)\\|[.=-]\\)")
                     (47 . ".\\(?:\\(?:\\*\\*\\|//\\|==\\)\\|[*/=>]\\)")
                     (48 . ".\\(?:x[a-zA-Z]\\)")
                     (58 . ".\\(?:::\\|[:=]\\)")
                     (59 . ".\\(?:;;\\|;\\)")
                     (60 . ".\\(?:\\(?:!--\\)\\|\\(?:~~\\|->\\|\\$>\\|\\*>\\|\\+>\\|--\\|<[<=-]\\|=[<=>]\\||>\\)\\|[*$+~/<=>|-]\\)")
                     (61 . ".\\(?:\\(?:/=\\|:=\\|<<\\|=[=>]\\|>>\\)\\|[<=>~]\\)")
                     (62 . ".\\(?:\\(?:=>\\|>[=>-]\\)\\|[=>-]\\)")
                     (63 . ".\\(?:\\(\\?\\?\\)\\|[:=?]\\)")
                     (91 . ".\\(?:]\\)")
                     (92 . ".\\(?:\\(?:\\\\\\\\\\)\\|\\\\\\)")
                     (94 . ".\\(?:=\\)")
                     (119 . ".\\(?:ww\\)")
                     (123 . ".\\(?:-\\)")
                     (124 . ".\\(?:\\(?:|[=|]\\)\\|[=>|]\\)")
                     (126 . ".\\(?:~>\\|~~\\|[>=@~-]\\)")
                     )
                   ))
        (dolist (char-regexp alist)
          (set-char-table-range composition-function-table (car char-regexp)
                                `([,(cdr char-regexp) 0 font-shape-gstring]))))
      ))
```

### Tools



#### Undo Tree

```
(use-package undo-tree
  :ensure t
  :config
  (progn
    (global-undo-tree-mode)
    (setq undo-tree-visualizer-timestamps t)
    (setq undo-tree-visualizer-diff t)
    ))
```

#### AutoCompany

```
(use-package company
  :ensure t
  :config
  (progn
    (add-hook 'after-init-hook 'global-company-mode)))
```

#### Display Keybind

```
(use-package which-key
  :config
  (progn
    (which-key-mode)
    (which-key-setup-side-window-bottom)))
```

#### Recent File

```
(use-package recentf
  :config
  (progn
    (setq recentf-max-saved-items 200
          recentf-max-menu-items 15)
    (recentf-mode)
    ))
```

#### Line Number

```
(use-package linum
  :init
  (progn
    (global-linum-mode t)
    (setq linum-format "%4d  ")
      (set-face-background 'linum nil)
    ))
```

#### Auto Pair Bracket

```
(use-package autopair
  :config (autopair-global-mode))
```

#### Neotree Sidebar

```
(use-package neotree
  :custom
  (neo-theme 'nerd2)
  :config
  (progn
    (setq neo-smart-open t)
    (setq neo-theme (if (display-graphic-p) 'icons 'nerd))
    (setq neo-window-fixed-size nil)
    ;; (setq-default neo-show-hidden-files nil)
    (global-set-key [f2] 'neotree-toggle)
    (global-set-key [f8] 'neotree-dir)))
```

#### Git Tool

```
(use-package magit)

(use-package git-gutter+
  :ensure t
  :config
  (progn
    (global-git-gutter+-mode)))
```

#### Sinppet Management

```
(use-package yasnippet
  :diminish yas-minor-mode
  :init (yas-global-mode)
  :config
  (progn
    (yas-global-mode)
    (add-hook 'hippie-expand-try-functions-list 'yas-hippie-try-expand)
    (setq yas-key-syntaxes '("w_" "w_." "^ "))
    ;; (setq yas-installed-snippets-dir "~/elisp/yasnippet-snippets")
    (setq yas-expand-only-for-last-commands nil)
    (yas-global-mode 1)
    (bind-key "\t" 'hippie-expand yas-minor-mode-map)
    (add-to-list 'yas-prompt-functions 'shk-yas/helm-prompt)))

(dolist (command '(yank yank-pop))
  (eval
   `(defadvice ,command (after indent-region activate)
      (and (not current-prefix-arg)
           (member major-mode
                   '(emacs-lisp-mode
                     lisp-mode
                     clojure-mode
                     scheme-mode
                     haskell-mode
                     ruby-mode
                     rspec-mode
                     python-mode
                     c-mode
                     c++-mode
                     objc-mode
                     latex-mode
                     js-mode
                     plain-tex-mode))
           (let ((mark-even-if-inactive transient-mark-mode))
             (indent-region (region-beginning) (region-end) nil))))))

(defun shk-yas/helm-prompt (prompt choices &optional display-fn)
  "Use helm to select a snippet. Put this into `yas-prompt-functions.'"
  (interactive)
  (setq display-fn (or display-fn 'identity))
  (if (require 'helm-config)
      (let (tmpsource cands result rmap)
        (setq cands (mapcar (lambda (x) (funcall display-fn x)) choices))
        (setq rmap (mapcar (lambda (x) (cons (funcall display-fn x) x)) choices))
        (setq tmpsource
              (list
               (cons 'name prompt)
               (cons 'candidates cands)
               '(action . (("Expand" . (lambda (selection) selection))))
               ))
        (setq result (helm-other-buffer '(tmpsource) "*helm-select-yasnippet"))
        (if (null result)
            (signal 'quit "user quit!")
          (cdr (assoc result rmap))))
    nil))
```

#### Smart Tab

```
(use-package smart-tab
  :config
  (progn
    (defun @-enable-smart-tab ()
      (smart-tab-mode))
    (add-hook 'prog-mode-hook '@-enable-smart-tab)
    ))
```

#### Helm

```
(use-package helm-swoop)
(use-package helm-gtags)
(use-package helm
  :diminish helm-mode
  :init
  (progn
    ;; (require 'helm-config)
    (setq helm-candidate-number-limit 100)
    ;; From https://gist.github.com/antifuchs/9238468
    (setq helm-idle-delay 0.0 ; update fast sources immediately (doesn't).
          helm-input-idle-delay 0.01  ; this actually updates things
                                        ; reeeelatively quickly.
          helm-yas-display-key-on-candidate t
          helm-quick-update t
          helm-M-x-requires-pattern nil
          helm-ff-skip-boring-files t)
    (helm-mode))
  :config
  (progn
    )
  :bind  (("C-c s" . helm-swoop)
          ("C-x C-f" . helm-find-files)
          ("C-x b" . helm-buffers-list)
          ("M-y" . helm-show-kill-ring)
          ("M-x" . helm-M-x)))
```

#### Fuzzy Searcha

```
(use-package fiplr)
```

#### Smart Move

```
(use-package mwim
  :bind
  ("C-a" . mwim-beginning-of-code-or-line)
  ("C-e" . mwim-end-of-code-or-line))
```

#### All The Icons

```
(use-package all-the-icons
  :after memoize
  :load-path "site-lisp/all-the-icons")
```

#### Treemacs

```
(use-package treemacs
  :ensure t
  :defer t
  :init
  (with-eval-after-load 'winum
    (define-key winum-keymap (kbd "M-0") #'treemacs-select-window))
  :config
  (progn
    (setq treemacs-collapse-dirs                 (if treemacs-python-executable 3 0)
          treemacs-deferred-git-apply-delay      0.5
          treemacs-directory-name-transformer    #'identity
          treemacs-display-in-side-window        t
          treemacs-eldoc-display                 t
          treemacs-file-event-delay              5000
          treemacs-file-extension-regex          treemacs-last-period-regex-value
          treemacs-file-follow-delay             0.2
          treemacs-file-name-transformer         #'identity
          treemacs-follow-after-init             t
          treemacs-git-command-pipe              ""
          treemacs-goto-tag-strategy             'refetch-index
          treemacs-indentation                   2
          treemacs-indentation-string            " "
          treemacs-is-never-other-window         nil
          treemacs-max-git-entries               5000
          treemacs-missing-project-action        'ask
          treemacs-no-png-images                 nil
          treemacs-no-delete-other-windows       t
          treemacs-project-follow-cleanup        nil
          treemacs-persist-file                  (expand-file-name ".cache/treemacs-persist" user-emacs-directory)
          treemacs-position                      'left
          treemacs-recenter-distance             0.1
          treemacs-recenter-after-file-follow    nil
          treemacs-recenter-after-tag-follow     nil
          treemacs-recenter-after-project-jump   'always
          treemacs-recenter-after-project-expand 'on-distance
          treemacs-show-cursor                   nil
          treemacs-show-hidden-files             t
          treemacs-silent-filewatch              nil
          treemacs-silent-refresh                nil
          treemacs-sorting                       'alphabetic-asc
          treemacs-space-between-root-nodes      t
          treemacs-tag-follow-cleanup            t
          treemacs-tag-follow-delay              1.5
          treemacs-user-mode-line-format         nil
          treemacs-width                         35)

    ;; The default width and height of the icons is 22 pixels. If you are
    ;; using a Hi-DPI display, uncomment this to double the icon size.
    ;;(treemacs-resize-icons 44)

    (treemacs-follow-mode t)
    (treemacs-filewatch-mode t)
    (treemacs-fringe-indicator-mode t)
    (pcase (cons (not (null (executable-find "git")))
                 (not (null treemacs-python-executable)))
      (`(t . t)
       (treemacs-git-mode 'deferred))
      (`(t . _)
       (treemacs-git-mode 'simple))))
  :bind
  (:map global-map
        ("M-0"       . treemacs-select-window)
        ("C-x t 1"   . treemacs-delete-other-windows)
        ("C-x t t"   . treemacs)
        ("C-x t B"   . treemacs-bookmark)
        ("C-x t C-t" . treemacs-find-file)
        ("C-x t M-t" . treemacs-find-tag)))

(use-package treemacs-evil
  :after treemacs evil
  :ensure t)

(use-package treemacs-projectile
  :after treemacs projectile
  :ensure t)

(use-package treemacs-icons-dired
  :after treemacs dired
  :ensure t
  :config (treemacs-icons-dired-mode))

(use-package treemacs-magit
  :after treemacs magit
  :ensure t)

(use-package treemacs-persp
  :after treemacs persp-mode
  :ensure t
  :config (treemacs-set-scope-type 'Perspectives))

(use-package lsp-treemacs
  :commands lsp-treemacs-errors-list
  :config
  (lsp-metals-treeview-enable t)
  (setq lsp-metals-treeview-show-when-views-received t))
```

### Programming



#### Lsp Mode

```
(use-package ccls
  :ensure t
  :config
  (setq ccls-executable (expand-file-name "~/.emacs.d/ccls"))
  )

;; (use-package eglot
  ;; :config
  ;; (add-hook 'prog-mode-hook 'eglot-ensure))

(use-package lsp-mode
  :ensure t
  :custom
  (lsp-enable-snippet t)
  (lsp-keep-workspace-alive t)
  (lsp-enable-xref t)
  (lsp-enable-imenu t)
  (lsp-enable-completion-at-point nil)

  :config  
  (add-hook 'go-mode-hook #'lsp)
  (add-hook 'python-mode-hook #'lsp)
  (add-hook 'c++-mode-hook #'lsp)
  (add-hook 'c-mode-hook #'lsp)
  (add-hook 'rust-mode-hook #'lsp)
  (add-hook 'html-mode-hook #'lsp)
  (add-hook 'js-mode-hook #'lsp)
  (add-hook 'typescript-mode-hook #'lsp)
  (add-hook 'json-mode-hook #'lsp)
  (add-hook 'yaml-mode-hook #'lsp)
  (add-hook 'dockerfile-mode-hook #'lsp)
  (add-hook 'shell-mode-hook #'lsp)
  (add-hook 'css-mode-hook #'lsp)

  (lsp-register-client
   (make-lsp-client :new-connection (lsp-stdio-connection "pyls")
                    :major-modes '(python-mode)
                    :server-id 'pyls))
  (setq company-minimum-prefix-length 1
        company-idle-delay 0.500) ;; default is 0.2
  (require 'lsp-clients) 
  :commands lsp)

(use-package company-lsp
  :ensure t
  :config
  (push 'company-lsp company-backends))

(use-package lsp-ui
  :ensure t
  :custom-face
  (lsp-ui-doc-background ((t (:background ni))))
  :init (setq lsp-ui-doc-enable t
              lsp-ui-doc-include-signature t               

              lsp-enable-snippet nil
              lsp-ui-sideline-enable nil
              lsp-ui-peek-enable nil

              lsp-ui-doc-position              'at-point
              lsp-ui-doc-header                nil
              lsp-ui-doc-border                "white"
              lsp-ui-doc-include-signature     t
              lsp-ui-sideline-update-mode      'point
              lsp-ui-sideline-delay            1
              lsp-ui-sideline-ignore-duplicate t
              lsp-ui-peek-always-show          t
              lsp-ui-flycheck-enable           nil
              )
  :bind (:map lsp-ui-mode-map
              ([remap xref-find-definitions] . lsp-ui-peek-find-definitions)
              ([remap xref-find-references] . lsp-ui-peek-find-references)
              ("C-c u" . lsp-ui-imenu))
  :config
  (setq lsp-ui-sideline-ignore-duplicate t)
  (add-hook 'lsp-mode-hook 'lsp-ui-mode))

(setq lsp-prefer-capf t)
```

#### Golang

```
(use-package go-mode
  :config
  (progn
    (setq gofmt-command "goimports")
    (add-hook 'before-save-hook 'gofmt-before-save)
    ))

;; (use-package auto-complete)
;; (use-package go-autocomplete
;;   :ensure t
;;   :config
;;   (require 'auto-complete-config)
;;   (ac-config-default)
;;   )

(when (memq window-system '(mac ns))
  (use-package exec-path-from-shell)
  (exec-path-from-shell-initialize)
  (exec-path-from-shell-copy-env "GOPATH"))

(use-package company-go
  :init
  (progn
    (setq company-go-show-annotation t)
    (setq company-tooltip-limit 20)                      ; bigger popup window
    (add-hook 'go-mode-hook 
              (lambda ()
                (set (make-local-variable 'company-backends) '(company-go))
                (company-mode)))
    )
  )

(use-package go-eldoc
  :config
  (progn
    (add-hook 'go-mode-hook 'go-eldoc-setup)
    ))

(use-package go-guru
  :defer t
  :hook (go-mode . go-guru-hl-identifier-mode))

;; go get -u -v golang.org/x/tools/cmd/...
;; go get -u -v github.com/rogpeppe/godef
;; go get -u -v golang.org/x/tools/cmd/goimports
;; go get -u -v golang.org/x/tools/gopls
;; go get -u -v github.com/mdempsky/gocode
```

#### Python

```
(use-package python
  :mode ("\\.py" . python-mode)
  :ensure t)

(use-package pyvenv)

(use-package python-black
  :demand t
  :after python
  :config
  (python-black-on-save-mode))

(use-package pyenv-mode
  :init
  (add-to-list 'exec-path "~/.pyenv/shims")
  (setenv "WORKON_HOME" "~/.pyenv/versions/")
  :config
  (pyenv-mode))
```

#### Webdev

```
;; web tools
(use-package emmet-mode)
;; (use-package web-mode
;;   :config
;;   (progn
;;     (defun @-web-mode-hook ()
;;       "Hooks for Web mode."
;;       (setq web-mode-markup-indent-offset 4)
;;       (setq web-mode-code-indent-offset 4)
;;       (setq web-mode-css-indent-offset 4))

;;     (add-to-list 'auto-mode-alist '("\\.ts\\'" . web-mode))
;;     (add-to-list 'auto-mode-alist '("\\.html?\\'" . web-mode))
;;     (add-to-list 'auto-mode-alist '("\\.css?\\'" . web-mode))
;;     (add-to-list 'auto-mode-alist '("\\.js\\'" . web-mode))

;;     (add-hook 'web-mode-hook  '@-web-mode-hook)    
;;     (setq tab-width 4)

;;     (add-hook 'web-mode-hook  'emmet-mode)))
(use-package web-beautify)

;; typescirpt tide
(use-package typescript-mode)
(use-package tide)

(defun setup-tide-mode ()
  (interactive)
  (tide-setup)
  (flycheck-mode +1)
  (setq flycheck-check-syntax-automatically '(save mode-enabled))
  (eldoc-mode +1)
  (tide-hl-identifier-mode +1)
  ;; company is an optional dependency. You have to
  ;; install it separately via package-install
  ;; `M-x package-install [ret] company`
  (company-mode +1))

;; aligns annotation to the right hand side
(setq company-tooltip-align-annotations t)
(add-to-list 'auto-mode-alist '("\\.tsx\\'" . web-mode))

;; formats the buffer before saving
(add-hook 'before-save-hook 'tide-format-before-save)
(add-hook 'typescript-mode-hook #'setup-tide-mode)
(add-hook 'web-mode-hook
          (lambda ()
            (when (string-equal "tsx" (file-name-extension buffer-file-name))
              (setup-tide-mode))))
```

#### Json

```
(use-package json-mode)
```

#### Yaml

```
(use-package yaml-mode)
```

#### Dockfile

```
(use-package dockerfile-mode)
```

#### Protobuf

```
(use-package protobuf-mode)
```

### OrgMode

```
(setq org-todo-keywords 
      '((sequence "TODO(t)" "INPROGRESS(i)" "WAITING(w)" "REVIEW(r)" "|" "DONE(d)" "CANCELED(c)")))

(setq org-todo-keyword-faces
      '(("TODO" . org-warning)
        ("INPROGRESS" . "yellow")
        ("WAITING" . "purple")
        ("REVIEW" . "orange")
        ("DONE" . "green")
        ("CANCELED" .  "red")))
(use-package org-bullets
  :config
  (progn
    (setq org-bullets-bullet-list '("☯" "✿" "✚" "◉" "❀"))
    (add-hook 'org-mode-hook (lambda () (org-bullets-mode 1)))
    ))

(use-package org-alert
  :defer t
  :config
  (progn
    (setq alert-default-style 'libnotify)
    ))
```

#### Org Publish

```
(use-package org
  :ensure org-plus-contrib
  :defer t)

(require 'ox-md)
(require 'ox-publish)

;; setup export theme
(defun @-publish-theme (theme fn &rest args)
  (let ((current-themes custom-enabled-themes))
    (mapcar #'disable-theme custom-enabled-themes)
    (load-theme theme t)
    (let ((result (apply fn args)))
      (mapcar #'disable-theme custom-enabled-themes)
      (mapcar (lambda (theme) (load-theme theme t)) current-themes)
      result)))

(advice-add #'org-export-to-file :around (apply-partially #'@-publish-theme 'doom-snazzy))
(advice-add #'org-export-to-buffer :around (apply-partially #'@-publish-theme 'doom-snazzy))

;; force publish whole site
(use-package htmlize)
(defun @-force-org-publish ()
  (interactive)
  (progn
    (org-reload)
    (org-publish-remove-all-timestamps)
    (org-publish-all t)
    (load-theme 'doom-molokai)    
    (set-face-background 'vertical-border (face-background 'default))
    (set-face-foreground 'vertical-border "grey")
    ))

;; read file content
(defun @-load-file-contents (filename)
  "Return the contents of FILENAME."
  (with-temp-buffer
    (insert-file-contents filename)
    (buffer-string)))

;; sitemap function
(defun @-org-publish-org-sitemap (title list)
  "Sitemap generation function."
  (concat (format "#+TITLE: %s\n" title)
          "#+OPTIONS: toc:nil\n"
          "#+KEYWORDS:技术博客,技术思考,机器学习,边缘计算,Kubernets,容器技术\n"
          "#+DESCRIPTION:前沿技术博客,记录技术生活点滴,Dont't Panic\n\n"
          "* Articals\n"
          (replace-regexp-in-string "\*" " " (org-list-to-subtree list))
          "\n\n"
          (@-load-file-contents (expand-file-name "~/.emacs.d/aboutme.org"))
          ))

(defun @-org-publish-org-sitemap-format (entry style project)
  "Custom sitemap entry formatting: add date"
  (cond ((not (directory-name-p entry))
         (format "- [[file:%s][ %s]]"
                 entry
                 (org-publish-find-title entry project)))
        ((eq style 'tree)
         ;; Return only last subdir.
         (concat "+ "
                 (capitalize (file-name-nondirectory (directory-file-name entry)))
                 "/"))
        (t entry)))

;; customize exported html
(setq org-html-head (@-load-file-contents (expand-file-name "~/.emacs.d/template.html")))
(setq org-html-preamble t)
(setq org-html-postamble (@-load-file-contents (expand-file-name "~/.emacs.d/footer.html")))
(setq org-publish-project-alist
      '(("orgfiles"
         :base-directory "/Users/deyuhua/Documents/org/notebooks/"
         :base-extension "org"
         :publishing-directory "/Users/deyuhua/Workspace/Documents/网站生成/notebooks/"
         :publishing-function org-html-publish-to-html
         :headline-levels 3
         :section-numbers nil
         :with-toc t
         :html-head-include-scripts nil  
         ;; :html-head site-header
         ;; :html-preamble t
         :recursive t
         :with-email "deyuhua@gmail.com"
         :with-title t
         :html-html5-fancy t
         :auto-sitemap t
         :sitemap-function @-org-publish-org-sitemap
         :sitemap-format-entry @-org-publish-org-sitemap-format
         :sitemap-filename "index.org"
         :sitemap-title "Don't Panic!"
         )

        ("images"
         :recursive t
         :base-directory "/Users/deyuhua/Documents/org/notebooks/images/"
         :base-extension "jpg\\|gif\\|png\\|jpeg\\|ico"
         :publishing-directory "/Users/deyuhua/Workspace/Documents/网站生成/notebooks/images/"
         :publishing-function org-publish-attachment)

        ("style"
         :base-directory "/Users/deyuhua/Documents/org/notebooks/style/"
         :base-extension "css\\|el\\|js"
         :publishing-directory "/Users/deyuhua/Workspace/Documents/网站生成/notebooks/style/"
         :publishing-function org-publish-attachment)

        ("fonts"
         :base-directory "/Users/deyuhua/Documents/org/notebooks/fonts/"
         :base-extension "eot\\|woff2\\|woff\\|ttf\\|svg"
         :publishing-directory "/Users/deyuhua/Workspace/Documents/网站生成/notebooks/fonts/"
         :publishing-function org-publish-attachment)   

        ("website" :components ("orgfiles" "images" "style" "fonts"))))
```

### Functions

```
(use-package ido-completing-read+)
(defun @-insert-src-block (src-code-type)
  "Insert a `SRC-CODE-TYPE' type source code block in org-mode."
  (interactive
   (let ((src-code-types
          '("emacs-lisp" "python" "C" "sh" "java" "js" "clojure" "C++" "css"
            "calc" "asymptote" "dot" "gnuplot" "ledger" "lilypond" "mscgen"
            "octave" "oz" "plantuml" "R" "sass" "screen" "sql" "awk" "ditaa"
            "haskell" "latex" "lisp" "matlab" "ocaml" "org" "perl" "ruby"
            "scheme" "sqlite" "html")))
     (list (ido-completing-read+ "Source code type: " src-code-types))))
  (progn
    (newline-and-indent)
    (insert (format "\n#+BEGIN_SRC %s\n" src-code-type))
    (newline-and-indent)
    (insert "#+END_SRC\n")
    (previous-line 2)
    (org-edit-src-code)))
(defun @-close-all-buffers ()
  (interactive)
  (mapc 'kill-buffer (buffer-list)))

(defun @-minify-buffer-contents()
  (interactive)
  (mark-whole-buffer)
  (goto-char (point-min))
  (while (search-forward-regexp "[\s\n]*" nil t) (replace-match "" nil t)))
```

### Keybind

```
(global-set-key (kbd "C-\\") 'comment-line)
;; F1 for tmux
;; F2 neotree toggle
(global-set-key (kbd "<f3>") 'helm-recentf)
(global-set-key (kbd "<f4>") 'fiplr-find-file)
(global-set-key (kbd "<f5>") 'grep-find)
(global-set-key (kbd "<f6>") 'goto-line)

;; F8 neotree-dir
(global-set-key (kbd "<f9>") 'bookmark-jump)
(global-set-key (kbd "<f10>") 'helm-M-x)
(global-set-key (kbd "<f12>") 'project-find-file)

(global-set-key (kbd "M-0") 'next-multiframe-window)
(global-set-key (kbd "M-9") 'previous-multiframe-window)
```