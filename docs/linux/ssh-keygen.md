# SSH Keys & `~/.ssh/config` — Vani's DevOps Story

> **Scenario:** Vani is a DevOps engineer.
> One laptop. Multiple GitHub accounts. Multiple servers. Multiple SSH keys.
> Her job is to make sure the **right key goes to the right machine**.

---

# 🚨 Day 1 — Vani Gets a Problem

Vani starts her new DevOps job.

Her setup looks like this:

```text
Vani's Laptop
│
├── Personal GitHub
│
├── Work GitHub
│
├── AWS Production Server
│
└── Legacy Client Server
```

She tries to manage everything with one SSH key.

Very quickly, things become messy.

```text
❌ Wrong GitHub account
❌ Wrong SSH key
❌ Permission denied
❌ Too many authentication failures
❌ "Why is GitHub using my personal account?!"
```

Her senior engineer tells her:

> "Stop using one key for everything. Give every identity a proper key and let SSH decide which one to use."

That's where `ssh-keygen` and `~/.ssh/config` come in.

---

# 🔑 1. First: What Exactly Is an SSH Key?

Vani runs:

```bash
ssh-keygen
```

SSH creates a **key pair**:

```text
Private Key
     +
Public Key
     =
SSH Identity
```

Think of it like a lock-and-key system.

```text
PUBLIC KEY
    ↓
The lock
Can be shared

PRIVATE KEY
    ↓
The actual key
Must stay with Vani
```

For example:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

The first one is private.

The second one is public.

### Vani's golden rule

```text
.pub       → Public → Safe to share
No .pub    → Private → NEVER share
```

---

# 🧠 2. How Does Vani Actually Log In?

Suppose Vani runs:

```bash
ssh ubuntu@server
```

The server doesn't receive her private key.

Instead, authentication roughly works like this:

```text
             VANI'S LAPTOP
                  │
                  │
          "I want to log in"
                  │
                  ▼
          ┌───────────────┐
          │ Remote Server │
          └───────────────┘
                  │
                  │ Challenge
                  ▼
          Vani's SSH client
                  │
                  │ Uses PRIVATE KEY
                  ▼
          Cryptographic proof
                  │
                  ▼
          Remote server
                  │
                  │ Verifies using
                  │ PUBLIC KEY
                  ▼
             ✅ Access
```

The important part:

> **Vani's private key never needs to be sent to the server.**

The server only needs her public key.

---

# 🔐 3. Vani Needs Four Different Identities

Now things get interesting.

Vani has:

```text
1. Personal GitHub
2. Work GitHub
3. AWS Production
4. Legacy Client Server
```

Her senior says:

> "Give each identity its own key."

So Vani creates:

```text
~/.ssh/
├── work/
│   ├── github_work
│   └── aws_prod
│
└── personal/
    └── github_personal
```

But what about the legacy server?

It requires RSA.

So Vani creates:

```text
legacy_server
```

Now she has different keys for different systems.

---

# ⚙️ 4. Choosing the Right Algorithm

Vani asks:

> "What kind of key should I create?"

Her senior gives her this simple rule:

| Situation           | Vani Uses                  |
| ------------------- | -------------------------- |
| Modern GitHub       | **Ed25519**                |
| Modern Linux server | **Ed25519**                |
| AWS                 | **Ed25519** when supported |
| Old/legacy system   | **RSA 4096**               |

So for modern systems:

```bash
ssh-keygen -t ed25519
```

For the legacy client:

```bash
ssh-keygen -t rsa -b 4096
```

### Why Ed25519?

It's modern, fast, compact, and widely supported.

### Why RSA 4096?

Because Vani's old client server doesn't support Ed25519.

So:

```text
Modern system
      ↓
Ed25519

Legacy system
      ↓
RSA 4096
```

---

# 🛠️ 5. Vani Creates Her Work GitHub Key

Vani needs a separate identity for her company's GitHub.

She runs:

```bash
ssh-keygen -t ed25519 \
  -f ~/.ssh/work/github_work \
  -C "vani@company.com"
```

SSH creates:

```text
~/.ssh/work/
├── github_work
└── github_work.pub
```

She adds:

```text
github_work.pub
```

to her **work GitHub account**.

The private key:

```text
github_work
```

stays on her laptop.

---

# 🏠 6. Vani Creates Her Personal GitHub Key

Her personal GitHub needs a completely different identity.

```bash
ssh-keygen -t ed25519 \
  -f ~/.ssh/personal/github_personal \
  -C "vani@personal.com"
```

Now:

```text
~/.ssh/
├── work/
│   └── github_work
│
└── personal/
    └── github_personal
```

She adds:

```text
github_personal.pub
```

to her personal GitHub account.

Now her two GitHub identities are separated.

---

# ☁️ 7. Vani Gets Production Access

Her company has an AWS production server:

```text
13.233.45.100
```

Vani creates another key:

```bash
ssh-keygen -t ed25519 \
  -f ~/.ssh/work/aws_prod \
  -C "vani@company.com"
```

Now:

```text
~/.ssh/
├── work/
│   ├── github_work
│   └── aws_prod
│
└── personal/
    └── github_personal
```

She installs the corresponding public key on the production server.

---

# 🏚️ 8. Then Comes the Legacy Server...

Unfortunately, an old client server says:

> "Ed25519? Never heard of it."

It requires RSA.

Vani doesn't argue.

She creates:

```bash
ssh-keygen -t rsa -b 4096 \
  -f ~/.ssh/work/legacy_server \
  -C "vani@company.com"
```

Now she has:

```text
~/.ssh/
├── work/
│   ├── github_work
│   ├── aws_prod
│   └── legacy_server
│
└── personal/
    └── github_personal
```

---

# 😵 9. Vani Has Too Many Keys

Now Vani has a new problem.

She could manually specify the key every time:

```bash
ssh -i ~/.ssh/work/aws_prod ubuntu@13.233.45.100
```

But that's ugly.

And imagine doing this every day.

She wants to type:

```bash
ssh aws-prod
```

and have SSH automatically know:

> "Oh, Vani means the AWS production server. Use `aws_prod`."

That's what:

```text
~/.ssh/config
```

is for.

---

# 🧠 10. Vani Creates `~/.ssh/config`

She runs:

```bash
nano ~/.ssh/config
```

And creates:

```sshconfig
# Work GitHub
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/work/github_work
    IdentitiesOnly yes

# Personal GitHub
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/personal/github_personal
    IdentitiesOnly yes

# AWS Production
Host aws-prod
    HostName 13.233.45.100
    User ubuntu
    IdentityFile ~/.ssh/work/aws_prod
    IdentitiesOnly yes

# Legacy Client
Host legacy-client
    HostName legacy.example.com
    User admin
    IdentityFile ~/.ssh/work/legacy_server
    IdentitiesOnly yes
```

Now Vani has created **aliases** for her machines.

---

# 🏷️ 11. What Does `Host` Actually Mean?

This:

```sshconfig
Host aws-prod
```

doesn't mean the actual server name.

It's simply Vani's nickname for the server.

So:

```text
Host aws-prod
       ↓
Vani's nickname
       ↓
13.233.45.100
```

Now she can type:

```bash
ssh aws-prod
```

instead of:

```bash
ssh -i ~/.ssh/work/aws_prod ubuntu@13.233.45.100
```

Much cleaner.

---

# 🗺️ 12. What Does `HostName` Mean?

This:

```sshconfig
HostName 13.233.45.100
```

is the **real destination**.

So:

```text
Host
  ↓
aws-prod

HostName
  ↓
13.233.45.100
```

Think:

```text
Host     = Nickname
HostName = Actual address
```

---

# 👤 13. What Does `User` Mean?

Vani's AWS server expects the username:

```text
ubuntu
```

So:

```sshconfig
User ubuntu
```

For GitHub, SSH uses:

```sshconfig
User git
```

That's why Git URLs look like:

```text
git@github.com
```

The GitHub username isn't the SSH user.

The SSH user is simply:

```text
git
```

---

# 🔑 14. What Does `IdentityFile` Mean?

This is the important one.

```sshconfig
IdentityFile ~/.ssh/work/aws_prod
```

means:

> "When Vani connects using this host, use this private key."

So:

```text
ssh aws-prod
      │
      ▼
~/.ssh/config
      │
      ▼
Host aws-prod
      │
      ▼
IdentityFile ~/.ssh/work/aws_prod
      │
      ▼
Use AWS private key
```

---

# 🚧 15. Why Does Vani Need `IdentitiesOnly yes`?

Vani has many keys.

SSH might otherwise try several identities from her SSH agent.

That can result in errors such as:

```text
Too many authentication failures
```

So she adds:

```sshconfig
IdentitiesOnly yes
```

This tells SSH:

> "For this host, don't randomly try every key I have. Use the identity I've configured."

---

# 🧪 16. Vani Tests Her AWS Connection

Instead of remembering:

```bash
ssh -i ~/.ssh/work/aws_prod ubuntu@13.233.45.100
```

she simply runs:

```bash
ssh aws-prod
```

SSH reads:

```text
~/.ssh/config
```

and finds:

```sshconfig
Host aws-prod
    HostName 13.233.45.100
    User ubuntu
    IdentityFile ~/.ssh/work/aws_prod
```

Result:

```text
Vani
 │
 │ ssh aws-prod
 ▼
SSH Config
 │
 ├── HostName → 13.233.45.100
 ├── User → ubuntu
 └── Key → aws_prod
 │
 ▼
AWS Production
```

---

# 🐙 17. Now Vani Has a GitHub Problem

Vani runs:

```bash
git clone git@github.com:company/project.git
```

But which GitHub account should SSH use?

Her personal account?

Her work account?

SSH sees:

```text
github.com
```

and doesn't know that Vani wants the **work key**.

This is where the `Host` aliases become extremely useful.

---

# 💼 18. Work GitHub

Vani configured:

```sshconfig
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/work/github_work
    IdentitiesOnly yes
```

So instead of:

```bash
git clone git@github.com:company/project.git
```

she uses:

```bash
git clone git@github-work:company/project.git
```

Notice:

```text
github-work
```

is NOT a real domain.

It's Vani's SSH alias.

SSH internally translates:

```text
github-work
      ↓
github.com
      ↓
github_work private key
```

---

# 🏠 19. Personal GitHub

Her personal configuration is:

```sshconfig
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/personal/github_personal
    IdentitiesOnly yes
```

So she clones personal repositories with:

```bash
git clone git@github-personal:vani-personal/project.git
```

Now:

```text
git@github-work
        ↓
Work GitHub Key

git@github-personal
        ↓
Personal GitHub Key
```

Same physical domain:

```text
github.com
```

Different SSH identities.

---

# 🧩 20. The Magic of `~/.ssh/config`

Vani's entire setup can now be visualized like this:

```text
                    ~/.ssh/config
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
    github-work    github-personal     aws-prod
          │               │                │
          ▼               ▼                ▼
     github.com       github.com      13.233.45.100
          │               │                │
          ▼               ▼                ▼
   github_work     github_personal     aws_prod
          │               │                │
          ▼               ▼                ▼
      Work GitHub    Personal GitHub    AWS Server
```

That's the entire purpose of SSH config:

> **Map a friendly alias to the correct destination, user, and private key.**

---

# 🛠️ 21. Vani's Debugging Day

One morning:

```bash
ssh aws-prod
```

fails.

She wants to know what SSH is doing.

She runs:

```bash
ssh -v aws-prod
```

For even more detail:

```bash
ssh -vvv aws-prod
```

Now she can see:

```text
Which config was loaded
Which host was selected
Which identity was attempted
Authentication methods
Server response
```

This is extremely useful when debugging SSH.

---

# 🔍 22. Vani Loses Her `.pub` File

One day she accidentally deletes:

```text
github_work.pub
```

She panics.

But the private key still exists:

```text
github_work
```

She can recreate the public key:

```bash
ssh-keygen -y \
  -f ~/.ssh/work/github_work \
  > ~/.ssh/work/github_work.pub
```

The private key remains the source of truth.

---

# 🆔 23. Vani Wants to Verify a Key

She can view the fingerprint:

```bash
ssh-keygen -l \
  -f ~/.ssh/work/github_work.pub
```

A fingerprint is a short representation of the key that can be used for verification.

---

# 🔒 24. Vani Protects Her Private Keys

She checks permissions:

```bash
ls -l ~/.ssh/work/github_work
```

Private keys should not be readable by everyone.

For example:

```bash
chmod 600 ~/.ssh/work/github_work
```

And remember:

```text
❌ Never commit private keys
❌ Never upload private keys
❌ Never send private keys to someone
❌ Never put private keys in GitHub
```

Public keys are different:

```text
✅ Public keys can be copied to servers
✅ Public keys can be added to GitHub
```

---

# 🧠 25. Vani's Final SSH Cheat Sheet

### Generate a modern key

```bash
ssh-keygen -t ed25519 -f ~/.ssh/key_name
```

### Generate RSA 4096 for legacy systems

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/key_name
```

### Change passphrase

```bash
ssh-keygen -p -f ~/.ssh/key_name
```

### Recreate public key

```bash
ssh-keygen -y -f ~/.ssh/key_name > ~/.ssh/key_name.pub
```

### Show fingerprint

```bash
ssh-keygen -l -f ~/.ssh/key_name.pub
```

### Test SSH

```bash
ssh alias-name
```

### Debug SSH

```bash
ssh -v alias-name
```

### Edit SSH configuration

```bash
nano ~/.ssh/config
```

---

# 🎯 26. Vani's Mental Model

Whenever Vani sees:

```bash
ssh something
```

she asks:

```text
"What does 'something' mean?"
        │
        ▼
Check ~/.ssh/config
        │
        ▼
Find Host something
        │
        ▼
Where is HostName?
        │
        ▼
Which User?
        │
        ▼
Which IdentityFile?
        │
        ▼
Use that private key
        │
        ▼
Authenticate using public key
        │
        ▼
       ✅
```

And when she sees:

```bash
git clone git@github-work:company/project.git
```

she knows:

```text
github-work
     ↓
SSH config alias
     ↓
github.com
     ↓
github_work private key
     ↓
Work GitHub account
```

---

# 🏆 The One Rule Vani Never Forgets

```text
                    SSH
                     │
          ┌──────────┴──────────┐
          │                     │
       WHERE?                 WHO?
          │                     │
       HostName              User
          │                     │
          └──────────┬──────────┘
                     │
                  WHICH KEY?
                     │
                IdentityFile
                     │
                     ▼
              Correct Identity
```

**`ssh-keygen` creates Vani's identities.**

**`~/.ssh/config` tells SSH which identity to use and where to use it.**

That's the entire game.
