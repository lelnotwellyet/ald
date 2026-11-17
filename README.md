# Practical 1: Version Control with Git (Windows Command Prompt + VS Code)

## Overview & Setup
- **Environment:** Windows Command Prompt (`cmd`) + Visual Studio Code.
- Run Git commands in Command Prompt.
- Create or edit files in VS Code.
- *Note:* Command Prompt does not have `cat` or `grep`; use `type` instead.
- Save files (`Ctrl + S`) before running `git add` or `git commit`. Unsaved changes will not be committed.

---

## Step-by-Step Instructions

### 1. Create the Repo and Set Name and Email (4 Marks)
```cmd
cd C:\
mkdir student-management
cd student-management
git init -b main
git config user.name "Your Name"
git config user.email "you@example.com"
git config --local --list
```
Then open the folder in VS Code:
```cmd
code .
```
> **Screenshot 1:** Output of `git config --local --list`

---

### 2. Create Three Files and Make the First Commit (5 Marks)
In VS Code (Explorer, New File icon), create these three files in the `student-management` folder and save each.

**`index.html`**
```html
<h1>Student Management System</h1>
```

**`style.css`**
```css
body { font-family: Arial; background: #f4f4f4; }
```

**`app.js`**
```javascript
console.log("Student Management App");
```

Back in Command Prompt / Git Bash:
```cmd
git add .
git status
git commit -m "Initial commit"
```
> **Screenshot 2:** `git status` and the commit output

---

### 3. Create `feature-login` and Add a Login Form (5 Marks)
```cmd
git checkout -b feature-login
```
In VS Code, replace the contents of `index.html` with:
```html
<h1>Student Management System</h1>
<h2>Login</h2>
<form>
  <input type="text" placeholder="Username">
  <input type="password" placeholder="Password">
  <button type="submit">Login</button>
</form>
```
Save the file, then run:
```cmd
git add index.html
git commit -m "Add login form"
```

---

### 4. Switch Back to `main`, Change `app.js` and Commit (4 Marks)
```cmd
git checkout main
```
*VS Code will reload `index.html` back to the original version, which is expected.*

Now open `app.js` and add this on a new second line:
```javascript
console.log("Main branch update");
```
Save the file, then run:
```cmd
git add app.js
git commit -m "Update app.js on main"
```

---

### 5. Merge `feature-login` into `main` (4 Marks)
```cmd
git merge feature-login -m "Merge feature-login into main"
type index.html
```
*The branches changed different files, so there is no conflict, and `index.html` now contains the login form.*

> **Screenshot 3:** Merge output and `type index.html`

---

### 6. Create a Merge Conflict and Resolve It (8 Marks)

#### a) Change `style.css` on a new branch
```cmd
git checkout -b feature-theme
```
In VS Code, edit `style.css` so the whole line reads:
```css
body { font-family: Arial; background: lightblue; }
```
Save, then:
```cmd
git commit -am "Blue background"
```

#### b) Change the same line differently on `main`
```cmd
git checkout main
```
Edit `style.css` to:
```css
body { font-family: Arial; background: lightgreen; }
```
Save, then:
```cmd
git commit -am "Green background"
git merge feature-theme
```
Git reports `CONFLICT (content): Merge conflict in style.css.`
> **Screenshot 4:** The `CONFLICT` message

#### c) View the conflict markers
```cmd
type style.css
```
You will see both versions between the `<<<<<<<`, `=======` and `>>>>>>>` markers.
> **Screenshot 5:** `type style.css` showing the markers (or screenshot VS Code's conflict view with its Accept Current / Incoming / Both buttons)

#### d) Resolve it
In VS Code, open `style.css`, delete all the marker lines and both versions, and leave just one final line:
```css
body { font-family: Arial; background: lightyellow; }
```
Save, then run:
```cmd
git add style.css
git commit -m "Resolve merge conflict in style.css"
git log --graph --oneline --all
```
*The graph shows both branches joining back into `main`.*

> **Screenshot 6:** Final merge commit  
> **Screenshot 7:** `git log --graph --oneline --all`

---

## Windows and VS Code Notes
- **`code .` not found?** Open VS Code manually and use **File > Open Folder** and pick the `student-management` folder (for example `C:\student-management`).
- **"LF will be replaced by CRLF" warnings?** Harmless line-ending messages on Windows. To silence them, run:
  ```cmd
  git config core.autocrlf true
  ```
- **Merge stopped with an editor?** If Git opens an editor for the commit message, save and close it. Using `-m` as above avoids this.
- **If `git commit -am` says "nothing to commit":** You forgot to save the file in VS Code (`Ctrl + S`).
- **`'cat'` or `'grep'` is not recognized?** You are in Command Prompt. Use `type` instead of `cat`.
- **`--local` flag:** Shows only this repo's settings.
- **`cd desktop` fails?** Not needed. The repo can live in any folder, such as `C:\student-management`.

---

## Screenshots Checklist
1. `git config --local --list`
2. First commit (and `git status`)
3. Merge of `feature-login` into `main`
4. The `CONFLICT` message
5. `type style.css` with the markers
6. Final merge commit
7. `git log --graph --oneline --all`
