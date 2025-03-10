# Updating DataHub

This file documents any backwards-incompatible changes in DataHub and assists people when migrating to a new version.

## Next

### Breaking Changes

### Potential Downtime

### Deprecations

### Other notable Changes

## 0.10.4

### Breaking Changes

### Potential Downtime

### Deprecations
- #8045: With the introduction of custom ownership types, the `Owner` aspect has been updated where the `type` field is deprecated in favor of a new field `typeUrn`. This latter field is an urn reference to the new OwnershipType entity. GraphQL endpoints have been updated to use the new field. For pre-existing ownership aspect records, DataHub now has logic to map the old field to the new field.

### Other notable Changes
- #8191: Updates GMS's health check endpoint to account for its dependency on external components. Notably, at this time, elasticsearch. This means that DataHub operators can now use GMS health status more reliably.

## 0.10.3

### Breaking Changes

- #7900: The `catalog_pattern` and `schema_pattern` options of the Unity Catalog source now match against the fully qualified name of the catalog/schema instead of just the name. Unless you're using regex `^` in your patterns, this should not affect you.
- #7942: Renaming the `containerPath` aspect to `browsePathsV2`. This means any data with the aspect name `containerPath` will be invalid. We had not exposed this in the UI or used it anywhere, but it was a model we recently merged to open up other work. This should not affect many people if anyone at all unless you were manually creating `containerPath` data through ingestion on your instance.
- #8068: In the `datahub delete` CLI, if an `--entity-type` filter is not specified, we automatically delete across all entity types. The previous behavior was to use a default entity type of dataset.
- #8068: In the `datahub delete` CLI, the `--start-time` and `--end-time` parameters are not required for timeseries aspect hard deletes. To recover the previous behavior of deleting all data, use `--start-time min --end-time max`.

### Potential Downtime

### Deprecations
- The signature of `Source.get_workunits()` is changed from `Iterable[WorkUnit]` to the more restrictive `Iterable[MetadataWorkUnit]`.
- Legacy usage creation via the `UsageAggregation` aspect, `/usageStats?action=batchIngest` GMS endpoint, and `UsageStatsWorkUnit` metadata-ingestion class are all deprecated.

### Other notable Changes

## 0.10.2

### Breaking Changes

- #7016 Add `add_database_name_to_urn` flag to Oracle source which ensure that Dataset urns have the DB name as a prefix to prevent collision (.e.g. {database}.{schema}.{table}). ONLY breaking if you set this flag to true, otherwise behavior remains the same.
- The Airflow plugin no longer includes the DataHub Kafka emitter by default. Use `pip install acryl-datahub-airflow-plugin[datahub-kafka]` for Kafka support.
- The Airflow lineage backend no longer includes the DataHub Kafka emitter by default. Use `pip install acryl-datahub[airflow,datahub-kafka]` for Kafka support.
- Java SDK PatchBuilders have been modified in a backwards incompatible way to align more with the Python SDK and support more use cases. Any application utilizing the Java SDK for patch building may be affected on upgrading this dependency.

### Deprecations

- The docker image and script for updating from Elasticsearch 6 to 7 is no longer being maintained and will be removed from the `/contrib` section of
  the repository. Please refer to older releases if needed.

## 0.10.0

### Breaking Changes

- #7103 This should only impact users who have configured explicit non-default names for DataHub's Kafka topics. The environment variables used to configure Kafka topics for DataHub used in the `kafka-setup` docker image have been updated to be in-line with other DataHub components, for more info see our docs on [Configuring Kafka in DataHub
  ](https://datahubproject.io/docs/how/kafka-config). They have been suffixed with `_TOPIC` where as now the correct suffix is `_TOPIC_NAME`. This change should not affect any user who is using default Kafka names.
- #6906 The Redshift source has been reworked and now also includes usage capabilities. The old Redshift source was renamed to `redshift-legacy`. The `redshift-usage` source has also been renamed to `redshift-usage-legacy` will be removed in the future.

### Potential Downtime

- #6894 Search improvements requires reindexing indices. A `system-update` job will run which will set indices to read-only and create a backup/clone of each index. During the reindexing new components will be prevented from start-up until the reindex completes. The logs of this job will indicate a % complete per index. Depending on index sizes and infrastructure this process can take 5 minutes to hours however as a rough estimate 1 hour for every 2.3 million entities.

#### Helm Notes

Helm without `--atomic`: The default timeout for an upgrade command is 5 minutes. If the reindex takes longer (depending on data size) it will continue to run in the background even though helm will report a failure. Allow this job to finish and then re-run the helm upgrade command.

Helm with `--atomic`: In general, it is recommended to not use the `--atomic` setting for this particular upgrade since the system update job will be terminated before completion. If `--atomic` is preferred, then increase the timeout using the `--timeout` flag to account for the reindexing time (see note above for estimating this value).

### Deprecations

## 0.9.6

### Breaking Changes

- #6742 The metadata file sink's output format no longer contains nested JSON strings for MCP aspects, but instead unpacks the stringified JSON into a real JSON object. The previous sink behavior can be recovered using the `legacy_nested_json_string` option. The file source is backwards compatible and supports both formats.
- #6901 The `env` and `database_alias` fields have been marked deprecated across all sources. We recommend using `platform_instance` where possible instead.

### Potential Downtime

### Deprecations

- #6851 - Sources bigquery-legacy and bigquery-usage-legacy have been removed

### Other notable Changes

- If anyone faces issues with login please clear your cookies. Some security updates are part of this release. That may cause login issues until cookies are cleared.

## 0.9.4 / 0.9.5

### Breaking Changes

- #6243 apache-ranger authorizer is no longer the core part of DataHub GMS, and it is shifted as plugin. Please refer updated documentation [Configuring Authorization with Apache Ranger](./configuring-authorization-with-apache-ranger.md#configuring-your-datahub-deployment) for configuring `apache-ranger-plugin` in DataHub GMS.
- #6243 apache-ranger authorizer as plugin is not supported in DataHub Kubernetes deployment.
- #6243 Authentication and Authorization plugins configuration are removed from [application.yml](../../metadata-service/factories/src/main/resources/application.yml). Refer documentation [Migration Of Plugins From application.yml](../plugins.md#migration-of-plugins-from-applicationyml) for migrating any existing custom plugins.
- `datahub check graph-consistency` command has been removed. It was a beta API that we had considered but decided there are better solutions for this. So removing this.
- `graphql_url` option of `powerbi-report-server` source deprecated as the options is not used.
- #6789 BigQuery ingestion: If `enable_legacy_sharded_table_support` is set to False, sharded table names will be suffixed with \_yyyymmdd to make sure they don't clash with non-sharded tables. This means if stateful ingestion is enabled then old sharded tables will be recreated with a new id and attached tags/glossary terms/etc will need to be added again. _This behavior is not enabled by default yet, but will be enabled by default in a future release._

### Potential Downtime

### Deprecations

### Other notable Changes

- #6611 - Snowflake `schema_pattern` now accepts pattern for fully qualified schema name in format `<catalog_name>.<schema_name>` by setting config `match_fully_qualified_names : True`. Current default `match_fully_qualified_names: False` is only to maintain backward compatibility. The config option `match_fully_qualified_names` will be deprecated in future and the default behavior will assume `match_fully_qualified_names: True`."
- #6636 - Sources `snowflake-legacy` and `snowflake-usage-legacy` have been removed.

## 0.9.3

### Breaking Changes

- The beta `datahub check graph-consistency` command has been removed.

### Potential Downtime

### Deprecations

- PowerBI source: `workspace_id_pattern` is introduced in place of `workspace_id`. `workspace_id` is now deprecated and set for removal in a future version.

### Other notable Changes

## 0.9.2

- LookML source will only emit views that are reachable from explores while scanning your git repo. Previous behavior can be achieved by setting `emit_reachable_views_only` to False.
- LookML source will always lowercase urns for lineage edges from views to upstream tables. There is no fallback provided to previous behavior because it was inconsistent in application of lower-casing earlier.
- dbt config `node_type_pattern` which was previously deprecated has been removed. Use `entities_enabled` instead to control whether to emit metadata for sources, models, seeds, tests, etc.
- The dbt source will always lowercase urns for lineage edges to the underlying data platform.
- The DataHub Airflow lineage backend and plugin no longer support Airflow 1.x. You can still run DataHub ingestion in Airflow 1.x using the [PythonVirtualenvOperator](https://airflow.apache.org/docs/apache-airflow/1.10.15/_api/airflow/operators/python_operator/index.html?highlight=pythonvirtualenvoperator#airflow.operators.python_operator.PythonVirtualenvOperator).

### Breaking Changes

- #6570 `snowflake` connector now populates created and last modified timestamps for snowflake datasets and containers. This version of snowflake connector will not work with **datahub-gms** version older than `v0.9.3`

### Potential Downtime

### Deprecations

### Other notable Changes

## 0.9.1

### Breaking Changes

- We have promoted `bigquery-beta` to `bigquery`. If you are using `bigquery-beta` then change your recipes to use the type `bigquery`.

### Potential Downtime

### Deprecations

### Other notable Changes

## 0.9.0

### Breaking Changes

- Java version 11 or greater is required.
- For any of the GraphQL search queries, the input no longer supports value but instead now accepts a list of values. These values represent an OR relationship where the field value must match any of the values.

### Potential Downtime

### Deprecations

### Other notable Changes

## `v0.8.45`

### Breaking Changes

- The `getNativeUserInviteToken` and `createNativeUserInviteToken` GraphQL endpoints have been renamed to
  `getInviteToken` and `createInviteToken` respectively. Additionally, both now accept an optional `roleUrn` parameter.
  Both endpoints also now require the `MANAGE_POLICIES` privilege to execute, rather than `MANAGE_USER_CREDENTIALS`
  privilege.
- One of the default policies shipped with DataHub (`urn:li:dataHubPolicy:7`, or `All Users - All Platform Privileges`)
  has been edited to no longer include `MANAGE_POLICIES`. Its name has consequently been changed to
  `All Users - All Platform Privileges (EXCEPT MANAGE POLICIES)`. This change was made to prevent all users from
  effectively acting as superusers by default.

### Potential Downtime

### Deprecations

### Other notable Changes

## `v0.8.44`

### Breaking Changes

- Browse Paths have been upgraded to a new format to align more closely with the intention of the feature.
  Learn more about the changes, including steps on upgrading, here: <https://datahubproject.io/docs/advanced/browse-paths-upgrade>
- The dbt ingestion source's `disable_dbt_node_creation` and `load_schema` options have been removed. They were no longer necessary due to the recently added sibling entities functionality.
- The `snowflake` source now uses newer faster implementation (earlier `snowflake-beta`). Config properties `provision_role` and `check_role_grants` are not supported. Older `snowflake` and `snowflake-usage` are available as `snowflake-legacy` and `snowflake-usage-legacy` sources respectively.

### Potential Downtime

- [Helm] If you're using Helm, please ensure that your version of the `datahub-actions` container is bumped to `v0.0.7` or `head`.
  This version contains changes to support running ingestion in debug mode. Previous versions are not compatible with this release.
  Upgrading to helm chart version `0.2.103` will ensure that you have the compatible versions by default.

### Deprecations

### Other notable Changes

## `v0.8.42`

### Breaking Changes

- Python 3.6 is no longer supported for metadata ingestion
- #5451 `GMS_HOST` and `GMS_PORT` environment variables deprecated in `v0.8.39` have been removed. Use `DATAHUB_GMS_HOST` and `DATAHUB_GMS_PORT` instead.
- #5478 DataHub CLI `delete` command when used with `--hard` option will delete soft-deleted entities which match the other filters given.
- #5471 Looker now populates `userEmail` in dashboard user usage stats. This version of looker connnector will not work with older version of **datahub-gms** if you have `extract_usage_history` looker config enabled.
- #5529 - `ANALYTICS_ENABLED` environment variable in **datahub-gms** is now deprecated. Use `DATAHUB_ANALYTICS_ENABLED` instead.
- #5485 `--include-removed` option was removed from delete CLI

### Potential Downtime

### Deprecations

### Other notable Changes

## `v0.8.41`

### Breaking Changes

- The `should_overwrite` flag in `csv-enricher` has been replaced with `write_semantics` to match the format used for other sources. See the [documentation](https://datahubproject.io/docs/generated/ingestion/sources/csv/) for more details
- Closing an authorization hole in creating tags adding a Platform Privilege called `Create Tags` for creating tags. This is assigned to `datahub` root user, along
  with default All Users policy. Notice: You may need to add this privilege (or `Manage Tags`) to existing users that need the ability to create tags on the platform.
- #5329 Below profiling config parameters are now supported in `BigQuery`:

  - profiling.profile_if_updated_since_days (default=1)
  - profiling.profile_table_size_limit (default=1GB)
  - profiling.profile_table_row_limit (default=50000)

  Set above parameters to `null` if you want older behaviour.

### Potential Downtime

### Deprecations

### Other notable Changes

## `v0.8.40`

### Breaking Changes

- #5240 `lineage_client_project_id` in `bigquery` source is removed. Use `storage_project_id` instead.

### Potential Downtime

### Deprecations

### Other notable Changes

## `v0.8.39`

### Breaking Changes

- Refactored the `health` field of the `Dataset` GraphQL Type to be of type **list of HealthStatus** (was type **HealthStatus**). See [this PR](https://github.com/datahub-project/datahub/pull/5222/files) for more details.

### Potential Downtime

### Deprecations

- #4875 Lookml view file contents will no longer be populated in custom_properties, instead view definitions will be always available in the View Definitions tab.
- #5208 `GMS_HOST` and `GMS_PORT` environment variables being set in various containers are deprecated in favour of `DATAHUB_GMS_HOST` and `DATAHUB_GMS_PORT`.
- `KAFKA_TOPIC_NAME` environment variable in **datahub-mae-consumer** and **datahub-gms** is now deprecated. Use `METADATA_AUDIT_EVENT_NAME` instead.
- `KAFKA_MCE_TOPIC_NAME` environment variable in **datahub-mce-consumer** and **datahub-gms** is now deprecated. Use `METADATA_CHANGE_EVENT_NAME` instead.
- `KAFKA_FMCE_TOPIC_NAME` environment variable in **datahub-mce-consumer** and **datahub-gms** is now deprecated. Use `FAILED_METADATA_CHANGE_EVENT_NAME` instead.

### Other notable Changes

- #5132 Profile tables in `snowflake` source only if they have been updated since configured (default: `1`) number of day(s). Update the config `profiling.profile_if_updated_since_days` as per your profiling schedule or set it to `None` if you want older behaviour.

## `v0.8.38`

### Breaking Changes

### Potential Downtime

### Deprecations

### Other notable Changes

- Create & Revoke Access Tokens via the UI
- Create and Manage new users via the UI
- Improvements to Business Glossary UI
- FIX - Do not require reindexing to migrate to using the UI business glossary

## `v0.8.36`

### Breaking Changes

- In this release we introduce a brand new Business Glossary experience. With this new experience comes some new ways of indexing data in order to make viewing and traversing the different levels of your Glossary possible. Therefore, you will have to [restore your indices](https://datahubproject.io/docs/how/restore-indices/) in order for the new Glossary experience to work for users that already have existing Glossaries. If this is your first time using DataHub Glossaries, you're all set!

### Potential Downtime

### Deprecations

### Other notable Changes

- #4961 Dropped profiling is not reported by default as that caused a lot of spurious logging in some cases. Set `profiling.report_dropped_profiles` to `True` if you want older behaviour.

## `v0.8.35`

### Breaking Changes

### Potential Downtime

### Deprecations

- #4875 Lookml view file contents will no longer be populated in custom_properties, instead view definitions will be always available in the View Definitions tab.

### Other notable Changes

## `v0.8.34`

### Breaking Changes

- #4644 Remove `database` option from `snowflake` source which was deprecated since `v0.8.5`
- #4595 Rename confusing config `report_upstream_lineage` to `upstream_lineage_in_report` in `snowflake` connector which was added in `0.8.32`

### Potential Downtime

### Deprecations

- #4644 `host_port` option of `snowflake` and `snowflake-usage` sources deprecated as the name was confusing. Use `account_id` option instead.

### Other notable Changes

- #4760 `check_role_grants` option was added in `snowflake` to disable checking roles in `snowflake` as some people were reporting long run times when checking roles.
