\__WIP
Before you unleash Flux to rule your cluster, a word of caution from the DevOps grimoires: **never** commit passwords, tokens, or secrets—not even to private Git repos. Just because your repo is hidden behind an access token doesn’t mean it’s safe from leaks, mistakes, or that one teammate who accidentally pushes everything to the wrong remote. Git is many things, but secure vault it is not. Treat secrets in Git like nuclear codes: don’t store them in plain text, don’t trust "I'll clean that up later," and never assume "private" means "safe."

Enter SOPS—the Mozilla-born encryption wizard that lets you safely store secrets in Git, but encrypted. It works by encrypting specific parts of your YAML files using age, GPG, or KMS backends, so the sensitive bits remain locked away even as the rest of your manifest is version-controlled. Combine this with the Flux SOPS integration, and your cluster gains the power to decrypt secrets at runtime, only when it actually needs them. This means you get all the GitOps benefits—auditable changes, full automation, version history—without broadcasting your API keys to the world (or your coworkers).

Of course, this means your cluster isn’t entirely self-managing. You’ll need to securely manage decryption keys and ensure they’re available to the right nodes at the right time. It's a small sacrifice for doing things the right way: a bit of manual setup for a massive gain in operational sanity and security. After all, true magic comes with responsibility, and we’d rather tame the beast than let it torch the village.

There’s something undeniably beautiful about learning new things — especially in the world of infrastructure, where every new tool or technique feels like unlocking a fresh layer of reality. It’s a strange kind of thrill: the excitement of cracking a complex problem, the adrenaline rush when something finally works, and yes, even the despair of debugging YAML that stares back at you with silent judgment. This path will take you into dark places - broken containers, lost packets, and mysterious logs — but with every stumble, you gain wisdom, confidence, and scars that quietly whisper “I’ve seen some real sht!” And while your future self may not remember every flag or workaround, your CV absolutely will. Because what starts as curiosity often ends as capability, and there's real power in being the person who can say, “Yeah, I’ve automated that.”


## [SOPS - https://getsops.io/](https://getsops.io/)

In the world of GitOps, where every configuration lives in a Git repository like a holy relic, we face a crucial dilemma: what do we do with secrets? Passwords, API tokens, TLS keys—all the juicy bits we don’t want publicly version-controlled, but still need for our cluster to run. This is where SOPS (Secrets OPerationS) steps in like a cryptographic guardian angel. Created by Mozilla, SOPS allows you to encrypt specific values inside YAML, JSON, ENV, or INI files, and store those encrypted files safely in your Git repo. Your Git history stays clean and trackable, and your secrets stay unreadable to all but the chosen ones — usually your cluster and you.

We need SOPS because GitOps without secret management is like castle walls with no gate: great visibility, no security. SOPS solves this without introducing another heavy subsystem — it plays nicely with tools like Flux, Age, GPG, and cloud KMS solutions (AWS, GCP, Azure). It encrypts only the secret values, not the whole file, so your manifests stay structured and diff-friendly. With SOPS and Flux working together, you can automate your entire cluster, including secrets, without ever committing plain-text passwords like a reckless YAML goblin.

⚠️ Wizard’s Warning! ⚠️

Beware, brave cluster conjurer: SOPS is not part of your Kubernetes cluster. It is a tool of the local realm, meant to be wielded on your own dev machine — where you already edit, enchant, and occasionally scream at YAML files. You use SOPS to encrypt the secrets before they ever touch Git, like sealing a scroll with magical wax before sending it to the archives. Only the encrypted files are committed; the plain ones must never leave your spellbook—or the shadow beasts of Git history will remember everything you've ever done.

In short: encrypt locally, commit cautiously, and never give your cluster a chance to leak your secrets like a drunk bard at a tavern.

## Install SOPS

SOPS is a standalone binary, and it runs happily on Linux, macOS, and even Windows. Most OS repos contains this app and you can install simply with zypper, pacman, rpm, etc.

Or, if you prefer the official binary (e.g., latest version), check the [GitHub releases page](https://github.com/getsops/sops/releases):
```bash
# Download the binary
curl -LO https://github.com/getsops/sops/releases/download/v3.10.2/sops-v3.10.2.linux.amd64

# Move the binary in to your PATH
mv sops-v3.10.2.linux.amd64 /usr/local/bin/sops

# Make the binary executable
chmod +x /usr/local/bin/sops
```
Verify it works:
```bash
sops --version
```

## Generate a GPG Key

If you don’t already have a GPG keypair, generate one now:

```bash
# Generate a GPG key specifically for SOPS
export KEY_NAME="flux-secrets"
export KEY_COMMENT="flux secrets"

gpg --batch --full-generate-key <<EOF
%no-protection
Key-Type: 1
Key-Length: 4096
Subkey-Type: 1
Subkey-Length: 4096
Expire-Date: 0
Name-Comment: ${KEY_COMMENT}
Name-Real: ${KEY_NAME}
EOF

# List your keys and find the fingerprint
gpg --list-secret-keys "${KEY_NAME}"

```

The long string (the fingerprint) is what you’ll use to tell SOPS who can decrypt secrets.

## Create a .sops.yaml Configuration File

To make your secret handling reproducible and automatic, add a .sops.yaml config to the root of your GitOps repo:

```yaml
# homelab-gitops/.sops.yaml
creation_rules:
  - path_regex: .*secret.*\.yaml$
    encrypted_regex: ^(data|stringData)$
    pgp: 71D663880FD6A35AD0B490CAD88AC9106AECCE2F  # Your fingerprint
```

## Create and Encrypt Your First Secret

