---
title: 'Snowflake Data Connector'
sidebar_label: 'Snowflake Data Conector'
description: 'Snowflake Data Conector Documentation'
pagination_prev: null
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

The Snowflake Data Connector enables federated SQL queries across datasets in the [Snowflake Cloud Data Warehouse](https://www.snowflake.com/).

```yaml
datasets:
  - from: snowflake:database.schema.table
    name: table
    params: 
      snowflake_warehouse: COMPUTE_WH
      snowflake_role: accountadmin
```

### Parameters
- `from`: a Snowflake fully qualified table name (database.schema.table). For instance `snowflake:snowflake_sample_data.tpch_sf1.lineitem` or `snowflake:TAXI_DATA."2024".TAXI_TRIPS`
- `snowflake_warehouse`: optional, specifies the [Snowflake Warehouse](https://docs.snowflake.com/en/user-guide/warehouses-tasks) to use
- `snowflake_role`: optional, specifies the role to use for accessing Snowflake data

### Auth

The connector supports Snowflake basic authentication (username and password) that must be configured using `spice login snowflake` or using [Secrets Stores](/secret-stores). Login requires the account identifier ('orgname-account_name' format) - use [Finding the organization and account name for an account](https://docs.snowflake.com/en/user-guide/admin-account-identifier#finding-the-organization-and-account-name-for-an-account) instructions.

<img width="800" src="/img/snowflake/ui-snowsight-account-identifier.png" />

<Tabs>
  <TabItem value="local" label="Local" default>
    ```bash
    spice login snowflake -a <account-identifier> -u <username> -p <password>
    ```

    Learn more about [File Secret Store](/secret-stores/file).
  </TabItem>
  <TabItem value="env" label="Env">
    ```bash
    SPICE_SECRET_SNOWFLAKE_ACCOUNT=<account-identifier> \
    SPICE_SECRET_SNOWFLAKE_USERNAME=<username> \
    SPICE_SECRET_SNOWFLAKE_PASSWORD=<password> \
    spice run
    ```

    `spicepod.yaml`
    ```yaml
    version: v1beta1
    kind: Spicepod
    name: spice-app

    secrets:
      store: env
    
    # <...>
    ```

    Learn more about [Env Secret Store](/secret-stores/env).
  </TabItem>
  <TabItem value="k8s" label="Kubernetes">
    ```bash
    kubectl create secret generic snowflake \
      --from-literal=account='<account-identifier>' \
      --from-literal=username='<username>' \
      --from-literal=password='<password>'
    ```

    `spicepod.yaml`
    ```yaml
    version: v1beta1
    kind: Spicepod
    name: spice-app

    secrets:
      store: kubernetes
    
    # <...>
    ```

    Learn more about [Kubernetes Secret Store](/secret-stores/kubernetes).
  </TabItem>
  <TabItem value="keyring" label="Keyring">
    Add new keychain entry (macOS), with user and password in JSON string

    ```bash
    security add-generic-password -l "Snowflake Secret" \
    -a spiced -s spice_secret_snowflake\
    -w $(echo -n '{"account":"<account-identifier>", "username": "<username>", "password": "<password>"}')
    ```

    `spicepod.yaml`
    ```yaml
    version: v1beta1
    kind: Spicepod
    name: spice-app

    secrets:
      store: keyring
    
    # <...>
    ```

    Learn more about [Keyring Secret Store](/secret-stores/keyring).
  </TabItem>
</Tabs>

## Example

```yaml
datasets:
  - from: snowflake:snowflake_sample_data.tpch_sf1.lineitem
    name: lineitem
    params: 
      snowflake_warehouse: COMPUTE_WH
      snowflake_role: accountadmin
```

## Limitations
1. Account identifier does not support the [Legacy account locator in a region format](https://docs.snowflake.com/en/user-guide/admin-account-identifier#format-2-legacy-account-locator-in-a-region). Use [Snowflake preferred name in organization format](https://docs.snowflake.com/en/user-guide/admin-account-identifier#format-1-preferred-account-name-in-your-organization).
1. Authentication: Only Snowflake basic authentication is supported at this time.