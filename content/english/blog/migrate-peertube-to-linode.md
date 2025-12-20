---
title: "Migrate Existing Docker Peertube Instance to Linode, Enable Object Storage"
meta_title: "Peertube Migration Guide"
description: "Relocate a Peertube video hosting platform to Linode while implementing object storage"
date: 2023-07-25T00:00:00Z
image: "/images/image-placeholder.png"
categories: ["How To"]
author: "Chris"
tags: ["peertube", "docker", "linode", "migration", "object-storage"]
draft: false
---

This guide walks through relocating a Peertube video hosting platform from one cloud provider to Linode while implementing object storage capabilities.

The migration transforms the setup from local Docker storage to cloud-based object storage, while maintaining core services like Nginx, PostgreSQL, Redis, and email relay functionality.

## Key Configuration Changes

**Original Setup (Peertube A):**
- Self-contained Docker volumes for storage and database
- Email relay via Google SMTP

**Target Setup (Peertube B):**
- Linode Object Storage for video files
- MailGun SMTP integration
- Database remains containerized on VPS

## Main Steps

1. **Provision Linode VPS** - Create instance with Debian 11, 4GB RAM
2. **Secure the Server** - Update packages, configure SSH, disable root access
3. **Install Docker** - Add Docker repositories and install Docker Compose
4. **Configure Services** - Deploy Peertube, PostgreSQL, Redis, and Nginx using provided compose files
5. **Migrate Data** - Use rsync to transfer existing database and storage folders
6. **Move Videos to Object Storage** - Execute Peertube's built-in migration script

## Notable Technical Details

The implementation uses Cloudflare Origin Certificates rather than Let's Encrypt, requiring custom Nginx configuration adjustments.

> **Important:** Domain names are definitive after initial Peertube startup, making pre-migration planning essential.
