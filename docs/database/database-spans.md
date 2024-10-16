<!--- Hugo front matter used to generate the website version of this page:
linkTitle: Client Calls
--->

# Semantic Conventions for Database Client Calls

**Status**: [Release Candidate][DocumentStatus], unless otherwise specified

<!-- Re-generate TOC with `markdown-toc --no-first-h1 -i` -->

<!-- toc -->

- [Name](#name)
- [Common attributes](#common-attributes)
  - [Notes and well-known identifiers for `db.system`](#notes-and-well-known-identifiers-for-dbsystem)
- [Sanitization of `db.query.text`](#sanitization-of-dbquerytext)
- [Semantic Conventions for specific database technologies](#semantic-conventions-for-specific-database-technologies)

<!-- tocstop -->

> **Warning**
>
> Existing database instrumentations that are using
> [v1.24.0 of this document](https://github.com/open-telemetry/semantic-conventions/blob/v1.24.0/docs/database/database-spans.md)
> (or prior):
>
> * SHOULD NOT change the version of the database conventions that they emit by default
>   until the database semantic conventions are marked stable.
>   Conventions include, but are not limited to, attributes,
>   metric and span names, and unit of measure.
> * SHOULD introduce an environment variable `OTEL_SEMCONV_STABILITY_OPT_IN`
>   in the existing major version which is a comma-separated list of values.
>   If the list of values includes:
>   * `database` - emit the new, stable database conventions,
>     and stop emitting the old experimental database conventions
>     that the instrumentation emitted previously.
>   * `database/dup` - emit both the old and the stable database conventions,
>     allowing for a seamless transition.
>   * The default behavior (in the absence of one of these values) is to continue
>     emitting whatever version of the old experimental database conventions
>     the instrumentation was emitting previously.
>   * Note: `database/dup` has higher precedence than `database` in case both values are present
> * SHOULD maintain (security patching at a minimum) the existing major version
>   for at least six months after it starts emitting both sets of conventions.
> * SHOULD drop the environment variable in the next major version.

**Span kind:** MUST always be `CLIENT`.

Span that describes database call SHOULD cover the duration of the corresponding call as if it was observed by the caller (such as client application).
For example, if a transient issue happened and was retried within this database call, the corresponding span should cover the duration of the logical operation
with all retries.

## Name

Database spans MUST follow the overall [guidelines for span names](https://github.com/open-telemetry/opentelemetry-specification/tree/v1.37.0/specification/trace/api.md#span).

<!-- markdown-link-check-disable -->
<!-- HTML anchors are not supported https://github.com/tcort/markdown-link-check/issues/225-->
The **span name** SHOULD be `{db.operation.name} {target}` if there is a
(low-cardinality) `{db.operation.name}` available (see below for the exact definition of the [`{target}`](#target-placeholder) placeholder).

If there is no (low-cardinality) `db.operation.name` available, database span names
SHOULD be [`{target}`](#target-placeholder).
<!-- markdown-link-check-enable -->

If neither `{db.operation.name}` nor `{target}` are available, span name SHOULD be `{db.system}`.

Semantic conventions for individual database systems MAY specify different span name format.

The <span id="target-placeholder">`{target}`</span> SHOULD describe the entity that the operation is performed against
and SHOULD adhere to one of the following values, provided they are accessible:

- `db.collection.name` SHOULD be used for data manipulation operations or operations on a database collection.
- `db.namespace` SHOULD be used for operations on a specific database namespace.
- `server.address:server.port` SHOULD be used for other operations not targeting any specific database(s) or collection(s)

If a corresponding `{target}` value is not available for a specific operation, the instrumentation SHOULD omit the `{target}`.
For example, for an operation describing SQL query on an anonymous table like `SELECT * FROM (SELECT * FROM table) t`, span name should be `SELECT`.

## Common attributes

These attributes will usually be the same for all operations performed over the same database connection.

<!-- semconv db -->
<!-- NOTE: THIS TEXT IS AUTOGENERATED. DO NOT EDIT BY HAND. -->
<!-- see templates/registry/markdown/snippet.md.j2 -->
<!-- prettier-ignore-start -->
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

| Attribute  | Type | Description  | Examples  | [Requirement Level](https://opentelemetry.io/docs/specs/semconv/general/attribute-requirement-level/) | Stability |
|---|---|---|---|---|---|
| [`db.system`](/docs/attributes-registry/db.md) | string | The database management system (DBMS) product as identified by the client instrumentation. [1] | `other_sql`; `adabas`; `cache` | `Required` | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| [`db.collection.name`](/docs/attributes-registry/db.md) | string | The name of a collection (table, container) within the database. [2] | `public.users`; `customers` | `Conditionally Required` [3] | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| [`db.namespace`](/docs/attributes-registry/db.md) | string | The name of the database, fully qualified within the server address and port. [4] | `customers`; `test.users` | `Conditionally Required` If available. | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| [`db.operation.name`](/docs/attributes-registry/db.md) | string | The name of the operation or command being executed. [5] | `findAndModify`; `HMSET`; `SELECT` | `Conditionally Required` [6] | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| [`db.response.status_code`](/docs/attributes-registry/db.md) | string | Database response status code. [7] | `102`; `ORA-17002`; `08P01`; `404` | `Conditionally Required` [8] | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| [`error.type`](/docs/attributes-registry/error.md) | string | Describes a class of error the operation ended with. [9] | `timeout`; `java.net.UnknownHostException`; `server_certificate_invalid`; `500` | `Conditionally Required` If and only if the operation failed. | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |
| [`server.port`](/docs/attributes-registry/server.md) | int | Server port number. [10] | `80`; `8080`; `443` | `Conditionally Required` [11] | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |
| [`db.operation.batch.size`](/docs/attributes-registry/db.md) | int | The number of queries included in a batch operation. [12] | `2`; `3`; `4` | `Recommended` | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| [`db.query.text`](/docs/attributes-registry/db.md) | string | The database query being executed. [13] | `SELECT * FROM wuser_table where username = ?`; `SET mykey "WuValue"` | `Recommended` [14] | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| [`network.peer.address`](/docs/attributes-registry/network.md) | string | Peer address of the database node where the operation was performed. [15] | `10.1.2.80`; `/tmp/my.sock` | `Recommended` If applicable for this database system. | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |
| [`network.peer.port`](/docs/attributes-registry/network.md) | int | Peer port number of the network connection. | `65123` | `Recommended` if and only if `network.peer.address` is set. | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |
| [`server.address`](/docs/attributes-registry/server.md) | string | Name of the database host. [16] | `example.com`; `10.1.2.80`; `/tmp/my.sock` | `Recommended` | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |
| [`db.query.parameter.<key>`](/docs/attributes-registry/db.md) | string | A query parameter used in `db.query.text`, with `<key>` being the parameter name, and the attribute value being a string representation of the parameter value. [17] | `someval`; `55` | `Opt-In` | ![Experimental](https://img.shields.io/badge/-experimental-blue) |

**[1]:** The actual DBMS may differ from the one identified by the client. For example, when using PostgreSQL client libraries to connect to a CockroachDB, the `db.system` is set to `postgresql` based on the instrumentation's best knowledge.
This attribute has stability level RELEASE CANDIDATE.

**[2]:** It is RECOMMENDED to capture the value as provided by the application without attempting to do any case normalization.
If the collection name is parsed from the query text, it SHOULD be the first collection name found in the query and it SHOULD match the value provided in the query text including any schema and database name prefix.
For batch operations, if the individual operations are known to have the same collection name then that collection name SHOULD be used, otherwise `db.collection.name` SHOULD NOT be captured.
This attribute has stability level RELEASE CANDIDATE.

**[3]:** If readily available. The collection name MAY be parsed from the query text, in which case it SHOULD be the first collection name found in the query.

**[4]:** If a database system has multiple namespace components, they SHOULD be concatenated (potentially using database system specific conventions) from most general to most specific namespace component, and more specific namespaces SHOULD NOT be captured without the more general namespaces, to ensure that "startswith" queries for the more general namespaces will be valid.
Semantic conventions for individual database systems SHOULD document what `db.namespace` means in the context of that system.
It is RECOMMENDED to capture the value as provided by the application without attempting to do any case normalization.
This attribute has stability level RELEASE CANDIDATE.

**[5]:** It is RECOMMENDED to capture the value as provided by the application without attempting to do any case normalization.
If the operation name is parsed from the query text, it SHOULD be the first operation name found in the query.
For batch operations, if the individual operations are known to have the same operation name then that operation name SHOULD be used prepended by `BATCH `, otherwise `db.operation.name` SHOULD be `BATCH` or some other database system specific term if more applicable.
This attribute has stability level RELEASE CANDIDATE.

**[6]:** If readily available. The operation name MAY be parsed from the query text, in which case it SHOULD be the first operation name found in the query.

**[7]:** The status code returned by the database. Usually it represents an error code, but may also represent partial success, warning, or differentiate between various types of successful outcomes.
Semantic conventions for individual database systems SHOULD document what `db.response.status_code` means in the context of that system.
This attribute has stability level RELEASE CANDIDATE.

**[8]:** If the operation failed and status code is available.

**[9]:** The `error.type` SHOULD match the `db.response.status_code` returned by the database or the client library, or the canonical name of exception that occurred.
When using canonical exception type name, instrumentation SHOULD do the best effort to report the most relevant type. For example, if the original exception is wrapped into a generic one, the original exception SHOULD be preferred.
Instrumentations SHOULD document how `error.type` is populated.

**[10]:** When observed from the client side, and when communicating through an intermediary, `server.port` SHOULD represent the server port behind any intermediaries, for example proxies, if it's available.

**[11]:** If using a port other than the default port for this DBMS and if `server.address` is set.

**[12]:** Operations are only considered batches when they contain two or more operations, and so `db.operation.batch.size` SHOULD never be `1`.
This attribute has stability level RELEASE CANDIDATE.

**[13]:** For sanitization see [Sanitization of `db.query.text`](../../docs/database/database-spans.md#sanitization-of-dbquerytext).
For batch operations, if the individual operations are known to have the same query text then that query text SHOULD be used, otherwise all of the individual query texts SHOULD be concatenated with separator `; ` or some other database system specific separator if more applicable.
Even though parameterized query text can potentially have sensitive data, by using a parameterized query the user is giving a strong signal that any sensitive data will be passed as parameter values, and the benefit to observability of capturing the static part of the query text by default outweighs the risk.
This attribute has stability level RELEASE CANDIDATE.

**[14]:** SHOULD be collected by default only if there is sanitization that excludes sensitive information. See [Sanitization of `db.query.text`](../../docs/database/database-spans.md#sanitization-of-dbquerytext).

**[15]:** Semantic conventions for individual database systems SHOULD document whether `network.peer.*` attributes are applicable. Network peer address and port are useful when the application interacts with individual database nodes directly.
If a database operation involved multiple network calls (for example retries), the address of the last contacted node SHOULD be used.

**[16]:** When observed from the client side, and when communicating through an intermediary, `server.address` SHOULD represent the server address behind any intermediaries, for example proxies, if it's available.

**[17]:** Query parameters should only be captured when `db.query.text` is parameterized with placeholders.
If a parameter has no name and instead is referenced only by index, then `<key>` SHOULD be the 0-based index.
This attribute has stability level RELEASE CANDIDATE.

The following attributes can be important for making sampling decisions
and SHOULD be provided **at span creation time** (if provided at all):

* [`db.collection.name`](/docs/attributes-registry/db.md)
* [`db.namespace`](/docs/attributes-registry/db.md)
* [`db.operation.name`](/docs/attributes-registry/db.md)
* [`db.query.text`](/docs/attributes-registry/db.md)
* [`db.system`](/docs/attributes-registry/db.md)
* [`server.address`](/docs/attributes-registry/server.md)
* [`server.port`](/docs/attributes-registry/server.md)

`db.system` has the following list of well-known values. If one of them applies, then the respective value MUST be used; otherwise, a custom value MAY be used.

| Value  | Description | Stability |
|---|---|---|
| `adabas` | Adabas (Adaptable Database System) | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `cassandra` | Apache Cassandra | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `clickhouse` | ClickHouse | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `cockroachdb` | CockroachDB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `cosmosdb` | Microsoft Azure Cosmos DB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `couchbase` | Couchbase | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `couchdb` | CouchDB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `db2` | IBM Db2 | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `derby` | Apache Derby | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `dynamodb` | Amazon DynamoDB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `edb` | EnterpriseDB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `elasticsearch` | Elasticsearch | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `filemaker` | FileMaker | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `firebird` | Firebird | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `geode` | Apache Geode | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `h2` | H2 | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `hanadb` | SAP HANA | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `hbase` | Apache HBase | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `hive` | Apache Hive | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `hsqldb` | HyperSQL DataBase | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `influxdb` | InfluxDB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `informix` | Informix | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `ingres` | Ingres | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `instantdb` | InstantDB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `interbase` | InterBase | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `intersystems_cache` | InterSystems Caché | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `mariadb` | MariaDB (This value has stability level RELEASE CANDIDATE) | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `maxdb` | SAP MaxDB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `memcached` | Memcached | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `mongodb` | MongoDB | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `mssql` | Microsoft SQL Server (This value has stability level RELEASE CANDIDATE) | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `mysql` | MySQL (This value has stability level RELEASE CANDIDATE) | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `neo4j` | Neo4j | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `netezza` | Netezza | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `opensearch` | OpenSearch | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `oracle` | Oracle Database | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `other_sql` | Some other SQL database. Fallback only. See notes. | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `pervasive` | Pervasive PSQL | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `pointbase` | PointBase | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `postgresql` | PostgreSQL (This value has stability level RELEASE CANDIDATE) | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `progress` | Progress Database | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `redis` | Redis | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `redshift` | Amazon Redshift | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `spanner` | Cloud Spanner | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `sqlite` | SQLite | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `sybase` | Sybase | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `teradata` | Teradata | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `trino` | Trino | ![Experimental](https://img.shields.io/badge/-experimental-blue) |
| `vertica` | Vertica | ![Experimental](https://img.shields.io/badge/-experimental-blue) |

`error.type` has the following list of well-known values. If one of them applies, then the respective value MUST be used; otherwise, a custom value MAY be used.

| Value  | Description | Stability |
|---|---|---|
| `_OTHER` | A fallback error value to be used when the instrumentation doesn't define a custom value. | ![Stable](https://img.shields.io/badge/-stable-lightgreen) |

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->
<!-- END AUTOGENERATED TEXT -->
<!-- endsemconv -->

### Notes and well-known identifiers for `db.system`

The list above is a non-exhaustive list of well-known identifiers to be specified for `db.system`.

If a value defined in this list applies to the DBMS to which the request is sent, this value MUST be used.
If no value defined in this list is suitable, a custom value MUST be provided.
This custom value MUST be the name of the DBMS in lowercase and without a version number to stay consistent with existing identifiers.

It is encouraged to open a PR towards this specification to add missing values to the list, especially when instrumentations for those missing databases are written.
This allows multiple instrumentations for the same database to be aligned and eases analyzing for backends.

The value `other_sql` is intended as a fallback and MUST only be used if the DBMS is known to be SQL-compliant but the concrete product is not known to the instrumentation.
If the concrete DBMS is known to the instrumentation, its specific identifier MUST be used.

Back ends could, for example, use the provided identifier to determine the appropriate SQL dialect for parsing the `db.query.text`.

When additional attributes are added that only apply to a specific DBMS, its identifier SHOULD be used as a namespace in the attribute key as for the attributes in the sections below.

## Sanitization of `db.query.text`

The `db.query.text` SHOULD be collected by default only if there is sanitization that excludes sensitive information.
Sanitization SHOULD replace all literals with a placeholder value.
Such literals include, but are not limited to, String, Numeric, Date and Time,
Boolean, Interval, Binary, and Hexadecimal literals.
The placeholder value SHOULD be `?`, unless it already has a defined meaning in the given database system,
in which case the instrumentation MAY choose a different placeholder.

Placeholders in a parameterized query SHOULD not be sanitized. E.g. `where id = $1` can be captured as is.

[IN-clauses](https://en.wikipedia.org/wiki/Where_(SQL)#IN) MAY be collapsed during sanitization,
e.g. from `IN (?, ?, ?, ?)` to `IN (?)`, as this can help with extremely long IN-clauses,
and can help control cardinality for users who choose to (optionally) add `db.query.text` to their metric attributes.

## Semantic Conventions for specific database technologies

More specific Semantic Conventions are defined for the following database technologies:

* [AWS DynamoDB](dynamodb.md): Semantic Conventions for *AWS DynamoDB*.
* [Cassandra](cassandra.md): Semantic Conventions for *Cassandra*.
* [Cosmos DB](cosmosdb.md): Semantic Conventions for *Microsoft Cosmos DB*.
* [CouchDB](couchdb.md): Semantic Conventions for *CouchDB*.
* [Elasticsearch](elasticsearch.md): Semantic Conventions for *Elasticsearch*.
* [HBase](hbase.md): Semantic Conventions for *HBase*.
* [MongoDB](mongodb.md): Semantic Conventions for *MongoDB*.
* [MSSQL](mssql.md): Semantic Conventions for *MSSQL*.
* [Redis](redis.md): Semantic Conventions for *Redis*.
* [SQL](sql.md): Semantic Conventions for *SQL* databases.

[DocumentStatus]: https://opentelemetry.io/docs/specs/otel/document-status
