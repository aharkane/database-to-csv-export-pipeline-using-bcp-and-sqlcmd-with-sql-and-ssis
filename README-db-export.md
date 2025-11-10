# SSIS Database Export Pipeline

SSIS implementation for automated database-to-CSV export using BCP and SqlCmd utilities.

## Overview

Automated data warehouse export solution with dynamic table processing and intelligent folder routing. Features two approaches using BCP (Bulk Copy Program) and SqlCmd utilities with variable-driven command construction.

## Architecture

**Dual-Package Strategy:**

```
Package 1: BCP Bulk Copy
Tables → Execute Process (BCP) → CSV Output

Package 2: SqlCmd with Smart Routing
Tables → Execute Process (SqlCmd) → DimTables/ or FactTables/
```

## Key Features

- Dynamic table enumeration from INFORMATION_SCHEMA
- Variable-driven command construction
- Intelligent folder routing (Dim vs Fact tables)
- Foreach Loop Container for sequential processing
- Expression-based parameter building

## Technology Stack

| Component | Technology |
|-----------|------------|
| **Platform** | SQL Server Integration Services |
| **Utilities** | BCP.EXE, SQLCMD.EXE |
| **Source** | AdventureWorks DWH |
| **Output** | CSV with UTF-8 encoding |

## Processing Approaches

### Package 1: BCP Bulk Export
- High-performance bulk copy operations
- Character mode for readable CSV
- Single directory output
- Optimized for large datasets

### Package 2: SqlCmd Flexible Export
- Smart routing: Dimensions → DimTables/, Facts → FactTables/
- Column headers included
- Wide column support (prevents truncation)
- Query-based extraction flexibility

## Skills Demonstrated

<table>
<tr>
<td width="55%" valign="top">

**SSIS Development**
- Execute Process Task
- Foreach Loop Container
- Variable and expression design
- Control flow orchestration

**Command-Line Integration**
- BCP utility operations
- SqlCmd automation
- Parameter construction
- Return code validation

</td>
<td width="45%" valign="top">

**Dynamic Processing**
- Metadata-driven enumeration
- Variable-based commands
- Expression evaluation
- Template-based design

**File Management**
- Dynamic path construction
- Folder organization logic
- Conditional routing
- File validation

</td>
</tr>
</table>