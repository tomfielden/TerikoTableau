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

## Transformation Details

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


### Detail+NVD_Step-1.tfl, "Product Detail" Portion

Renaming:

    * COMPONENT                 -> Member Name
    * MFGR                      -> Manufacturer
    * PC_STATE_CD               -> State
    * PRODUCT_MASTER_ID         -> Product Master ID (string from int)
    * PURCHASES                 -> Purchase $'s
    * VOLUME                    -> Cases
    * GPO_MEMBER_ID             -> IPS Member ID
    * PARENT_MANUFACTURER_NAME  -> Parent Manufacturer
    * MFGR#                     -> #SKU (trimmed to remove extra spaces)
    * PERIOD_YEAR               -> Year
    * DISTRIBUTOR               -> Distributor House
    * Capacity (Salesforce)     -> Student Count (int to string)

Calculated Fields:

    * Weight (LBS)

```
    If     [ITEM_UOM] = "LBS" THEN [PACK] * [ITEM]
    ELSEIF [ITEM_UOM] = "LB"  THEN [PACK] * [ITEM]
    ELSEIF [ITEM_UOM] = "OZ"  THEN [PACK] * [ITEM] / 16
    ELSEIF [ITEM_UOM] = "OZS" THEN [PACK] * [ITEM] / 16
    END
```

    * Total Weight (LBS)

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

    * FiscalDate

```
    IF INT([PERIOD_MONTH]) < 10 THEN
        DATE(DATEPARSE("MM/yyyy", STR(INT([PERIOD_MONTH])+3) + "/" + [PERIOD_YR]))
    ELSE
        DATE(DATEPARSE("MM/yyyy", STR(INT([PERIOD_MONTH])-9) + "/" + STR(INT([PERIOD_YR])+1)))
    END
```

    * Qtr

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
        * Group by "IPS Member ID"
        * MAX of Student Count
        * MAX of Name of Co-Op


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


### Detail+NVD_Step-1.tfl, "Billed NVD" Portion

Renaming:

    * IPS_MEMBER_ID                 -> IPS Member ID
    * ITEM_PACK_SIZE                -> Pack
    * ITEM_SIZE                     -> ITEM
    * TOTAL_PKG_QUANTITY            -> Cases
    * TOTAL_WT_QUANTITY             -> Total Weight (LBS)
    * TOTAL_PURCH_AMT               -> Purchase $'s
    * MANUFACTURER_ID               -> MFGR_ID
    * ARA_PRODUCT_ID                -> ARA_PRODUCT_ID (int to string)
    * BRAND_ID                      -> BRAND_ID (int to string)
    * REBATE_INVOICE_NUMBER         -> REBASE_INVOICE_NUMBER (int to string)
    * MANUFACTURER_PRODUCT_CD       -> #SKU (timmed to remove extra spaces)
    * PROFIT_CENTER_CD              -> PC#

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

    * FiscalDate

```
    DATE(DATEPARSE ("MM/yyyy", STR([FISCAL_MONTH]) + "/" + STR([FISCAL_YEAR])))
```

*NOTE 1:*

We will join Billed NVD with Product Details using the following keys,

* MFGR_ID                           (MFGR_ID)
* MFGR#                             (#SKU)
* GPO_MEMBER_ID                     (IPS Member ID)
* CALENDAR_MONTH + CALENDAR_YEAR    (date)

We need to be aware that the actual records are indexed by yet another field,

* CLIENT_ID

*NOTE 2:*

We will join Billed NVD with Product Details using the following keys,

* MANUFACTURER_ID                   (MFGR_ID)
* MANUFACTURER_PRODUCT_CD           (#SKU)
* IPS_MEMBER_ID                     (IPS Member ID)
* CALENDAR_MONTH + CALENDAR_YEAR    (date)

We need to be aware that records in this table are keyed more finely.
In particular,

REBATEABLE_PURCH_AMT and TOTAL_PKG_QUANTITY are indexed by the above and also,

* ITEM_PACK_SIZE + ITEM_UOM
* FISCAL_MONTH + FISCAL_YEAR

BILLED_REBATE_AMT is indexed all the above plus,

* NVD_RATE, NVD_RATE_BASIS_TYPE, INCOME_PROVISION, REBATE_ID

```
If NVD_RATE_BASIS_TYPE = SPD Then
    BILLED_REBATE_AMT = NVD_RATE * REBATEABLE_PURCH_AMT

If NVD_RATE_BASIS_TYPE = PKG Then
    BILLED_REBATE_AMT = NVD_RATE * TOTAL_PKG_QUANTITY
```

### Detail+NVD_Step-2.tfl

The purpose of this workflow is to merge the "Product Billed (Clean)" and "Billed NVD (clean)". 

Specifically we use the following source tables,

    * server: Teriko's Projects/Product Detail (Clean)
    * server: Teriko's Projects/Billed NVD (Clean)


Renaming of "Billed NVD (Clean)" fields,

    * PRODUCT_DESCRIPTION   -> Product Description (NVD)
    * Purchase $'s          -> Purchase $'s (NVD)
    * Total Weight (LBS)    -> Total Weight (LBS)(NVD)
    * Manufacturer          -> Manufacturer (NVD)
    * PC#                   -> PC# (NVD)

The join fields are,

    * MFGR_ID
    * IPS Member ID
    * #SKU
    * Date

Noticing that the above fields do not uniquely identify records in the Billed NVD table, the table needs to first be aggregated before joining on the above keys with the Product Detail table,

The aggregations for Product Detail are,

    * MAX of BRAND
    * MAX of Distributor
    * MAX of Distributor House
    * MAX of MAJOR_CAT
    * MAX of MINOR_CAT
    * MAX of Manufacturer
    * MAX of Member Name
    * MAX of Name of Co-Op
    * MAX of Pack Size
    * MAX of Pack Size Orig
    * MAX of Parent Manufacturer
    * MAX of PC#
    * MAX of PCZIP
    * MAX of Product Description
    * MAX of Product Master ID
    * MAX of Qtr
    * MAX of SECTOR
    * MAX of State
    * MAX of Student Count
    * MAX of SY
    * MAX of SY-Half
    * MAX of Year
    * SUM of Cases (Product Detail)
    * SUM of Purchase $'s
    * SUM of Weight (LBS)

The Billed NVS requires two levels of aggregation

The first level aggregation is grouped by the above join fields as well as,

    * Item UOM (NVD)
    * FiscalDate

The first level aggregations are,

    * MAX of ARA_PRODUCT_ID
    * MAX of Manufacturer (NVD)
    * MAX of Product Description (NVD)
    * MAX of PRODUCT_CENTER_CD
    * MAX of REBATE_ID
    * MAX of REBATE_INVOICE_DATE
    * MAX of Cases (NVD)
    * MAX of Purchase $'s (NVD)
    * MAX of REBATEABLE_PURCH_AMT
    * MAX of Total Weight (LBS)(NVD)
    * SUM of BILLED_REBATE_AMT

The second level aggregation is grouped by the same fields used by the join and the following aggregrations,

    * MAX of ARA_PRODUCT_ID
    * MAX of Manufacturer (NVD)
    * MAX of Product Description (NVD)
    * MAX of PRODUCT_CENTER_CD
    * MAX of REBATE_ID
    * MAX of REBATE_INVOICE_DATE
    * SUM of Cases (NVD)
    * SUM of Purchase $'s (NVD)
    * SUM of REBATEABLE_PURCH_AMT
    * SUM of Total Weight (LBS)(NVD)
    * SUM of BILLED_REBATE_AMT

## Fields Explained

| Product Detail + NVD (Clean) | Product Detail (Clean)<BR>Billed NVD (Clean) | IPS Product Detail<BR>IPS_BILLED_NVD_DATA | Notes |
| :------------------------ | :-------------------- | :---------------- | :------------ |
| #SKU                      | PD : #SKU | PD : MFGR# | trimmed to remove extra spaces  |
| ARA_PRODUCT_ID            | NVD : ARA_PRODUCT_ID | NVD : ARA_PRODUCT_ID  | convert integer to string |
| BRAND                     | PD : BRAND  | PD : BRAND  |   |
| Date                      | PD : Date  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| Distributor               | PD : Distributor  | PD : DISTRIBUTOR  | (formula)  |
| Distributor House         | PD : Distributor House  | PD : DISTRIBUTOR |   |
| FiscalDate                | PD : FiscalDate  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| IPS Member ID             | PD : IPS Member ID  | PD : GPO_MEMBER_ID  |   |
| MAJOR_CAT                 | PD : MAJOR_CAT  | PD : MAJOR_CAT  |   |
| Manufacturer              | PD : Manufacturer  | PD : MFGR  | (formula)  |
| Manufacturer (NVD)        | NVD : Manufacturer  | NVD : MANUFACTURER_NAME  | (formula)  |
| Member Name               | PD : Member Name  | PD : COMPONENT  |   |
| MFGR_ID                   | PD : MFGR_ID  | PD : MFGR_ID  |   |
| MINOR_CAT                 | PD : MINOR_CAT  | PD : MINOR_CAT  |   |
| Name of Co-Op             | PD : Name of Co-Op  | Salesforce : Name of Co-Op  |   |
| Pack Size                 | PD : Pack Size  | PD : Pack Size Orig  | (formula)  |
| Pack Size Orig            | PD : Pack Size Orig  | PD : ITEM<BR>PD : ITEM_UOM<BR>PD : PACK  | (formula)  |
| Parent Manufacturer       | PD : Parent Manufacturer  | PD : PARENT_MANUFACTURER_NAME  |   |
| PC#                       | PD : PC#  | PD : PC#  |   |
| PC# (NVD)                 | NVD : PC#  | NVD : PROFIT_CENTER_CD  |   |
| PCZIP                     | PD : PCZIP  | PD : PCZIP  |   |
| Product Description       | PD : Product Description  | PD : PRODUCT_DESCRIPTION  |   |
| Product Description (NVD) | NVD : Product Description  | NVD : PRODUCT_DESCRIPTION  |   |
| Product Master ID         | PD : Product Master ID  | PD : PRODUCT_MASTER_ID  |   |
| Qtr                       | PD : Qtr  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| REBATE_ID                 | NVD : REBATE_ID  | NVD : REBATE_ID  |   |
| REBATE_INVOICE_DATE       | NVD : REBATE_INVOICE_DATE  | NVD : REBATE_INVOICE_DATE  |   |
| SECTOR                    | PD : SECTOR  | PD : SECTOR  |   |
| State                     | PD : State  | PD : PC_STATE_CD  |   |
| Student Count             | PD : Student Count  | Salesforce : Capacity  | convert integer to string  |
| SY                        | PD : SY  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| SY-Half                   | PD : SY-Half  | PD : PERIOD_MONTH<BR>PD : PERIOD_YEAR  | (formula)  |
| Year                      | PD : Year  | PD : PERIOD_YEAR  | (formula)  |
| BILLED_REBATE_AMT         | NVD : BILLED_REBATE_AMT  | NVD : BILLED_REBATE_AMT  |   |
| Cases                     | PD : Cases  | PD : VOLUME  |   |
| Cases (NVD)               | NVD : Cases  | NVD : TOTAL_PKG_QUANTITY  |   |
| Purchase $'s              | PD : Purchase $'s  | PD : PURCHASES  |   |
| Purchase $'s (NVD)        | NVD : Purchase $'s  | NVD : TOTAL_PURCH_AMT  |   |
| REBATEABLE_PURCH_AMT      | NVD : REBATEABLE_PURCH_AMT  | NVD : REBATEABLE_PURCH_AMT   |   |
| Total Weight (LBS)        | PD : Total Weight (LBS)  | PD : Cases<BR>PD : Weight (LBS)  | Cases * Weight (LBS) |
| Total Weight (LBS)(NVD)   | NVD : Total Weight (LBS)  | NVD : TOTAL_WT_QUANTITY   |   |
| Weight (LBS)              | PD : Total Weight (LBS)  | PD : ITEM<BR>PD : ITEM_UOM<BR>PD : PACK<BR>  | (formula)  |
