# GitHub Release Checklist

**Nothing in this project has been published. No repository has been modified.**
Every step below is yours to perform.

---

## 1. Before you touch the repository

- [ ] The redesigned Home page has passed acceptance (`IMPLEMENTATION_SOP.md` Phase 7)
- [ ] The PBIX is saved and closed
- [ ] A dated backup exists in `_backups`
- [ ] Screenshots captured at 1280 × 720

### Check the PBIX file size first

```bash
ls -lh "Sales_Analysis_Report.pbix"
```

| Size | What to do |
|---|---|
| Under 50 MB | Commit normally |
| 50–100 MB | Commits, but GitHub warns. Consider Git LFS |
| Over 100 MB | **GitHub will reject the push.** Use Git LFS, or publish a download link and omit the file |

Finding this out after writing a commit message is a waste of ten minutes.

---

## 2. Branch

```bash
git clone <your-repo-url>
cd <your-repo>
git checkout -b redesign/home-page
```

Never commit this directly to `main`. The branch is your third rollback route.

---

## 3. Place the files

```
<repo root>/
├── README.md
├── PROJECT_OVERVIEW.md
├── DATA_DICTIONARY.md
├── METRIC_DEFINITIONS.md
├── METHODOLOGY.md
├── DATA_VALIDATION.md
├── KNOWN_LIMITATIONS.md
├── DESIGN_SYSTEM.md
├── IMPLEMENTATION_SOP.md
├── GITHUB_RELEASE_CHECKLIST.md
├── .gitignore
├── data/Sales_Dataset.xlsx
├── theme/Cream_Forest_Theme_v2.json
├── report/Sales_Analysis_Report.pbix
└── docs/images/*.png
```

- [ ] All eleven Markdown files at the root
- [ ] `.gitignore` in place **before** the first `git add`
- [ ] Screenshots in `docs/images/` with the exact filenames the README references

---

## 4. Resolve the placeholders

```bash
grep -rn "CONFIRM" *.md
```

- [ ] Every `<!-- CONFIRM: ... -->` resolved or deleted
- [ ] Your name, contact and LinkedIn in the README
- [ ] Licence chosen
- [ ] `METRIC_DEFINITIONS.md` DAX column filled from the PBIX formula bar

Shipping a README containing the word `CONFIRM` is the kind of detail a reviewer
notices.

---

## 5. Check the links before committing

- [ ] Every image path in the README matches a file that exists, case included
- [ ] Every relative Markdown link resolves
- [ ] No absolute paths from your own machine
- [ ] No link to a private or local resource

GitHub is case-sensitive. `01_Home.png` and `01_home.png` are different files, and this
is the most common cause of a broken portfolio README. Your current repository already
has six broken image links.

```bash
grep -oP '!\[.*?\]\(\K[^)]+' README.md | while read p; do
  [ -f "$p" ] && echo "OK   $p" || echo "MISS $p"
done
```

---

## 6. Review what you are about to commit

```bash
git add -A
git status
git diff --cached --stat
```

- [ ] No credentials, tokens or connection strings
- [ ] No personal data beyond what you intend to publish
- [ ] No `.pbix` temp files or `~$` lock files
- [ ] File count and sizes look right

The source workbook contains 25 manager names and photo URLs. They are fictional sample
data — confirm you are content to publish them.

---

## 7. Commit and push

```bash
git commit -m "Redesign Home page as income-statement layout; add project documentation

- Rebuild Home: forest masthead and footer, cream canvas, P&L chain
- Add five label measures so sub-line ratios respond to filters
- Fix Net Proft typo; add Help route from Home
- Document data dictionary, metrics, methodology, validation, limitations
- Add Cream & Forest theme"

git push -u origin redesign/home-page
```

---

## 8. Review on GitHub before merging

- [ ] Open the branch on github.com
- [ ] **Every image renders.** This is the step that catches broken paths
- [ ] Tables render correctly
- [ ] Code blocks are syntax-highlighted
- [ ] No raw HTML comments visible in the rendered page
- [ ] The README reads well to someone who has never seen the project

Read the rendered README as a stranger would. If the first screen does not say what the
project is and what it demonstrates, fix that before merging.

---

## 9. Merge and tag

```bash
git checkout main
git merge redesign/home-page
git push
git tag -a v2.0 -m "Home page redesign and full documentation"
git push --tags
```

---

## 10. Repository presentation

- [ ] Description set — one sentence, what it is and what it demonstrates
- [ ] Topics added: `power-bi`, `dax`, `data-analytics`, `business-intelligence`,
      `data-visualization`, `portfolio`
- [ ] A release created from the `v2.0` tag with short notes
- [ ] Social preview image set (Settings → General → Social preview) — use the Home
      screenshot

The social preview is what appears when you share the link on LinkedIn. Without it the
link renders as a grey box.

---

## 11. Final honesty check

- [ ] The README does not claim anything untested
- [ ] The sample-data disclosure is present and easy to find
- [ ] No screenshot shows a defect you know about
- [ ] No documented validation result is one you did not actually run
- [ ] The limitations file is linked from the README

---

## Rollback

| Problem | Fix |
|---|---|
| Pushed too early | `git revert <sha>` and push. Do not force-push a shared branch |
| Committed a large file | `git rm --cached <file>`, commit, push. The blob stays in history — rewrite only if it must be gone |
| Published something private | Remove it, push, then **rotate any exposed credential**. Assume it was seen |
| Merged a broken README | Fix on a branch and merge again. Do not rewrite `main` history |

---

## A note on what this checklist cannot verify

This file was written without access to your repository. It does not know your remote
name, your default branch, your existing file layout, or what is already committed.
Check the first `git status` output against what you expect before running `git add -A`.
