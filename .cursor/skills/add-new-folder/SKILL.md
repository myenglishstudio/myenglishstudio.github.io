---
name: add-new-folder
description: >-
  Adds a new material folder to My English Studio (myenglishstudio.github.io):
  syncs with remote, works on branch addingnewfolder, creates the folder under
  Materials or Special Topics, updates index.html, optionally adds .gitkeep or
  hands off to add-new-file. Use when the user asks to add a new folder, create
  a new level/topic section, or run the add-new-folder workflow.
---

# Add New Folder

Workflow for adding a new material folder to [My English Studio](https://myenglishstudio.github.io/).

Answer the user in Russian. Follow the steps in order. Do not skip confirmation stops (steps 6–7, 9). Prefer remote as source of truth.

## Progress checklist

Copy and update as you go:

```
Add New Folder Progress:
- [ ] 1. Compare local vs remote
- [ ] 2. Confirm remote is the latest source of truth
- [ ] 3. Update local if it does not match remote
- [ ] 4. Create and switch to branch addingnewfolder
- [ ] 5. Push branch to remote
- [ ] 6. Ask how to name the new folder
- [ ] 7. Ask which component; show structure for approval
- [ ] 8. Create the folder in the chosen component
- [ ] 9. Ask whether to add a file into the new folder now
- [ ] 10. If no file yet — add .gitkeep so the folder is tracked
- [ ] 11. Update index.html for the new folder (and placeholder if any)
- [ ] 12. Verify path to the new folder and file
- [ ] 13. Push latest changes to remote
- [ ] 14. Merge into main on remote
- [ ] 15. Delete temporary branches locally and remotely
- [ ] 16. Confirm folder is visible on https://myenglishstudio.github.io/
- [ ] 17. If adding a real file next — run skill add-new-file after the folder is live
```

## Components (site structure)

Top-level sections in `index.html` are the **components**:

1. **Materials** — CEFR levels (`A1` … `C1`), section eyebrow `Materials`
2. **Special Topics** — standalone topics (e.g. `Job Interview`), section eyebrow `Special Topics`

Each component is a `<section class="sheet materials">` with a `<ul class="level-grid">` of folder cards (`<li>`).

When showing structure for approval (step 7), list like:

```
Components:
1. Materials
   - A1 (Starting out)
   - A2 (Building blocks)
   - B1 (Intermediate)
   - B2 (Upper-intermediate)
   - C1 (Advanced)
2. Special Topics
   - Job Interview (JI)
```

Refresh the list from the real repo + `index.html` every run (folders may change).

## Steps (follow exactly)

### 1. Проверь соответствует ли статус удаленного репозитория с локальным

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

```bash
git checkout main
git pull origin main
```

Prefer remote content for already-published pages unless the user says otherwise.

### 4. Локально создай ветку `addingnewfolder` и перейди на нее

```bash
git checkout -b addingnewfolder
```

If the branch already exists, ask whether to reuse it or recreate it from updated `main`.

### 5. Пушни эту ветку в удаленный репозиторий

```bash
git push -u origin addingnewfolder
```

### 6. Спроси как назвать новую папку

Ask for the folder name. Wait for the answer.

Naming rules:
- Prefer short, clear names matching existing style (`A1`, `Job Interview`, …)
- Prefer spaces only when the display name needs them (like `Job Interview`); otherwise avoid spaces
- Disk folder name = approved name; URL-encode spaces in `href` (`Job%20Interview`)

### 7. Спроси в какой компонент положить новую папку. Покажи структуру компонентов, чтоб можно было выбрать и апрувнуть

Show the live component structure (Materials / Special Topics + existing folders). Ask the user to choose a component and confirm. Do not create the folder until approved.

### 8. Создай папку в указанном компоненте

Create the directory at the **repo root** (same level as `A1`, `C1`, `Job Interview`):

```bash
mkdir -p "<Folder Name>"
```

Components are logical groups in `index.html`, not nested disk paths — do not nest under `A1/` unless the user explicitly asks for a subfolder.

### 9. Спроси будем ли сразу добавлять в эту папку какой-нибудь файл

Ask yes/no. Wait for the answer.

- **Yes** → still finish creating the folder + `index.html` entry + merge (steps 10–16) first with `.gitkeep` if no file is provided yet in this workflow; then go to step 17
- **No** → continue with step 10

If the user already attached a file in the same turn as “yes”, note the path for step 17; do not run `add-new-file` until the folder is live on the site.

### 10. Если сразу не добавляем в эту папку ничего, создай в ней файл-пустышку типа `.gitkeep` чтоб папка была видна на сайте

```bash
touch "<Folder Name>/.gitkeep"
```

Git does not track empty directories; `.gitkeep` keeps the folder in the repo. Do **not** add a public link to `.gitkeep` in `index.html`.

### 11. Обнови `index.html` с учетом добавленной папки и файла

Add a new `<li>` inside the chosen component’s `<ul class="level-grid">`.

**Empty folder (`.gitkeep` only)** — visible card, no material links:

```html
<li>
  <span class="level-num">XX</span>
  <div class="level-body">
    <span class="level-name">Display Name</span>
    <span class="level-empty">No materials yet</span>
  </div>
</li>
```

Ask the user for short code (`level-num`, e.g. `B2`, `JI`, `BP`) and display name if not obvious from the folder name.

**If a real HTML file was placed in this same workflow** (rare — preferred path is step 17):

```html
<li class="is-ready">
  <span class="level-num">XX</span>
  <div class="level-body">
    <span class="level-name">Display Name</span>
    <div class="level-links">
      <a href="Folder%20Name/File_Name.html">Link Title</a>
    </div>
  </div>
</li>
```

Link rules:
- Encode spaces in folder segment of `href`
- Filename must match disk exactly (prefer underscores over spaces in filenames)
- Never link to `.gitkeep`

### 12. Проверь путь к новой папке и файлу

```bash
ls -la "<Folder Name>/"
# expect .gitkeep and/or the real html file
rg -n "level-name|>Display Name|Folder" index.html
```

Confirm the new card appears under the correct component section.

### 13. Добавь последние изменения в удаленный репозиторий

```bash
git add "<Folder Name>/" index.html
git commit -m "$(cat <<'EOF'
Add folder <Folder Name> under <Component>.

EOF
)"
git push origin addingnewfolder
```

If commit fails with `unknown option trailer`, use `/usr/bin/git`. Do not commit `.DS_Store` or secrets.

### 14. На удаленном репозитории вмержи изменения в `main`

Preferred:

```bash
gh pr create --base main --head addingnewfolder --title "Add folder <Folder Name>" --body "$(cat <<'EOF'
## Summary
- Create folder <Folder Name> under <Component>
- Update index.html card

## Test plan
- [ ] Open https://myenglishstudio.github.io/
- [ ] Confirm the new folder/card is visible

EOF
)"
gh pr merge --merge
```

If `gh` is unavailable:

```bash
git checkout main
git pull origin main
git merge addingnewfolder
git push origin main
```

### 15. Удали временные ветки локально и удаленно

```bash
git checkout main
git pull origin main
git branch -d addingnewfolder
git push origin --delete addingnewfolder
```

### 16. Ожидаемый результат это новая папка добавлена в проект и видна на главной странице https://myenglishstudio.github.io/

Confirm to the user:
- Disk path of the folder
- Which component / `level-num` / display name in `index.html`
- That the card should appear after GitHub Pages deploys (usually within a minute)

Optionally fetch the live homepage and check the new name appears.

### 17. Если добавляем новый файл в новую папку используй skill `add-new-file` после того как папка будет создана и видна на https://myenglishstudio.github.io/

Only after steps 1–16 are done and the folder card is live (or deploy has had time to finish):

1. Read and follow `.cursor/skills/add-new-file/SKILL.md`
2. When `add-new-file` asks for the folder, use the folder created in this workflow
3. Do not start `add-new-file` on `addingnewfolder` — that branch must already be deleted

If the user said “no file now” at step 9, stop after step 16.

## Hard stops

- Do not create the folder before name + component are approved (steps 6–7)
- Do not skip `.gitkeep` when no real file is added yet (step 10)
- Do not link `.gitkeep` from `index.html`
- Do not merge before path verification (step 12)
- Do not leave `addingnewfolder` after a successful merge (step 15)
- Do not run `add-new-file` before the new folder is live on the site (step 17)
- Prefer remote `main` when syncing (steps 2–3)
