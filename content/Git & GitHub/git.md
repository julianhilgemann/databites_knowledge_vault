Here is a comprehensive **General Git Primer**.

### 1. The Mental Model: The Three Zones

Git is not just "Save" and "Undo." It moves files between three specific zones.

| Zone | Definition | The Command to Enter |
| :--- | :--- | :--- |
| **1. Working Directory** | **"The Workbench."** The files currently open in VS Code. You are editing them. They are messy and unsaved. | (You are here by default) |
| **2. Staging Area** | **"The Shopping Cart."** You pick specific files from the workbench that you want to include in the next snapshot. | `git add <file>` |
| **3. Repository (.git)** | **"The Photo Album."** The permanent history. Once data is here, it is very hard to lose. | `git commit` |

---

### 2. The Daily Workflow (The "Happy Path")

This is the cycle you will repeat 20 times a day.

#### Step 1: Status Check
Always start here. It tells you which zone your files are in.
```bash
git status
```
*   *Red files:* In Working Directory (Modified but not staged).
*   *Green files:* In Staging Area (Ready to commit).

#### Step 2: Stage Changes
Move files from Working Directory to Staging.
```bash
git add .           # Stages EVERYTHING (Use with caution)
git add file.txt    # Stages just one file (Best practice)
```

#### Step 3: Commit (Save)
Take a snapshot of the Staging Area.
```bash
git commit -m "Fix logic in Sales Calculation"
```
*   **The `-m` flag:** Allows you to write the message inline.
*   **Best Practice:** Use present tense imperative verbs (*"Fix bug"*, not *"Fixed bug"*).

#### Step 4: Sync with Cloud (Remote)
Send your local commits to the server (GitHub / Azure DevOps).
```bash
git push origin main
```
*   `origin`: The nickname for the URL of the cloud repository.
*   `main`: The branch you are pushing to.

---

### 3. Branching (The Parallel Universe)

**Concept:** Branching takes a copy of the code so you can experiment without breaking the main application.

| Command | Action |
| :--- | :--- |
| `git branch` | Lists all local branches. |
| `git branch <name>` | Creates a new branch (but doesn't switch to it). |
| `git switch <name>` | **Switches** you to that branch. (Replaces the old `checkout`). |
| `git switch -c <name>`| **Creates AND Switches** in one move. |
| `git merge <name>` | Combines the history of `<name>` into your *current* branch. |

**The Workflow:**

1.  `git switch -c feature/new-kpi` (Create safe space).
2.  *Do work... Commit work...*
3.  `git switch main` (Go back to the production code).
4.  `git pull` (Make sure Main is up to date).
5.  `git merge feature/new-kpi` (Bring your work into Main).

---

### 4. Remote Operations (Teamwork)

#### `git clone <url>`
Downloads a repository for the first time. You only do this once per project.

#### `git fetch` vs `git pull` (Common Question)
*   **`git fetch`**: "Go to the internet and **look** at what changed. Download the info, but **do not touch my files**." (Safe).
*   **`git pull`**: "Go to the internet, download changes, and **immediately overwrite** my files with them." (Fetch + Merge).

---

### 5. Undoing Mistakes ( The "Oh Sh*t" Commands)

#### A. "I typed the commit message wrong"
```bash
git commit --amend -m "New correct message"
```
*   *Condition:* Only works if you haven't pushed to the cloud yet.

#### B. "I staged a file I didn't mean to"
```bash
git restore --staged <file>
```
*   Moves the file from **Staging** back to **Working Directory**.

#### C. "I ruined this file. Reset it to how it was this morning."
```bash
git restore <file>
```
*   **Warning:** Destructive. Changes in that file are gone forever.

#### D. "Everything is broken. Reset the whole folder to the last clean commit."
```bash
git reset --hard HEAD
```
*   **Warning:** **Nuclear Option.** Deletes all uncommitted work.

---

### 6. The `.gitignore` File

**Concept:** A text file that tells Git which files to **intentionally ignore**.

**Why it matters for Data Engineers:**
You never want to commit:
1.  **Credentials:** `.env` files with passwords.
2.  **Binary Data:** `.pbix` files, `.csv` exports, `.abf` cache files.
3.  **System Files:** `.DS_Store` (Mac), `Thumbs.db` (Windows).

**Example `.gitignore`:**
```text
# Ignore local settings
.vscode/
localSettings.json

# Ignore data
*.csv
*.parquet
Data/

# Ignore logs
*.log
```

---

### 7. Resolving Conflicts (The Human Element)

**Scenario:**
*   You changed Line 10 of `Sales.tmdl`.
*   Your colleague changed Line 10 of `Sales.tmdl`.
*   You try to `git merge`.

**Result:**
Git pauses and says: **"CONFLICT"**. It cannot decide who is right.

**The Fix:**
1.  Open the file in VS Code.
2.  You will see markers:
    ```text
    <<<<<<< HEAD
    Measure = SUM(Sales)
    =======
    Measure = AVERAGE(Sales)
    >>>>>>> feature/branch
    ```
3.  **Decide:** Delete the lines you don't want. Remove the `<<<<` markers.
4.  **Finish:** `git add .` and `git commit`.

---

### Summary Checklist

1.  **"I commit often."** (Small, atomic commits are easier to debug than one massive commit).
2.  **"I use Branches."** (I never commit directly to Main).
3.  **"I protect Secrets."** (I check my `.gitignore` to ensure no API keys or CSVs enter the repo).
4.  **"I prefer PRs."** (I use Pull Requests in Azure DevOps/GitHub to merge code, creating a paper trail of code reviews).

### Mnemonic: **"Add, Commit, Push"**
Just remember the rhythm:
1.  **Add** to the cart.
2.  **Commit** to the purchase.
3.  **Push** it to the delivery truck.