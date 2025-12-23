---
title: "Customizing iTop to Add Extra Fields to the Software Licence Object"
meta_title: "Customizing iTop Software Licence Fields"
description: "A comprehensive guide for extending iTop's CMDB by adding custom fields to Software Licence objects"
date: 2025-08-03T00:00:00Z
image: "/images/solutions/customizing-itop.png"
categories: ["How To"]
author: "Chris"
tags: ["itop", "cmdb", "customization"]
draft: false
---

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="kilpen" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>

SKILL LEVEL:  Advanced.


## Background ##
In iTop’s CMDB, Software Licence objects are defined by the Configuration Management (CMDB) module. By default they hold a small set of properties: Name, Software, Organization, Usage limit, Description, Perpetual flag, Start date, End date and Key. For many asset‑management activities, IT managers need extra attributes (e.g., renewal date, vendor, license cost, support end date or PO number) that are not present in the standard schema. iTop is designed to be extended through custom modules; these modules modify the data model without altering core files. The open‑source (community) edition lacks a graphical editor, so fields are added via XML definitions.

The instructions below describe how to create an extension module and add new fields to the SoftwareLicence class. They are based on Combodo’s official tutorial for adding fields and the data model reference. Always work on a development instance and keep backups before applying changes in production.

## Create an extension module ##
1. **Install a development copy of iTop.** Never experiment directly on your production CMDB. The development instance should have write permissions on the conf/production/config-itop.php file.
2. **Download the “module creation wizard”.** Combodo provides an online wizard that generates a skeleton module. Supply a unique module name (e.g., software-licence-custom) and version; specify the category business and add a dependency on itop-config-mgmt (the module that defines the licence classes) so your module loads after the core module.
3. **Extract the module into the extensions/ folder of your development iTop.** You should now have a directory containing files such as:
    1. datamodel.<module_name>.xml – the XML definition of your data model customisations
    2. module.<module_name>.php – registration stub
    3. en.dict.<module_name>.php – dictionary for labels and translations
    4. model.<module_name>.php – empty PHP file for advanced customisations (not needed for extra fields)
4. **Run the iTop setup (/setup/)** and choose Update an existing instance to install the empty module. After installation, your module will appear in the list of extensions.

## Define the new fields ##
1. **Open the file datamodel.<module>.xml** in a UTF‑8‑capable text editor.
2. **Remove unnecessary tags (e.g., <menus>).** Within the <classes> tag, define modifications to the SoftwareLicence class. New fields must specify _delta="define" to tell iTop that they are additions. 

```xml
<itop_design xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="1.0">
  <classes>
    <class id="SoftwareLicence">
      <fields>
        <!-- Renewal date: date field -->
        <field id="renewal_date" xsi:type="AttributeDate" _delta="define">
          <sql>renewal_date</sql>
          <default_value/>
          <is_null_allowed>true</is_null_allowed>
        </field>
        <!-- Vendor name: short text -->
        <field id="vendor" xsi:type="AttributeString" _delta="define">
          <sql>vendor</sql>
          <default_value/>
          <is_null_allowed>true</is_null_allowed>
        </field>
        <!-- Licence cost: decimal value with 2 decimals -->
        <field id="licence_cost" xsi:type="AttributeDecimal" _delta="define">
          <sql>licence_cost</sql>
          <default_value/>
          <is_null_allowed>true</is_null_allowed>
          <digits>10</digits>
          <decimals>2</decimals>
        </field>
        <!-- Support end date: date field -->
        <field id="support_end" xsi:type="AttributeDate" _delta="define">
          <sql>support_end</sql>
          <default_value/>
          <is_null_allowed>true</is_null_allowed>
        </field>
        <!-- Purchase order number: free‑form text -->
        <field id="po_number" xsi:type="AttributeString" _delta="define">
          <sql>po_number</sql>
          <default_value/>
          <is_null_allowed>true</is_null_allowed>
        </field>
      </fields>
    </class>
  </classes>
</itop_design>
```

1. **id** – the internal attribute code; avoid spaces and accents.
2. **xsi:type** – the attribute type from the XML data model reference. Common types include AttributeString, AttributeText (multiline), AttributeDate, AttributeDateTime, AttributeDecimal, AttributeEnum (for drop‑down lists) and AttributeExternalKey (for links to other objects).
3. **`<sql>`** – the column name in the database table. It is good practice to use the same name as the attribute code.
4. **`<is_null_allowed>`** – whether the field can be empty. Set to false if you want the field mandatory.
5. **Additional tags** (e.g., <digits> and <decimals> for decimals; <values> for enumerations). For an enum field, use:
```xml
<field id="maintenance_type" xsi:type="AttributeEnum" _delta="define">
  <sql>maintenance_type</sql>
  <default_value/>
  <is_null_allowed>true</is_null_allowed>
  <values>
    <value id="renewal">Renewal</value>
    <value id="subscription">Subscription</value>
    <value id="perpetual">Perpetual</value>
  </values>
</field>
```

## Make the fields visible in the user interface ##
Defining a field creates a database column, but iTop will not display it automatically. You must adjust the presentation section for the SoftwareLicence class so the fields appear in the details, list and search forms.

1. Locate the definition of SoftwareLicence in iTop’s original data model (datamodel/2.x/itop-config-mgmt/datamodel.itop-config-mgmt.xml). Copy the entire <presentation> block for the class. It contains nested <details>, <list>, <search>, and possibly <dashboard> sections.
2. Paste the copied <presentation> block into your module’s XML file after the <fields> section and wrap it with <presentation> tags. Change the opening <details> (and other sub‑sections) to add _delta="redefine" so iTop knows you are overriding the default presentation. For instance:
```xml
<presentation>
  <details _delta="redefine">
    <items>
      <item id="name"><rank>10</rank></item>
      <item id="software_id"><rank>20</rank></item>
      <item id="org_id"><rank>30</rank></item>
      <item id="usage_limit"><rank>40</rank></item>
      <item id="description"><rank>50</rank></item>
      <!-- insert new fields at appropriate positions -->
      <item id="renewal_date"><rank>60</rank></item>
      <item id="vendor"><rank>70</rank></item>
      <item id="licence_cost"><rank>80</rank></item>
      <item id="support_end"><rank>90</rank></item>
      <item id="po_number"><rank>100</rank></item>
      <item id="perpetual"><rank>110</rank></item>
      <item id="start_date"><rank>120</rank></item>
      <item id="end_date"><rank>130</rank></item>
      <item id="key"><rank>140</rank></item>
    </items>
  </details>
  <!-- Optionally redefine list and search views to include the fields -->
  <list _delta="redefine">
    <items>
      <item id="name"><rank>10</rank></item>
      <item id="software_id"><rank>20</rank></item>
      <item id="renewal_date"><rank>30</rank></item>
      <item id="vendor"><rank>40</rank></item>
      <item id="licence_cost"><rank>50</rank></item>
      <item id="support_end"><rank>60</rank></item>
      <item id="po_number"><rank>70</rank></item>
      <item id="start_date"><rank>80</rank></item>
      <itepm id="end_date"><rank>90</rank></item>
    </items>
  </list>
  <search _delta="redefine">
    <items>
      <item id="name"><rank>10</rank></item>
      <item id="software_id"><rank>20</rank></item>
      <item id="vendor"><rank>30</rank></item>
      <item id="po_number"><rank>40</rank></item>
      <!-- etc. -->
    </items>
  </search>
</presentation>
```

Each <item> entry declares the attribute to display and its relative position (rank). Use ranks spaced by 10 to allow future insertions.

## Add labels to the dictionary ##
Without dictionary entries, new attributes will be displayed using their internal codes. To define user‑friendly labels:
1. Open en.dict.<module>.php and locate the comment “// Dictionary entries go here”.
2. Add translations for each attribute using the pattern Class:<ClassName>/Attribute:<attribute_code>, for example:
```php
<?php
Dict::Add('EN US', 'English', 'English', array(
  // Dictionary entries go here
  'Class:SoftwareLicence/Attribute:renewal_date' => 'Renewal Date',
  'Class:SoftwareLicence/Attribute:vendor'       => 'Vendor',
  'Class:SoftwareLicence/Attribute:licence_cost' => 'Licence Cost',
  'Class:SoftwareLicence/Attribute:support_end'  => 'Support End',
  'Class:SoftwareLicence/Attribute:po_number'    => 'PO Number',
  'Class:SoftwareLicence/Attribute:maintenance_type' => 'Maintenance Type',
  // dictionary values for enum options
  'Class:SoftwareLicence/Attribute:maintenance_type/Value:renewal'     => 'Renewal',
  'Class:SoftwareLicence/Attribute:maintenance_type/Value:subscription' => 'Subscription',
  'Class:SoftwareLicence/Attribute:maintenance_type/Value:perpetual'   => 'Perpetual',
));
?>
```

The tutorial on adding a new field shows that adding an entry like 'Class:Server/Attribute:notes' => 'Additional Notes' changes the label displayed in the UI.

## Compile the changes and update the database ##
1. **Check your modifications** using the iTop toolkit. Browse to /toolkit/ on your development instance. The toolkit can parse your XML and dictionary files; click Refresh to validate the module. Resolve any errors until the toolkit reports a clean state.
2. **Apply the modifications** via the toolkit. On the Write tab, choose Update iTop code and database. This compiles the XML into PHP classes and updates the MySQL schema by creating new columns (e.g., renewal_date, vendor, etc.).
3. **Re‑run the setup** (/setup/) and select Update an existing instance; ensure your custom module is checked before proceeding. This step updates the configuration file so your module is loaded and clears caches. After completion, navigate to an existing Software Licence record—your new fields should appear in the details, list and search views.
4. **Test** creating and editing a licence, verifying that:
Data entry works for the new fields.
Values are saved and displayed correctly.
List and search screens include the new columns when you configure the list.
5. **Deploy to production.** Copy the entire module directory into the extensions/ folder of the production instance and run the setup there. Keep backups of the database and configuration file before the upgrade.

## Best practices and alternatives ##
* **Plan your data model carefully.** Decide which data type suits each attribute (string, text, date, decimal, enum or external key) using the XML data model reference. Changing the type later requires database migrations.
* **Use enumeration fields for controlled vocabularies** (e.g., license types or vendors). Define allowed values inside <values> and add dictionary entries for each value. If values may change at run‑time, consider using AttributeExternalKey linked to a typology class so administrators can manage the list from the UI.
* **Run on a test system first.** Upgrading your data model touches the database and may create downtime. Always test the upgrade and verify backups before running it on production.
* **ITSM Designer (Professional/Essential editions only).** Combodo’s paid editions include an ITSM Designer tool that allows adding fields through the web UI. If you have access to it, you can create new attributes from the “Classes” tab and drag them into the presentation; the changes are then deployed automatically. The community edition requires the manual XML approach described above.
* **Update lists and search screens.** If a new field does not appear in the “Configure This List” dialog, verify that you redefined the <list> and <search> sections in your module’s <presentation> block and that the module was applied. You may also need to clear caches by reloading the setup.

## Summary ##
To track additional licence information in iTop’s CMDB, you must extend the SoftwareLicence class. Use the module creation wizard to generate a skeleton extension, then define new fields in the module’s XML (_delta="define"), adjust the presentation to display the fields, add labels to the dictionary, compile the changes with the toolkit and run the setup to update your instance. This approach is fully supported by iTop and ensures your changes are upgrade‑safe and easy to deploy across environments.
