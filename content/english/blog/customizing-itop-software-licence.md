---
title: "Customizing iTop to Add Extra Fields to the Software Licence Object"
meta_title: "Customizing iTop Software Licence Fields"
description: "A comprehensive guide for extending iTop's CMDB by adding custom fields to Software Licence objects"
date: 2025-08-03T00:00:00Z
image: "/images/image-placeholder.png"
categories: ["How To"]
author: "Chris"
tags: ["itop", "cmdb", "customization"]
draft: false
---

This guide provides a comprehensive walkthrough for extending iTop's Configuration Management Database (CMDB) by adding custom fields to Software Licence objects. By default, these objects contain limited properties, so this tutorial walks developers through creating extension modules to add attributes like renewal dates, vendor information, and support end dates.

## Module Creation

Users must install a development iTop instance and utilize Combodo's module wizard to generate skeleton extensions. The module should depend on `itop-config-mgmt` to ensure proper loading sequence.

## Field Definition

New attributes are declared in XML with the `_delta="define"` attribute. The guide demonstrates adding five sample fields:

- Renewal date
- Vendor name
- Licence cost (decimal)
- Support end date
- Purchase order number

Each field specifies appropriate data types.

## User Interface Configuration

Fields must be made visible by redefining the presentation block with `_delta="redefine"`, allowing developers to customize details, list, and search views with ranked item positioning.

## Dictionary Entries

User-friendly labels require entries following the pattern `'Class:SoftwareLicence/Attribute:field_name'` in language dictionary files.

## Compilation & Testing

The toolkit validates XML and dictionary files, then updates both code and database schema. Setup must run again to register the module and clear caches.

## Best Practices

- Test on non-production systems first
- Use enumeration fields for controlled vocabularies
- Note that professional editions offer graphical ITSM Designer tools as alternatives to manual XML editing
