
# Snowflake DCM Projects — Infrastructure as Code

## Project Overview

This project demonstrates how to manage Snowflake infrastructure declaratively using **Snowflake Declarative Change Management (DCM) Projects**.

Based on the official Snowflake Quickstart, the implementation covers infrastructure provisioning, Jinja templating, role-based access control (RBAC), change planning, deployment, and configuration verification.

## Technologies

- Snowflake
- Snowflake CLI
- Declarative Change Management (DCM)
- SQL
- Jinja Templating
- Streamlit
- GitHub Codespaces
- Git

## Architecture

The DCM project manages the following Snowflake resources:

| Component | Implementation |
|---|---|
| Database | `DCM_DEMO_1_DEV` |
| Warehouse | `DCM_DEMO_1_WH_DEV` |
| Raw Layer | `RAW` schema |
| Analytics Layer | `ANALYTICS` schema |
| Serving Layer | `SERVE` schema |
| Dynamic Table | `ENRICHED_ORDER_DETAILS` |
| Semantic View | `ORDER_ANALYTICS` |
| Streamlit Application | `ORDERS_DASHBOARD` |
| Team Schemas | `DEV_TEAM_1`, `DEV_TEAM_2` |
| Access Control | Snowflake RBAC |

## Implementation

### 1. Environment Setup

Configured Snowflake CLI connectivity and initialized the DCM project environment using the provided setup SQL script.

### 2. Infrastructure Deployment

Used `snow dcm plan` to preview infrastructure changes and `snow dcm deploy` to provision the required Snowflake resources.

The initial deployment created **17 entities** and altered **1 existing entity**.

### 3. Data Loading and Validation

Loaded sample data into the RAW tables:

| Table | Inserted Rows |
|---|---:|
| MENU | 14 |
| ORDER_HEADER | 10 |
| ORDER_DETAIL | 20 |

Refreshed the dynamic table and validated the resulting analytical data through SQL queries.

### 4. Declarative Infrastructure Changes

Modified the DCM manifest to resize the warehouse from `X-SMALL` to `SMALL`.

DCM detected the configuration difference and generated a targeted `ALTER WAREHOUSE` operation.

### 5. Jinja-Based Team Provisioning

Enabled a second development team through the manifest's Jinja configuration.

The deployment created:

- `DEV_TEAM_2` schema
- `DEV_TEAM_2_OWNER_DEV` role
- `DEV_TEAM_2_USAGE_DEV` role

It also updated the relevant role grants.

### 6. Configuration Verification

Executed a final DCM plan after deployment.

**Result:**

```text
No changes detected.
```

This confirmed that DCM detected no remaining differences between the declared configuration and managed Snowflake resources.

## Key Learnings

- Declarative infrastructure management
- Infrastructure as Code (IaC) principles
- Configuration drift detection
- Idempotent deployment workflows
- Jinja-based infrastructure generation
- Snowflake role-based access control (RBAC)
- Incremental infrastructure updates
- PLAN → DEPLOY → VERIFY workflows

## Cleanup

The original Quickstart includes a cleanup script:

`Quickstarts/get-started-snowflake-dcm-projects/scripts/03_cleanup.sql`

Review its contents and dependencies before execution.

## References

- [Official Snowflake Quickstart](https://www.snowflake.com/en/developers/guides/get-started-snowflake-dcm-projects/)
- [Snowflake DCM Documentation](https://docs.snowflake.com/en/user-guide/dcm-projects/dcm-projects-overview)
