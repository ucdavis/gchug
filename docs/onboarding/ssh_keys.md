# 1. Creating and uploading SSH keys

An SSH key comes in two parts:

- **Private key** - stays on your laptop, never shared.
- **Public key** - uploaded to HIPPO, which installs it on Hive.

When you connect, Hive checks that your laptop holds the matching private key. If it does, you're in.

```
[Your laptop: private key] --ssh--> [Hive: public key via HiPPO]
```

## 1.1 Mac/Linux

**Step 1: Generate a Key Pair (on your laptop)**

First, check if you already have one:

```sh
ls ~/.ssh/id_ed25519*
```

If those files exist, skip to Step 2. Otherwise, generate a new key:

```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- Press **Enter** to accept the default location (`~/.ssh/id_ed25519`).
- Set a passphrase (recommended) or leave empty for none.

You'll get two files:

- `~/.ssh/id_ed25519`      - private key (keep secret)
- `~/.ssh/id_ed25519.pub`  - public key (upload this to HIPPO)

**Step 2: Upload Your Public Key to HIPPO**

Print your public key and copy the full output:

```sh
cat ~/.ssh/id_ed25519.pub
```

It will look like:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... your_email@example.com
```

Then:

1. Log into HiPPO.
2. Navigate to the SSH keys section for your Hive account.
3. Paste the **entire line** (including `ssh-ed25519` at the start and your comment at the end).
4. Save.

HiPPO propagates the key to Hive within a few minutes.

**Step 3: Connect**

From your laptop:

```sh
ssh username@hive.hpc.ucdavis.edu
```

You should log in immediately (or, if you set a passphrase, you'll be asked for *that* - not your HPC password, which doesn't exist for SSH).

## 1.2 Windows (MobaXterm tutorial)

https://www.youtube.com/watch?v=vGbzOq3fDqU

ADD PICTURE OF WHERE IT SHOULD GO

## 1.3 Troubleshooting

**Permission denied (publickey)**

Hive rejected your key. Common causes:

- The key hasn't fully propagated yet - wait a few minutes after uploading to HIPPO.
- You uploaded the **private** key by mistake. Only the `.pub` file goes to HIPPO.
- The pasted key is truncated or has extra whitespace. Re-copy the full line.

**Wrong key being offered**

If you have multiple keys, SSH may try the wrong one first. Point it at the right one explicitly:

```sh
ssh -i ~/.ssh/id_ed25519 your_username@hive.hpc.ucdavis.edu
```

**Passphrase asked every time**

Load your key into the SSH agent once per session:

```sh
# macOS / Linux
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

On macOS, store it in the Keychain so you don't have to repeat this:

```sh
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

## 1.4 Security Notes

- **Never share your private key** (`id_ed25519`). Only the `.pub` file is uploaded or shared.
- **Use a passphrase** on your private key - it's your last defense if your laptop is lost or stolen.
- **Use `ed25519` keys** over older `rsa` keys - smaller, faster, more secure.
