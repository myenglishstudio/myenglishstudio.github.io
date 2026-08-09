---
name: add-new-file
description: >-
  Adds one or more HTML lesson/material files to My English Studio
  (myenglishstudio.github.io): syncs with remote, works on branch addingnewfile,
  places file(s) in the chosen folder, updates index.html links, merges to main,
  and cleans up temporary branches. Use when the user asks to add a new file,
  add several files, upload a new lesson, or run the add-new-file workflow.
---

# Add New File

Workflow for adding **one or more** material HTML files to [My English Studio](https://myenglishstudio.github.io/) in a single run.

Answer the user in Russian. Follow the steps in order. Do not skip confirmation stops (steps 6–8). Prefer remote as source of truth.

Supports:
- **One file** → one link in `index.html`
- **Several files** → all go into the **same** approved folder in one batch; one commit / one merge

## Progress checklist

Copy and update as you go:

```
Add New File Progress:
- [ ] 1. Compare local vs remote
- [ ] 2. Confirm remote is the latest source of truth
- [ ] 3. Update local if it does not match remote
- [ ] 4. Create and switch to branch addingnewfile
- [ ] 5. Push branch to remote
- [ ] 6. Ask user to upload the new file(s) (if not already provided)
- [ ] 7. Ask which folder; list existing folders
- [ ] 8. After confirmation, place all file(s) in the approved folder
- [ ] 9. Update index.html for every added file
- [ ] 10. Verify paths for every new file
- [ ] 11. Push latest changes to remote
- [ ] 12. Merge into main on remote
- [ ] 13. Delete temporary branches locally and remotely
- [ ] 14. Confirm file(s) open from https://myenglishstudio.github.io/
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

### 4. Локально создай ветку `addingnewfile` и перейди на нее

```bash
git checkout -b addingnewfile
```

If the branch already exists locally, ask the user whether to reuse it or recreate it from updated `main`.

### 5. Пушни эту ветку в удаленный репозиторий

```bash
git push -u origin addingnewfile
```

### 6. На этой ветке локально попроси загрузить новый файл, если еще не загружен

Accept **one file or several files** in the same run.

If nothing is provided yet, ask the user to upload or give path(s). Make clear they can send:
- one file, or
- several files at once (list / multiple attachments / several paths)

Wait until the batch for this run is complete before continuing. If the user sends files in multiple messages, keep collecting until they confirm the list is ready (or clearly intend to proceed with what was sent).

List the pending files back to the user before step 7.

### 7. Спроси в какую папку добавить новый файл. Покажи какие есть папки

List top-level material folders (exclude `.git`, `.cursor`, etc.). Refresh from disk each run, for example:

- `A1`
- `A2`
- `B1`
- `B2`
- `C1`
- `Job Interview`
- `Global Communication Dynamics Curriculum`

All files in this run go into **one** folder. Ask which folder to use. Do not move any files until the user confirms.

If the user wants different folders for different files, finish this batch for one folder first, then run the skill again for the rest (or ask them to split into separate runs).

### 8. После подтверждения добавь предложенный файл в апрувнутую папку

Copy/move **every** uploaded file into the approved folder.

Filename rules (avoid 404s) — apply to each file:
- Prefer underscores over spaces in filenames (`Recap_Lesson_1.html`, not `Recap_Lesson 1.html`)
- Keep a human-readable display title for the link text in `index.html`
- If renaming for URL safety, tell the user the final filename for each renamed file

If the folder only had `.gitkeep` and you are adding real HTML materials, you may leave `.gitkeep` or remove it; either is fine. Do not link `.gitkeep` in `index.html`.

### 9. Обнови `index.html` с учетом добавленного файла

Edit `index.html` once for the whole batch: add **one `<a>` per file**.

**CEFR levels (`A1`–`C1`) and Special Topics cards** — inside the corresponding `<li>`:
- If the card shows `<span class="level-empty">No materials yet</span>`, replace it with a `<div class="level-links">…</div>` and add class `is-ready` on the `<li>`
- Append each new `<a>` inside `level-links` (order: keep existing links, then new files in the order the user provided)

Link conventions (per file):
- Folder with a space → encode in `href`: `Job%20Interview/...`, `Global%20Communication%20Dynamics%20Curriculum/...`
- Filename must match the real file exactly (including underscores)
- Link text: human-readable title (spaces OK)

Examples:

```html
<a href="C1/Vague_Language.html">Vague Language</a>
<a href="Job%20Interview/Tell_Us_about_Yourself.html">Tell Us about Yourself</a>
```

### 10. Проверь путь к новому файлу

Verify **every** new file before commit:

```bash
ls -la "<folder>/"
# each new file exists

rg -n "href=.*<filename>" index.html
# each new filename appears in index.html
```

For each file, resolve `https://myenglishstudio.github.io/<folder-encoded>/<filename>` and confirm it matches disk.

### 11. Добавь последние изменения в удаленный репозиторий

One commit for the whole batch:

```bash
git add "<folder>/<file1>.html" "<folder>/<file2>.html" ... index.html
git commit -m "$(cat <<'EOF'
Add <Title1>[, <Title2>, ...] to <Folder>.

EOF
)"
git push origin addingnewfile
```

If `git commit` fails with `unknown option trailer`, use `/usr/bin/git` for commit/push.

Do not commit secrets or `.DS_Store`.

### 12. На удаленном репозитории вмержи изменения в `main`

Preferred (GitHub CLI):

```bash
gh pr create --base main --head addingnewfile --title "Add materials to <Folder>" --body "$(cat <<'EOF'
## Summary
- Add <file1>[, <file2>, ...] to <folder>
- Update index.html link(s)

## Test plan
- [ ] Open https://myenglishstudio.github.io/
- [ ] Open each new link and confirm it loads (no 404)

EOF
)"
gh pr merge --merge
```

If `gh` is unavailable, merge via git:

```bash
git checkout main
git pull origin main
git merge addingnewfile
git push origin main
```

### 13. Удали временные ветки локально и удаленно

After merge succeeds:

```bash
git checkout main
git pull origin main
git branch -d addingnewfile
git push origin --delete addingnewfile
```

### 14. Ожидаемый результат

Все добавленные в этом запуске файлы есть в проекте и открываются с главной страницы https://myenglishstudio.github.io/

Confirm to the user:
- Disk path for each file
- Link text and `href` for each entry in `index.html`
- That materials should appear after GitHub Pages deploys (usually within a minute)

## Hard stops

- Do not place files before the user confirms the folder (steps 7–8)
- Do not merge to `main` before path verification passes for **all** new files (step 10)
- Do not leave `addingnewfile` behind after a successful merge (step 13)
- Prefer remote `main` over divergent local history when syncing (steps 2–3)
- Do not split one multi-file batch across different folders in a single run
