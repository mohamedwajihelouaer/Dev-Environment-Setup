# Sharing SSH keys between Windows and WSL

- Quick guide on how to use git ssh settings between WSL and windows to avoid creating ducplicate keys

## Make sure Windows has SSH keys:

```bash Get-ChildItem ~\.ssh ```

- If you haven't created one in Windows yet, run 

```bash ssh-keygen -t ed25519 ``` 

in PowerShell and add ~/.ssh/id_ed25519.pub to GitHub/GitLab.

## Configure /etc/wsl.conf for permissions

- Windows mounted filesystems default to 777 permissions, but OpenSSH in Linux will refuse to run if key permissions are not restricted (600). Ensure metadatais enabled in WSL so Linux permissions apply to Windows.

```bash sudo nano /etc/wsl.conf ```

- Add or ensure the ```[automount]``` block exists:

```text
[automount]
options = "metadata"
```

## Copy or symlink keys into WSL

- Run from inside WSL terminal. Create the .ssh folder in your Linux home directory and link your Windows keys.
  
```bash 
mkdir -p ~/.ssh
cp -r /mnt/c/Users/<WindowsUser>/.ssh/* ~/.ssh/
```
Replace <WindowsUser> with your Windows username

## Set the strict file permissions OpenSSH requires

```bash  ```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/*.pub 2>/dev/null || true
chmod 644 ~/.ssh/known_hosts 2>/dev/null || true

## Test from both environments

```powershell ssh -T git@github.com```

```bash ssh -T git@github.com```

## SSH Agent auto start in WSL

```bash 
nano ~/.bashrc
```

```bash
# Ensure ssh-agent is running in WSL
SSH_ENV="$HOME/.ssh/agent-environment"

start_agent() {
    echo "Initializing new SSH agent..."
    /usr/bin/ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
    chmod 600 "${SSH_ENV}"
    . "${SSH_ENV}" > /dev/null
    /usr/bin/ssh-add
}

# Source agent settings if available; check if process is alive
if [ -f "${SSH_ENV}" ]; then
    . "${SSH_ENV}" > /dev/null
    ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
        start_agent;
    }
else
    start_agent;
fi
```

```bash
# check
ssh-add -l
```
