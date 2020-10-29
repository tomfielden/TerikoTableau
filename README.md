# TerikoTableau
Tableau Prep and Workbooks 

## Overview

### The "TerikoTableau" Directory

This README file is located in the "TerikoTableau" directory. This directory is controlled by the *git* version control system and is linked to the "tomfielden/TerikoTableau" repository on *www.GitHub.com*. Version control is discussed below.

### The "Prep Flows" Subdirectory

There are several Tableau Prep Builder workflows in the "Prep Flows" subdirectory,

* Detail_Teriko-1.0.tfl
* NVD_Teriko-1.0.tfl
* Details+NVD.tfl

The "Detail_Teriko-1.0" amd "NVD_Teriko-1.0" workflows each have two files; ".tfl" and ".tflx" explained below.

There is also an Excel file,

* IPS_Product_Details.xls

This Excel workbook contains two sheets/tables,

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

Each of the three Prep Builder workflows consumes several tables and produces a single table which is then stored on the Tableau Server. The server address is,

* https://10az.online.tableau.com

Here are the inputs and output tables for each workflow,

### Detail_Teriko-1.0

Purpose: To preprocess the Product Detail data correcting several issues including manufacturer and distributor name substitutions.

Inputs,
* server: default/IPS Product Detail
* server: Salesforce/Salesforce Accounts Extract
* IPS_Product_Details.xls: Manufacturer
* IPS_Product_Details.xls: Distributor

Output,
* server: Teriko's Projects/Product Detail (Clean)

### NVD_Teriko-1.0

Purpose: To preprocess the Billed NVD data similarly to the Product Detail data so they match more accurately.

Inputs,
* server: default/IPS_BILLED_NVD_DATA
* IPS_Product_Details.xls: Manufacturer

Output,
* server: Teriko's Projects/Billed NVD (Clean)

### Details+NVD

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

1. Open "Details_Teriko-1.0.tfl", File -> Export to "Details_Teriko-1.0.tflx" (default, replace existing)
2. Open "NVD_Teriko-1.0.tfl", File -> Export to "NVD_Teriko-1.0.tflx" (default, replace existing)

Else you can skip these steps. Their purpose is to pull in the local Excel mapping data into the Prep Builder package (.tflx) file(s).

### Monthly Steps
Note: Be sure the Tableau Prep Builder application is closed and not running. Eases a server login issue.

1. Open "Detail_Teriko-1.0.tflx",   Flow -> Run All (~ 1 to 1.5 hours)
2. Open "NVD_Teriko-1.0.tflx",      Flow -> Run All (~ 7 to 10 minutes)
3. Open "Details+NVD.tfl",          Flow -> Run All (~ 1 hour)

Expect the total run time to be 2 hours 45 minutes assuming no delay in between steps.

Note: There may be a way to publish flows to the server and have them automatically scheduled.

## Transformation Details

### Excel IPS_Product_Details.xls

There are two mapping tables in the associated Excel workbook,

* Manufacturers

The Manufacturer sheet/table contains two fields,

    * Old
    * New

The "Old" field is intended to join with the "IPS Product Details".MFGR field.
The "New" field either contains a name that is intended to substituted for an existing name (this is an opportunity to group names together into a common name) or left blank. In the left blank case, a formula in the workflow will fill in a computed value in place of the original name, originally this was BRAND or, if null, MFGR_ID.

* Distributors

The Distributor sheet/table contains two fields

    * Old
    * New

The "Old" field is intended to join with the "IPS Product Details".DISTRIBUTOR field
The "New" field contains substitute names for the given distributor. Please see the actual formula for specifics.


### Detail_Teriko-1.0.tfl / Product Detail (Clean)

Renaming:

    * COMPONENT                 -> Member Name
    * MFGR                      -> Manufacturer
    * PC_STATE_CD               -> State
    * PRODUCT_MASTER_ID         -> Product Master ID (string from int)
    * PURCHASES                 -> Purchase $'s
    * VOLUME                    -> Cases (Product Detail)
    * GPO_MEMBER_ID             -> IPS Member ID
    * PARENT_MANUFACTURER_NAME  -> Parent Manufacturer
    * MFGR#                     -> #SKU (trimmed to remove extra spaces)
    * PERIOD_YEAR               -> Year
    * DISTRIBUTOR               -> Distributor House
    * Capacity (Salesforce)     -> Student Count (int to string)

Calculated Fields:

    * Weight (LBS)
```
    If     [ITEM_UOM] = "LBS" THEN ROUND([PACK] * [ITEM])
    ELSEIF [ITEM_UOM] = "OZ"  THEN ROUND([PACK] * ([ITEM]/16))
    ELSEIF [ITEM_UOM] = "LB"  THEN ROUND([PACK] * [ITEM])
    ELSEIF [ITEM_UOM] = "OZS" THEN ROUND([PACK] * ([ITEM]/16))
    END
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

    * Pack Size
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
            [MFGR_ID]
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
    Notice: When a name substiturion (Old to New) is not found, the original DISTRIBUTOR name is used, but is prefixed with an asterix so that missing values can be spotted quickly. The DISTRIBUTOR field is renamed to "Distributor House".

    * Join "Salesforce/Salesforce Accounts Extract" on "IPS Member ID"
        * First aggregate table.
        * Group by "IPS Member ID"
        * MAX of Student Count
        * MAX of Name of Co-Op

    * Inner Join on Manufacturer, #SKU
        * MAX of Product Description
        * MAX of Pack Size

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



### NVD_Teriko-1.0.tfl / Billed NVD (Clean)

Renaming:

    * IPS_MEMBER_ID                 -> IPS Member ID
    * ITEM_PACK_SIZE                -> PACK
    * ITEM_SIZE                     -> ITEM
    * TOTAL_PKG_QUANTITY            -> Cases (NVD)
    * MANUFACTURER_ID               -> MFGR_ID
    * ARA_PRODUCT_ID                -> ARA_PRODUCT_ID (int to string)
    * BRAND_ID                      -> BRAND_ID (int to string)
    * REBATE_INVOICE_NUMBER         -> REBASE_INVOICE_NUMBER (int to string)
    * MANUFACTURER_PRODUCT_CD       -> #SKU (timmed to remove extra spaces)

Calculated Fields:

    * Manufacturer
    Join "Manufacturer" to Excel.Manufacturer.Old to pick up new names or compute missing names (detailed in Excel)
```
    IF ISNULL([Old]) THEN
        [MANUFACTURER_NAME]
    ELSEIF ISNULL([New]) THEN
        IF ISNULL([BRAND_NAME]) THEN
            [MFGR_ID]
        ELSE
            [BRAND_NAME]
        END
    ELSE
        [New]
    END
```
    Note: This is a similar process to the "Detail_Teriko-1.0" calculation for Manufacturer.

    * Date
```
    DATE(DATEPARSE ("MM/yyyy", [CALENDAR_MONTH] + "/" + [CALENDAR_YEAR]))
```

    * FiscalDate
```
    DATE(DATEPARSE ("MM/yyyy", STR([FISCAL_MONTH]) + "/" + STR([FISCAL_YEAR])))
```

*NOTE:*

We will join Billed NVD with Product Details using the following keys,

* MANUFACTURER_ID
* MANUFACTURER_PRODUCT_CD
* IPS_MEMBER_ID
* CALENDAR_MONTH + CALENDAR_YEAR (date)

We need to be aware that records in this table are keyed more finely.
In particular,

REBATEABLE_PURCH_AMT and TOTAL_PKG_QUANTITY are indexed by the above and also,

* ITEM_PACK_SIZE + ITEM_UOM
* FISCAL_MONTH + FISCAL_YEAR

BILLED_REBATE_AMT is indexed all the above plus,

* NVD_RATE, NVD_RATE_BASIS_TYPE, INCOME_PROVISION

```
If NVD_RATE_BASIS_TYPE = SPD Then
    BILLED_REBATE_AMT = NVD_RATE * REBATEABLE_PURCH_AMT

If NVD_RATE_BASIS_TYPE = PKG Then
    BILLED_REBATE_AMT = NVD_RATE * TOTAL_PKG_QUANTITY
```

### Details+NVD.tfl / Product Detail + NVD (Clean)

The purpose of this workflow is to selectively pick up (aggregated: see below) records from the "Billed NVD (Clean)" table and add fields, but not rows to the existing "Product Detail (Clean)" table.

This workflow is a controlled join of the two tables constructed by the workflows above,

    * server: Teriko's Projects/Product Detail (Clean)
    * server: Teriko's Projects/Billed NVD (Clean)

Renaming of "Billed NVD (Clean)" fields,

    * PRODUCT_DESCRIPTION   -> Product Description (NVD)
    * TOTAL_PURCH_AMT       -> Purchase $'s (NVD)
    * TOTAL_WT_QUANTITY     -> Weight (LBS)(NVD)
    * Manufacturerer        -> Manufacturer (NVD)

The join fields are,

    * MFGR_ID
    * IPS Member ID
    * #SKU
    * Date

Noticing that the above fields do not uniquely identify records in the Billed NVD table, the table needs to first be aggregated before joining on the above keys with the Product Detail table,

The aggregations are,

    * MAX of ARA_PRODUCT_ID
    * MAX of Manufacturer (NVD)
    * MAX of Product Description (NVD)
    * MAX of PRODUCT_CENTER_CD
    * MAX of REBATE_ID
    * SUM of BILLED_REBATE_AMT
    * SUM of Cases (NVD)
    * SUM of Purchase $'s (NVD)
    * SUM of REBATEABLE_PURCH_AMT
    * SUM of Weight (LBS)(NVD)

*Note:*
All of the fields and records of the "Product Details (Clean)" appear in the final result.
Only the aggregation fields of the "Billed NVD" table will appear in the final result.
