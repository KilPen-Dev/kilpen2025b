---
title: "Image Conversion Using API"
meta_title: "Automated Image Conversion Solution"
description: "Automated solution for batch image format conversion using Google Workspace and APIs"
date: 2023-06-01T00:00:00Z
image: "/images/image-placeholder.png"
categories: ["Solutions"]
author: "Chris"
tags: ["automation", "api", "google-workspace"]
draft: false
---

## Business Problem

The client required converting over 1,000 images from PNG to TIFF format. A manual approach would demand substantial time and resources.

## Mission

Create an automated solution integrating into the current workflow that would seamlessly convert images into the TIFF format and sort output files into appropriate folders according to an information management scheme.

## Solution

KilPen implemented a system leveraging Google Workspace and an API-based image converter. The automation:

- Ran hourly to detect newly generated images
- Submitted images to the conversion API
- Used Google Apps Script to retrieve converted files
- Stored converted files in Google Drive
- Applied a sorting algorithm to organize files into designated locations

## Timeline

**Completion:** One week
