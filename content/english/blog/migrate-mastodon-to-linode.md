---
title: "Migrate Existing Mastodon from Masto.host to Linode"
meta_title: "Mastodon Migration Guide"
description: "Guide for migrating a production Mastodon instance from Masto.host to a self-managed Linode VPS"
date: 2024-01-11T00:00:00Z
image: "/images/image-placeholder.png"
categories: ["How To"]
author: "Chris"
tags: ["mastodon", "linode", "migration", "self-hosting"]
draft: false
---

This guide addresses migrating a production Mastodon instance from Masto.host to a self-managed Linode VPS. There is nothing inherently wrong with Masto.host, but this tutorial serves a specific use case requiring independent infrastructure.

## Pre-Migration Setup

### Infrastructure Preparation

Provision a Debian 11 VPS on Linode with 4GB shared CPU, then configure hostname and timezone settings. Establish system security through:

- SSH key authentication
- Multi-factor authentication via Google Authenticator
- Disabled root login and password authentication

### Supporting Services

**Object Storage:** Create a Linode Object Storage bucket with read/write access credentials

**Email Service:** Set up Mailgun with domain verification through Cloudflare DNS

**Web Server:** Nginx configuration for the primary domain and a separate proxy for cached object storage

### Mastodon Installation

The comprehensive installation process involves:

1. Installing Node.js
2. Setting up PostgreSQL
3. Installing system packages
4. Installing Ruby via rbenv
5. Cloning the official Mastodon repository

## Migration Day Procedures

1. Request a data backup from Masto.host
2. Transfer the database via secure copy to the new server
3. Upload media files to the object storage bucket
4. Issue certificates through Certbot
5. Enable systemd services

The `.env.production` configuration file should include required environment variables for S3 storage, PostgreSQL, Redis, and SMTP settings.
