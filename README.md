# dbt-snowflake-datops-materilizations
Repo contains the materializations for Data Engineers DataOps Framework

Conatins the following materializations for Snowflake:

* File Format
* Stages
* Stored Procedures
* Tasks
* Streams
* Tables
* Generic

Adds the ability to create the raw tables based on the yml file

## Stored Procedures

Usage

```sql
{{ 
    config(materialized='stored_procedure',
    preferred_language = 'sql',
    override_name = 'SAMPLE_STORE_PROC',
    parameters = 'status varchar',
    return_type = 'NUMBER(38, 0)')
}}
```

| property             | description                                                                         | required | default            |
| -------------------- | ----------------------------------------------------------------------------------- | -------- | ------------------ |
| `materialized`       | specifies the type of materialisation to run                                        | yes      | `stored_procedure` |
| `preferred_language` | describes the language the stored procedure is written in                           | no       | `sql`              |
| `override_name`      | specifies the name of the stored procedure if this is an overrider stored procedure | no       | `model['alias']`   |
| `parameters`         | specifes the parameters that needs to be passed when calling the stored procedure   | no       |                    |
| `return_type`        | specifies the stored procedure return type                                          | no       | `varchar`          |

## Tasks

Usage

```sql
{{ 
    config(materialized='task',
    is_serverless = true,
    warehouse_name_or_size = 'xsmall',
    schedule = 'using cron */2 6-20 * * * Pacific/Auckland',
    stream_name = 'stm_orders',
    is_enabled = false)
 }}
```

| property                 | description                                                                           | required | default  |
| ------------------------ | ------------------------------------------------------------------------------------- | -------- | -------- |
| `materialized`           | specifies the type of materialisation to run                                          | yes      | `task`   |
| `is_serverless`          | specifies if the warehouse should be serverless or dedicated                          | no       | `true`   |
| `warehouse_name_or_size` | specifies the warehouse size if serverless otherwise the name of the warehouse to use | no       | `xsmall` |
| `schedule`               | specifies the schedule which the task should be run on using CRON expressions         | no *     |          |
| `task_after`             | specifies the task which this task should be run after                                | no *     |          |
| `stream_name`            | specifies the stream which the task should run only if there is data available        | no       |          |
| `is_enabled`             | specifies if the task should be enabled or disabled at the end of the run             | yes      | `true`   |

* only one of `schedule` or `task_after` is required.

## Streams

Usage

```sql
{{
    config(materialized='stream',
    source_schema='sales',
    source_model='raw_orders')
}}
```

| property        | description                                                                    | required | default  |
| --------------- | ------------------------------------------------------------------------------ | -------- | -------- |
| `materialized`  | specifies the type of materialisation to run                                   | yes      | `stream` |
| `source_schema` | specifies the source table or view schema if different to the current location | yes      |          |
| `source_model`  | specifies the source table or view model name to add the stream to             | yes      |          |

## Tables

Usage

```yml
    tables:
      - name: raw_customers
        description: Customer Information
        external:
          auto_create_table: true
          auto_create_stream: false
```

| property               | description                                                      | required | default |
| ---------------------- | ---------------------------------------------------------------- | -------- | ------- |
| `auto_create_table`    | specifies if the table should be maintianed by dbt or not        | yes      | `false` |
| `auto_create_stream` * | specifies if the table should have a stream created on it or not | no       | `false` |

* it's recommended that a separate stream object is created instead of setting up the stream via the table object as the stream doesn't appear on the DAG when created via this method, nor can you reference it using the `ref` macro.

## File Formats

Usage

```sql
{{
    config(materialized='file_format')
}}
```

| property       | description                                     | required | default       |
| -------------- | ----------------------------------------------- | -------- | ------------- |
| `materialized` | specifies the type of materialisation to run    | yes      | `file_format` |

View [Snowflake `create file format` documentation](https://docs.snowflake.com/en/sql-reference/sql/create-file-format.html) for more information on the available options.

example

```sql
{{ config(materialized='file_format') }}

    type = json
    null_if = ()
    compression = none
    ignore_utf8_errors = true
```

## Stages

```sql
{{
    config(materialized='stage')
}}
```

| property       | description                                  | required | default |
| -------------- | -------------------------------------------- | -------- | ------- |
| `materialized` | specifies the type of materialisation to run | yes      | `stage` |

View [Snowflake `create stage` documentation](https://docs.snowflake.com/en/sql-reference/sql/create-stage.html) for more information on the available options.

[Storage Integrations](https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration.html) need to be maintained separately as you require `Create integration` privilage on the role you are using to set those up and they are global to snowflake instead of per database.

example

```sql
{{ config(materialized='stage') }}

{% if target.name == 'prod' %}
  url='azure://xxxxxxprod.blob.core.windows.net/external-tables'
{% elif target.name == 'test' %}
  url='azure://xxxxxxtest.blob.core.windows.net/external-tables'
{% elif target.name == 'dev' %}
  url='azure://xxxxxxdev.blob.core.windows.net/external-tables'
{% else %}
  url='azure://xxxxxxsandbox.blob.core.windows.net/external-tables'
{% endif %} 
  storage_integration = DATAOPS_TEMPLATE_EXTERNAL
```

## Generic

Usage

```sql
{{
    config(materialized='generic')
}}
```

| property       | description                                  | required | default   |
| -------------- | -------------------------------------------- | -------- | --------- |
| `materialized` | specifies the type of materialisation to run | yes      | `generic` |

example

```sql
{{ config(materialized='generic') }}

CREATE OR REPLACE api integration SnowWatch_Prod_API
    api_provider = azure_api_management
    azure_tenant_id = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
    azure_ad_application_id = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
    api_allowed_prefixes = ('https://de-snowwatch-prod-ae-apim.azure-api.net')
    API_KEY = 'xxxxxxxxxxxxxxxxxxxx'
    enabled = true; 
```
