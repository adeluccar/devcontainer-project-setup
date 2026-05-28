# Devcontainer Setup

## 1. SSH on the host machine

This is per developer, per machine. You only do this once.

If you already have GitHub connected via SSH, you likely just need to add your existing key to the macOS Keychain (step 2) and update your SSH config (step 3). Start there.

### 1.1 Generate an SSH key (skip if you already have one)

Check for an existing key:

```bash
ls ~/.ssh/*.pub
```

If you see one (e.g. `id_ed25519.pub`) and it's already added to GitHub, skip to 1.2.

Otherwise, generate one:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_github
```

Then add the public key to GitHub:

```bash
cat ~/.ssh/id_ed25519_github.pub
```

Copy the output and add it at [github.com/settings/ssh/new](https://github.com/settings/ssh/new).

### 1.2 Add your key to macOS Keychain

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_github
```

### 1.3 Update `~/.ssh/config`

Add this at the very top of `~/.ssh/config` (create the file if it doesn't exist):

```
IgnoreUnknown UseKeychain

Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  AddKeysToAgent yes
  UseKeychain yes

Host *
  AddKeysToAgent yes
  UseKeychain yes
```

`IgnoreUnknown UseKeychain` is required — `UseKeychain` is macOS-only and without it the container (Linux) will reject the config.

### 1.4 Verify

```bash
ssh -T git@github.com
# Hi <username>! You've successfully authenticated...
```

Once this works on the host, you're ready to move on.

## 2. Install prerequisites

```bash
brew install colima docker docker-compose devcontainer
```

## 3. Configure Colima

These steps apply to every Colima instance you use — the default one and any named instances (e.g. `colima start complete-css`). Each instance has its own config file and needs to be configured separately.

Open the config for the instance you're setting up:

```bash
# Default instance
~/.colima/default/colima.yaml

# Named instance (e.g. complete-css)
~/.colima/complete-css/colima.yaml
```

If the file doesn't exist, start the instance once to generate it, then stop it:

```bash
colima start            # default
colima start complete-css  # named
colima stop
colima stop complete-css
```

**Set `forwardAgent` to `true`:**

```yaml
forwardAgent: true
```

**Find `provision: null` and replace it with:**

```yaml
provision:
  - mode: system
    script: apt-get install -y socat

  - mode: user
    script: |
      cat > "$HOME/.ssh_agent.sh" << 'EOF'
      REAL_AGENT=$(find /tmp -maxdepth 2 -name 'agent.*' -type s 2>/dev/null | head -1)
      if [ -S "$REAL_AGENT" ]; then
        pkill -f "socat.*ssh-agent.sock" 2>/dev/null || true
        rm -f /tmp/ssh-agent.sock
        socat UNIX-LISTEN:/tmp/ssh-agent.sock,fork UNIX-CONNECT:$REAL_AGENT &
      fi
      export SSH_AUTH_SOCK=/tmp/ssh-agent.sock
      EOF
      grep -q 'ssh_agent.sh' "$HOME/.profile" || echo '. $HOME/.ssh_agent.sh' >> "$HOME/.profile"
```

Repeat for each named instance. If you've already started an instance without these settings, stop it, update the config, then restart it with the `--provision` flag to force the script to re-run:

```bash
colima stop complete-css
colima start complete-css --provision
```

This forwards your Mac's SSH agent into the Colima VM and creates a stable socket at `/tmp/ssh-agent.sock` that the container binds.

## 4. Choose a base image

The default image in `devcontainer.json` is `mcr.microsoft.com/devcontainers/base:debian` — a generic Debian base with no language tooling. Swap it out for something suited to your stack before provisioning. Examples:

- Node.js: `mcr.microsoft.com/devcontainers/javascript-node:22`
- Python: `mcr.microsoft.com/devcontainers/python:3.12`
- Go: `mcr.microsoft.com/devcontainers/go:1.22`

Full list at [mcr.microsoft.com/devcontainers](https://mcr.microsoft.com/devcontainers).

If your project has a dev server, also add `appPort` so the port is forwarded to your Mac:

```json
"appPort": [3000]
```

Without it, the server runs inside the container but you won't be able to reach it from the browser.

## 5. Start Colima and provision the container

Start Colima with at least 12 GB of memory — Claude Code's installer will fail with less:

```bash
colima start --memory 12
```

Then provision the container from the project root:

```bash
devcontainer up --workspace-folder . --remove-existing-container
```

The `--remove-existing-container` flag is useful for the first few runs while you're tweaking the config — it ensures a clean reprovision every time. Claude Code is installed automatically as part of the post-create step.

Drop in once it's up:

```bash
devcontainer exec --workspace-folder . bash
```

## 6. Verify Claude, git, and GitHub from inside the container

**Claude:**

```bash
claude --version
```

**Git:**

```bash
git config user.name
git config user.email
```

Your identity comes from the `~/.gitconfig` mounted from the host — it should already be set.

**GitHub CLI:**

```bash
gh auth login
```

Choose **HTTPS**. The container has no browser, so you'll see:

```
! First copy your one-time code: XXXX-XXXX
! Failed opening a web browser...
  Please try entering the URL in your browser manually
```

Copy the code, open [github.com/login/device](https://github.com/login/device) on your Mac, and paste it. Then verify:

```bash
gh auth status
```

**SSH to GitHub** (confirms git push/pull will work):

```bash
ssh -T git@github.com
# Hi <username>! You've successfully authenticated...
```

## 7. Stopping

```bash
exit
docker stop $(docker ps -q --filter label=devcontainer.local_folder=$(pwd))
colima stop
```

## Afterthoughts

### .gitignore

The `.gitignore` ships with sensible defaults but needs tailoring to your project's stack — add build output, generated files, secrets, and anything else your toolchain produces that shouldn't be committed.

### npm projects

If your project uses npm, uncomment `node_modules/` and `package-lock.json` in `.claudeignore` — both are context dead weight that Claude doesn't need to read.

## Troubleshooting

### `git push` fails with a public key error

Run this from inside the container and enter your passphrase when prompted:

```bash
ssh -T git@github.com
# Hi <username>! You've successfully authenticated...
```

Once it goes through, `git push` will work.
