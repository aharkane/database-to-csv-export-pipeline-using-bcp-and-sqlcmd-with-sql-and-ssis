# Database-to-CSV Export Pipeline using BCP and SqlCmd with SQL and SSIS

## Table of Contents

1. [Overview](#overview)
2. [Package 1: ETL DB-to-CSV using BCP](#package-1-etl-db-to-csv-using-bcp)
3. [Package 2: ETL DB-to-CSV using SqlCmd](#package-2-etl-db-to-csv-using-sqlcmd)


---

## Overview

Automated data warehouse to csv files export solution, with dynamic table processing and intelligent folder routing. Features two approaches using BCP (Bulk Copy Program) and SqlCmd utilities with variable-driven command construction, using Microsoft ETL tool SSIS,  for extracting data from AdventureWorks 2022 Data Warehouse (AdvWrks2022_DWH)

---


## Package 1: ETL DB-to-CSV using BCP
Database extraction using Bulk Copy Program (BCP) utility, optimized for rapid export of database tables to CSV format with minimal transformation overhead.

**User Variables**

| Variable Name | Data Type | Purpose | Example Value |
|---------------|-----------|---------|---|
| `ServerName` | String | SQL Server instance | `.` (local) |
| `DatabaseName` | String | Target database | `AdvWrks2022_DWH` |
| `CurrentSchema` | String | Database schema | `prod` |
| `DataFolder` | String | Root export directory | `D:\...\Data\` |
| `CurrentTable` | String | Current table being processed | (Empty - set by loop) |
| `CurrentFile` | String (Expression) | Full file path | `D:\...\Data\[TableName].csv` |
| `TableList` | Array | Enumerated table names | (Populated by SQL query) |
| `BCPCommand` | String (Expression) | Complete BCP command line | See below |

**Execution Workflow Summary**

```
Initialize
    ↓
Execute "Get Table List" SQL Query
    ↓
Populate TableList variable with all prod schema tables
    ↓
BEGIN FOREACH LOOP (each table in TableList)
    ├── Set CurrentTable = [table name]
    ├── Construct BCPCommand expression
    ├── Execute BCP.EXE with command
    ├── Write CSV file to DataFolder\[TableName].csv
    └── Continue to next table
END FOREACH LOOP
    ↓
All tables exported as CSV files
```

**Critical Variable:**

*BCPCommand Expression*:
```
"SELECT * FROM [" + @[User::CurrentSchema] + "].[" + @[User::CurrentTable] + 
"]" queryout "" + @[User::CurrentFile] + 
"" -c -t, -r\n -q -S " + @[User::ServerName] + 
" -d " + @[User::DatabaseName] + " -T -C 65001"
```

*Breakdown*:
- `SELECT * FROM [prod].[TableName]`: Dynamic query selecting all columns
- `queryout`: BCP mode for exporting query results to file
- `""`: Quoted file path with dynamic concatenation
- `-c`: Character mode (readable text format) `-t,`: Tab character as field terminator (comma-separated) `-r\n`: Newline character as row terminator `-q`: Quoted identifiers
- `-S .`: Server name (local instance)
- `-d AdvWrks2022_DWH`: Database specification
- `-T`: Trusted (Windows) authentication
- `-C 65001`: UTF-8 code page for international characters

*Sample Execution for DimProduct*:
```
bcp.exe "SELECT * FROM [prod].[DimProduct]" queryout "D:\Data\DimProduct.csv" 
-c -t, -r\n -q -S . -d AdvWrks2022_DWH -T -C 65001
```

**Advantages of BCP Approach**

- **Extreme Performance**: BCP is fastest native SQL Server bulk operation tool
- **Minimal Overhead**: Direct database-to-file with no intermediate staging
- **Character Mode Simplicity**: Clean, readable CSV format

---

## Package 2: ETL DB-to-CSV using SqlCmd

Flexible database extraction using the versatile SqlCmd command-line utility, with intelligent table classification (Dimensions vs. Facts) and organized output folder structure for business intelligence workflows.

**User Variables**

| Variable Name | Data Type | Purpose | Example Value |
|---|---|---|---|
| `_ServerName` | String | SQL Server instance | `.` (local) |
| `_DatabaseName` | String | Target database | `AdvWrks2022_DWH` |
| `_SchemaName` | String | Database schema | `prod` |
| `_DimTables_FolderName` | String | Dimension output folder | `D:\...\Data\DimTables\` |
| `_FactTables_FolderName` | String | Fact tables output folder | `D:\...\Data\FactTables\` |
| `CurrentTable` | String | Current table being processed | (Empty - set by loop) |
| `CurrentFile` | String (Expression) | Dynamic file path based on table type | See below |
| `SqlCmd_loadingScript` | String (Expression) | Dynamic T-SQL query | See below |
| `SqlCmd_Arg` | String (Expression) | Complete SqlCmd arguments | See below |
| `TableList` | Array | Enumerated table names | (Populated by SQL query) |


**Execution Workflow Summary**

```
Initialize
    ↓
Execute "Get Table List" SQL Query
    ↓
Populate TableList variable with all prod schema tables
    ↓
BEGIN FOREACH LOOP (each table in TableList)
    ├── Set CurrentTable = [table name]
    ├── Evaluate CurrentFile expression (Dim vs. Fact routing)
    ├── Evaluate SqlCmd_loadingScript expression
    ├── Construct SqlCmd_Arg expression
    ├── Execute SQLCMD.EXE with command
    ├── Write CSV file to DataFolder\[TableName].csv
    └── Continue to next table
END FOREACH LOOP
    ↓
All tables exported as CSV files
```

**Critical Variable:**   
*1. CurrentFile Expression*
```
LEFT(@[User::CurrentTable], 3) == "Dim" 
? @[User::_DimTables_FolderName] + @[User::CurrentTable] + ".csv" 
: @[User::_FactTables_FolderName] + @[User::CurrentTable] + ".csv"
```
*Breakdown*: 
- **If** table name starts with "Dim" → Route to DimTables folder
- **Else** (Fact tables) → Route to FactTables folder
- 

*2. SqlCmd_loadingScript Expression*

```
"SET NOCOUNT ON; SELECT * FROM " + @[User::_SchemaName] + "." + @[User::CurrentTable]
```
*Breakdown*:
- `SET NOCOUNT ON`: Suppresses row count message from result set (cleaner output)
- `SELECT * FROM prod.[CurrentTable]`: Dynamic query selecting all columns


*3. SqlCmd_Arg Expression*

```
"-S " + @[User::_ServerName] +
" -d " + @[User::_DatabaseName] +
" -E -W -s\";\" -w 65535" +
" -Q \"" + @[User::SqlCmd_loadingScript] + "\"" +
" -o\"" + @[User::CurrentFile] + "\"" +
" -f 65001"
```

*Breakdown*:
- `-S .`: Server name
- `-d AdvWrks2022_DWH`: Database name
- `-E`: Trusted connection (Windows authentication); `-W`: Remove trailing spaces from each field; `-s`;`: Separator character (semicolon); `-w 65535`: Column width (very wide to prevent truncation)
- `-Q "SET NOCOUNT ON; SELECT ..."`: Query to execute
- `-o"D:\...\[TableName].csv"`: Output file path
- `-f 65001`: Code page (UTF-8)


**Advantages of SqlCmd Approach**

- **Flexibility**: General-purpose T-SQL execution with complex query support
- **Organized Separation**: Intelligent folder routing
- **Query Logic**: Support for complex T-SQL including NOCOUNT and formatting


---




## Conclusion

**Processing Approaches Comparison**

| Aspect | BCP Package | SqlCmd Package |
|--------|-------------|----------------|
| **Utility** | BCP.EXE (Bulk Copy Program) | SQLCMD.EXE (Command-line Tool) |
| **Specialization** | Dedicated bulk data movement | General-purpose T-SQL execution |
| **Performance** | Optimized for high-volume bulk operations | Flexible query execution |
| **Delimiter Control** | Native `-t` flag for separators | `-s` flag for separator specification |
| **Output Control** | Query output formatting | File redirection via `-o` flag |
| **Authentication** | Integrated Windows auth (`-T`) | Windows (`-E`) or SQL auth |
| **Use Case** | Large-scale bulk extraction | Complex queries, specific formatting |
| **Table Separation** | Single output folder | Dimension vs. Fact folder separation |
| **Width Control** | Default or fixed widths | Explicit column width (`-w`) |
