---
title: "Automating PGP Key Issuance and Distribution with GPG and 1Password"
meta_title: "Automated PGP Key Management"
description: "Centralizing PGP key management within an organization using Debian, GnuPG, and 1Password CLI"
date: 2023-08-25T00:00:00Z
image: "/images/image-placeholder.png"
categories: ["How To"]
author: "Chris"
tags: ["security", "pgp", "gpg", "1password", "automation"]
draft: false
---

This article describes an approach to centralizing PGP key management within an organization using a Debian 12 server, GnuPG, and 1Password CLI for automated issuance and distribution.

## Infrastructure Setup

The implementation uses a hardened Debian 12 "Bookworm" instance with restricted network access via Cloudflare WARP and SSH authentication requiring certificates, passwords, and two-factor OTP.

### Required Packages

- openssh-server
- libpam-google-authenticator
- nano
- sudo
- gnupg
- curl
- 1password-cli
- jq

```bash
sudo apt install -y openssh-server libpam-google-authenticator nano gnupg curl
```

The 1Password CLI requires separate installation from their official repository using GPG key verification and debsig-verify policy configuration.

## Automation Process

The workflow creates vaults, generates PGP credentials, and distributes keys through these steps:

1. Create user-specific 1Password vault
2. Generate random PGP passphrase stored in 1Password
3. Create master key with ECC subkey for encryption
4. Generate signing subkey
5. Export and upload public key to user vault and company key ring
6. Export and upload private subkeys to user vault

## The Script

A BASH script (`issueKey.sh`) automates the entire process using syntax:

```bash
./issueKey [Full Name] [email address]
```

The script generates ECC keys with Curve25519 encryption and Ed25519 signing, storing passphrases securely in 1Password vaults with appropriate permissions for users and administrators.
