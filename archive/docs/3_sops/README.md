Before you unleash Flux to rule your cluster, a word of caution from the DevOps grimoires: **never** commit passwords, tokens, or secrets — not even to private Git repos. Just because your repo is hidden behind a password or an access token doesn’t mean it’s safe from leaks, mistakes, or that one teammate who accidentally pushes everything to the wrong remote. As Master Yoda once said: Git is many things, but secure vault it is not!

## [SOPS - https://getsops.io/](https://getsops.io/)

In the world of GitOps, where every configuration lives in a Git repository like a holy relic, we face a crucial dilemma: what do we do with secrets? Passwords, API tokens, TLS keys — all the juicy bits we don’t want publicly version-controlled, but still need for our cluster to run. This is where SOPS (Secrets OPerationS) steps in like a cryptographic guardian angel. Created by Mozilla, SOPS allows you to encrypt specific values inside YAML, JSON, ENV, or INI files, and store those encrypted files safely in your Git repo. Your Git history stays clean and trackable, and your secrets stay unreadable to all but the chosen ones — usually your cluster and you.

We need SOPS because GitOps without secret management is like castle walls with no gate: great visibility, no security. SOPS solves this without introducing another heavy subsystem — it plays nicely with tools like Flux, Age, GPG, and cloud KMS solutions (AWS, GCP, Azure). It encrypts only the secret values, not the whole file, so your manifests stay structured and diff-friendly. With SOPS and Flux working together, you can automate your entire cluster, including secrets, without ever committing plain-text passwords like a reckless YAML goblin.

⚠️ Wizard’s Warning! ⚠️

Beware, brave cluster conjurer: SOPS is not part of your Kubernetes cluster. It is a tool of the local realm, meant to be wielded on your own dev machine — where you already edit, enchant, and occasionally scream at YAML files. You use SOPS to encrypt the secrets before they ever touch Git, like sealing a scroll with magical wax before sending it to the archives. Only the encrypted files are committed; the plain ones must never leave your spellbook — or the shadow beasts of Git history will remember everything you've ever done.

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

### a) Generate a GPG Key

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

### b) WIP_Use [age (https://github.com/FiloSottile/age)](https://github.com/FiloSottile/age)
age is a simple, modern and secure file encryption tool, format, and Go library. It's recommended to use age over PGP, if possible.  
It features small explicit keys, no config options, and UNIX-style composability. Just like SOPS, most OS repos contains it and you can install simply with zypper, pacman, rpm, etc.

age-keygen -o secrets/age-key.txt
..TODO


## Create a .sops.yaml Configuration File

To make your secret handling reproducible and automatic, add a .sops.yaml config to your $HOME folder of your local machine:

```yaml
# $HOME/.sops.yaml
creation_rules:
  - path_regex: .*secret.*\.yaml$
    encrypted_regex: ^(data|stringData)$
    pgp: 71D663880FD6A35AD0B490CAD88AC9106AECCE2F  # Your fingerprint
    age: 
```

So, in this arcane glyph:
 - path_regex: .*secret.*\.yaml$ — This tells SOPS to watch all files with "secret" in the name and ending in .yaml. It doesn’t matter where they hide — in secrets/, cluster/, or buried in /mnt/yaml/mordor/doom/ — if they smell like secrets, they shall be encrypted.    
 - encrypted_regex: ^(data|stringData)$ — This narrows SOPS's power: it only encrypts the data and stringData fields, where Kubernetes secrets usually stash their precious values. This means the rest of the file (metadata, kind, etc.) remains visible for GitOps and version control—visible structure, hidden content.  
 - pgp: 71D6...CE2F — This is the GPG fingerprint of the key used to lock and unlock your secrets. Think of it as your magical sigil—only those who hold the matching private key may reveal what lies within the encrypted scrolls.

## Create and Encrypt Your First Secret

It all starts with a simple secret, where we want to store the login credentials of the Archmage.
```yaml
# ../secrets/default/my-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
stringData:
  username: admin
  password: super-secret-magical-password
```

Use your prefered text editor and save the file for example: secrets/default/my-secret.yaml  
And now encrypt it.
```bash
sops -e -i secrets/default/my-secret.yaml
```

After this, open the file and admire the magical ciphertext — your cluster will later decrypt it, but it’s unreadable to mere mortals.
```bash
$ cat my-secret.yaml 
apiVersion: v1
kind: Secret
metadata:
    name: my-secret
    namespace: default
type: Opaque
stringData:
    username: ENC[AES256_GCM,data:Fy4ymyw=,iv:enenntj+ou1nwRQm2r20E7RGoVqhA7LNf3FNijzVf/8=,tag:cYS67nRY8ww44F74lvgnXA==,type:str]
    password: ENC[AES256_GCM,data:8xnmtAA75cbupScJPeXypykGfBfcCaQDdL41Wfg=,iv:DpzHVCHpvYMnXzLCU7NoCQrZfF25GrF3k/nH/EfqDxw=,tag:JIOL4Szm6XmYJDUaYcpLGg==,type:str]
sops:
    lastmodified: "2025-07-25T09:03:10Z"
    mac: ENC[AES256_GCM,data:1mfqv+pqAwc/HlM1E+fGNuEBOwIHNUzu+Q+RoKoWfmM8HvFO2cVlqXrPZBCmdiQ8M/xN2xFfQIum4mmle7zMXV/qWTAaxK3jZUhuQLOUzYtdiR0FI95Q5LmzTpFrIr28eU7WzW/EkB9WlyNWmITpxSeIV+eCZ4CBCxAVqeOXU5Y=,iv:IikDDZ+R85AkYM+npJN0qUVk7BKop2wlEziTxTSu6ek=,tag:uR9VqseaHUJFuGhcQ9v5rA==,type:str]
    pgp:
        - created_at: "2025-07-25T09:03:10Z"
          enc: |-
            -----BEGIN PGP MESSAGE-----

            hQIMAwjUHBw2Q75RARAAp0Wef70G++auYDmgdoYbDg/RHYUvSKvjTqu5Z4f4tA8q
            f7dn94dOJpHQ/r3wAvXXK1iVj9qWZT2M52x32nY1FlogE1759eIzexAA69sJtv4I
            BaRQNQREvqHn7hD1z11rwGPEmwYnzhBOEa+PvXOKkCebdp4NkHoN0FovrOdz1YNd
            pV0gFCqO38yI+SwLLVN0ZXc4I9zF07F/WA6/OskxEthFgRVw+U0A+8cZsgd0NFO+
            XBvN2SVqm/LHvZeL9/wl6TPtuPKgJGgwDnpHyHP5VfulLMzZhvXOtr6+OHPYuVIs
            oSnK9E6VjduR47VpTAAQu+29RUlRPbexDMql1eCCqRj/Jr9XL+oJhRV4gaOl1GLr
            YLzVeVIhpAwqKoITx5rDAVjp6YNkjJ7LP1MglqOItXzxAHt5DvUxhRTaX949/t6j
            b6h9IWIAJ7xuZIg5Kb1A2yDoIRH6bex19HwSJAVtcanVBzQEcdo5l9SOe2RjAmg2
            PvIzosuJdK9TwHwAZKjpXqa/vvouYKFamZE5GDm7NnzeXuZyEIWIJ6BpSGA1bCyk
            5hwcRz+nts+FJld9P8bplTKypv39tYlbFh2qUi5ig4RFlbWQ2M3lTuNkwPhvNHJ0
            uHjuK1tDezrtkUQLO1DmUKKXO3NJV9s8UMA0CjgS/App2VU0UBLcFtBnnI6fW1nS
            XgFERSFdFrHMeaP3MqlHi4KmGjowEuqUK4whbd2iBB0yEtfocMXaqGCC9s6HtWOr
            0dQrFPWPhf8YUgQKptxKx3ohbqIsKaGr8KuKTuFCmUXmEbl/1wDDPN0WGhssDjo=
            =/elf
            -----END PGP MESSAGE-----
          fp: 71D663880FD6A35AD0B490CAD88AC9106AECCE2F
    encrypted_regex: ^(data|stringData)$
    version: 3.10.2
```

SOPS handles the encrypt/decrypt procedure, to make changes and save encrypted automatically, use:
```bash
sops secrets/default/my-secret.yaml.
```


### What happens, when you need to edit secrets but don't have the key?

Well... 

When you try to edit secrets without the key, the encrypted YAML just stares back at you — cold, unreadable, and mocking your lack of magical credentials. You can push changes, sure... but your cluster won’t decrypt them, and Flux will quietly ignore your feeble mortal scribbles.

#### Import you PGP key on a new machine

```bash
$ gpg --import flux-secrets.asc

$ gpg --list-keys flux-secrets
pub   rsa4096 2025-07-19 [SCEAR]
      71D663880FD6A35AD0B490CAD88AC9106AECCE2F
uid           [ unknown] flux-secrets (flux secrets)
sub   rsa4096 2025-07-19 [SEA]
```

## Create a kube secret from the key

Export your GPG private key (the one matching the fingerprint in .sops.yaml) and create a sealed artifact Flux can use:

```bash
gpg --export-secret-keys --armor "${KEY_NAME}" > flux-secrets.asc
```
Then, apply it to your cluster as a Kubernetes secret, in the same namespace where Flux lives (usually flux-system). The name sops-gpg is special — Flux will automatically look for it. If you name it differently, you’ll have to configure more settings, and the logs will get cryptic fast.

```bash
kubectl create secret generic sops-gpg --namespace=flux-system --from-file=sops.asc=flux-secrets.asc
```

From now on, Flux recognizes your encrypted secret by the presence of a magical glyph — specifically, the sops: metadata block embedded in the YAML file. When it sees this enchanted signature, it knows, "Ah, this is no ordinary config… this one is sealed." Then it invokes SOPS during reconciliation, decrypts the values using your sops-gpg secret, and applies the result as if it had always known the password. No special annotation, no secret handshake—just proper encryption and a key in the right place.

The final binding — a sacred connection between your secrets/ folder and Flux, so your encrypted secrets are always watched, decrypted, and applied without you lifting a terminal finger.

```yaml
# k3s-homelab/cluster/secrets.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: secrets
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./secrets
  prune: true
  wait: true
  timeout: 5m
  decryption:
    provider: sops
    secretRef:
      name: sops-gpg
  dependsOn:
    - name: helm-repositories # Wait for at least the Helm repositories
```

This tells Flux:
 - "Check the ./secrets folder every 10 minute."
 - "Use the sops-gpg secret to decrypt things."
 - "Apply everything into the defined namespace."
 - "And if I remove a file from Git, remove it from the cluster too (prune: true)."

 And after a few seconds...

```bash
k3s1:~ # k get secrets
NAME        TYPE     DATA   AGE
my-secret   Opaque   2      19s


k3s1:~ # k get secret my-secret -o yaml
apiVersion: v1
data:
  password: c3VwZXItc2VjcmV0LW1hZ2ljYWwtcGFzc3dvcmQ=
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2025-07-25T10:10:26Z"
  labels:
    kustomize.toolkit.fluxcd.io/name: secrets
    kustomize.toolkit.fluxcd.io/namespace: flux-system
  name: my-secret
  namespace: default
  resourceVersion: "6170661"
  uid: 5d295e92-139d-492f-9c6b-289559d1dc57
type: Opaque
```
