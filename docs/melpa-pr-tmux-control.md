# Draft: melpa/melpa pull request for tmux-control

For Clay to send; nothing has been opened. Before sending:

1. Merge [tmux-control#194](https://github.com/csheaff/tmux-control/pull/194)
   and tag `v0.8.2` on its merge commit, so MELPA builds 0.8.2.
2. Add an `Assisted-by:` line under `;; Author:` in `tmux-control.el`, which
   MELPA's template now asks for when LLMs wrote some of the code. Commits
   name Claude Opus 4.8, Fable 5, Opus 5.5 and Sonnet 5.5, and Copilot; how
   to list them is yours to decide. Then tick that box below.
3. Fork melpa/melpa, add `recipes/tmux-control` with the recipe below, and
   open the pull request with the title and body below.

Recipe, `recipes/tmux-control`:

```elisp
(tmux-control :fetcher github :repo "csheaff/tmux-control")
```

Title: `Add recipe for tmux-control`

Body:

---

### Brief summary of what the package does

tmux-control is a client for tmux's control mode (`tmux -C`), the protocol
iTerm2's tmux integration uses. It connects to a tmux server, local or over
SSH, and shows each tmux window as an Emacs buffer, rendered with Eat's
terminal emulator. tmux's sessions outlive Emacs: close Emacs or lose the
connection, reconnect, and the programs are still running. Scrollback is an
Emacs buffer you can search and copy from, and split tmux windows show as
Emacs windows.

Similar packages: [emamux](https://github.com/emacsorphanage/emamux) sends
commands to tmux, and [tmux-pane](https://github.com/laishulu/emacs-tmux-pane)
moves between Emacs windows and tmux panes when Emacs runs inside tmux.
Neither shows tmux's panes as Emacs buffers. It depends on
[Eat](https://codeberg.org/akib/emacs-eat), from NonGNU ELPA.

### Direct link to the package repository

https://github.com/csheaff/tmux-control

### Your association with the package

I'm the author and maintainer.

### Relevant communications with the upstream package maintainer

**None needed**

### Checklist

- [x] The package is released under a [GPL-Compatible Free Software License](https://www.gnu.org/licenses/license-list.en.html#GPLCompatibleLicenses)
- [x] I've read [CONTRIBUTING.org](https://github.com/melpa/melpa/blob/master/CONTRIBUTING.org)
- [ ] LLMs were used to generate some of the code, and if so, I've added an `Assisted-by:` line as described in [CONTRIBUTING.org](https://github.com/melpa/melpa/blob/master/CONTRIBUTING.org#attribution-for-ai-generated-code)
- [x] I understand the package must have been maintained in a public repository for 1 month or more
- [x] I've used the latest version of [package-lint](https://github.com/purcell/package-lint) to check for packaging issues, and addressed its feedback
- [x] My elisp byte-compiles cleanly
- [x] I've used `M-x checkdoc` to check the package's documentation strings
- [x] I've built and installed the package using the instructions in [CONTRIBUTING.org](https://github.com/melpa/melpa/blob/master/CONTRIBUTING.org#test-your-recipe)

package-lint reports no issues, and the file byte-compiles with no warnings
on Emacs 29.1 to 32. checkdoc's remaining notes are keys written into
docstrings and messages that start with the package's name. melpazoid lists
nine top-level `advice-add` calls on Eat; `tmux-control-unload-function`
removes them all.

---
