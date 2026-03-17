**Prominent ETL Tools**
- Oracle Warehouse Builder (OWB)
- SAP Data Services
- IBM Inforshpere Information Server
- SAS Data Management
- Informatica Powercenter
- Elixir Repertoire for Data ETL
- Data Migrator (IBI)
- SQL Server Integration Services (SSIS)

**Informatica Powercenter**
- Informatica is a software development comapny, which offers data integration products.
- It offers products for ETL, data virtualization, data masking, data quality, data replica, master data management, etc.

**Why Informatica**
- When we have a data system available at the backend where we want to perform certain operations on the data.
- The operations may include data cleaning, data modification, etc. based on certain set of rules or simply loading a bulk set of data from one system to another.

**Informatica Datawarehousing**
- Data warehousing is the process of constructing and using a data warehouse.
- A data warehouse is constructed by integrating data from multiple heterogeneous sources that support analytical reporting, structured and/or ad hoc queries, and decision making.
- Data warehousing involves data cleaning, data integration, and data consolidations.

> [!Important]
> Include the data warehousing diag

**Data Mart**
- A data mart is the access layer of the data warehouse environment that is used to get data out to the users.
- It is a subset of the data warehouse and is usually oriented to a specific business line or team.

# `PowerCenter` Components
## Powercenter Server-Side Components (The Engine)
These are the backend services that actually run and manage ETL jobs.
The Server-side contains the following 4 components:
### Repository Service
- Manages metadata repository (stores source definitions, target definitions, mappings, and workflows)
- Provides access to metadata for client tools and other services.
- Enusres consistency and version control of ETL objects.
### Integration Service
- Executes workflows and sessions.
- Responsible for data extraction, transformation, and loading (ETL).
- Moves data from source systems to target systems according to mappings.
### Domain
- It is the top-level container in the `PowerCenter` architecture.
- Groups nodes and services together for management.
- Controlled via the Admin Console.
### Node
- A physical server within the domain.
- Runs services like Repository Service and Integration Service.
- Multiple nodes can be configured for scalability and load balancing.

## Powercenter Client-Side Components (The Workbench)
These are the frontend tools used by the developers and administrators to design, manage, and monitor ETL processes.
The below 4 components together make a repository service. If for any reason the repository service is not runnig, the data can not be loaded to the target.
### Powercenter Designer
- Mappings (data flow logic) are created here.
- Defines how data moves, source -> transformations -> target.
- Includes transformations like filter, joiner, aggregator, lookup, etc.
### Powercenter Workflow Manager
- Used to create workflows (execution logic).
- Defines tasks such as sessions, commands, and events.
- Controls the order and dependencies of ETL jobs.
### Powercenter Workflow Monitor
- Used to monitor execution of workflows and sessions.
- Provides logs, statistics and performance matrix.
- Helps in debugging and performance tuning.
### Powercenter Repository Manager
- Used to manage the repository (metadata storage).
- Handles user permissions, object management, and versioning.
- Ensures secure access to ETL objects.

## Data Flow Between Components Visualized
```text
[ Client Tools ]
   Designer → Build mappings
   Workflow Manager → Create workflows
   Repository Manager → Manage metadata
   Workflow Monitor → Track execution
        |
        v
[ Repository Service ] → Stores metadata
        |
        v
[ Integration Service ] → Executes workflows/mappings
        |
        v
[ Domain & Nodes ] → Provide infrastructure and scalability
        |
        v
[ Source Systems → Target Systems ]
```
