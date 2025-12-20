---
title: "Running Ubuntu Desktop in Google Cloud Platform"
meta_title: "Ubuntu Desktop on GCP"
description: "Set up a virtual Ubuntu Desktop environment on Google Cloud Platform's Compute Engine"
date: 2025-03-27T00:00:00Z
image: "/images/image-placeholder.png"
categories: ["How To"]
author: "Chris"
tags: ["gcp", "ubuntu", "cloud", "remote-desktop"]
draft: false
---

This tutorial guides you through setting up a virtual Ubuntu Desktop environment on Google Cloud Platform's Compute Engine service.

## Project and VM Creation

The process begins by establishing a new GCP project, then creating a VM instance using an E2 series General Purpose machine (approximately $25/month operational cost) with Ubuntu 20.04 LTS.

## Remote Desktop Setup

The installation involves three key components:

- Chrome Remote Desktop software installation
- Simply Login Manager (SLiM) for graphical display management
- Service configuration through Google's headless setup portal

### Installation Commands

```bash
sudo apt update && sudo apt upgrade
```

Install Chrome Remote Desktop and SLiM desktop manager following Google's official documentation.

## Connection Process

1. Authenticate Chrome Remote Desktop via Google's authorization system
2. Set a personal 6-digit PIN
3. Access your virtual desktop remotely through the Chrome browser interface

This practical guide targets users seeking cloud-based Ubuntu desktop access without extensive infrastructure management overhead.
