===================
Data Query Protocol
===================

Introduction
============

Data are usually stored in systems like an RDBMS (relational database management system),
 a NoSQL-cluster or a triple-store. When applications are developed on top of these systems,
developers need a tool for retrieving data segments by sending the system relevant questions,
 called queries. Common query languages are SQL, SPARQL or NoSQL hash-maps (?).

Companies and organisations are moving fast towards a (Linked) Open Data 
policy. This new envelopment affects how people use data. As the data owners will not push
the data to the data consumers' databases, but the data consumers will pull the data from the data owner,
the data will have to be accessed over a transfer protocol, most commonly HTTP, as if the data
were webpages identified by URIs.

In the past, many HTTP web-services have tried to mimic the idea of RDBMS or triple-stores
 by allowing SQL/SPARQL queries to be sent inside a HTTP message body or encoded inside 
the URL. Others have tried to define their own query protocol, although they weren't always 
aware of this, by extending their web-service with extra flags and filters.

The kind of use cases that might benefit from a query protocol are:

* Data viewers calling databases to get data to display.
* Visualisation tools calling databases or data scraping tools.
* Crowd sourcing tools augmenting information dynamically pulled from a data
  catalogue.

In the rest of this document, all possible use cases are called applications. After discussing 
the term data query and after discussing why we need a protocol and not a language,  a proposal 
will be made. At the end of this document, existing work will be discussed taking some  predefined
 criteria into account.


Different aspects of querying data (Or: Defining the term data query protocol)
==============================================================================

Before diving deeper into a proposal and existing work, this chapter will discuss various
 aspects of querying data over HTTP. It gives an answer to why a protocol is needed and 
not just a query language.

Querying
--------

Querying a dataset over HTTP, whether these data are tabular or not, involves five main
features:

1. selecting
2. limiting
3. ordering by
4. filtering
5. aggregating (sum, count, distinct)

A sixth feature is a nice addition to query protocols:

6. joining with other datasets

Identifying resources
---------------------

A first question that needs to be brought forward is how the dataset that is going to be
manipulated is identified. Many languages, such as SQL, identify a dataset using a
simple name. Other languages, such as SPARQL, identify a dataset using URIs.

When URIs are chosen to identify resources, the data query protocol might also choose to embrace
the RESTful principles. It can become an extra layer upon REST (these protocols usually
come with an end-point), or it can become part of REST (extra parameters are sent within the request).

Big Data vs. Small Data
-----------------------

Small Data are data that can be returned within one HTTP response. These data can be 
scraped from a website, can be the result from a query on a RDBMS, can be a static CSV file...

Big Data are data that cannot be maintained on one single machine. Datasets between big data
and small data are usually contained in one file (for instance in a CSV file) or a simple system
like an RDBMS (sqlite, mysql, 4store...).

Not all protocols can handle all kinds of data. Some protocols only perform queries on
what would have been returned in a HTTP response. Others translate the HTTP query string
to the query language of their big data or RDBMS back-end.

The Semantic Web
----------------

The Semantic Web has been the subject of academic research for several years. RDF triples 
are used to store data in triple-stores. A triple is a series of 3 URIs which identify an 
object, a predicate and a subject.

Example:

::
  subject predict object
  <http://...> <http:///> <http:///>
  <http://...> is schema:...



The Semantic Web gives a new dimension to query languages. For instance, it would be possible
to filter on all elements that happen to be a schema:Library. Usually triple-stores also come, mostly
exclusively, with a semantic query languages called SPARQL (virtuoso, 4store, sesame...).

Proposal
========

The proposal divides into 2 parts. First, the definition of a JSON-serializable
query object. Second, the presentation of that data to a web accessible query
endpoint.

Query Object
------------

The Proposal is heavily based on `ElasticSearch query language`_

.. _ElasticSearch query language: http://www.elasticsearch.org/guide/reference/api/search/

Query object has the following key attributes:

* size (=limit): number of results to return
* from (=offset): offset into result set -
  http://www.elasticsearch.org/guide/reference/api/search/from-size.html
* sort: sort order -
  http://www.elasticsearch.org/guide/reference/api/search/sort.html
* query: Query in ES Query DSL
  http://www.elasticsearch.org/guide/reference/api/search/query.html
* fields: set of fields to return -
  http://www.elasticsearch.org/guide/reference/api/search/fields.html
* facets: - see http://www.elasticsearch.org/guide/reference/api/search/facets/

Additions:

* q: either straight text or a hash will map directly onto a [query_string
  query](http://www.elasticsearch.org/guide/reference/query-dsl/query-string-query.html)
  in back-end

  * Of course this can be re-interpreted by different back-ends. E.g. some may
    just pass this straight through e.g. for an SQL back-end this could be the
    full SQL query

* filters: dict of fields with for each one specified a filter like term,
  terms, prefix, range. This provides a quick way to do filtering.

  * Value for a field can just be text in which case this becomes a term query
    on that field

    * E.g. my-field: 'abc' - would only match results with abc in that field


Examples
~~~~~~~~

::

  {
     q: 'quick brown fox',
     filters: {
       'owner': 'jones'
     }
  }


Existing Work
============= 

ElasticSearch
-------------

JSON oriented document store and search index.

* http://www.elasticsearch.org/guide/reference/api/search/
* http://www.elasticsearch.org/guide/reference/query-dsl/

Open Search
-----------

Open Search is a standard for searching inside webpages. It can be extended to work for
any RESTful web-service.

* http://ope...? TODO

Webstore
--------

Designed to expose RDBMS over RESTful HTTP.

* http://github.com/okfn/webstore
* Documentation (includes spec of query format): http://webstore.readthedocs.org/en/latest/index.html
* Supports RESTful style as well as full SQL

.. _Webstore: http://github.com/okfn/webstore

CouchDB
-------

A RESTful client 

SQL
---

Raw SQL over HTTP.

This is one in Scraperwiki and the Webstore_.

DAP
---

DAP is a data transmission protocol designed speciﬁcally for science data. The
protocol relies on the widely used and stable HTTP and MIME standards, and
provides data types to accommodate gridded data, relational data, and time
series, as well as allowing users to deﬁne their own data types.

* http://opendap.org/pdf/ESE-RFC-004v1.2.pdf
* http://opendap.org/

Unstructured Query Language
---------------------------

* UnQL means Unstructured Query Language. It's an open query language for JSON, semi-structured and document databases.
* http://www.unqlspec.org/display/UnQL/Home

UnQL is a query language not a query protocol so provides no information on how clients and servers interact.

HTSQL
-----

* http://htsql.org/
* A database query language based on SQL

  * HTSQL is a URI-based high-level query language for relational databases. HTSQL wraps your database with a web service layer, translating HTTP requests into SQL and returning results as HTML, JSON, etc.

URI Fragment Identifiers for the text/csv Media Type
----------------------------------------------------

* Method for addressing (and hence possibly querying) into csv documents
* http://tools.ietf.org/html/draft-hausenblas-csv-fragment-00
* Status: draft
* Published: 26 April 2011

Google Visualization API Query Language
---------------------------------------

Another restricted SQL. Has advantage of one existing implementation - so would
immediately work with Google Spreadsheets and Fusion Tables, presumably? Also

* http://code.google.com/apis/chart/interactive/docs/querylanguage.html#Language_Syntax

SPARQL
------

SPARQL is the de facto standard query language for triple stores. It uses URIs to identify
resources. Anyone can directly execute SPARQL queries over HTTP using the end-point.

The DataTank and SPECTQL
------------------------

The DataTank is a 5 minute RESTful API. It comes with a query language, based on HTSQL, which
provides an easy way to structure the response to be able to directly use it inside your app
or visualisation.

For example:

 http://data.irail.be/spectql/Airports/Liveboard/LCY/2012/03/04/12/00/departures{iso8601,delay-,direction}:csv

Selects the time, delay and direction of planes leaving at the airport of London. Sorted by delay (DESC) 
and with CSV as the output format.

