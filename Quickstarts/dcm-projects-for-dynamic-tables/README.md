# Snowflake DCM Projects — Dynamic Tables Lifecycle

A hands-on data engineering project demonstrating how to manage Snowflake Dynamic Tables and their supporting infrastructure using **Declarative Change Management (DCM)**, Infrastructure as Code principles, and the Snowflake CLI.

This project is based on the official [DCM Projects for Dynamic Tables Quickstart](https://www.snowflake.com/en/developers/guides/dcm-projects-for-dynamic-tables/) and was implemented in GitHub Codespaces.

## 1. Project Overview

Managing data pipelines through individual SQL scripts can become challenging as the number of database objects, dependencies, environments, and schema changes increases.

Snowflake DCM addresses this problem by allowing developers to define the desired state of Snowflake infrastructure in version-controlled files.

Instead of manually tracking and executing every database change, developers can use DCM to compare the declared configuration against the deployed environment and apply the required changes.

This project explores:

- Declarative infrastructure management with Snowflake DCM
- Infrastructure as Code and version control
- Dynamic Table creation and lifecycle management
- Data ingestion, transformation, and serving layers
- Jinja templating and reusable SQL macros
- Role-based access control (RBAC)
- Schema evolution using incremental deployments
- Frozen Regions for Dynamic Tables
- AI-based classification using Snowflake Cortex AI

## 2. Technology Stack

| Technology | Purpose |
|---|---|
| Snowflake | Cloud data platform |
| Snowflake DCM | Declarative infrastructure management |
| Snowflake CLI | Planning and deploying DCM projects |
| Dynamic Tables | Declarative data transformations |
| SQL | Data modeling and transformation |
| YAML | Environment and project configuration |
| Jinja | Parameterization and reusable definitions |
| Git & GitHub | Version control and project documentation |
| GitHub Codespaces | Cloud-based development environment |

## 3. Architecture

The project follows a layered data architecture.

```text
                  GitHub Repository
                         |
                         v
                 DCM Manifest (YAML)
                         |
                         v
                DCM SQL Definitions
                         |
                  snow dcm plan
                         |
                         v
                 Review Changes
                         |
                  snow dcm deploy
                         |
                         v
                  Snowflake DEV
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
         RAW         ANALYTICS        SERVE
          |              ^              ^
          |              |              |
          +--------------+              |
                         |              |
                  Dynamic Tables -------+
                         |
                         v
                Analytical Data
```

### RAW Layer

Stores the source data required by the pipeline, including orders, order details, menu items, customers, inventory, and truck information.

### ANALYTICS Layer

Contains Dynamic Tables that transform and enrich the source data.

The `ENRICHED_ORDER_DETAILS` Dynamic Table combines order-related data with additional business attributes.

### SERVE Layer

Exposes analytical views designed for downstream consumption and reporting.

### Infrastructure Management

The project also defines supporting infrastructure, access controls, SQL functions, tasks, and reusable grants through DCM-managed definitions.

## 4. Repository Structure

```text
dcm-projects-for-dynamic-tables/
|
|-- README.md
|
|-- DCM_Projects_DT_Lifecycle/
|   |-- manifest.yml
|   |
|   `-- sources/
|       |-- definitions/
|       |   |-- access.sql
|       |   |-- analytics.sql
|       |   |-- expectations.sql
|       |   |-- ingest.sql
|       |   |-- jinja_demo.sql
|       |   |-- raw.sql
|       |   `-- serve.sql
|       |
|       `-- macros/
|           `-- grants_macro.sql
|
`-- scripts/
    |-- 01_pre_deploy.sql
    |-- 02_post_deploy.sql
    |-- 03_verify_evolution.sql
    `-- 04_cleanup.sql
```

The DCM project definitions describe the desired infrastructure, while the supporting scripts handle preparation, sample data loading, verification, and cleanup.

## 5. Implementation

### Step 1 — Environment Preparation

The project was developed using GitHub Codespaces and the Snowflake CLI.

A Snowflake connection was configured and verified before deployment.

The `DCM_DEVELOPER` role was created and granted the privileges required for the development environment.

The development target in `manifest.yml` was configured for the Snowflake account.

### Step 2 — Initial DCM Plan

The initial DCM plan compared the declared infrastructure with the existing Snowflake environment.

**Plan results:**

| Operation | Count |
|---|---:|
| CREATE | 36 |
| ALTER | 1 |
| DROP | 0 |
| Total | 37 |

The plan was reviewed before deployment.

### Step 3 — Initial Deployment

The DCM project was deployed to the `DCM_DEV` target.

**Deployment results:**

| Operation | Count |
|---|---:|
| Created | 36 |
| Altered | 1 |
| Dropped | 0 |

The deployment provisioned the development database, warehouse, schemas, source tables, Dynamic Tables, views, functions, tasks, and access-control objects.

The primary development resources included:

- Database: `DCM_DEMO_3_DEV`
- Warehouse: `DCM_DEMO_3_WH_DEV`
- Schemas: `RAW`, `ANALYTICS`, `SERVE`

### Step 4 — Sample Data and Pipeline Validation

The post-deployment SQL script was executed to populate the source tables and refresh the Dynamic Tables.

**Sample data inserted:**

| Dataset | Rows |
|---|---:|
| Trucks | 8 |
| Menu items | 14 |
| Customers | 20 |
| Inventory | 20 |
| Order headers | 20 |
| Order details | 30 |
| **Total** | **112** |

Four Dynamic Tables were refreshed.

The enriched order details were successfully queried, confirming that the initial deployment and transformation pipeline were operational.

## 6. Dynamic Table Schema Evolution

After the initial deployment, the `ENRICHED_ORDER_DETAILS` Dynamic Table definition was modified to demonstrate how DCM handles changes to existing infrastructure.

Two modifications were introduced.

### Frozen Region

A Frozen Region condition was added:

```sql
FROZEN WHERE (
    ORDER_TS < CURRENT_TIMESTAMP() - INTERVAL '1 day'
)
```

The intention was to distinguish historical records from records eligible for ongoing refresh processing.

### AI-Based Classification

A new calculated column was introduced:

```sql
AI_CLASSIFY(
    m.MENU_ITEM_NAME,
    ['Healthy', 'Indulgent']
):labels[0]::VARCHAR AS DIET_CLASSIFICATION
```

This expression uses Snowflake Cortex AI to classify menu items into two categories:

- `Healthy`
- `Indulgent`

The classification was added to the Dynamic Table query as the `DIET_CLASSIFICATION` column.

## 7. Incremental Deployment

Following the modifications, a second DCM plan was executed.

DCM detected the changes to the existing Dynamic Table rather than proposing a complete recreation of the infrastructure.

**Second deployment results:**

| Operation | Count |
|---|---:|
| Created | 0 |
| Altered | 1 |
| Dropped | 0 |

The deployment successfully altered:

`DCM_DEMO_3_DEV.ANALYTICS.ENRICHED_ORDER_DETAILS`

The deployment plan included:

- Adding the Frozen Region condition
- Updating the Dynamic Table query
- Introducing the `DIET_CLASSIFICATION` column

This demonstrated DCM's ability to detect and deploy targeted changes to an existing Snowflake object.

## 8. Validation Results and Limitations

The initial infrastructure deployment, sample data insertion, Dynamic Table refreshes, and schema evolution deployment completed successfully.

However, the final evolution verification could not be completed because of a Snowflake Trial account restriction.

During the manual refresh of `ENRICHED_ORDER_DETAILS`, Snowflake returned:

```text
SQL compilation error:
Target table failed to refresh:
AI function AI_CLASSIFY is not available for trial accounts.
```

### What Was Verified

- Initial DCM planning and deployment
- Creation of development infrastructure
- Sample data insertion
- Initial Dynamic Table refreshes
- Querying enriched analytical data
- Detection of schema evolution changes
- Successful deployment of the modified Dynamic Table definition

### What Remains Unverified

- Successful execution of `AI_CLASSIFY`
- Population of `DIET_CLASSIFICATION`
- Runtime behavior of Frozen Regions following the modified query
- Successful refresh of the evolved Dynamic Table

The limitation was caused by account-level availability of the AI function, not by a reported DCM deployment failure.

No substitute classification implementation was introduced, in order to preserve the original tutorial's intended implementation.

## 9. Key Takeaways

This project provided practical experience with declarative infrastructure management in Snowflake.

The main outcomes were:

1. **Infrastructure as Code:** Snowflake objects were managed through version-controlled definitions.
2. **Change Planning:** DCM identified differences between the declared and deployed infrastructure before execution.
3. **Controlled Deployment:** Infrastructure changes were applied through a repeatable deployment workflow.
4. **Dynamic Table Management:** Data transformations were deployed and refreshed as part of a layered analytical architecture.
5. **Schema Evolution:** DCM identified and deployed a targeted alteration to an existing Dynamic Table.
6. **Platform Limitations:** The project demonstrated the importance of distinguishing successful infrastructure deployment from successful runtime execution.

## 10. Project Status

| Component | Status |
|---|---|
| Snowflake CLI configuration | Completed |
| DCM project setup | Completed |
| Initial DCM plan | Completed |
| Initial DCM deployment | Completed |
| Sample data loading | Completed |
| Initial Dynamic Table validation | Completed |
| Schema evolution plan | Completed |
| Schema evolution deployment | Completed |
| Cortex AI classification validation | Blocked by Trial account restriction |
| Frozen Region runtime verification | Not completed |
| Resource cleanup | Deferred |

Snowflake development resources have intentionally been retained for potential further investigation and validation.

## 11. References

- [Official Snowflake DCM Projects for Dynamic Tables Quickstart](https://www.snowflake.com/en/developers/guides/dcm-projects-for-dynamic-tables/)
- [Snowflake DCM Documentation](https://docs.snowflake.com/en/user-guide/dcm-projects/dcm-projects-overview)
- [Snowflake Labs — snowflake-dcm-projects](https://github.com/snowflake-labs/snowflake-dcm-projects)

---

**Project type:** Data Engineering / Infrastructure as Code / Snowflake DCM

**Implementation environment:** GitHub Codespaces

**Origin:** Adapted from the official Snowflake Labs quickstart repository.