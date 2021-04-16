# My exam notes

You may find this useful, or not :confused:

:warning: A work in progress :warning:

These notes are my work towards the Elastic Certified Engineer exam.
The exam is hands on, thus no multiple choice senarios.  You actually need to know it and be able to walk the walk. :D

Elastic are up front with the exam content with a large topic list you can see below.  Thus this enables candidates to be able to work on senarios that are close to what will be in the exam. :joy:

## Docker images

TBC

## Exam prep guide from Elastic

:star: Video By Rich Raposa https://www.youtube.com/watch?v=hsaLZSKCkF0

:sparkles: Certification FAQ
https://www.elastic.co/training/certification/faq

:warning: Pay particular attention to the version of ElasticSearch used in the exam.
(this changes 1st July 2021)


# Using the elastic.co docs

:warning: It is very important you start to use the elastic.co website instead of google to search for solutions.  The website is available to you in the exam (google is not :blush:).

Use the Search icon at the right hand top of the site (highlighted in Yellow)
![search engine](images/WebSiteSearchIcon.jpg)

# Topics (pre-July 2021)
:question: Based on https://www.elastic.co/training/elastic-certified-engineer-exam

Each topic below will have example questions and solutions. Click on the Link to follow.

## Installation and Configuration
:point_right: [Link](Installation_and_Configuration.md)

- Deploy and start an Elasticsearch cluster that satisfies a given set of requirements
- Configure the nodes of a cluster to satisfy a given set of requirements
- Secure a cluster using Elasticsearch Security
- Define role-based access control using Elasticsearch Security

## Indexing Data
:point_right: [Link](Indexing_Data.md)

- Define an index that satisfies a given set of requirements
- Perform index, create, read, update, and delete operations on the documents of an index
- Define and use index aliases
- Define and use an index template for a given pattern that satisfies a given set of requirements
- Define and use a dynamic template that satisfies a given set of requirements
- Use the Reindex API and Update By Query API to reindex and/or update documents
- Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents

## Queries
:point_right: [Link](Queries.md)

- Write and execute a search query for terms and/or phrases in one or more fields of an index
- Write and execute a search query that is a Boolean combination of multiple queries and filters
- Highlight the search terms in the response of a query
- Sort the results of a query by a given set of requirements
- Implement pagination of the results of a search query
- Apply fuzzy matching to a query
- Define and use a search template
- Write and execute a query that searches across multiple clusters

## Aggregations
:point_right: [Link](Aggregations.md)

- Write and execute metric and bucket aggregations
- Write and execute aggregations that contain sub-aggregations

## Mappings and Text Analysis
:point_right: [Link](Mappings_and_Text_Analysis.md)

- Define a mapping that satisfies a given set of requirements
- Define and use a custom analyzer that satisfies a given set of requirements
- Define and use multi-fields with different data types and/or analyzers
- Configure an index so that it properly maintains the relationships of nested arrays of objects

## Cluster Administration
:point_right: [Link](Cluster_Administration.md)

- Allocate the shards of an index to specific nodes based on a given set of requirements
- Configure shard allocation awareness and forced awareness for an index
- Diagnose shard issues and repair a cluster's health
- Backup and restore a cluster and/or specific indices
- Configure a cluster for use with a hot/warm architecture
- Configure a cluster for cross cluster search
