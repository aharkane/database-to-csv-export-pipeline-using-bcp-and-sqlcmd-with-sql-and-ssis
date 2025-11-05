# SSIS Data Extraction Packages: Database-to-CSV Export Pipeline

Advanced SQL Server Integration Services (SSIS) implementation showcasing enterprise-grade database extraction and file export operations using two distinct command-line approaches: BCP (Bulk Copy) and SqlCmd utilities.

---

## Table of Contents

1. [Overview](#overview)
2. [Technical Skills Demonstrated](#technical-skills-demonstrated)
3. [Project Architecture](#project-architecture)
4. [Package 1: ETL DB-to-CSV using BCP](#package-1-etl-db-to-csv-using-bcp)
5. [Package 2: ETL DB-to-CSV using SqlCmd](#package-2-etl-db-to-csv-using-sqlcmd)
6. [Command-Line Integration Patterns](#command-line-integration-patterns)
7. [Variable-Driven Architecture](#variable-driven-architecture)
8. [Database Schema Reference](#database-schema-reference)
9. [Implementation Highlights](#implementation-highlights)
10. [Data Export Workflows](#data-export-workflows)

---

## Overview

This project demonstrates a comprehensive database export solution built with SQL Server Integration Services (SSIS) for extracting data from dimensional and fact tables into CSV format. The implementation showcases two professional approaches to command-line based data extraction: the specialized BCP (Bulk Copy Program) utility and the versatile SqlCmd command-line tool.

Both packages target the AdventureWorks 2022 Data Warehouse (AdvWrks2022_DWH), processing dimensional tables and fact tables with dynamic table enumeration and parameterized export operations. The solution exemplifies enterprise-grade data distribution, business intelligence data extraction, and external reporting workflows.

---

## Technical Skills Demonstrated

### SSIS Advanced Techniques

- **Package Development**: Production-ready SSIS packages with version control (v17.0.1008.3) and build numbers
- **Expression-Based Variables**: Complex SSIS expressions for dynamic command string generation
- **Process Task Execution**: Execute Process Task implementations for command-line utility integration
- **Control Flow Containers**: Foreach loops for batch table enumeration and iteration
- **Dynamic Command Generation**: Runtime construction of BCP and SqlCmd parameters
- **Variable Expressions**: Multi-line expressions with string concatenation and conditional logic

### Command-Line Database Tools

- **BCP (Bulk Copy Program) Integration**:
  - Native SQL Server bulk data mover
  - Character mode output (`-c` flag)
  - Custom delimiter handling (`-t,` for comma-separated values)
  - Row terminator configuration (`-r\n` for newline)
  - Authentication and encryption options
  - Code page specification (UTF-8 via `-C 65001`)

- **SqlCmd Command-Line Utility**:
  - T-SQL query execution from command line
  - Custom delimiters (`-s` separator flag)
  - Output redirection to file (`-o` parameter)
  - Column width control (`-w` for wide output)
  - Scripting variables and expressions
  - Error handling and query results formatting

### Data Extraction Patterns

- **Bulk Export Operations**: High-performance data extraction to flat files
- **Schema-Based Table Enumeration**: Dynamic discovery and processing of all schema tables
- **Fact vs. Dimension Separation**: Intelligent routing based on table naming conventions
- **Table-Level Iteration**: Foreach loop processing of multiple database tables
- **Dynamic File Naming**: Expression-driven CSV file naming with schema-qualified table names
- **Parameterized Queries**: Query templates populated with current table context

### SQL Server Integration

- **Source Database**: AdventureWorks 2022 Data Warehouse (AdvWrks2022_DWH)
- **Source Schema**: `prod` (production schema containing dimensional and fact tables)
- **Authentication**: Windows authentication (Trusted Connection via `-T` flag)
- **Character Encoding**: UTF-8 support (code page 65001) for international character handling
- **Query Execution**: Direct T-SQL SELECT queries without intermediate staging

### Data Quality & Transformation

- **NULL Handling**: Proper representation of missing values in CSV format
- **Data Type Preservation**: Numeric, date, and string types maintained through export
- **Delimiter Management**: Custom delimiter specification to avoid conflicts with data
- **Column Mapping**: Automatic or explicit column sequence preservation
- **Large Dataset Support**: Optimized for bulk operations and high-volume exports
- **Character Encoding**: UTF-8 standard for global compatibility

### Development & Automation

- **Version Control**: SSIS projects ready for Git/GitHub integration
- **Scheduled Execution**: Designed for integration with SQL Server Agent
- **Parametric Configuration**: Server, database, schema, and folder path configuration via variables
- **Batch Processing**: Sequential processing of multiple tables in single execution
- **Error Handling**: Command exit code validation and logging capabilities
- **Reusability**: Template-based design for adapting to other databases and tables

---

## Project Architecture

### High-Level Export Pattern

```
AdventureWorks 2022 DWH Database
    ├── prod.DimProduct
    ├── prod.DimCustomer
    ├── prod.DimDate
    ├── prod.DimGeography
    ├── prod.FactInternetSales
    └── prod.FactReseller
    
         ↓ [Get Table List]
    
    [Execute SQL Query]
    ↓ [Retrieves list of all tables in prod schema]
    
    [Dynamic Variable: TableList]
    
         ↓ [Foreach Loop Container]
    
    For Each Table in TableList:
    ├── Set CurrentTable variable
    ├── Generate dynamic export command
    ├── Execute command-line utility (BCP or SqlCmd)
    └── Output: [TableName].csv
    
         ↓ [File Output]
    
    Data\DimTables\ (for Dimension tables)
    Data\FactTables\ (for Fact tables)
    Data\ (for mixed output)
```

### Processing Approaches Comparison

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

---

## Package 1: ETL DB-to-CSV using BCP

### Purpose

High-performance database extraction using the specialized Bulk Copy Program (BCP) utility, optimized for rapid export of database tables to CSV format with minimal transformation overhead.

### Package Metadata

- **Name**: ETL Package (BCP Implementation)
- **Creation Date**: August 18, 2025
- **SSIS Version**: 17.0.1008.3
- **Version Build**: 21
- **Source Database**: AdvWrks2022_DWH
- **Source Schema**: prod

### User Variables

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

### Critical Variable: BCPCommand Expression

**Expression Logic**:
```
"SELECT * FROM [" + @[User::CurrentSchema] + "].[" + @[User::CurrentTable] + 
"]" queryout "" + @[User::CurrentFile] + 
"" -c -t, -r\n -q -S " + @[User::ServerName] + 
" -d " + @[User::DatabaseName] + " -T -C 65001"
```

**Breakdown**:
- `SELECT * FROM [prod].[TableName]`: Dynamic query selecting all columns
- `queryout`: BCP mode for exporting query results to file
- `""`: Quoted file path with dynamic concatenation
- `-c`: Character mode (readable text format)
- `-t,`: Tab character as field terminator (comma-separated)
- `-r\n`: Newline character as row terminator
- `-q`: Quoted identifiers
- `-S .`: Server name (local instance)
- `-d AdvWrks2022_DWH`: Database specification
- `-T`: Trusted (Windows) authentication
- `-C 65001`: UTF-8 code page for international characters

**Example Expansion**:
```
"SELECT * FROM [prod].[DimProduct]" queryout "D:\Data\DimProduct.csv" 
-c -t, -r\n -q -S . -d AdvWrks2022_DWH -T -C 65001
```

### Control Flow Structure

#### 1. **Get Table List** (Execute SQL Task)

**Purpose**: Populate the TableList variable with all table names from the production schema

**SQL Query**:
```sql
SELECT TABLE_NAME 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_SCHEMA = 'prod' 
ORDER BY TABLE_NAME
```

**Output**: Resultset mapped to `TableList` variable (array of table names)

**Execution Timing**: Pre-loop initialization

#### 2. **Foreach Loop Container**

**Enumerator Type**: Foreach Enumerator from Variable
- **Variable**: `User::TableList` (string array)
- **Iteration**: Sequential processing of each table name
- **Current Table Variable**: `User::CurrentTable` (assigned during each iteration)

**Nested Components**:

##### a) **Execute Process Task** (implied/referenced)

**Task Purpose**: Execute BCP command-line utility

**Configuration**:
- **Executable**: `bcp.exe` (path from system PATH or full path)
- **Arguments**: `@[User::BCPCommand]` (dynamically constructed per loop iteration)
- **Working Directory**: Output folder for CSV files
- **Error Handling**: Capture exit code for validation

**Process Execution Flow**:
```
1. Construct BCPCommand for current table
2. Launch BCP.EXE with constructed parameters
3. BCP connects to SQL Server
4. BCP executes SELECT query
5. BCP formats output as character CSV
6. BCP writes to file with table name
7. Verify exit code (0 = success)
```

**Sample Execution for DimProduct**:
```
bcp.exe "SELECT * FROM [prod].[DimProduct]" queryout "D:\Data\DimProduct.csv" 
-c -t, -r\n -q -S . -d AdvWrks2022_DWH -T -C 65001
```

### BCP Output Format

**Generated CSV File**: `[TableName].csv`

**Format Characteristics**:
- **Delimiter**: Comma (`,`)
- **Row Terminator**: Newline (`\n`)
- **Mode**: Character mode (readable text)
- **Encoding**: UTF-8 (code page 65001)
- **Headers**: No column headers (BCP default; headers can be added in post-processing)
- **NULLs**: Empty fields for NULL values

**Example Output** (DimProduct.csv):
```
1,AR-5381,Adjustable Race,0.00,0.00,(NULL),(NULL),(NULL),NULL,2008-04-30,NULL,NULL,...
2,BA-8327,Bearing Ball,0.00,0.00,(NULL),(NULL),(NULL),NULL,2008-04-30,NULL,NULL,...
3,BE-2349,BB Ball Bearing,0.00,0.00,(NULL),(NULL),(NULL),NULL,2008-04-30,NULL,NULL,...
```

### Advantages of BCP Approach

- **Extreme Performance**: BCP is fastest native SQL Server bulk operation tool
- **Minimal Overhead**: Direct database-to-file with no intermediate staging
- **Character Mode Simplicity**: Clean, readable CSV format
- **Native Authentication**: Integrated Windows authentication support
- **Delimiter Flexibility**: Custom separator specification via `-t` flag
- **Encoding Support**: UTF-8 and other code pages for international data

### Execution Workflow Summary

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

---

## Package 2: ETL DB-to-CSV using SqlCmd

### Purpose

Flexible database extraction using the versatile SqlCmd command-line utility, with intelligent table classification (Dimensions vs. Facts) and organized output folder structure for business intelligence workflows.

### Package Metadata

- **Name**: ETL Package (SqlCmd Implementation)
- **Creation Date**: August 18, 2025
- **SSIS Version**: 17.0.1008.3
- **Version Build**: 36
- **Source Database**: AdvWrks2022_DWH
- **Source Schema**: prod

### User Variables

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

### Critical Variable: CurrentFile Expression

**Expression Logic**:
```
LEFT(@[User::CurrentTable], 3) == "Dim" 
? @[User::_DimTables_FolderName] + @[User::CurrentTable] + ".csv" 
: @[User::_FactTables_FolderName] + @[User::CurrentTable] + ".csv"
```

**Interpretation**: 
- **If** table name starts with "Dim" → Route to DimTables folder
- **Else** (Fact tables) → Route to FactTables folder

**Example Evaluations**:
```
DimProduct → D:\...\Data\DimTables\DimProduct.csv
FactInternetSales → D:\...\Data\FactTables\FactInternetSales.csv
FactReseller → D:\...\Data\FactTables\FactReseller.csv
```

### Critical Variable: SqlCmd_loadingScript Expression

**Expression Logic**:
```
"SET NOCOUNT ON; SELECT * FROM " + @[User::_SchemaName] + "." + @[User::CurrentTable]
```

**Components**:
- `SET NOCOUNT ON`: Suppresses row count message from result set (cleaner output)
- `SELECT * FROM prod.[CurrentTable]`: Dynamic query selecting all columns

**Example Expansion**:
```
SET NOCOUNT ON; SELECT * FROM prod.DimCustomer
```

### Critical Variable: SqlCmd_Arg Expression

**Expression Logic** (Multi-line):
```
"-S " + @[User::_ServerName] +
" -d " + @[User::_DatabaseName] +
" -E -W -s\";\" -w 65535" +
" -Q \"" + @[User::SqlCmd_loadingScript] + "\"" +
" -o\"" + @[User::CurrentFile] + "\"" +
" -f 65001"
```

**Component Breakdown**:
- `-S .`: Server name
- `-d AdvWrks2022_DWH`: Database name
- `-E`: Trusted connection (Windows authentication)
- `-W`: Remove trailing spaces from each field
- `-s`;`: Separator character (semicolon)
- `-w 65535`: Column width (very wide to prevent truncation)
- `-Q "SET NOCOUNT ON; SELECT ..."`: Query to execute
- `-o"D:\...\[TableName].csv"`: Output file path
- `-f 65001`: Code page (UTF-8)

**Example Expansion**:
```
-S . -d AdvWrks2022_DWH -E -W -s";" -w 65535 
-Q "SET NOCOUNT ON; SELECT * FROM prod.DimGeography" 
-o"D:\Data\DimTables\DimGeography.csv" -f 65001
```

### Control Flow Structure

#### 1. **Get Table List** (Execute SQL Task)

**Purpose**: Identical to BCP package - populate TableList variable

**SQL Query**:
```sql
SELECT TABLE_NAME 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_SCHEMA = 'prod' 
ORDER BY TABLE_NAME
```

**Output**: `TableList` array variable

#### 2. **Foreach Loop Container**

**Enumerator Type**: Foreach Enumerator from Variable
- **Variable**: `User::TableList` (string array)
- **Current Item Mapping**: `User::CurrentTable` (assigned per iteration)

**Nested Components**:

##### a) **Execute Process Task** (implied/referenced)

**Task Purpose**: Execute SqlCmd command-line utility

**Configuration**:
- **Executable**: `sqlcmd.exe` (path from system PATH or full path)
- **Arguments**: Constructed from `@[User::SqlCmd_Arg]` expression
- **Working Directory**: Data export folders
- **Error Handling**: Exit code validation

**Process Execution Flow**:
```
1. Evaluate CurrentFile expression (Dim vs. Fact routing)
2. Evaluate SqlCmd_loadingScript expression (generate query)
3. Evaluate SqlCmd_Arg expression (construct full command)
4. Launch SQLCMD.EXE with constructed parameters
5. SqlCmd connects to SQL Server with Windows auth
6. SqlCmd executes T-SQL query
7. SqlCmd formats output with semicolon separator
8. SqlCmd removes trailing spaces from fields
9. SqlCmd redirects output to CSV file
10. Verify exit code (0 = success)
```

**Sample Execution for DimCustomer**:
```
sqlcmd.exe -S . -d AdvWrks2022_DWH -E -W -s";" -w 65535 
-Q "SET NOCOUNT ON; SELECT * FROM prod.DimCustomer" 
-o"D:\Data\DimTables\DimCustomer.csv" -f 65001
```

### SqlCmd Output Format

**Generated CSV File**: `[TableName].csv` (in appropriate folder based on table type)

**Format Characteristics**:
- **Delimiter**: Semicolon (`;`)
- **Column Width**: 65535 characters (prevents truncation)
- **Trailing Spaces**: Removed (`-W` flag)
- **Encoding**: UTF-8 (code page 65001)
- **Headers**: Included (SqlCmd default - query results with column names)
- **NULLs**: Empty fields or NULL literal depending on data type

**Example Output** (DimCustomer.csv):
```
CustomerKey;GeographyKey;CustomerAlternateKey;Title;FirstName;MiddleName;LastName;...
1;1;13001;Mr.;Orlando;N;Gundersen;...
2;1;13002;Mr.;Gustavo;N;Achong;...
3;1;13003;Dr.;Catherine;R;Abel;...
```

### Advantages of SqlCmd Approach

- **Flexibility**: General-purpose T-SQL execution with complex query support
- **Output Control**: `-W` flag for clean whitespace handling
- **Organized Separation**: Intelligent folder routing for Dimensions vs. Facts
- **Wide Columns**: `-w` parameter prevents data truncation
- **Query Logic**: Support for complex T-SQL including NOCOUNT and formatting
- **Format Consistency**: StandardSQL formatting across all databases
- **Error Messages**: Detailed error output for debugging

---

## Command-Line Integration Patterns

### Pattern 1: Dynamic Command Construction via Expressions

Instead of static command strings, both packages build commands at runtime:

```
Static (Bad):
Task: Execute "C:\bcp.exe SELECT * FROM DimProduct ..."

Dynamic (Good):
Variable: BCPCommand = Expression that builds:
  "SELECT * FROM [" + CurrentSchema + "].[" + CurrentTable + "]" ...
Task: Execute sqlcmd @[User::SqlCmd_Arg]
```

**Advantages**:
- Single template handles all tables
- No hardcoding of table names
- Easily adapts to schema changes
- Reusable across different databases

### Pattern 2: Table Enumeration via SQL Query

Pre-populate list of all tables dynamically:

```sql
-- Executed once at package start
SELECT TABLE_NAME 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_SCHEMA = 'prod'
```

**Result**: TableList variable contains all table names

**Benefit**: Package discovers tables automatically without maintenance

### Pattern 3: Intelligent Output Routing (SqlCmd)

Conditional folder assignment based on naming convention:

```
IF table name starts with "Dim"
  → Output to DimTables folder
ELSE (Fact tables)
  → Output to FactTables folder
```

**Expression**:
```
LEFT(CurrentTable, 3) == "Dim" 
? DimFolder + Table + ".csv" 
: FactFolder + Table + ".csv"
```

**Benefit**: Organized output structure matching data warehouse dimensional model

### Pattern 4: Foreach Loop for Batch Processing

Iterate over all tables sequentially:

```
For Each Table in TableList:
  1. Set CurrentTable = Table
  2. Expressions evaluate with new context
  3. Execute command with current table parameters
  4. Output file created
  5. Continue to next table
```

**Result**: N table exports in single package execution

### Pattern 5: Process Exit Code Validation

Capture and validate utility execution:

```
Command execution → Exit Code (0=success, non-zero=error)
If success: Continue
If error: Log error, continue or fail based on configuration
```

**Enable Debugging**:
```
Precedence Constraint: On success, continue
Error handling: Redirect to error task
```

---

## Variable-Driven Architecture

### Three-Layer Variable Structure

#### Layer 1: Configuration Variables (Static)
```
_ServerName = "."
_DatabaseName = "AdvWrks2022_DWH"
_SchemaName = "prod"
_DimTables_FolderName = "D:\Data\DimTables\"
_FactTables_FolderName = "D:\Data\FactTables\"
```
**Purpose**: Static configuration for environment/database setup

#### Layer 2: Iteration Variables (Loop Context)
```
CurrentTable = "" (populated by foreach loop)
TableList = [DimCustomer, DimDate, DimGeography, ..., FactInternetSales, ...]
```
**Purpose**: Track current table and enumerate all tables

#### Layer 3: Expression Variables (Dynamic)
```
CurrentFile = Expression (evaluate at runtime based on table type)
BCPCommand = Expression (construct complete command string)
SqlCmd_Arg = Expression (build all arguments)
SqlCmd_loadingScript = Expression (dynamic T-SQL query)
```
**Purpose**: Build runtime parameters based on context

### Variable Evaluation Timing

```
Package Start:
  ├─ Configuration variables initialized
  └─ Execute SQL to populate TableList
    
Foreach Loop - Iteration 1:
  ├─ CurrentTable = "DimCustomer"
  ├─ Expression variables evaluate:
  │   ├─ CurrentFile → "D:\Data\DimTables\DimCustomer.csv"
  │   ├─ BCPCommand/SqlCmd_Arg → Complete command with DimCustomer
  │   └─ SqlCmd_loadingScript → "SELECT * FROM prod.DimCustomer"
  ├─ Execute process with evaluated values
  └─ Output: DimCustomer.csv created
    
Foreach Loop - Iteration 2:
  ├─ CurrentTable = "DimDate"
  ├─ All expressions re-evaluate with new table name
  ├─ Execute with DimDate context
  └─ Output: DimDate.csv created
  
[Continue for all tables in list]
```

---

## Database Schema Reference

### Source Database: AdventureWorks 2022 Data Warehouse

**Database Name**: AdvWrks2022_DWH  
**Source Schema**: prod  
**Connection**: Windows Trusted Authentication

### Sample Table Structure

#### Dimension Tables (Dim prefix)

```
Dimension Table Example - DimProduct:
├── ProductKey (Primary Key, Int)
├── ProductAlternateKey (String)
├── ProductName (String)
├── StandardCost (Decimal)
├── ListPrice (Decimal)
├── Color (String)
├── Size (String)
├── SizeUnitCode (String)
├── Weight (Decimal)
├── WeightUnitCode (String)
├── ProductLine (String)
├── DealerPrice (Decimal)
├── Class (String)
├── Style (String)
├── ModelName (String)
├── ProductDescription (String)
├── ProductCategoryKey (Foreign Key)
├── ProductSubcategoryKey (Foreign Key)
├── DateKey (Foreign Key)
└── Status (String)

Dimension Table Example - DimCustomer:
├── CustomerKey (Primary Key, Int)
├── CustomerAlternateKey (String)
├── FirstName (String)
├── MiddleName (String)
├── LastName (String)
├── NameStyle (Bit)
├── BirthDate (Date)
├── MaritalStatus (String)
├── Suffix (String)
├── Gender (String)
├── EmailAddress (String)
├── YearlyIncome (Money)
├── TotalChildren (Int)
├── NumberChildrenAtHome (Int)
├── EnglishEducation (String)
├── SpanishEducation (String)
├── FrenchEducation (String)
├── Occupation (String)
├── HouseOwnerFlag (Bit)
├── NumberCarsOwned (Int)
├── AddressLine1 (String)
├── AddressLine2 (String)
├── Phone (String)
├── DateFirstPurchase (Date)
└── CommuteDistance (String)
```

#### Fact Tables (Fact prefix)

```
Fact Table Example - FactInternetSales:
├── ProductKey (Foreign Key)
├── OrderDateKey (Foreign Key)
├── DueDateKey (Foreign Key)
├── ShipDateKey (Foreign Key)
├── CustomerKey (Foreign Key)
├── PromotionKey (Foreign Key)
├── CurrencyKey (Foreign Key)
├── SalesTerritoryKey (Foreign Key)
├── SalesOrderNumber (String)
├── SalesOrderLineNumber (Int)
├── RevisionNumber (Int)
├── OrderQuantity (SmallInt)
├── UnitPrice (Money)
├── ExtendedAmount (Money)
├── UnitPriceDiscountPct (Float)
├── DiscountAmount (Money)
├── ProductStandardCost (Money)
├── TotalProductCost (Money)
├── SalesAmount (Money)
├── TaxAmt (Money)
├── Freight (Money)
└── CarrierTrackingNumber (String)

Fact Table Example - FactReseller:
├── ProductKey (Foreign Key)
├── OrderDateKey (Foreign Key)
├── DueDateKey (Foreign Key)
├── ShipDateKey (Foreign Key)
├── ResellerKey (Foreign Key)
├── EmployeeKey (Foreign Key)
├── PromotionKey (Foreign Key)
├── CurrencyKey (Foreign Key)
├── SalesTerritoryKey (Foreign Key)
├── SalesOrderNumber (String)
├── SalesOrderLineNumber (Int)
├── RevisionNumber (Int)
├── OrderQuantity (SmallInt)
├── UnitPrice (Money)
├── ExtendedAmount (Money)
├── UnitPriceDiscountPct (Float)
├── DiscountAmount (Money)
├── ProductStandardCost (Money)
├── TotalProductCost (Money)
├── SalesAmount (Money)
├── TaxAmt (Money)
└── Freight (Money)
```

---

## Implementation Highlights

### 1. Enterprise-Grade Parametrization

Both packages eliminate hardcoding through comprehensive variables:
- Server, database, schema all configurable
- Output folders parameterized for easy relocation
- Commands dynamically constructed for any table
- Single package template serves entire schema

### 2. Expression-Based Command Generation

Complex multi-line expressions build complete command strings:
```
BCPCommand = [Full BCP.EXE command with quoted parameters]
SqlCmd_Arg = [Complete SQLCMD.EXE argument string]
```

This approach enables:
- Single point of change for command format
- Runtime evaluation based on loop context
- Complex string manipulation within SSIS
- Debugging via expression preview

### 3. Intelligent Table Classification

SqlCmd package implements business logic in variable expressions:
```
IF LEFT(TableName, 3) = "Dim"
THEN DimTables folder
ELSE FactTables folder
```

Real-world scenarios:
- Dimensions output to analytics folder
- Facts output to reporting folder
- Audit tables to archive folder
- Other tables to default folder

### 4. Foreach Loop Scalability

Single package handles any number of tables:
- Add new tables to schema → automatically included
- No package modification needed
- Linear scalability with table count
- Sequential or parallelizable processing

### 5. CSV Output Optimization

**BCP Approach**:
- Minimal overhead (raw binary operation)
- Comma delimiter (standard CSV)
- Character mode (readable)
- No headers (add in post-processing if needed)

**SqlCmd Approach**:
- Semicolon delimiter (avoids conflict if data contains commas)
- Column headers included (SQL column names as first row)
- `-W` flag removes trailing whitespace
- Wide columns prevent truncation

### 6. Character Encoding & Internationalization

Both packages support UTF-8:
- `-C 65001` (BCP)
- `-f 65001` (SqlCmd)

Handles:
- International character sets
- Multilingual product names
- Regional data formats
- Global business requirements

### 7. Windows Authentication Integration

Both packages use Trusted Connection:
- `-T` flag (BCP)
- `-E` flag (SqlCmd)

Benefits:
- No password storage in package
- Integrated with Active Directory
- Enterprise security standards
- No credentials in deployment

### 8. Error Handling & Logging

Exit code validation enables:
- Success/failure determination
- Error task routing
- Logging to event viewer
- Package failure on critical errors
- Continue-on-error for non-critical tables

---

## Data Export Workflows

### BCP Package Execution Flow

```
INITIALIZATION PHASE:
1. Package starts
2. All configuration variables initialized
   ├─ ServerName = "."
   ├─ DatabaseName = "AdvWrks2022_DWH"
   ├─ CurrentSchema = "prod"
   ├─ DataFolder = "D:\...\Data\"
   └─ TableList = Empty array

DISCOVERY PHASE:
3. Execute SQL Task: "Get Table List"
   └─ Query: SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='prod'
   └─ Result: TableList = [DimAccount, DimCustomer, DimDate, ..., FactInternetSales, FactReseller, ...]

FOREACH LOOP INITIALIZATION:
4. Configure Foreach Loop Enumerator
   ├─ Source: User::TableList array
   ├─ Current item mapping: User::CurrentTable

EXPORT ITERATION - ITERATION 1 (DimCustomer):
5. CurrentTable = "DimCustomer"
6. Evaluate BCPCommand expression:
   BCPCommand = "SELECT * FROM [prod].[DimCustomer]" queryout 
                "D:\...\Data\DimCustomer.csv" 
                -c -t, -r\n -q -S . -d AdvWrks2022_DWH -T -C 65001
7. Execute Process Task:
   └─ Executable: bcp.exe
   └─ Arguments: [BCPCommand value]
   └─ BCP executes:
      ├─ Connects to SQL Server via Windows auth
      ├─ Executes SELECT query
      ├─ Formats result as character CSV (comma-delimited)
      ├─ Writes to file
      └─ Returns exit code (0 = success)
8. Output file created: D:\Data\DimCustomer.csv
9. Continue to next iteration

EXPORT ITERATION - ITERATION N (All remaining tables):
10. Repeat steps 5-8 for each table in TableList
    ├─ DimDate → DimDate.csv
    ├─ DimGeography → DimGeography.csv
    ├─ DimProduct → DimProduct.csv
    ├─ FactInternetSales → FactInternetSales.csv
    ├─ FactReseller → FactReseller.csv
    └─ ... all tables processed

COMPLETION:
11. All tables exported
12. All CSV files in: D:\Data\*.csv
13. Package completes with success status
```

### SqlCmd Package Execution Flow

```
INITIALIZATION PHASE:
1. Package starts
2. Configuration variables initialized
   ├─ _ServerName = "."
   ├─ _DatabaseName = "AdvWrks2022_DWH"
   ├─ _SchemaName = "prod"
   ├─ _DimTables_FolderName = "D:\...\Data\DimTables\"
   ├─ _FactTables_FolderName = "D:\...\Data\FactTables\"
   └─ TableList = Empty array

DISCOVERY PHASE:
3. Execute SQL Task: "Get Table List"
   └─ Query: SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='prod'
   └─ Result: TableList = [DimAccount, DimCustomer, ..., FactInternetSales, ...]

FOREACH LOOP INITIALIZATION:
4. Configure Foreach Loop Enumerator
   ├─ Source: User::TableList array
   ├─ Current item mapping: User::CurrentTable

EXPORT ITERATION - ITERATION 1 (DimCustomer):
5. CurrentTable = "DimCustomer"
6. Evaluate CurrentFile expression:
   LEFT("DimCustomer", 3) == "Dim" ? TRUE
   CurrentFile = "D:\...\Data\DimTables\DimCustomer.csv"
7. Evaluate SqlCmd_loadingScript expression:
   SqlCmd_loadingScript = "SET NOCOUNT ON; SELECT * FROM prod.DimCustomer"
8. Evaluate SqlCmd_Arg expression:
   SqlCmd_Arg = "-S . -d AdvWrks2022_DWH -E -W -s";" -w 65535 
                -Q "SET NOCOUNT ON; SELECT * FROM prod.DimCustomer" 
                -o"D:\Data\DimTables\DimCustomer.csv" -f 65001"
9. Execute Process Task:
   └─ Executable: sqlcmd.exe
   └─ Arguments: [SqlCmd_Arg value]
   └─ SqlCmd executes:
      ├─ Connects to SQL Server via Windows auth (-E)
      ├─ Executes T-SQL query
      ├─ Suppresses row count (SET NOCOUNT ON)
      ├─ Removes trailing spaces (-W)
      ├─ Uses semicolon as separator (-s";")
      ├─ Formats output with wide columns (-w 65535)
      ├─ Redirects to file (-o"...")
      └─ Returns exit code (0 = success)
10. Output file created: D:\Data\DimTables\DimCustomer.csv
    (Note: File includes column headers as first row)

EXPORT ITERATION - ITERATION 2 (FactInternetSales):
11. CurrentTable = "FactInternetSales"
12. Evaluate CurrentFile expression:
    LEFT("FactInternetSales", 3) == "Dim" ? FALSE
    CurrentFile = "D:\...\Data\FactTables\FactInternetSales.csv"
13. Evaluate expressions with new context
14. Execute SqlCmd with FactInternetSales parameters
15. Output file created: D:\Data\FactTables\FactInternetSales.csv

EXPORT ITERATIONS - REMAINING TABLES:
16. Continue iterations for all remaining tables
    ├─ Dimension tables → DimTables folder
    ├─ Fact tables → FactTables folder
    ├─ Includes headers in each file
    ├─ Semicolon-delimited format
    └─ Wide columns for no truncation

COMPLETION:
17. All tables exported with organized folder structure
18. Folder hierarchy:
    D:\Data\
    ├── DimTables\
    │   ├─ DimAccount.csv
    │   ├─ DimCustomer.csv
    │   ├─ DimDate.csv
    │   ├─ DimGeography.csv
    │   ├─ DimProduct.csv
    │   └─ ... other dimension tables
    └── FactTables\
        ├─ FactInternetSales.csv
        ├─ FactReseller.csv
        └─ ... other fact tables
19. Package completes with success status
```

---

## Real-World Use Cases

### Use Case 1: Data Warehouse Distribution

**Scenario**: Export AdventureWorks data for external analytics partner

**Process**:
1. Execute SqlCmd package for organized folder structure
2. Zip DimTables and FactTables folders
3. Deliver to partner via SFTP
4. Partner imports CSV files into their analytics platform

**Advantage**: Partner receives pre-organized, properly formatted data

### Use Case 2: Backup & Archival

**Scenario**: Daily export of critical tables for backup/compliance

**Process**:
1. Schedule BCP package to run nightly
2. Export all tables to timestamped folder
3. Archive CSV files to Blob Storage
4. Maintain 7-year historical archive

**Advantage**: BCP's raw speed ensures fast daily backups

### Use Case 3: BI Tool Integration

**Scenario**: Feed Tableau/Power BI with current database snapshot

**Process**:
1. Execute SqlCmd package
2. Import CSV files to shared drive accessible to BI tools
3. BI tools refresh datasets from CSV sources
4. Dashboards display latest data

**Advantage**: SqlCmd includes headers, making BI import automatic

### Use Case 4: Data Migration

**Scenario**: Migrate from SQL Server to cloud data warehouse (Azure Synapse)

**Process**:
1. Run BCP package to export all tables
2. Upload CSV files to Azure Data Lake
3. Azure Synapse imports CSVs via COPY statement
4. Validation confirms data integrity

**Advantage**: CSV is universal migration format

### Use Case 5: Regulatory Compliance Export

**Scenario**: Export sales data for tax/compliance audit

**Process**:
1. Execute SqlCmd package with specific table filter
2. Generate organized folder structure for auditors
3. Include column headers for clarity
4. Deliver via secure file transfer

**Advantage**: SqlCmd headers eliminate manual documentation

---

## Skills Summary

| Domain | Skills Demonstrated |
|--------|---------------------|
| **SSIS Advanced** | Expression-based variables, dynamic command generation, process execution, foreach loops |
| **Command-Line Tools** | BCP.EXE parameter mastery, SqlCmd.EXE argument construction, exit code handling |
| **Database Extraction** | High-performance bulk export, UTF-8 encoding, authentication integration, delimiter handling |
| **Scripting** | Multi-line expressions, conditional logic, string concatenation, template patterns |
| **SQL Server** | System catalog queries (INFORMATION_SCHEMA), T-SQL execution, schema navigation |
| **Data Integration** | CSV export formats, encoding standards, folder organization, batch processing |
| **Automation** | Parameterized packages, reusable templates, scheduled execution ready |
| **Troubleshooting** | Command-line debugging, variable expression testing, exit code validation |

---

## Conclusion

These SSIS packages represent enterprise-grade database extraction solutions showcasing two powerful approaches to command-line integration. The BCP-based package emphasizes raw performance for large-scale exports, while the SqlCmd-based package demonstrates flexibility and organization for business intelligence workflows. Together, they exemplify mastery of SSIS advanced techniques, command-line tool integration, and data warehousing best practices.

The variable-driven architecture, expression-based command generation, and intelligent table classification demonstrate production-ready engineering suitable for complex enterprise data integration scenarios.

---

**Created**: August 18, 2025  
**SSIS Version**: 17.0.1008.3 (SQL Server 2022)  
**Database**: AdventureWorks 2022 Data Warehouse  
**Development Environment**: SQL Server Data Tools (SSDT)  
**Portfolio Status**: Ready for GitHub publication and professional showcase
