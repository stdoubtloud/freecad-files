# FreeCAD Version Control with Git and Zippey

This guide sets up git version control for FreeCAD `.FCStd` files using **zippey**, a tool that allows git to efficiently store and diff the ZIP-based FreeCAD format as readable text rather than binary blobs.

>**None of this is my work!** The concept and scripts were created and documented by Dante Catalfamo: [blog.lambda.cx](https://blog.lambda.cx)
---

## Prerequisites

- Linux machine
- Git installed (`git --version` to check)
- Python 3 installed (`python3 --version` to check)
- A GitHub account

---

## Step 1 — Download Zippey

```bash
mkdir -p /shared/projects/freecadgit
cd /shared/projects/freecadgit
git clone https://bitbucket.org/sippey/zippey.git
chmod +x zippey/zippey.py
```

---

## Step 2 — Fix the Python Shebang

The zippey script may reference `python` rather than `python3`. Fix this with:

```bash
sed -i 's|#!/usr/bin/env python|#!/usr/bin/env python3|' /shared/projects/freecadgit/zippey/zippey.py
```

Verify it worked:

```bash
head -1 /shared/projects/freecadgit/zippey/zippey.py
# Should show: #!/usr/bin/env python3
```

---

## Step 3 — Configure Global Git Settings

Open `~/.gitconfig` in a text editor and add the following sections, keeping anything already there:

```ini
[diff "zip"]
    textconv = unzip -c -a
[core]
    attributesfile = ~/.gitattributes
[filter "zippey"]
    smudge = /shared/projects/freecadgit/zippey/zippey.py d
    clean = /shared/projects/freecadgit/zippey/zippey.py e
```

---

## Step 4 — Create the Global Gitattributes File

```bash
echo "*.FCStd filter=zippey" >> ~/.gitattributes
echo "*.FCStd diff=zip" >> ~/.gitattributes
```

---

## Step 5 — Set Up SSH Authentication for GitHub

This avoids needing to enter a password every time you push.

**Generate an SSH key:**

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Accept the default location and set a passphrase if desired.

**Add the key to your SSH agent:**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**Copy your public key:**

```bash
cat ~/.ssh/id_ed25519.pub
```

**Add it to GitHub:**

1. Go to github.com → click your profile picture → **Settings**
2. **SSH and GPG keys → New SSH key**
3. Give it a title (e.g. `my-linux-machine`), paste the key, click **Add SSH key**

**Test it:**

```bash
ssh -T git@github.com
# Should show: Hi username! You've successfully authenticated...
```

---

## Step 6 — Create the GitHub Repository

1. Go to github.com → click **+** in the top right → **New repository**
2. Name it `freecad-files`
3. Make it **Private**
4. **Do not** tick any initialisation options (no README, no .gitignore)
5. Click **Create repository**

---

## Step 7 — Set Up the Local Repository

```bash
mkdir /shared/projects/freecad-files
cd /shared/projects/freecad-files
git init
git remote add origin git@github.com:YOUR_GITHUB_USERNAME/freecad-files.git
```

---

## Step 8 — Ignore FreeCAD Backup Files

FreeCAD creates `.FCBak` files every time you save. Exclude them from git:

```bash
echo "*.FCBak" > /shared/projects/freecad-files/.gitignore
```

---

## Step 9 — Verify Zippey is Working

Before committing, confirm zippey can process your files:

```bash
cat /path/to/MyFile.FCStd | /shared/projects/freecadgit/zippey/zippey.py e | head -20
```

You should see text output with headers like `79492|59618|B|Document.xml` rather than binary gibberish. If you get an error, check the shebang fix in Step 2 and permissions in Step 1.

---

## Step 10 — Add Files and Push

Copy your `.FCStd` files into `/shared/projects/freecad-files/`, then:

```bash
cd /shared/projects/freecad-files
git add .
git commit -m "initial commit"
git push -u origin master
```

---

## Day-to-Day Usage

After making changes in FreeCAD, save the file, then:

```bash
cd /shared/projects/freecad-files
git add .
git commit -m "description of what you changed"
git push
```

---

## Restoring Files

**On a new machine or after data loss:**

> ⚠️ Zippey must be installed and configured (Steps 1–4) on the new machine first, otherwise files will be restored as garbled text.

```bash
git clone git@github.com:YOUR_GITHUB_USERNAME/freecad-files.git /shared/projects/freecad-files
```

**Restoring a previous version of a specific file:**

Find the commit you want:

```bash
git log --oneline
```

Restore that file from a specific commit:

```bash
git checkout <commit-hash> -- MyFile.FCStd
```

---

## Notes

- These global git settings (Steps 3–4) must be repeated on every machine you use.
- The SSH key setup (Step 5) must also be repeated per machine, with a new key added to GitHub for each.
- FreeCAD `.FCStd` files contain XML internally. Zippey unpacks them before storing in git so that diffs show meaningful changes rather than binary differences.
