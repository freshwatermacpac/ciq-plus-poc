# Git Bash Cheat Sheet
*For pushing projects to GitHub from any folder*

---

## First time ever (one-off)
Run once on your machine to set your identity:
```
git config --global user.name "Mick Barrett"
git config --global user.email "mick@vineiq.com.au"
```

---

## New project setup (one-off per folder)
Run these once to link a folder to a new GitHub repo.

**Step 1** — Create a new empty repo on GitHub:
- Go to github.com/new
- Name it, set to Public or Private, **don't add a README**
- Click Create repository
- Copy the repo URL (looks like `https://github.com/freshwatermacpac/your-repo-name.git`)

**Step 2** — In Git Bash, navigate to your folder:
```
cd "C:/path/to/your/folder"
```

**Step 3** — Run the setup commands:
```
git init
git add .
git commit -m "First commit"
git branch -M main
git remote add origin https://github.com/freshwatermacpac/YOUR-REPO-NAME.git
git push -u origin main
```
> On the first push, a browser window will open asking you to sign into GitHub. Do it once — you'll never be asked again for any repo.

---

## Everyday use (push updates)
After the one-off setup above, this is all you ever need:
```
git add .
git commit -m "Describe what you changed"
git push
```

---

## Navigating to your folder in Git Bash
Replace the path with your actual folder location:
```
cd "C:/Users/MickBarrett/OneDrive - Michael Barrett/RDV Ventures/VineIQ/Client work/PROJECT-NAME"
```
**Tip:** You can drag a folder from File Explorer into the Git Bash window and it will paste the path for you.

---

## Useful extras

| What you want | Command |
|---|---|
| See what's changed (not yet saved) | `git status` |
| See history of saves | `git log --oneline` |
| Undo unsaved changes to a file | `git checkout -- filename.html` |

---

*Authentication is stored permanently by Windows Credential Manager — no passwords needed after the first push.*
