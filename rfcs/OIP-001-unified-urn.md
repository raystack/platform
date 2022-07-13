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

**TBA**
