---
name: delete-file
description: >-
  Removes one or more HTML lesson/material files from My English Studio
  (myenglishstudio.github.io): syncs with remote, works on branch deletingfile,
  deletes file(s) from the chosen folder, updates index.html links, merges to main,
  and cleans up temporary branches. Use when the user asks to delete a file, remove
  a lesson, or run the delete-file workflow.
---

# Delete File

Workflow for removing **one or more** material HTML files from [My English Studio](https://myenglishstudio.github.io/) in a single run.

Answer the user in Russian. Follow the steps in order. Do not skip confirmation stops (steps 6–8). Prefer remote as source of truth.

Supports:
- **One file** → remove one link from `index.html`
- **Several files** → all from the **same** approved folder in one batch; one commit / one merge

## Progress checklist

Copy and update as you go:

```
Delete File Progress:
- [ ] 1. Compare local vs remote
- [ ] 2. Confirm remote is the latest source of truth
- [ ] 3. Update local if it does not match remote
- [ ] 4. Create and switch to branch deletingfile
- [ ] 5. Push branch to remote
- [ ] 6. Ask which file(s) to delete (if not already provided)
- [ ] 7. Ask which folder; show existing files for confirmation
- [ ] 8. After confirmation, delete all file(s) from the approved folder
- [ ] 9. Update index.html for every removed file
- [ ] 10. Verify paths for every deleted file
- [ ] 11. Push latest changes to remote
- [ ] 12. Merge into main on remote
- [ ] 13. Delete temporary branches locally and remotely
- [ ] 14. Confirm file(s) no longer open from https://myenglishstudio.github.io/
```

## Steps (follow exactly)

### 1. Проверь соответствует ли статус удаленного репозитория с локальным

From the repo root:

```bash
git fetch origin
git status
git rev-parse HEAD
git rev-parse origin/main
```

Report whether local `main` and `origin/main` match. Prefer `/usr/bin/git` if the default git fails with `unknown option trailer`.

### 2. Последняя и актуальная версия должна быть на удаленном

Treat `origin/main` as the source of truth. Do not overwrite remote with older local commits.

### 3. Если локальная не соответствует удаленной, обнови локальную версию

If local is behind or diverged in a way that remote should win:

```bash
git checkout main
git pull origin main
```

Resolve conflicts only if needed; prefer remote content for already-published pages unless the user says otherwise.

### 4. Локально создай ветку `deletingfile` и перейди на нее

```bash
git checkout -b deletingfile
```

If the branch already exists locally, ask the user whether to reuse it or recreate it from updated `main`.

### 5. Пушни эту ветку в удаленный репозиторий

```bash
git push -u origin deletingfile
```

### 6. Спроси какой файл удалить, если еще не указан

Accept **one file or several files** in the same run.

If nothing is provided yet, ask the user to name the file(s) by:
- filename (e.g. `LESSON_1.html`), or
- link title from the site (e.g. `Lesson 1`), or
- full path in the repo (e.g. `Job Interview/Concept_Check.html`)

Make clear they can delete one file or several at once.

Wait until the batch for this run is complete before continuing. List the pending deletions back to the user before step 7.

**Do not delete anything until the user explicitly confirms in step 8.**

### 7. Спроси из какой папки удалить файл. Покажи какие есть папки и файлы

List top-level material folders and the HTML files inside each (exclude `.git`, `.cursor`, `.gitkeep`). Refresh from disk each run, for example:

- `A1` — …
- `Job Interview` — …
- `Global Communication Dynamics Curriculum` — …
- `Phrasal Verbs for Business & Tech Presentations` — …

All files in this run must come from **one** folder. Ask which folder and confirm the exact filename(s). Match user input to real disk paths (account for `%20` encoding in `index.html` vs spaces in folder names).

If the user wants to delete files from different folders, finish one folder first, then run the skill again for the rest.

### 8. После подтверждения удали предложенный файл из апрувнутой папки

After explicit user confirmation, delete **every** approved file from disk:

```bash
git rm "<folder>/<file1>.html" "<folder>/<file2>.html" ...
```

Or `rm` + `git add` if files are not yet tracked in this branch.

Rules:
- Delete only the confirmed HTML material files — never delete `.gitkeep`, `index.html`, or skill files unless the user explicitly asks
- If the user meant to remove an entire folder/section, use skill `delete-folder` (if it exists) or `add-new-folder` in reverse — do not delete the folder card in this skill unless all materials are gone and the user confirms empty-card behaviour

### 9. Обнови `index.html` с учетом удаленного файла

Edit `index.html` once for the whole batch: remove **one `<a>` per deleted file**.

**CEFR levels (`A1`–`C1`) and Special Topics cards** — inside the corresponding `<li>`:
- Remove each matching `<a href="…">…</a>` line
- If **no links remain** in `level-links`:
  - Replace `<div class="level-links">…</div>` with `<span class="level-empty">No materials yet</span>`
  - Remove class `is-ready` from the `<li>`

Match links by filename in `href` (decode `%20`, `%26`, etc. when comparing).

Do not remove the folder card itself — only the material link(s).

### 10. Проверь что файл удален и ссылка убрана

Verify **every** deleted file before commit:

```bash
# file must NOT exist
test ! -f "<folder>/<filename>.html" && echo "OK deleted"

# link must NOT appear in index.html
rg -n "<filename>" index.html || echo "OK no link"

# no broken href pointing to deleted file
rg -n "href=.*<filename>" index.html
# should return nothing
```

For each file, confirm `https://myenglishstudio.github.io/<folder-encoded>/<filename>` would 404 after deploy.

### 11. Добавь последние изменения в удаленный репозиторий

One commit for the whole batch:

```bash
git add -u "<folder>/" index.html
git commit -m "$(cat <<'EOF'
Remove <Title1>[, <Title2>, ...] from <Folder>.

EOF
)"
git push origin deletingfile
```

If `git commit` fails with `unknown option trailer`, use `/usr/bin/git` for commit/push.

Do not commit unrelated changes.

### 12. На удаленном репозитории вмержи изменения в `main`

Preferred (GitHub CLI):

```bash
gh pr create --base main --head deletingfile --title "Remove materials from <Folder>" --body "$(cat <<'EOF'
## Summary
- Remove <file1>[, <file2>, ...] from <folder>
- Update index.html link(s)

## Test plan
- [ ] Open https://myenglishstudio.github.io/
- [ ] Confirm removed link(s) are gone
- [ ] Confirm deleted URL returns 404

EOF
)"
gh pr merge --merge
```

If `gh` is unavailable, merge via git:

```bash
git checkout main
git pull origin main
git merge deletingfile
git push origin main
```

### 13. Удали временные ветки локально и удаленно

After merge succeeds:

```bash
git checkout main
git pull origin main
git branch -d deletingfile
git push origin --delete deletingfile
```

### 14. Ожидаемый результат

Все удаленные в этом запуске файлы отсутствуют в проекте и больше не открываются с главной страницы https://myenglishstudio.github.io/

Confirm to the user:
- Which file(s) were removed (disk path)
- Which link(s) were removed from `index.html`
- Whether the folder card now shows remaining links or `No materials yet`
- That changes should appear after GitHub Pages deploys (usually within a minute)

## Hard stops

- **Always ask for explicit confirmation before deleting** (steps 6–8) — this is destructive
- Do not delete files before the user confirms folder + filename(s) (steps 7–8)
- Do not merge to `main` before verification passes for **all** deleted files (step 10)
- Do not leave `deletingfile` behind after a successful merge (step 13)
- Prefer remote `main` over divergent local history when syncing (steps 2–3)
- Do not split one multi-file batch across different folders in a single run
- Do not remove the entire folder/section card unless all materials are gone and empty-card rules apply (step 9)
