# Submitting to MELPA

Roost depends on tmux-control, so tmux-control goes first; submit Roost once
tmux-control is on MELPA. Each submission is a pull request to
[melpa/melpa](https://github.com/melpa/melpa) adding one file under `recipes/`.

## Recipes

`recipes/tmux-control`:

```elisp
(tmux-control :fetcher github :repo "csheaff/tmux-control")
```

`recipes/roost`:

```elisp
(roost :fetcher github :repo "csheaff/roost"
       :files (:defaults ("scripts" "scripts/roost_remote.py")))
```

Roost's `:files` matter: Roost copies `scripts/roost_remote.py` to each host,
and finds it next to `roost.el`. The default file list includes only Emacs
Lisp, so without it a MELPA install would have no helper.

## Status (2026-10-16)

Checked with package-lint 20260903.2119, checkdoc, and
[melpazoid](https://github.com/riscy/melpazoid) at 6200dec (2026-10-10), on
Emacs 30.2 and 32.0.50, which gave the same results; melpazoid's container now
runs Emacs 31.1. MELPA's `package-build` built both recipes: the Roost archive
holds `roost.el` and `scripts/roost_remote.py`, and the tmux-control archive
holds `tmux-control.el`, which needs no other file.

| Check | tmux-control | Roost |
| --- | --- | --- |
| `package-lint` | No issues | "tmux-control is not installable", which clears once tmux-control is on MELPA; and that `with-eval-after-load` is for configurations, which `roost-watch-mode` uses to add Roost's sections to Magit once Magit, an optional package, loads |
| Byte compilation and loading | No warnings | No warnings |
| `checkdoc` | 29 advisory notes, two of which [tmux-control#191](https://github.com/csheaff/tmux-control/pull/191) fixes; the rest are keys written into docstrings (14), messages starting with "tmux-control:" (12), and `\\[dired]` taken for a name to capitalise | None, since the six that came in after 0.8.1 were fixed for 0.9.0 |
| melpazoid | The eight top-level `advice-add` calls, which `tmux-control-unload-function` undoes; no hooks are added at load. Hints to use `end-of-line` (2) and to avoid `ignore-errors` (5) | Hints to avoid `ignore-errors` (24), around lookups and clean-ups allowed to fail, such as Git over TRAMP, and `with-suppressed-warnings` (3), where Roost still reads the options 0.9 made obsolete |
| License | GPL-3.0-or-later, in `LICENSE` and the file's header | GPL-3.0-or-later, in `LICENSE` and the headers of `roost.el` and the helper, added for 0.9.0: without them melpazoid fails Roost |
| `check-declare` | Consult and Evil, optional and not installed there | tmux-control, Magit, perspective.el and Evil, not installed there, and `canvas-refresh`, which Emacs 32 has in C |
| Tests | 357 ERT, source and compiled, and 42 live integration scenarios, in CI | 273 ERT and 156 Python lifecycle tests, in CI on Emacs 29.1 and 30.1 |

melpazoid also notes, for both, that `;; Author:` gives no address for
`lm-authors` to read.

MELPA's reviewers run melpazoid. Its Emacs Lisp checks ran here outside its
container, with native compilation off as in the container, and its license
and packaging checks on a copy of each repository. Run it again before
submitting.

## Before submitting Roost

- Confirm that tmux-control's recipe merged and that `M-x package-install`
  finds it.
- Re-run `package-lint` on `roost.el`; the dependency error should be gone.
- MELPA builds from the default branch, and numbers each build from the
  latest tag, not the Version header: until `v0.9.0` is tagged, it calls
  0.9.0 something like 0.8.1.0.20261016.182. Tags also feed MELPA Stable.
