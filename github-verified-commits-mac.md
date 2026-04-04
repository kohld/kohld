# GitHub Verified Commits on macOS

## Overview
This handout explains how to sign your GitHub commits with GPG to get the "Verified" badge.

---

## 1. Install GPG

```bash
brew install gnupg pinentry-mac
```

---

## 2. Generate GPG Key Pair

```bash
gpg --full-generate-key
```

**Inputs:**
- **Kind of key:** RSA and RSA (default)
- **Key size:** 4096
- **Validity:** 0 (never expire) or e.g., 1 year
- **Real name:** Your name
- **Email:** Your GitHub email (must match exactly!)
- **Passphrase:** Choose securely and remember it

---

## 3. Find Your Key ID

```bash
gpg --list-secret-keys --keyid-format=long
```

**Output looks like:**
```
pub   rsa4096 2026-04-04 [SC]
      XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
uid                      Your Name <your.email@github.com>
sub   rsa4096 2026-04-04 [E]
      YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
```

**Your Key ID is the last 16 characters of the `pub` line:**
```
XXXXXXXXXXXXXXXX
```

---

## 4. Upload Public Key to GitHub

```bash
gpg --armor --export XXXXXXXXXXXXXXXX
```

**Copy the entire output** (from `-----BEGIN PGP PUBLIC KEY BLOCK-----` to `-----END PGP PUBLIC KEY BLOCK-----`).

**On GitHub:**
1. Settings → SSH and GPG keys
2. New GPG key
3. Paste & Save

---

## 5. Configure Git Locally

```bash
# Set GPG key for signing (USE YOUR KEY ID!)
git config --global user.signingkey XXXXXXXXXXXXXXXX

# Specify GPG program
git config --global gpg.program /usr/local/bin/gpg

# Automatically sign all commits
git config --global commit.gpgsign true
```

---

## 6. Configure GPG_TTY and pinentry

**Open `~/.zshrc` (or `~/.bash_profile`):**
```bash
nano ~/.zshrc
```

**Add at the top:**
```bash
export GPG_TTY=$(tty)
```

**Save (Ctrl+O, Enter, Ctrl+X).**

**Reload:**
```bash
source ~/.zshrc
```

**Configure gpg-agent – create/edit `~/.gnupg/gpg-agent.conf`:**
```bash
nano ~/.gnupg/gpg-agent.conf
```

**Add:**
```
pinentry-program /usr/local/bin/pinentry-mac
```

(For M1/M2 Macs: `/opt/homebrew/bin/pinentry-mac`)

---

## 7. Restart GPG Agent

```bash
killall gpg-agent
gpg-agent --daemon
```

---

## 8. Test: Sign a Commit

```bash
git commit --allow-empty -S -m "Test verified commit"
```

**Expected:** A macOS dialog window for your GPG passphrase.

If everything works:
```bash
git push
```

**Check on GitHub** – your commit should now have a green "Verified" ✅ badge.

---

## Troubleshooting

### "Inappropriate ioctl for device"
- Run `killall gpg-agent`
- Run `source ~/.zshrc`
- Passphrase dialog should now appear

### Passphrase not accepted
```bash
killall gpg-agent
rm ~/.gnupg/S.gpg-agent*
gpg-agent --daemon
```

Then try again.

### Test GPG directly
```bash
echo "test" | gpg --clearsign
```

Should bring up a dialog. If yes but Git doesn't work → check Git config.

---

## Time Investment

| Step | Duration |
|------|----------|
| Installation | ~2 min |
| Key generation | ~10 min |
| GitHub setup | ~2 min |
| Git config | ~2 min |
| **Total** | **~15-20 min** |

---

## After Setup

✅ All your commits are automatically signed  
✅ GitHub shows "Verified" badge  
✅ Zero additional overhead  

**First time:** Passphrase dialog (20 seconds)  
**Later:** Cached – no dialog needed

---

## Quick Reference

```bash
# Show your Key ID
gpg --list-secret-keys --keyid-format=long

# Export public key
gpg --armor --export XXXXXXXXXXXXXXXX

# Restart GPG agent
killall gpg-agent && gpg-agent --daemon

# Test: signed empty commit
git commit --allow-empty -S -m "Test"

# Show git config
git config --global --list | grep sign
```

---

**Last updated:** April 2026  
**OS:** macOS  
**Tools:** Git + GPG + GitHub
