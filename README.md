# Playing with the Cloud Data Stack (dbt + Snowflake + Airflow)

This project, I'm going cloud-native. Builing it to understand how dbt works with a real cloud data warehouse (Snowflake) and how to orchestrate it all with Airflow using Astronomer's Astro CLI (which was cool and powerful and made this part of the project a breeze.)

**The Why:** Less about complex data transformations, more about learning the tooling and getting comfortable with a modern data stack. Thought of it as a "hello world" for cloud data engineering.

## What This Is

A simple dbt project that:
- Connects to Snowflake (cloud data warehouse)
- Transforms data with dbt models
- Tests data quality
- Orchestrates everything with Airflow via Astro CLI

**Not included:** Complex business logic, massive datasets, or production-grade security. This is a learning sandbox with a free snowflake account and their available data sets.

## The Stack

| Tool | What It Does | Why I Used It |
|------|--------------|---------------|
| **Snowflake** | Cloud data warehouse | Industry standard, wanted to learn it |
| **dbt Core** | SQL transformations | Open source version, runs locally |
| **Astronomer Astro CLI** | Airflow development tool | Way easier than setting up Airflow from scratch |

## What I Built

### Snowflake Setup
Created the whole infrastructure in Snowflake:
```sql
-- User for dbt to connect as
-- Role with appropriate permissions
-- Database to hold the data
-- Warehouse (compute) to run queries
-- Schema to organize tables
```

This is the unglamorous setup work that every project needs. Got a good feel for snowflake at a hight level here.

### dbt Models
Created views and tables to transform the data. Nothing fancy - just learning how dbt works:
- Source definitions (pointing to raw data)
- Staging models (cleaning & basic transformations)
- Mart models (business-ready data)

**The Gotcha:** My models kept materializing as views instead of tables even though I set `materialized='table'` in `dbt_project.yml`. Had to explicitly set it per model. Still not sure why - food for thought. Worked as expected in another project..

### Data Tests
Added two types of tests (both in the tests folder and the marts folder):
- **Generic tests** - Built-in dbt tests (not_null, unique, etc)
- **Singular tests** - Custom SQL tests for business logic. Things like is the order date valid / reasonable?

I've worked in tech long enough to not sleep on basic constraints and checks.

### Airflow Orchestration
Used Astro CLI to spin up Airflow locally, then connected it to both dbt and Snowflake. The DAG triggers dbt runs on a schedule.

**Why Airflow for this simple project?** Honestly, probably overkill. But in real projects you'd have dependencies, scheduling, monitoring - so good to learn the setup now.

## How It Works

```
Snowflake (raw data)
    ↓
dbt models run (transform data)
    ↓
dbt tests run (validate quality)
    ↓
Transformed tables land back in Snowflake
    ↓
All orchestrated by Airflow
```

Simple flow, but all the pieces are there.

## Running It Yourself

### Prerequisites
- Snowflake account (free trial works)
- Python 3.8+
- dbt Core installed
- Astro CLI installed
- VS Code (optional but recommended)

### Setup Steps

**1. Snowflake Setup**
```sql
-- Create user, role, warehouse, database, schema
-- Grant appropriate permissions
-- Save your connection details
```

**2. dbt Configuration**
```bash
# Configure profiles.yml with Snowflake credentials
dbt debug  # Test connection
dbt run    # Run models
dbt test   # Run tests
```

**3. Airflow Setup**
```bash
# Initialize Astro project
astro dev init

# Start Airflow
astro dev start

# Access UI at localhost:8080
```

**4. Connect Everything**
- Add Snowflake connection in Airflow UI
- Configure dbt connection
- Trigger the DAG

### Commands I Used

```bash
# dbt commands
dbt run              # Run all models
dbt test             # Run all tests
dbt run --select model_name   # Run specific model
dbt docs generate    # Generate documentation
dbt docs serve       # View docs in browser

# Astro commands
astro dev start      # Start Airflow
astro dev stop       # Stop Airflow
astro dev restart    # Restart (when you change code)
astro dev logs       # View logs
```

## Things I Learned

### 1. Snowflake's User Creation

Snowflake's security model felt weird at first. You don't just create a user - you create:
- User
- Role (with specific grants)
- Warehouse (compute layer)
- Database + Schema (storage layer)

Then you grant the role to the user, grant permissions to the role, and set default warehouse/database/schema.

### 2. dbt Materialization Config Is Finicky

Set `materialized='table'` in `dbt_project.yml` but models kept creating as views. Had to add:

```sql
{{ config(materialized='table') }}
```

...at the top of each model file.

**The Lesson:** Model-level config overrides project-level. Or maybe I messed up the YAML indentation. Either way, explicit is better than implicit.

### 3. Astro CLI vs. Raw Airflow Is Night and Day

Raw Airflow setup involves:
- Installing packages
- Configuring databases
- Setting up executors
- Managing environment variables
- Debugging cryptic errors

Astro CLI:
```bash
astro dev init
astro dev start
```

Done. It just works.

**The Lesson:** Use the right tool for the job. Astro is made for local Airflow development. Don't fight it.

### 4. Airflow UI Changed Since The Tutorial

The video showed one version of the Airflow UI, mine looked different. Specifically:
- Snowflake connection setup had different fields
- UI navigation was reorganized
- Some buttons moved around

Spent way too long troubleshooting what I thought were connection errors but was actually just "where did they move this field?"

**The Lesson:** Airflow 2.x had major UI changes. If following old tutorials, expect UI differences.

### 5. dbt + Cloud Warehouse = Really Fast

On PostgreSQL (another project), dbt runs took seconds. On Snowflake? Milliseconds.

Why? Snowflake warehouses are pre-warmed compute clusters. No cold starts.

**The Lesson:** Cloud data warehouses are fast because they're always-on and massively parallel. It costs money, but it's fast.

## What I'd Do Differently

### If I Were Starting Over

**More Realistic Data**
Used Snowflake's sample data (TPCH dataset). It's fine for learning but doesn't reflect real business problems. Would've been cooler to use real-ish data (like Faker to generate fake customer data).

**Incremental Models**
Everything rebuilds every time. In production you'd use incremental models to only process new/changed data.

**Better Tests**
My tests are basic (not null, unique), pluse relationship tests and accepted value tests. Real projects need:
- Custom business logic tests
- Data freshness tests

### What I'd Add Next

**Short Term:**
- More models (show multi-layer transformations)
- dbt snapshots (track changes over time)
- Source freshness checks
- Better documentation with descriptions

**Medium Term:**
- Cost monitoring dashboards
- Alert on failed tests

**Long Term:**
- Multi-environment setup (dev/staging/prod)
- dbt mesh architecture (multiple projects)
- Reverse ETL back to operational systems

## Project Structure

```
intro-dbt-snowflake-airflow-project/
├── dags/                    # Airflow DAGs
│   └── dbt_snowflake_dag.py
├── dbt_project.yml          # dbt configuration
├── models/
│   ├── staging/            # Raw → clean
│   └── marts/              # Clean → business ready
├── tests/
│   └── [custom tests]
├── profiles.yml            # Connection to Snowflake
└── packages.yml            # dbt packages
```

## Why This Project Matters

This isn't about building something complex. It's about understanding the toolchain:

✅ **Snowflake** - How cloud data warehouses work  
✅ **dbt** - How to structure transformations  
✅ **Airflow** - How to orchestrate workflows  
✅ **Astro CLI** - How to develop locally  

These are the tools every modern data team uses. This project is me building muscle memory with them.

## Differences from other projects

| Aspect | Other Projects | Snowflake Project |
|--------|-------------------|-------------------|
| **Database** | Self-hosted PostgreSQL | Cloud Snowflake |
| **Cost** | Free (local Docker) | Free trial ends in a month |
| **Performance** | Seconds | Milliseconds |
| **Setup Complexity** | Docker networking hell | Create account, done |
| **Airflow Setup** | Docker Compose | Astro CLI |
| **Learning Focus** | ELT flow, Docker, orchestration | Cloud tooling, dbt best practices |

Both projects taught me different things. PostgreSQL one was about understanding the fundamentals. This one was about using production tools.

## Resources That Helped

- [Tutorial Video](https://www.youtube.com/watch?v=OLXkGB7krGo) - Original walkthrough
- [dbt Documentation](https://docs.getdbt.com/) - Actually really good docs
- [Astronomer Docs](https://docs.astronomer.io/) - For Astro CLI setup
- [Snowflake Docs](https://docs.snowflake.com/) - Especially RBAC section
- Random Stack Overflow posts
- Claude, to help me summarize and cement learnings

## Final Thoughts

This project was less about "look at this complex thing I built" and more about "look at these tools I learned."

The code itself is simple. The value is in:
- Understanding how dbt works with cloud warehouses
- Learning Snowflake's architecture
- Getting comfortable with Astro CLI
- Building end-to-end even when it's simple

Still just trying to understand how the pieces fit together, adn get some muscle memory built up.


*Built while figuring out why everything was a view when I explicitly said table. Still not 100% sure why.*

### my notes from running through the project, stream-of-consciousness style
Intro to some DE tooling.
Referenced https://www.youtube.com/watch?v=OLXkGB7krGo
Need to figure out why my models didn't materialize as tables, only views, as expected from the config file - I had to expressly tell them per model.

Used Snowflake data, set up a user and role and db and wh and schema.
Installed and used dbt core w/ vs code to create views, and transform data with models, and add some generalized tests and singular tests.
Ran, populated the views and tables within Snowflake.

Installed Astro, used it to spin up Airflow.
Connected it to dbt project and Snowflake account.
Tasks ran successfully when I triggered the dag - after a lot of troublshooting w/ different UI than the video showed re: snowflake connection.

Not complicated, focused on tooling here vs. what we did in the SQL scripts and tranformations.
Not sure why we'd use Airflow on this, or if we'd need to just pull changes to the data, whatever - not focused on use case, just getting some familiarity.
