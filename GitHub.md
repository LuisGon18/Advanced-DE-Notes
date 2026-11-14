# **Github**

Git is an open-source version control system, version control is a software practice and system that tracks, manages, and stores changes to files.

## Downloading Git

[https://www.atlassian.com/git/tutorials/install-git](https://www.atlassian.com/git/tutorials/install-git)

## Important Terms

- **Directory:** Folder
- **Terminal or Command Line:** Interface for text commands
- **CLI:** Command Line Interface
- **cd:** Change directory
- **Code Editor:** Word processor for writing code, just like VSCode
- **Repository:** Project, or the folder/ place where your project is kept
- **Github:** A website to host your repositories online

## Commands

- ```clone ``` : Bring a repository hosted somewhere like Github into a folder on your local machine
- ```add``` : Track your files and changes in Git
- ```commit``` : save your files and changes in Git
- ```push``` : Upload Git commits to a remote repo, like Github, usually done with ```git push origin main``` but an upstream can be set like ```git push -u origin <branch>``` after setting it up, you can use only ```git push``` to push to the default branch you have set
- ```pull``` : Download changes from the remote repo to your local machine, the opposite of push
- ```dir -Force``` : Lists all files in a folder, including hidden files, when a .git folder is present you know you are in a git repository
- ```status``` : Shows the current state of your directory, like which files are modified, staged or untracked, and where your HEAD is pointing
- ```. ``` : Means all in git
- ```-m <"message_text">``` : Message, sometimes required like in commits
- ```init``` : Creates a new git repository from the folder you are working on
- ```remote``` :  remote means somewhere else ```remote add origin``` is used to connect to a remote repository, which is necessary to push to it, ```remote -v``` shows any remote repositories connected to your local repo
 
## SSH Keys

In order to push into github with your account, you need to prove to github you own the account, this is done with an SSH key:

### How the SSH key system works

**SSH** (Secure Shell) is a way for two computers to talk securely. It uses **asymmetric cryptography**: instead of one password that both sides know, you have **two keys that are made together and only work as a pair**.

- **Private key** (`id_ed25519`): stays on your PC, **never share it, never upload it**. It's like your signature, only you can make it
- **Public key** (`id_ed25519.pub`): you can give it to anyone (GitHub). It can't make your signature, it can only **check** that a signature really came from your private key


**Why SSH instead of HTTPS:**

- With HTTPS you have to use a username + Personal Access Token (GitHub no longer accepts your normal password)
- With SSH you set it up once per computer and then push/pull without typing anything
- Each computer should have its **own key**. If you lose a laptop, you just delete that key in GitHub Settings and the others keep working

### Step by step SSH connection (Windows, PowerShell)

Windows 10/11 already comes with OpenSSH, so nothing extra to install. Check it with:

```powershell
ssh -V
```

**Step 0: Check if you already have a key**

```powershell
dir $HOME\.ssh
```

- If you see `id_ed25519` and `id_ed25519.pub`, you already have a key. You can reuse it (skip to Step 2) or make a new one
- If the folder doesn't exist or is empty, go to Step 1

**Step 1: Create the key**

```powershell
ssh-keygen -t ed25519 -C "your_github_email@example.com"
```

- `-t ed25519` = type of key (the modern, recommended one)
- `-C` = comment, a label so you know whose key it is. Use the email of your GitHub account
- Press **Enter** to save it in the default place (`C:\Users\<you>\.ssh\id_ed25519`)
- Passphrase: a password for the key. Optional, press Enter twice to leave it empty

**Step 2: Turn on the ssh-agent** (the program that holds your key in memory)

On Windows the agent is a service that is **off by default**. Open **PowerShell as Administrator** (right click → Run as administrator) and run:

```powershell
Get-Service -Name ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

- You only do this once, after that it starts by itself every time Windows boots
- Check it's running with: `Get-Service ssh-agent` → Status should say **Running**

**Step 3: Add your key to the agent** (normal PowerShell is fine)

```powershell
ssh-add $HOME\.ssh\id_ed25519
```

- If you set a passphrase, it asks for it here once
- Check it was added with: `ssh-add -l`

**Step 4: Copy the public key**

```powershell
Get-Content $HOME\.ssh\id_ed25519.pub | Set-Clipboard
```

- This copies the `.pub` (public) key to your clipboard. Never copy the file without `.pub`
- It should start with `ssh-ed25519 ...` and end with your email

**Step 5: Add the key to GitHub**

1. GitHub → click your profile picture → **Settings**
2. Left menu → **SSH and GPG keys** → **New SSH key**
3. **Title:** something to recognize the machine, e.g. `Windows laptop`
4. **Key type:** Authentication Key
5. **Key:** paste (Ctrl + V) → **Add SSH key**

**Step 6: Test the connection**

```powershell
ssh -T git@github.com
```

- First time it asks `Are you sure you want to continue connecting (yes/no)?` → type `yes`
- Success message: `Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.`

**Step 7: Use the SSH link, not the HTTPS one**

On GitHub, in the green **Code** button pick the **SSH** tab:

```powershell
git clone git@github.com:<your-username>/<repo-name>.git
```

If a repo was already cloned with HTTPS, switch it to SSH:

```powershell
git remote -v                                                          # shows the current link
git remote set-url origin git@github.com:<your-username>/<repo-name>.git
```

**Step 8 (only if you used a passphrase): make Git use Windows' SSH**

Git for Windows brings its own copy of SSH, which doesn't talk to the Windows ssh-agent, so it keeps asking for your passphrase on every push. Tell Git to use the Windows one:

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

## Git Branching

Branching is useful to work in a kind of sandbox area to make sure the changes you made work, before merging them to the main branch, or like a backup. Branching is especially useful when collaborating with other people.

Branching looks really similar to snowflake micropartitions, when you create a new branch, it is created from a specific commit, you basically create a clone of the project until then, changes to one branch can't be seen in another, each branch only keeps track of the changes in it.

### branching commands

- ```branch``` : shows branches, a * is shown in the branch you are using.
- ```checkout``` : used to change branch, also used to create a new branch with ```checkout -b <branch_name>```
- ```diff``` : shows what changes have been made
- ```merge``` : combines changes from different branches into a single branch, integrating work while preserving history. It helps unify development without losing progress, it's best to merge from pull request in github.
- ```merge main``` : Updates your feature branch with main.
- ```branch -d <branch_name>``` deletes branch only if merged, use ```-D``` to force delete even if unmerged

### Pull Request

It's common practice to not only merge the changes made to a branch back to the main branch, but to first save the changes and then create a pr (pull request) from the branch you saved the changes to, to the main one, after a pr anyone can review it, comment on it and ask for changes. After a pr is made you can also update the code in that same branch with simple additional commits, after the pr is resolved and the branch is merged to the main one, the additional branch is usually deleted and you switch back to the main branch.

After approving a pr and merging the branch to the main one, you need to update the repository by pulling from main, and deleting the branch used to make the changes

## Undoing

If a mistake is made, system or human, you can use ```reset HEAD~1``` this tells git to point one commit further back than the most recent one by unstaging and uncommitting the changes just committed.

### Undoing Commands

- ```log``` : shows you a list of all commits, including their hash or id, you can use the hash to go back to the state of the repo after that specific commit using ```reset <hash>``` (press q to get out of log view)
- ```reset --hard <hash>``` : this also moves the pointer back to that commit, but while normal reset keeps your changes in the files as unstaged edits, hard discards all changes and makes your files match that commit exactly
  
## Forking

This makes a complete copy of a repository, this is useful to copy other people's repos, where you don't have any access to make changes on the original.