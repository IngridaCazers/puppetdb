---
title: "PuppetDB: Catalog inputs endpoint"
layout: default
canonical: "/puppetdb/latest/api/query/v4/catalog_inputs.html"
---

# Catalog inputs endpoint

[curl]: ../curl.markdown#using-curl-from-localhost-non-sslhttp
[paging]: ./paging.markdown
[query]: query.markdown
[subqueries]: ./ast.markdown#subquery-operators
[ast]: ./ast.markdown
[nodes]: ./nodes.markdown

> **Experimental Endpoint**: This endpoint is designated as
> experimental. It may be altered or removed in a future release.

## `/pdb/query/v4/catalog-inputs`

The `/catalog-inputs` endpoint returns a JSON array containing
all of the most recent catalog inputs for the catalogs in your
infrastructure.

### URL parameters

* `query`: optional. A JSON array containing the query in prefix
  notation (`["<OPERATOR>", "<FIELD>", "<VALUE>"]`). See the sections
  below for the supported operators and fields. For general info about
  queries, see [our guide to query structure.][query]

If a query parameter is not provided, all catalog inputs will be returned.

### Query operators

See [the AST query language page][ast].

### Query fields

* `certname` (string): the certname associated with the input.
* `producer_timestamp` (string): a string representing the time at
  which the `replace catalog inputs` command containing the input was
  submitted from the Puppet Server.
* `catalog_uuid` (string): the unique ID of the catalog to which the
  input corresponds.
* `inputs` (array): An array ofinputs for a catalog in the form `[ ["<type>", "<name>"] ... ]`
  such as `[ ["hiera", "puppetdb::globals::version"] ... ]`.

### Subquery Relationships

Here is a list of related entities that can be used to constrain the result set
using implicit subqueries. For more information consult the documentation for
[subqueries][subqueries].

* [`nodes`][nodes]: Node for a catalog.

### Response format

Successful responses will be in `application/json`.

The result will be a JSON array with one entry per certname. Each entry is of
the form:

    {
      "certname" : <node certname>,
      "producer_timestamp": <time of catalog transmission by Puppet Server>,
      "catalog_uuid" : <unique id of related catalog>,
      "inputs" : [[<catalog input type>, <catalog inputname>]]
    }

### Examples

This query will return the complete list of catalog inputs:

    curl -X GET http://localhost:8080/pdb/query/v4/catalog-inputs

    [{
       "certname" : "yo.delivery.puppetlabs.net",
       "producer_timestamp": "2014-10-13T20:46:00.000Z",
       "catalog_uuid" : "53b72442-3b73-11e3-94a8-1b34ef7fdc95",
       "inputs" : [["hiera", "puppetdb::globals::version"]]
    },
    {
       "certname" : "foo.delivery.puppetlabs.net",
       "catalog_uuid" : "9a3c8da6-f48c-4567-b24e-ddae5f80a6c6",
       "producer_timestamp": "2014-11-20T02:15:20.861Z",
       "inputs" : [["hiera", "puppetdb::disable_ssl"]]
    }]

This query will return all catalogs with producer_timestamp after 2014-11-19:

    curl -X GET http://localhost:8080/pdb/query/v4/inputs \
      --data-urlencode 'query=[">","producer_timestamp","2014-11-19"]'

    [{
       "certname" : "foo.delivery.puppetlabs.net",
       "catalog_uuid" : "9a3c8da6-f48c-4567-b24e-ddae5f80a6c6",
       "producer_timestamp": "2014-11-20T02:15:20.861Z",
       "inputs" : [["hiera", "puppetdb::disable_ssl"]]
    }]

## Paging

The v4 catalog-inputs endpoint supports all the usual paging
URL parameters described in the documents on
[paging][paging]. Ordering is allowed on every queryable field.
