---
title: Active and inactive items
author: jhutchings1
ms.author: justhu
ms.date: 04/12/2017
ms.topic: article
ms.prod: healthvault
description: Learn about the best way to determine if a HealthVault Thing is current or historical. 
---

# Determining which HealthVault items are active

Retrieving and displaying a list of active items is a common application scenario, such as showing a list of a user's active Conditions or current Medications. 

In the case of a data type such as Condition, there are two fields that one might reasonably expect to reflect whether it is active or not: **status**, which may contain values such as **Current**, **Intermittent**, or **Past**, and **stop-date**, which is an approximate date-time field. 

This presents a challenge. If the fields have conflicting values, such as a status of **Current** and **stop-date** in the past, should the application consider the item active? What if the **stop-date** is a descriptive value, such as "when I was a child" instead of a date value? These issues make determining which items are active a challenge to application developers, and applications may use inconsistent logic to determine this, resulting in confusion for users that use multiple HealthVault-connected applications.

Instead of using these properties, however, we recommend that you read the **updated-end-date** property. This property represents the primary mechanism for indicating whether an item is active. Applications can now store a date value representing the last date at which an item is considered active. This field is a xsd:datetime type to allow easy date-based comparisons, as opposed to the approximate date-time type. Combined with the existing **effective-date** field, applications can now consistently determine the date range during which an item was considered active.

## Updated-end-date vs. existing end date fields

A few HealthVault thing types define an end date as part of their definition, such as Condition, with the **stop-date** field, and Medication, with the **date-discontinued** field. To ensure consistency across the various HealthVault-connected applications, Microsoft recommends: 
- The **updated-end-date** field should be considered the primary end date field for all types. When you query for active items or edit data to indicate the active state of an item, use the **updated-end-date** field.
- Existing end date fields on datatypes that define them should be considered original or source data, such as data from clinical systems. Clinical systems can set this field to keep a record of the original data. User edits should occur on the **updated-end-date** field and not update the existing end date field, which may not always be kept up to date, so it should not be used in determining whether an item is active or not.

## Setting and Clearing updated-end-date
There are some special considerations for setting this field that developers should be aware of.
- For types that have an end date field, such as Asthma Inhaler, Condition, Medication, and so on:
    - When creating an item, if you specify an end date but do not specify an updated end date, the **updated-end-date** element will be set to the same value as the end date. On update, the same logic applies, except that if the **updated-end-date** element already has a value it will not be overwritten by specifying an end date.
    - If you specify an end date and also specify an updated end date, each field will retain its specified value.
- For all types, to clear the **updated-end-date** field, specify any date greater than <span class="literalValue">9999-12-31T00:00:00Z</span>.

## Read-Only data and updated-end-date

When an item in HealthVault is [read-only](/healthvault/concepts/data/read-only-data), it cannot be modified. 

The **updated-end-date** field is defined in a section of the thing schema where the read-only restriction doesn't apply, so it can be modified even if an item is read-only. This allows applications or users to specify an updated end date to indicate whether a read-only item is active or not.

## Querying for items by active status

To query for active or inactive items you can use a query filter.

Active items have an updated end date in the future. To query for these, set the **updated end date minimum** filter to the current date.

Inactive items have an updated end date in the past. To query for these, set the **updated end date maximum** filter to the current date.

## .NET SDK code samples
The following example creates an inactive item.

```c#
// This example shows how to mark an item as inactive 
Medication med = new Medication(new CodableValue("My medication"));
med.UpdatedEndDate = new DateTime(2013, 6, 8); 
PersonInfo.SelectedRecord.NewItem(med);
```

The following example clears the updated end date of an item.

```c#
// In this example, "9b3ef8ad-dd4a-484c-aa05-66f967470e0f" is the thing id 
// for an existing Medication instance. 
HealthRecordItem med = PersonInfo.SelectedRecord.GetItem(new Guid("9b3ef8ad-dd4a-484c-aa05-66f967470e0f"));
med.UpdatedEndDate = DateTime.MaxValue;
PersonInfo.SelectedRecord.UpdateItem(med);
```

The following example queries for active items.

```c#
// This example shows how to query for active items, which have 
// an updated end date in the future 
HealthRecordItemCollection GetActiveItems(Guid typeId) 
{    
    HealthRecordSearcher searcher = PersonInfo.SelectedRecord.CreateSearcher();    
    HealthRecordFilter filter = new HealthRecordFilter(typeId);    
    filter.UpdatedEndDateMin = DateTime.Today;     
    searcher.Filters.Add(filter);    
    HealthRecordItemCollection items = searcher.GetMatchingItems()[0];
}
```
