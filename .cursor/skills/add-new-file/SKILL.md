---
name: add-new-file
description: >-
  Adds a new HTML lesson/material file to My English Studio (myenglishstudio.github.io):
  syncs with remote, works on branch addingnewfile, places the file in the chosen folder,
  updates index.html links, merges to main, and cleans up temporary branches. Use when
  the user asks to add a new file, upload a new lesson, or run the add-new-file workflow.
---

# Add New File

Workflow for adding a new material HTML file to [My English Studio](https://myenglishstudio.github.io/).

Answer the user in Russian. Follow the steps in order. Do not skip confirmation stops (steps 6–8). Prefer remote as source of truth.

## Progress checklist

Copy and update as you go:

```
Add New File Progress:
- [ ] 1. Compare local vs remote
- [ ] 2. Confirm remote is the latest source of truth
- [ ] 3. Update local if it does not match remote
- [ ] 4. Create and switch to branch addingnewfile
- [ ] 5. Push branch to remote
- [ ] 6. Ask user to upload the new file (if not already provided)
- [ ] 7. Ask which folder; list existing folders
- [ ] 8. After confirmation, place file in the approved folder
- [ ] 9. Update index.html for the new file
- [ ] 10. Verify the path to the new file
- [ ] 11. Push latest changes to remote
- [ ] 12. Merge into main on remote
- [ ] 13. Delete temporary branches locally and remotely
- [ ] 14. Confirm file opens from https://myenglishstudio.github.io/
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

Report whether local `main` and `origin/main` match.

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

If the user has not already attached/provided the new file, ask them to upload or provide the file path now. Wait for the file before continuing.

### 7. Спроси в какую папку добавить новый файл. Покажи какие есть папки

List top-level material folders (exclude `.git`, `.cursor`, etc.), for example:

- `A1`
- `A2`
- `B1`
- `B2`
- `C1`
- `Job Interview`

Ask which folder to use. Do not move the file until the user confirms.

### 8. После подтверждения добавь предложенный файл в апрувнутую папку

Copy/move the uploaded file into the approved folder.

Filename rules (avoid 404s):
- Prefer underscores over spaces in filenames (`Recap_Lesson_1.html`, not `Recap_Lesson 1.html`)
- Keep the original display title for the link text in `index.html`
- If renaming for URL safety, tell the user the final filename

### 9. Обнови `index.html` с учетом добавленного файла

Edit `index.html` in the matching section:

**CEFR levels (`A1`–`C1`)** — inside the corresponding `<li>` under Materials:
- If the level shows `<span class="level-empty">No materials yet</span>`, replace it with a `<div class="level-links">…</div>` and add class `is-ready` on the `<li>`
- Append a new `<a>` inside existing `level-links`

**Special topics (`Job Interview`)** — append a new `<a>` inside that section’s `level-links`

Link conventions:
- Folder with a space → encode in `href`: `Job%20Interview/...`
- Filename must match the real file exactly (including underscores)
- Link text: human-readable title (spaces OK)

Example:

```html
<a href="Job%20Interview/Tell_Us_about_Yourself.html">Tell Us about Yourself</a>
```

### 10. Проверь путь к новому файлу

Verify before commit:

```bash
# file exists at the linked path
ls -la "<folder>/<filename>.html"

# href in index.html matches (account for %20 encoding)
rg -n "href=.*<filename>" index.html
```

Mentally resolve the URL: `https://myenglishstudio.github.io/<folder-encoded>/<filename>` and confirm it matches disk.

### 11. Добавь последние изменения в удаленный репозиторий

Commit on `addingnewfile` (only when this workflow is running — user invoked the skill), then push:

```bash
git add "<folder>/<filename>.html" index.html
git commit -m "$(cat <<'EOF'
Add <Title> to <Folder>.

EOF
)"
git push origin addingnewfile
```

If `git commit` fails with `unknown option trailer` (old git wrapper), use `/usr/bin/git` for commit/push.

Do not commit secrets or `.DS_Store`.

### 12. На удаленном репозитории вмержи изменения в `main`

Preferred (GitHub CLI):

```bash
gh pr create --base main --head addingnewfile --title "Add <Title>" --body "$(cat <<'EOF'
## Summary
- Add <filename> to <folder>
- Update index.html link

## Test plan
- [ ] Open https://myenglishstudio.github.io/
- [ ] Click the new link and confirm the page loads (no 404)

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

Новый файл добавлен в проект и открывается при открытии на главной странице https://myenglishstudio.github.io/

Confirm to the user:
- File path on disk
- Link text and `href` in `index.html`
- That the material should appear on the live site after GitHub Pages deploys (usually within a minute)

## Hard stops

- Do not place the file before the user confirms the folder (steps 7–8)
- Do not merge to `main` before path verification (step 10) passes
- Do not leave `addingnewfile` behind after a successful merge (step 13)
- Prefer remote `main` over divergent local history when syncing (steps 2–3)
