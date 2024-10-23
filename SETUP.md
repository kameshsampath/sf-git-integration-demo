# Snowflake Git Integration: 60-Second Guide for Devs ❄️🐙⏱️

A quick demo of [Snowflake's Git integration](https://docs.snowflake.com/en/developer-guide/git/git-setting-up) , version control for your data workflows in under a minute? You bet! ⚡❄️

## Pre-requisites

- Snowflake [Trial Account](https://signup.snowflake.com/)
- [GitHub](https://github.com) Account
- Familiar with Snowflake [SQL Worksheets](https://docs.snowflake.com/en/user-guide/ui-worksheet)

## Setup

For the this [repository](https://github.com/kameshsampath/sf-git-integration-demo.git) to your GitHub account.

### Setup Git Integration

>[!NOTE]
> All the commands in this demo will be executed from SQL worksheet.

### Setup Database/Schema/WareHouse

All objects/resources that we create in Snowflake are confined to a DB. Let us create demo database for this exercise,

```sql
-- Use role that has permissions to create API Integration
USE ROLE ACCOUNTADMIN;

-- Craete and use the warehouse that your session
CREATE WAREHOUSE IF NOT EXISTS KAMESH_DEMOS_S;
USE WAREHOUSE KAMESH_DEMOS_S;

-- Database to hold all the objects
CREATE OR REPLACE DATABASE KAMESH_GIT_REPOS;
-- Use the created database
USE DATABASE KAMESH_GIT_REPOS;

-- Create schema to hold all github repositories
CREATE OR REPLACE SCHEMA GITHUB;
USE SCHEMA GITHUB;
```

### API Integration

To interact with Git we need to create a [API Integration](https://docs.snowflake.com/en/sql-reference/sql/create-api-integration) object. We can specify the prefixes i.e. GitHub organisation or user url.

In the following example we create an integration to my GitHub user url `https://github.com/kameshsampath`. 

> [!IMPORTANT]
> Make use to update `https://github.com/kameshsampath` to your org/user user.

```sql
CREATE API INTEGRATION IF NOT EXISTS  kameshsampath_git
    API_PROVIDER = git_https_api
    -- allowed orgs and repositories
    API_ALLOWED_PREFIXES = ('https://github.com/kameshsampath')
    ENABLED = TRUE;
```

### Create Git Repository

With integration created let us use it to create the Git repository object,

```sql
CREATE GIT REPOSITORY IF NOT EXISTS git_integration_demo
    API_INTEGRATION = kameshsampath_git
    ORIGIN = 'https://github.com/kameshsampath/sf-git-integration-demo.git';
```

> [!IMPORTANT]
> Make use to update `https://github.com/kameshsampath/sf-git-integration-demo.git` to your fork.

If we want to pull from private repository we need to pass the `GIT_CREDENTIALS` parameter. (e.g.) 

Create a secret to hold GitHub PAT,

```sql
CREATE SECRET IF NOT EXISTS my_gh_pat
   TYPE = password
   USERNAME='your gh user'
   PASSWORD='ghp_xxxxxxxx';
```

Then use the `my_gh_pat` to the Git Repository creation command like,

```sql
CREATE GIT REPOSITORY IF NOT EXISTS git_integration_demo
    API_INTEGRATION = your_integration_name
    GIT_CREDENTIALS = my_gh_pat
    ORIGIN = 'some private GH repo';
```

### Run the SQL

Refresh the respository to pull the latest chages or commits,

```sql
ALTER GIT REPOSITORY git_integration_demo FETCH;
```

We can list the files from branches, tags or even commit hashes,

#### From branch

List all files on branch `main`,

```sql
ls @git_integration_demo/branches/main;
```

#### From tags

List all files on tag `v0.0.1`

```sql
ls @git_integration_demo/tags/v0.0.1;
```

#### From commit hashes

List all files on a commit with hash `7729e75`,

```sql
ls @git_integration_demo/commits/7729e75;
```

For this demo we can use the branch `main` and run the file called [demo.sql](./demo.sql) which creates a table named `TODOS`.

```sql
EXECUTE IMMEDIATE FROM @git_integration_demo/branches/main/demo.sql;
```

If all went well you should see the results with rows from `TODOS` table.

## Cleanup 

To clear all the objects created by the demo script run,

```sql
EXECUTE IMMEDIATE FROM @git_integration_demo/branches/main/cleanup.sql;
```