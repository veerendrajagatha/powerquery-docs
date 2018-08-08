---
title: Data source functions
description: How to use data source functions to wrap connector behavior
author: cpopell
manager: kfile
ms.reviewer: ''

ms.service: powerquery
ms.component: power-query
ms.topic: overview
ms.date: 08/10/2018
ms.author: gepopell

LocalizationGroup: reference
---

### Data Source Functions

A Data Connector wraps and customizes the behavior of a [data source function in the M Library](https://msdn.microsoft.com/library/mt253322.aspx#Anchor_15).
For example, an extension for a REST API would make use of the [Web.Contents](https://msdn.microsoft.com/library/mt260892.aspx) function to make HTTP requests.
Currently, a limited set of data source functions have been enabled to support extensibility.

- [Web.Contents](https://msdn.microsoft.com/library/mt260892.aspx)
- [OData.Feed](https://msdn.microsoft.com/library/mt260868.aspx)
- [Odbc.DataSource](https://msdn.microsoft.com/library/mt708843.aspx)
- [AdoDotNet.DataSource](https://msdn.microsoft.com/library/mt736964)
- [OleDb.DataSource](https://msdn.microsoft.com/library/mt790573.aspx)

**Example:**

```
[DataSource.Kind="HelloWorld", Publish="HelloWorld.Publish"]
shared HelloWorld.Contents = (optional message as text) =>
    let
        message = if (message <> null) then message else "Hello world"
    in
        message;
```

### Data Source Kind

Functions marked as `shared` in your extension can be associated with a specific data source by including a `DataSource.Kind` metadata record on the function with the name of a Data Source definition record. 
The Data Source record defines the authentication types supported by your data source, and basic branding information (like the display name / label).
The name of the record becomes is unique identifier. 

Functions associated with a data source must have the same required function parameters (including name, type, and order). Functions for a specific Data Source Kind can only use credentials associated with that Kind.
Credentials are identified at runtime by performing a lookup based on the combination of the function's required parameters.
For more information about how credentials are identified, please see [Data Source Paths] below.

**Example:**

```
HelloWorld = [
    Authentication = [
        Implicit = []
    ],
    Label = Extension.LoadString("DataSourceLabel")
];
```

#### Properties

The following table lists the fields for your Data Source definition record.

| Field              | Type     | Details                                                                                                                                                                                                                                                                   |
|:-------------------|:---------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Authentication     | record   | Specifies one or more types of authentication supported by your data source. At least one kind is required. Each kind will be displayed as an option in the Power Query credential prompt. For more information, see [Authentication Kinds](#authentication-kinds) below. |
| Label              | text     | **(optional)** Friendly display name for this extension in credential dialogs.                                                                                                                                                                                            |
| SupportsEncryption | logical  | **(optional)** When true, the UI will present the option to connect to the data source using an encrypted connection. This is typically used for data sources with a non-encrypted fallback mechanism (generally ODBC or ADO.NET based sources).                          |


### Publish to UI

Similar to the (Data Source)[#data-source-kind] definition record, the Publish record provides the Power Query UI the information it needs to expose this extension in the Get Data dialog.

**Example:**

```
HelloWorld.Publish = [
    Beta = true,
    ButtonText = { Extension.LoadString("FormulaTitle"), Extension.LoadString("FormulaHelp") },
    SourceImage = HelloWorld.Icons,
    SourceTypeImage = HelloWorld.Icons
];

HelloWorld.Icons = [
    Icon16 = { Extension.Contents("HelloWorld16.png"), Extension.Contents("HelloWorld20.png"), Extension.Contents("HelloWorld24.png"), Extension.Contents("HelloWorld32.png") },
    Icon32 = { Extension.Contents("HelloWorld32.png"), Extension.Contents("HelloWorld40.png"), Extension.Contents("HelloWorld48.png"), Extension.Contents("HelloWorld64.png") }
];
```

#### Properties

The following table lists the fields for your Publish record.

| Field               | Type    | Details                                                                                                                                                                                                                                                                                                                    |
|:--------------------|:--------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ButtonText          | list    | List of text items that will be displayed next to the data source's icon in the Power BI Get Data dialog.                                                                                                                                                                                                                  |
| Category            | text    | Where the extension should be displayed in the Get Data dialog. Currently the only category values with special handing are `Azure` and `Database`. All other values will end up under the Other category.                                                                                                               |
| Beta                | logical | **(optional)** When set to true, the UI will display a Preview/Beta identifier next to your connector name and a warning dialog that the implementation of the connector is subject to breaking changes.                                                                                                                   |
| LearnMoreUrl        | text    | **(optional)** Url to website containing more information about this data source or connector.                                                                                                                                                                                                                             |
| SupportsDirectQuery | logical | **(optional)** Enables Direct Query for your extension.<br>**This is currently only supported for ODBC extensions.**                                                                                                                                                                                                       |
| SourceImage         | record  | **(optional)** A record containing a list of binary images (sourced from the extension file using the **Extension.Contents** method). The record contains two fields (Icon16, Icon32), each with its own list. Each icon should be a different size.                                                                       |                                                                                                                                                                                                                              |
| SourceTypeImage     | record  | **(optional)** Similar to SourceImage, except the convention for many out of the box connectors is to display a sheet icon with the source specific icon in the bottom right corner. Having a different set of icons for SourceTypeImage is optional - many extensions simply reuse the same set of icons for both fields. |