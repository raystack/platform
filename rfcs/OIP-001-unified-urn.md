# Unifying URN format across tools

URN or Uniform Resource Name is what we are using across most of ODPF tools and libraries. URN should not be ambiguous and only represents a single resource. That is why having a good URN format is crucial as it will prevent conflict or duplication of identifiers.

The goal of this RFC is to decide what is a good and persistent URN format that can be used across our tools.

## Background

### Current formats

To understand the needs of this initiation better, let's take a look at how each of our tools generate their URN format.

| Resource            | Format                                       | Example                                            |
| :------             | :----                                        | :-----                                             |
| Meteor's RDBMS      | `{service}::{host}/{database}/{table}`       | `postgres::10.283.86.19:5432/user_db/user_role`    |
| Meteor's BigQuery   | `{service}::{project}/{dataset}/{table}`     | `bigquery::odpf-prod/datamart/daily_booking`       |
| Meteor's Metabase   | `{service}::{host}/dashboards/{dashboardID}` | `metabase::my-metabase-server.com/dashboards/872`  |
| Shield's Resource   | `{resource_type}/{namespace}/{resource_id}`  | `r/namespace-id/resource-name`                     |
| Guardian's BigQuery | `{resource_id}`                              | `metabase:293`                                     |

There are few things that we can improve here:

- Using `{host}` as part of the URN will damage persistency (mostly on meteor's). Resource location should be allowed to change without causing its generated URN to be invalid. The reason behind using `{host}` as URN component is for removing ambiguity on URN.
- Using `/` as a separator has a few issues:
    1. When passing a resource URN as route parameter via `http` protocol, this URN will need to be encoded.
    2. Even if it is encoded, some **services** or **proxies** may not be able to `route-match` properly. (e.g [gorillamux](https://github.com/gorilla/mux/issues/639) default behaviour)

### Limited resource referencing between tools

Instead of each tools defining their own URN formats, it will be better, if possible, all tools or services have the same urn format when talking about a resource or asset.

Since different tools are using different format, this would prevent resource referencing (or potentially sharing?) between tools without helps from an extra mapping layer (either by service or library).

## Requirements

Our final unified URN should handle these cases:
1. Persist through change of resource location. (e.g. DB is moved to another server)
2. Can easily be used on URL without relying on services to handle the encoding/decoding.
3. Should be globally unique, or at least within an organization (AKAB / DKAB).
4. SAMPLE CASE: If we somehow have two different Metabases, we should be able to differentiate which metabase it is from URN without relying on `host`.

And these are cases that are great to have:
1. -

## Proposal

**1.** `urn:{NID}:{NSS}:{project}:{kind}:{name}` - by [spy16](https://github.com/spy16)

I highly recommend we stick to the IETF standard definition of URN from [RFC8141](https://datatracker.ietf.org/doc/html/rfc8141) (even if we take only a subset of it). 

[RFC 8141: Section 2](https://datatracker.ietf.org/doc/html/rfc8141#section-2) defines the syntax for URNs.

1. For all ODPF products, we can use `odpf` (or `ODPF`) as the [Namespace Identifier (NID)](https://datatracker.ietf.org/doc/html/rfc8141#section-2.1).
2. Every ODPF product should use the product name as [Namespace Specific String](https://datatracker.ietf.org/doc/html/rfc8141#section-2.2). For example, all resources managed by Entropy would have `entropy` as the NSS. 

NID and NSS combined forms the `assigned-name`: `urn:<NID>:<NSS>` --> `urn:odpf:entropy`. This assigned-name uniquely identifies every product within odpf.

Optional components (which are defined by the entity that owns the NSS) can be appended to `assigned-name` to form resource-level identifiers. For example: `urn:odpf:siren:alert1`

Optional components can have some generic restrictions that we follow. For example:
* all optional components following the namespace should match the pattern `^[A-Za-z0-9-]+$`.  
* no components in the URN are allowed to have `/` character.
* components must be ordered to match reducing scope (i.e., `urn` matches everything globally, `urn:odpf` matches everything within ODPF, `urn:odpf:entropy` matches everything within entropy product of odpf, `urn:odpf:entropy:project-foo` matches everything within `project-foo` and so on).  

With all these combined: The URN for  a "resource"  of kind "firehose" in project "foo" with name "f1" managed by "entropy" will be `urn:odpf:entropy:foo:firehose:f1`


**2.** `{namespace}:{label}:{source}:{identifier}` - by [StewartJingga](https://github.com/StewartJingga)

- **namespace** represents which org (or even environment) the resource belongs to. This is especially useful if you are maintaining resources from different organizations or entities. Example: `odpf`, `odpf-prod`.
- **label** can be used in a case where for example you have two different postgres servers in a namespace. Label is used to differentiate those two, without labels, we can only use the server address. Example: `transaction_storage`, `optimus`, `main-database`, `production`.
- **source** is the service/tool/storage that generate the resource. Example: `postgres`, `bigquery`, `metabase`, `kafka`.
- **identifier** should be unique inside the `source`. The simplest approach is to just use the identifier generated by the `source` itself. In case of `metabase's collection`, we can use `collection:321` or `card:88` for representing a card.

Examples

- **metabase** - `odpf:main-dashboard:metabase:collection:321`
- **bigquery** - `odpf:default:bigquery:myproject:mydataset:mytable` - default is used for an urn that does not require label for uniqueness
- **postgres** - `odpf:stencil-integration:postgres:descriptors` - this is to represents a postgres table that is used by stencil integration
- **elasticsearch** - `odpf-prod:compass:elasticsearch:index:table` - this is to represents an elasticsearch index that is used by compass in production
- **hadoop** - `odpf:datalake:hadoop:index:table` - this is to represents a hadoop table that is being used as a datalake
