---
title: "Extending iTop's WebApplication Object with Custom Fields"
meta_title: "Extending iTop WebApplication Object"
description: "A comprehensive guide for extending iTop's CMDB by adding custom fields to the WebApplication object"
date: 2025-12-24T00:00:00Z
image: "/images/solutions/extending-itop-webapplication-object.png"
categories: ["How To"]
author: "Chris"
tags: ["itop", "cmdb", "customization"]
draft: false
---

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="kilpen" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>

SKILL LEVEL:  Advanced.

# Extending iTop's WebApplication Object with Custom Fields

iTop is a powerful open-source IT Service Management (ITSM) tool, but sometimes the default object properties don't capture all the information you need. In this guide, we'll create a custom extension to add new fields to the WebApplication object, perfect for tracking SaaS subscriptions, costs, and ownership.

## What We're Building

We'll add the following fields to the WebApplication object:

| Field | Type | Purpose |
|-------|------|---------|
| Payment Method | String | How the service is paid |
| 1Password Vault | String | Credential storage location |
| Action to Take | Dropdown | keep, cancel, unsure, downgrade |
| Plan Type | Dropdown | monthly, annual, by use |
| Renewal Date | Date | Next renewal/expiration |
| Annual Cost | Decimal | Expected yearly cost |
| Year to Date Cost | Decimal | Spent this year |
| Accounting Tag | String | Expense tracking tag |
| Accounting Category | Dropdown | General, Programmatic, Fundraising, Other |
| Purpose | Text | What the app is used for |
| Technical Notes | Text | Additional documentation |
| Technical Owner | Link to Person | Responsible person |
| Privacy Policy URL | URL | Vendor privacy policy |
| Terms of Service URL | URL | Vendor ToS |

## Extension Structure

Create a folder called `webapp-extended-attributes` with the following files:

```bash
webapp-extended-attributes/
├── module.webapp-extended-attributes.php
├── datamodel.webapp-extended-attributes.xml
├── en.dict.webapp-extended-attributes.php
└── extension.xml
```

## Extension Files

### module.webapp-extended-attributes.php

```php
<?php
/**
 * Module definition for WebApplication Extended Attributes
 */

SetupWebPage::AddModule(
    __FILE__,
    'webapp-extended-attributes/1.0.0',
    array(
        'label' => 'WebApplication Extended Attributes',
        'category' => 'business',
        'dependencies' => array(
            'itop-config-mgmt/2.0.0',
        ),
        'mandatory' => false,
        'visible' => true,
        'auto_select' => 'SetupInfo::ModuleIsSelected("itop-config-mgmt")',
        'datamodel' => array(
            'datamodel.webapp-extended-attributes.xml',
        ),
        'data.struct' => array(),
        'data.sample' => array(),
        'dictionary' => array(
            'en.dict.webapp-extended-attributes.php',
        ),
        'webservice' => array(),
        'doc.manual_setup' => '',
        'doc.more_information' => '',
    )
);
```

### datamodel.webapp-extended-attributes.xml

> **Note:** Do not include the `<?xml version="1.0"?>` declaration at the top of this file. Some PHP configurations have `short_open_tag` enabled, which causes PHP to misinterpret `<?xml` as a PHP opening tag.

```xml
<itop_design xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="3.0">
  <classes>
    <class id="WebApplication" _delta="must_exist">
      <fields>
        <!-- Payment & Subscription -->
        <field id="payment_method" xsi:type="AttributeString" _delta="define">
          <sql>payment_method</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <field id="onepassword_vault" xsi:type="AttributeString" _delta="define">
          <sql>onepassword_vault</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <field id="action_to_take" xsi:type="AttributeEnum" _delta="define">
          <sql>action_to_take</sql>
          <values>
            <value id="keep">
              <code>keep</code>
            </value>
            <value id="cancel">
              <code>cancel</code>
            </value>
            <value id="unsure">
              <code>unsure</code>
            </value>
            <value id="downgrade">
              <code>downgrade</code>
            </value>
          </values>
          <default_value>unsure</default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <field id="plan_type" xsi:type="AttributeEnum" _delta="define">
          <sql>plan_type</sql>
          <values>
            <value id="monthly">
              <code>monthly</code>
            </value>
            <value id="annual">
              <code>annual</code>
            </value>
            <value id="by_use">
              <code>by_use</code>
            </value>
          </values>
          <default_value>monthly</default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <field id="renewal_date" xsi:type="AttributeDate" _delta="define">
          <sql>renewal_date</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <!-- Cost Information -->
        <field id="annual_cost" xsi:type="AttributeDecimal" _delta="define">
          <sql>annual_cost</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
          <digits>12</digits>
          <decimals>2</decimals>
        </field>

        <field id="year_to_date_cost" xsi:type="AttributeDecimal" _delta="define">
          <sql>year_to_date_cost</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
          <digits>12</digits>
          <decimals>2</decimals>
        </field>

        <!-- Accounting -->
        <field id="accounting_tag" xsi:type="AttributeString" _delta="define">
          <sql>accounting_tag</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <field id="accounting_category" xsi:type="AttributeEnum" _delta="define">
          <sql>accounting_category</sql>
          <values>
            <value id="general">
              <code>general</code>
            </value>
            <value id="programmatic">
              <code>programmatic</code>
            </value>
            <value id="fundraising">
              <code>fundraising</code>
            </value>
            <value id="other">
              <code>other</code>
            </value>
          </values>
          <default_value>general</default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <!-- Documentation & Purpose -->
        <field id="purpose" xsi:type="AttributeText" _delta="define">
          <sql>purpose</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <field id="technical_notes" xsi:type="AttributeText" _delta="define">
          <sql>technical_notes</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
        </field>

        <!-- Legal URLs -->
        <field id="privacy_policy_url" xsi:type="AttributeURL" _delta="define">
          <sql>privacy_policy_url</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
          <target>_blank</target>
        </field>

        <field id="tos_url" xsi:type="AttributeURL" _delta="define">
          <sql>tos_url</sql>
          <default_value></default_value>
          <is_null_allowed>true</is_null_allowed>
          <target>_blank</target>
        </field>

        <!-- Technical Owner (link to Person) -->
        <field id="technical_owner_id" xsi:type="AttributeExternalKey" _delta="define">
          <sql>technical_owner_id</sql>
          <target_class>Person</target_class>
          <is_null_allowed>true</is_null_allowed>
          <on_target_delete>DEL_MANUAL</on_target_delete>
          <filter><![CDATA[SELECT Person]]></filter>
        </field>

        <field id="technical_owner_name" xsi:type="AttributeExternalField" _delta="define">
          <extkey_attcode>technical_owner_id</extkey_attcode>
          <target_attcode>friendlyname</target_attcode>
        </field>
      </fields>

      <presentation>
        <details>
          <items>
            <!-- Subscription & Payment fieldset -->
            <item id="fieldset:WebApplication:subscription" _delta="define">
              <rank>30</rank>
              <items>
                <item id="payment_method">
                  <rank>10</rank>
                </item>
                <item id="onepassword_vault">
                  <rank>20</rank>
                </item>
                <item id="action_to_take">
                  <rank>30</rank>
                </item>
                <item id="plan_type">
                  <rank>40</rank>
                </item>
                <item id="renewal_date">
                  <rank>50</rank>
                </item>
              </items>
            </item>

            <!-- Cost fieldset -->
            <item id="fieldset:WebApplication:costs" _delta="define">
              <rank>40</rank>
              <items>
                <item id="annual_cost">
                  <rank>10</rank>
                </item>
                <item id="year_to_date_cost">
                  <rank>20</rank>
                </item>
                <item id="accounting_tag">
                  <rank>30</rank>
                </item>
                <item id="accounting_category">
                  <rank>40</rank>
                </item>
              </items>
            </item>

            <!-- Documentation fieldset -->
            <item id="fieldset:WebApplication:documentation" _delta="define">
              <rank>50</rank>
              <items>
                <item id="purpose">
                  <rank>10</rank>
                </item>
                <item id="technical_notes">
                  <rank>20</rank>
                </item>
                <item id="technical_owner_id">
                  <rank>30</rank>
                </item>
                <item id="privacy_policy_url">
                  <rank>40</rank>
                </item>
                <item id="tos_url">
                  <rank>50</rank>
                </item>
              </items>
            </item>
          </items>
        </details>

        <search>
          <items>
            <item id="action_to_take" _delta="define">
              <rank>110</rank>
            </item>
            <item id="accounting_category" _delta="define">
              <rank>120</rank>
            </item>
            <item id="plan_type" _delta="define">
              <rank>130</rank>
            </item>
            <item id="technical_owner_id" _delta="define">
              <rank>140</rank>
            </item>
            <item id="renewal_date" _delta="define">
              <rank>150</rank>
            </item>
          </items>
        </search>

        <list>
          <items>
            <item id="action_to_take" _delta="define">
              <rank>110</rank>
            </item>
            <item id="renewal_date" _delta="define">
              <rank>120</rank>
            </item>
            <item id="annual_cost" _delta="define">
              <rank>130</rank>
            </item>
          </items>
        </list>
      </presentation>
    </class>
  </classes>
</itop_design>
```

### en.dict.webapp-extended-attributes.php

```php
<?php
/**
 * English dictionary for WebApplication Extended Attributes
 */

// Fieldset labels
Dict::Add('EN US', 'English', 'English', array(
    'Class:WebApplication/Attribute:fieldset:subscription' => 'Subscription & Payment',
    'Class:WebApplication/Attribute:fieldset:costs' => 'Cost Information',
    'Class:WebApplication/Attribute:fieldset:documentation' => 'Documentation & Ownership',
));

// Field labels
Dict::Add('EN US', 'English', 'English', array(
    // Subscription & Payment
    'Class:WebApplication/Attribute:payment_method' => 'Payment Method',
    'Class:WebApplication/Attribute:payment_method+' => 'How this service is paid for (e.g., credit card, invoice)',

    'Class:WebApplication/Attribute:onepassword_vault' => '1Password Vault',
    'Class:WebApplication/Attribute:onepassword_vault+' => 'Name of the 1Password vault containing credentials',

    'Class:WebApplication/Attribute:action_to_take' => 'Action to Take',
    'Class:WebApplication/Attribute:action_to_take+' => 'Planned action for this subscription',
    'Class:WebApplication/Attribute:action_to_take/Value:keep' => 'Keep',
    'Class:WebApplication/Attribute:action_to_take/Value:keep+' => 'Continue the subscription',
    'Class:WebApplication/Attribute:action_to_take/Value:cancel' => 'Cancel',
    'Class:WebApplication/Attribute:action_to_take/Value:cancel+' => 'Cancel the subscription',
    'Class:WebApplication/Attribute:action_to_take/Value:unsure' => 'Unsure',
    'Class:WebApplication/Attribute:action_to_take/Value:unsure+' => 'Decision pending',
    'Class:WebApplication/Attribute:action_to_take/Value:downgrade' => 'Downgrade',
    'Class:WebApplication/Attribute:action_to_take/Value:downgrade+' => 'Downgrade to a lower tier',

    'Class:WebApplication/Attribute:plan_type' => 'Plan Type',
    'Class:WebApplication/Attribute:plan_type+' => 'Billing cycle for this service',
    'Class:WebApplication/Attribute:plan_type/Value:monthly' => 'Monthly',
    'Class:WebApplication/Attribute:plan_type/Value:monthly+' => 'Billed monthly',
    'Class:WebApplication/Attribute:plan_type/Value:annual' => 'Annual',
    'Class:WebApplication/Attribute:plan_type/Value:annual+' => 'Billed annually',
    'Class:WebApplication/Attribute:plan_type/Value:by_use' => 'By Use',
    'Class:WebApplication/Attribute:plan_type/Value:by_use+' => 'Billed based on usage',

    'Class:WebApplication/Attribute:renewal_date' => 'Renewal Date',
    'Class:WebApplication/Attribute:renewal_date+' => 'Next renewal or expiration date',

    // Cost Information
    'Class:WebApplication/Attribute:annual_cost' => 'Annual Cost',
    'Class:WebApplication/Attribute:annual_cost+' => 'Expected annual cost for this service',

    'Class:WebApplication/Attribute:year_to_date_cost' => 'Year to Date Cost',
    'Class:WebApplication/Attribute:year_to_date_cost+' => 'Amount spent this year so far',

    'Class:WebApplication/Attribute:accounting_tag' => 'Accounting Tag',
    'Class:WebApplication/Attribute:accounting_tag+' => 'Tag used for accounting/expense tracking',

    'Class:WebApplication/Attribute:accounting_category' => 'Accounting Category',
    'Class:WebApplication/Attribute:accounting_category+' => 'Category for financial reporting',
    'Class:WebApplication/Attribute:accounting_category/Value:general' => 'General',
    'Class:WebApplication/Attribute:accounting_category/Value:general+' => 'General operating expense',
    'Class:WebApplication/Attribute:accounting_category/Value:programmatic' => 'Programmatic',
    'Class:WebApplication/Attribute:accounting_category/Value:programmatic+' => 'Program-related expense',
    'Class:WebApplication/Attribute:accounting_category/Value:fundraising' => 'Fundraising',
    'Class:WebApplication/Attribute:accounting_category/Value:fundraising+' => 'Fundraising expense',
    'Class:WebApplication/Attribute:accounting_category/Value:other' => 'Other',
    'Class:WebApplication/Attribute:accounting_category/Value:other+' => 'Other category',

    // Documentation & Ownership
    'Class:WebApplication/Attribute:purpose' => 'Purpose',
    'Class:WebApplication/Attribute:purpose+' => 'What this application is used for',

    'Class:WebApplication/Attribute:technical_notes' => 'Technical Notes',
    'Class:WebApplication/Attribute:technical_notes+' => 'Additional technical information or notes',

    'Class:WebApplication/Attribute:technical_owner_id' => 'Technical Owner',
    'Class:WebApplication/Attribute:technical_owner_id+' => 'Person responsible for technical oversight',
    'Class:WebApplication/Attribute:technical_owner_name' => 'Technical Owner Name',
    'Class:WebApplication/Attribute:technical_owner_name+' => 'Name of the technical owner',

    'Class:WebApplication/Attribute:privacy_policy_url' => 'Privacy Policy URL',
    'Class:WebApplication/Attribute:privacy_policy_url+' => 'Link to the vendor\'s privacy policy',

    'Class:WebApplication/Attribute:tos_url' => 'Terms of Service URL',
    'Class:WebApplication/Attribute:tos_url+' => 'Link to the vendor\'s terms of service',
));
```

### extension.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<extension format="1.0">
    <extension_code><![CDATA[webapp-extended-attributes]]></extension_code>
    <company><![CDATA[]]></company>
    <author><![CDATA[Custom Extension]]></author>
    <label><![CDATA[WebApplication Extended Attributes]]></label>
    <description><![CDATA[Adds additional properties to WebApplication objects including payment information, cost tracking, accounting categories, ownership, and legal documentation links.]]></description>
    <version><![CDATA[1.0.0]]></version>
    <release_date><![CDATA[2024-12-23]]></release_date>
    <itop_version_min><![CDATA[3.0.0]]></itop_version_min>
    <status>stable</status>
    <mandatory>false</mandatory>
    <more_info_url><![CDATA[]]></more_info_url>
</extension>
```

## Installation

1. Copy the `webapp-extended-attributes` folder to your iTop `extensions/` directory

2. Run the setup wizard at `https://your-itop/setup/`

3. Select "WebApplication Extended Attributes" in the extensions list

4. Complete the setup wizard

Your WebApplication objects will now display three new fieldsets: Subscription & Payment, Cost Information, and Documentation & Ownership.

## Bonus: Syncing Production to Development

If you maintain separate production and development iTop instances, you can use this script to sync your production database to development. This is useful for testing with real data.

### sync-prod-to-dev.sh

```bash
#!/bin/bash
# Sync iTop production database to development
# Runs unattended for cron scheduling

set -e

DB_PASSWORD='your-database-password'
BACKUP_DIR="/tmp"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "=== iTop Production to Dev Sync - $TIMESTAMP ==="

# Step 1: Backup dev (safety net)
echo "[$(date)] Backing up dev database..."
docker exec itop-dev-db mariadb-dump -u root -p"$DB_PASSWORD" itop > "$BACKUP_DIR/dev_backup_$TIMESTAMP.sql"

# Step 2: Export production
echo "[$(date)] Exporting production database..."
docker exec itop-db mariadb-dump -u root -p"$DB_PASSWORD" itop > "$BACKUP_DIR/prod_export.sql"

# Step 3: Stop dev iTop
echo "[$(date)] Stopping dev iTop..."
docker stop itop-dev

# Step 4: Import to dev
echo "[$(date)] Importing to dev database..."
cat "$BACKUP_DIR/prod_export.sql" | docker exec -i itop-dev-db mariadb -u root -p"$DB_PASSWORD" itop

# Step 5: Start dev and clear cache only (don't touch env-production)
echo "[$(date)] Starting dev iTop and clearing cache..."
docker start itop-dev
sleep 10

docker exec itop-dev rm -rf /var/www/html/data/cache-*

# Step 6: Cleanup temp export file (keep backups)
rm -f "$BACKUP_DIR/prod_export.sql"

# Step 7: Cleanup old backups (keep last 7 days)
find "$BACKUP_DIR" -name "dev_backup_*.sql" -mtime +7 -delete 2>/dev/null || true

echo "[$(date)] === Sync complete ==="
echo "Dev backup saved to: $BACKUP_DIR/dev_backup_$TIMESTAMP.sql"
```

### Setting Up Automated Sync

1. Make the script executable:
```bash
chmod +x sync-prod-to-dev.sh
```

2. Add a cron job for nightly sync at 2 AM:
```bash
crontab -e
```

Add this line:
```bash
0 2 * * * /path/to/sync-prod-to-dev.sh >> /var/log/itop-sync.log 2>&1
```

**Prerequisites:**
- Both instances must have the same iTop version
- Both instances must have the same extensions installed
- Adjust container names (`itop`, `itop-db`, `itop-dev`, `itop-dev-db`) to match your environment

## Resources

- [iTop Documentation](https://www.itophub.io/wiki/page?id=3_2_0:start)
- [XML Data Model Reference](https://www.itophub.io/wiki/page?id=3_2_0:customization:xml_reference)
- [iTop Toolkit](https://github.com/Combodo/itop-toolkit-community/releases)
