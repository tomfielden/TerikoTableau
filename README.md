# TerikoTableau
Tableau Prep and Workbooks -- For IPS

## Overview

### The "TerikoTableau" Directory

This README file is located in the "TerikoTableau" directory. This directory is controlled by the *git* version control system and is linked to the "tomfielden/TerikoTableau" repository on *www.GitHub.com*. Version control is discussed below.

### The "Prep Flows" Subdirectory

There are two Tableau Prep Builder workflows in the "Prep Flows" subdirectory. They must be run in sequence

* Detail+NVD_Step-1.tfl
* Detail+NVD_Step-2.tfl

The "Detail+NVD_Step-1.tfl" should not be run directly. Instead, open it and export it to "Detail+NVD_Step-1.tflx" and run that instead. What happening is that the ".tfl" file needs to pick up and "package" some local data found in the Excel file/workbook,

* IPS_Product_Details.xls

The Excel workbook contains two sheets/tables,

* Manufacturers
* Distributors

These sheets/tables are used by the Prep Builder workflows. In general local data (such as Excel) is read dynamically by the ".tfl" workflows and then packaged into the ".tflx" workflows. The significance of these files will be explained in detail below.

For historical and technical reference there is a "Prep Flows/Backup" subdirectory containing early versions of the main Prep Builder workflows.

### Prep Builder Workflow Inputs + Outputs

The fundamental concept is that Tableau Prep Builder is an ETL (Extract Transform Load) system. It's purpose is to take existing data tables, primarily located on the Tableau Server, transform them into a more useful form and load them back onto the Tableau server.

There source tables for Prep Builder workflows are,

* IPS Product Detail
* IPS_BILLED_NVD_DATA
* Salesforce Account Extract

As mentioned elsewhere, local Excel data is also used to construct the output tables. The output tables are all located in "Teriko's Projects" on the Tableau Server,

* Product Detail (Clean)
* Billed NVD (Clean)
* Product Detail + NVD (Clean)

### Tableau Application(s)

Tableau has several applications of relevance,

* Tableau Server
* Tableau Online
* Tableau Desktop
* Tableau Prep Builder

The Tableau Server contains all the dynamic source data and output data.
The Tableau Online tool is used to interrogate and visualize data, mostly located on the Tableau Server. It is reasonably fast and convenient.
The Tableau Desktop is a more powerful alternative to the Tableau Online tool and allows more complex table joins and it useful for downloaded large datasets.
The Tableau Prep Builder is largly the focus of this directory/repository. The Prep Builder is used to transform upstream/source data into a more useful form for investigations and reporting.

## The Prep Builder Workflows Data

Each of the Prep Builder workflows consumes several tables and produces a single table which is then stored on the Tableau Server. The server address is,

* https://10az.online.tableau.com

Here are the inputs and output tables for each workflow,

### Detail+NVD_Step-1.tfl

*Purpose:*

To preprocess the "IPS Product Detail" and "IPS_BILLED_NVD_DATA" data correcting several issues including manufacturer and distributor name substitutions.

Inputs,

* server: default/IPS Product Detail
* server: default/IPS_BILLED_NVD_DATA
* server: Salesforce/Salesforce Accounts Extract
* IPS_Product_Details.xls: Manufacturer
* IPS_Product_Details.xls: Distributor

Outputs,

* server: Teriko's Projects/Product Detail (Clean)
* server: Teriko's Projects/Billed NVD (Clean)

### Detail+NVD_Step-2.tfl

Purpose: To perform the join of Product Detail and Billed NVD and pick up only the useful fields for speed and convenience.

Inputs,
* server: Teriko's Projects/Product Detail (Clean)
* server: Teriko's Projects/Billed NVD (Clean)

Output,
* server: Teriko's Projects/Product Detail + NVD (Clean)


## Running Prep Builder Workflows

### Conditional Steps
If you alter anything in the IPS_Product_Details.xls file (Manufacturer or Distributor mapping tables)
Then you will need to first,

1. Open "Detail+NVD_Step-1.tfl"
    * File -> Export to "Detail+NVD_Step-1.tflx" (default, replace existing)

Else you can skip these steps. Their purpose is to pull in the local Excel mapping data into the Prep Builder package (.tflx) file(s).

### Monthly Steps
Note: Be sure the Tableau Prep Builder application is closed and not running. Eases a server login issue.

1. Open "Detail+NVD_Step-1.tflx"
    * Flow -> Run All (~ 1.5 hours)
2. Open "Detail+NVD_Step-2.tfl"
    * Flow -> Run All (~ 45 minutes)

*Note:* There is a way to publish flows to the server and have them automatically scheduled. Publication to the server requires another licence for Tableau Conductor

## Fields Explained

| Product Detail + NVD (Clean) | Product Detail (Clean)<BR>Billed NVD (Clean) | IPS Product Detail<BR>IPS_BILLED_NVD_DATA | Notes |
| :------------------------ | :-------------------- | :---------------- | :------------ |
| Aramark Product ID        | NVD : Aramark Product ID | NVD : ARA_PRODUCT_ID  | convert integer to string |
| Brand                     | PD : Brand  | PD : BRAND  |   |
| Category Major            | PD : Category Major  | PD : MAJOR_CAT  |   |
| Category Minor            | PD : Category Minor  | PD : MINOR_CAT  |   |
| Co-Op Name                | PD : Co-Op Name | Salesforce : Name of Co-Op  |   |
| Date                      | PD : Date  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| Distributor               | PD : Distributor  | PD : DISTRIBUTOR  | (formula)  |
| Distributor House         | PD : Distributor House  | PD : DISTRIBUTOR |   |
| Fiscal Date               | PD : Fiscal Date  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| Manufacturer              | PD : Manufacturer  | PD : MFGR  | (formula)  |
| Manufacturer (NVD)        | NVD : Manufacturer  | NVD : MANUFACTURER_NAME  | (formula)  |
| Manufacturer ID           | PD : Manufacturer ID  | PD : MFGR_ID  |   |
| Manufacturer Parent       | PD : Manufacturer Parent  | PD : PARENT_MANUFACTURER_NAME  |   |
| Member                    | PD : Member  | PD : COMPONENT  |   |
| Member ID                 | PD : Member ID  | PD : GPO_MEMBER_ID  |   |
| Pack Size                 | PD : Pack Size  | PD : Pack Size Orig  | Selected Pack Size for Man+SKU group  |
| Pack Size Orig            | PD : Pack Size Orig  | PD : ITEM<BR>PD : ITEM_UOM<BR>PD : PACK  | (formula)  |
| Product Description       | PD : Product Description  | PD : PRODUCT_DESCRIPTION  | Selected Product Description for Man+SKU group |
| Product Description (NVD) | NVD : Product Description  | NVD : PRODUCT_DESCRIPTION  |   |
| Product Master ID         | PD : Product Master ID  | PD : PRODUCT_MASTER_ID  |   |
| Profit Center ID          | PD : Profit Center ID  | PD : PC#  |   |
| Profit Center ID (NVD)    | NVD : Profit Center ID  | NVD : PROFIT_CENTER_CD  |   |
| Profit Center ZIP         | PD : Profit Center ZIP  | PD : PCZIP  |   |
| QTR                       | PD : QTR  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| Rebate Invoice Date (NVD) | NVD : Rebate Invoice Date  | NVD : REBATE_INVOICE_DATE  |   |
| Sector                    | PD : Sector  | PD : SECTOR  |   |
| SKU                       | PD : #KU | PD : MFGR# | trimmed to remove extra spaces  |
| State                     | PD : State  | PD : PC_STATE_CD  |   |
| Student Count             | PD : Student Count  | Salesforce : Capacity  | convert integer to string  |
| SY                        | PD : SY  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| SY-Half                   | PD : SY-Half  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| Year                      | PD : Year  | PD : PERIOD_YEAR  | (formula)  |
| Billed $'s (NVD)          | NVD : Billed $'s  | NVD : BILLED_REBATE_AMT  |   |
| Cases                     | PD : Cases  | PD : VOLUME  |   |
| Cases (NVD)               | NVD : Cases  | NVD : TOTAL_PKG_QUANTITY  |   |
| LBS / Case                | PD : LBS / Case  | PD : PACK<BR>PD : ITEM<BR>PD : ITEM_UOM<BR>  | PACK * ITEM (in lbs) |
| LBS Total                 | PD : LBS Total  | PD : Cases<BR>PD : LBS / Case  | Cases * LBS / Case |
| LBS Total (NVD)           | NVD : LBS Total  | NVD : TOTAL_WT_QUANTITY   | Stated directly in the NVD table  |
| Purchase $'s              | PD : Purchase $'s  | PD : PURCHASES  |   |
| Purchase $'s (NVD)        | NVD : Purchase $'s  | NVD : TOTAL_PURCH_AMT  |   |
| Rebateable $'s (NVD)      | NVD : Rebateable $'s  | NVD : REBATEABLE_PURCH_AMT   |   |


### Notes:

* In the table above, categorical data is sorted at the top, measure data is sorted at the bottom as it appears in a Tableau Desktop worksheet.
* (IPS Product Detail) ITEM is weight per item in some units stated in UNIT_UOM.
* (IPS Product Detail) PACK is number of items in a package described by SOLD_BY, typically a case.
* (IPS Product Detail) VOLUME is the number of packs sold in each row of the table.
* The "Product Detail (Clean)" table is created by mixing in several other tables. The number of rows are the same as the main source table, "IPS Product Detail"
* The "Billed NVD (clean)" table shares a least one mixin table. The number of rows is the same as the main source table, "IPS_BILLED_NVD_DATA"
* The "Product Detail + NVD (Clean)" table is created by several aggregations and "left" joins the "Product Detail (Clean)" and "Billed NVD (clean)" tables. The direct result of aggregating tables is that the number of resulting rows is significantly less than the main "Product Detail (Clean)" table.
* The "Product Detail (Clean)" table is indexed by the following fields,
    * Manufacturer ID
    * Member ID
    * Date
    * SKU
    * CLIENT_ID
* The "Billed NVD (Clean)" table is indexed by the following fields,
    * Manufacturer ID
    * Member ID
    * DISTIBUTION_CENTER_ID
    * Date
    * Fiscal Date
    * Rebate Invoice ID
    * Aramark Product ID
    * PKG_BASIS
    * WT_BASIS
    * NVD_RATE
        * There are only 10 pairs of records where this matters and not for INCOME_PROVISION
        * Recommendation: Drop it as index field
    * Rebate ID
        * This seems to be a split field
        * There are 2216 pairs of records, mostly KENS and LA BREA, that index by Rebase ID.
    * INCOME_PROVISION
        * This seems to be a split field
        * Fields (Cases, Purchase $'s, LBS Total) each are duplicated.
        * Must do MAX of Rebateable $'s
        * There are 764 pairs of records (Nearly all DART / SOLO CUP CPC) where records are not duplicated.


### Excel IPS_Product_Details.xls

There are two mapping tables in the associated Excel workbook,

* Manufacturers

The Manufacturer sheet/table contains two fields,

    * Old
    * New

The "Old" field is intended to join with the "IPS Product Detail" MFGR/Manufacturer field.
The "New" field either contains a name that is intended to substituted for an existing name (this is an opportunity to group names together into a common name) or left blank. In the left blank case, a formula in the workflow will fill in a computed value in place of the original name, originally this was BRAND or, if null, MFGR_ID.

* Distributors

The Distributor sheet/table contains two fields

    * Old
    * New

The "Old" field is intended to join with the "IPS Product Detail" DISTRIBUTOR/"Distributor House" field
The "New" field contains substitute names for the given distributor. Please see the actual formula for specifics.


### Product Detail (Clean) Formulas

Calculated Fields:

* LBS / Case

```
    If     [ITEM_UOM] = "LBS" THEN [PACK] * [ITEM]
    ELSEIF [ITEM_UOM] = "LB"  THEN [PACK] * [ITEM]
    ELSEIF [ITEM_UOM] = "OZ"  THEN [PACK] * [ITEM] / 16
    ELSEIF [ITEM_UOM] = "OZS" THEN [PACK] * [ITEM] / 16
    END
```

* LBS Total

```
    [Cases] * [Weight (LBS)]
```

* ITEM_STR

```
    IF INT([ITEM]) = [ITEM] THEN
       STR([ITEM])
    ELSE
        IF ([ITEM] - INT(ITEM))*100 < 10 THEN
            STR(INT([ITEM]))+'.0'+STR(ROUND(([ITEM] - INT(ITEM))*100))
        ELSE
            STR(INT([ITEM]))+'.'+ STR(ROUND(([ITEM]-INT([ITEM]))*100))
        END
    END
```

* Pack Size Orig

```

    IF [ITEM_STR] = '0' OR ISNULL([ITEM_STR]) THEN
        IF [PACK] = 0 OR ISNULL([PACK]) THEN
            [ITEM_UOM]
        ELSE
            STR([PACK])+"/"+[ITEM_UOM]
        END
    ELSE
        IF [PACK] = 0 OR ISNULL([PACK]) THEN
            [ITEM_STR]+" "+[ITEM_UOM]
        ELSE
            [ITEM_STR]+"/"+STR([PACK])+" "+[ITEM_UOM]
        END
    END
```

* Manufacturer
    Join "Manufacturer" to Excel.Manufacturer.Old to pick up new names or compute missing names (detailed in Excel)

```
    IF ISNULL([Old]) THEN
        [Manufacturer]
    ELSEIF ISNULL([New]) THEN
        IF ISNULL([BRAND]) THEN
            IF ISNULL([MFGR_ID]) THEN
                [Manufacturer]
            ELSE
                [MFGR_ID]
            END
        ELSE
            [BRAND]
        END
    ELSE
        [New]
    END
```
    Notice that when Excel.Manufacturer.New is left empty, the BRAND or, if null, the MFGR_ID value will be substituted for the Manufacturer.

* Date

```
    DATE(DATEPARSE ( "MM/yyyy", [PERIOD_MONTH] + "/" + [PERIOD_YR] ))
```

* Fiscal Date

```
    IF INT([PERIOD_MONTH]) < 10 THEN
        DATE(DATEPARSE("MM/yyyy", STR(INT([PERIOD_MONTH])+3) + "/" + [PERIOD_YR]))
    ELSE
        DATE(DATEPARSE("MM/yyyy", STR(INT([PERIOD_MONTH])-9) + "/" + STR(INT([PERIOD_YR])+1)))
    END
```

* QTR

```
    "Q"+STR(INT((INT([PERIOD_MONTH])-1)/3)+1)+"-"+[PERIOD_YR]
```

* SY-Half

```
    IF INT([PERIOD_MONTH]) <= 6 THEN
        "2H-"+STR(INT([PERIOD_YR])-1)+"-"+STR(INT([PERIOD_YR])-2000)
    ELSE
        "1H-"+[PERIOD_YR]+"-"+STR(INT([PERIOD_YR])-2000+1)
    END
```

* SY

```
    IF INT([PERIOD_MONTH]) <= 6 THEN
        STR(INT([PERIOD_YR])-1)+"-"+STR(INT([PERIOD_YR])-2000)
    ELSE
        [PERIOD_YR]+"-"+STR(INT([PERIOD_YR])-2000+1)
    END
```

* Distributor
    Join "DISTRIBUTOR" to Excel.Distributor.Old to pick up new names (detailed in Excel)

```
    IF ISNULL([Old]) THEN
        "* "+[DISTRIBUTOR]
    ELSE
        [New]
    END
```

 Notice: When a name substitution (Old to New) is not found, the original DISTRIBUTOR name is used, but is prefixed with an asterix so that missing values can be spotted quickly. The DISTRIBUTOR field is renamed to "Distributor House".


* Join "Salesforce/Salesforce Accounts Extract" on "IPS Member ID"
    * First aggregate table.
    * Group by "Member ID"
    * MAX of Student Count
    * MAX of Co-Op Name


* Inner Join on Manufacturer, #SKU
    * MAX of Product Description    -> Unique PD
    * MAX of Pack Size              -> Unique PS


* Pack Size
This formula makes more sense in context. It says; If the #SKU is empty, use the current/original Pack Size else use the one chose by the aggregation above (i.e. the unique one in the Manufacturer, #SKU group)

```
    IF ISNULL([#SKU]) or [#SKU] = "" THEN
        [Pack Size Orig]
    ELSE
        [Unique PS]
    END
```

    * Product Description
    Same effect as for "Pack Size", we choose the aggregated value unless #SKU is null/empty in which case we stick with the original value,

```
    IF ISNULL([#SKU]) or [#SKU] = "" THEN
        [PRODUCT_DESCRIPTION]
    ELSE
        [Unique PD]
    END
```

### Billed NVD (Clean) Forumlas

Calculated Fields:

* Manufacturer
    Join "Manufacturer" to Excel.Manufacturer.Old to pick up new names or compute missing names (detailed in Excel)

```
    IF ISNULL([Old]) THEN
        [MANUFACTURER_NAME]
    ELSEIF ISNULL([New]) THEN
        IF ISNULL([BRAND_NAME]) THEN
            IF ISNULL([MFGR_ID]) THEN
                [MANUFACTURER_NAME]
            ELSE
                [MFGR_ID]
            END
        ELSE
            [BRAND_NAME]
        END
    ELSE
        [New]
    END
```

    Note: This is a similar process above calculation for Manufacturer.

* Date

```
    DATE(DATEPARSE ("MM/yyyy", [CALENDAR_MONTH] + "/" + [CALENDAR_YEAR]))
```

* Fiscal Date

```
    DATE(DATEPARSE ("MM/yyyy", STR([FISCAL_MONTH]) + "/" + STR([FISCAL_YEAR])))
```

### Product Detail + NVD (Clean) Formulas

The "Billed NVD (Clean)" table contains "split" records where some field values are duplicated in order to allow other fields to be more detailed. The records are split by the following two key fields,

* Rebate ID
* INCOME_PROVISION

To aggregate correctly, first aggregrate using fields,

* Manufacturer ID
* Member ID
* DISTIBUTION_CENTER_ID
* Date
* Fiscal Date
* Rebate Invoice ID
* Aramark Product ID
* PKG_BASIS
* WT_BASIS

Notice, these are the index fields of the table excluding "Rebate ID" and "INCOME_PROVISION".

Then, aggregate remaining fields by "MAX" for categorical (string, date) fields and,

* MAX of Rebateable $'s
* SUM of Billed $'s
* MAX of Purchase $'s
* MAX of LBS Total

The "Rebateable $'s" values are not duplicated in the split records, but the maximum value seems to be the correct one.

Note:

```
If NVD_RATE_BASIS_TYPE = "SPD" Then
    Billed $'s = NVD_RATE * Rebateable $'s

If NVD_RATE_BASIS_TYPE = PKG Then
    Billed $'s = NVD_RATE * TOTAL_PKG_QUANTITY
```

The "Product Detail (Clean)" and the partially aggregated (to resolve split records) "Billed NVD (Clean)" tables are then each aggregated by

* Manufacturer ID
* Member ID
* Date
* SKU

All categorical fields are aggregated as "MAX" and all measure fields are aggregated as "SUM"
