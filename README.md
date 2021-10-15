# My exam notes

You may find this useful, or not :confused:

These notes are my work towards the Elastic Certified Engineer exam.
The exam is hands on, thus no multiple choice senarios.  You actually need to know it and be able to walk the walk. :D

Elastic are up front with the exam content with a large topic list you can see below.  Thus this enables candidates to be able to work on senarios that are close to what will be in the exam. :joy:

I passed on 14th October 2021. :joy: :joy: :joy:

I had 10 questions with two clusters to work on in 3 hours.

## Docker images

There are a number of docker images to use in the [DockerComposeExamples](DockerComposeExamples) folder in this repo.  The Clustering will be useful.

## Exam prep guide from Elastic

:star: Video By Rich Raposa https://www.youtube.com/watch?v=hsaLZSKCkF0

:sparkles: Certification FAQ
https://www.elastic.co/training/certification/faq

:warning: Pay particular attention to the version of ElasticSearch used in the exam.
(this changes 1st July 2021) Currently, it is v7.13.

:star: What is in the exam? https://www.elastic.co/training/elastic-certified-engineer-exam

## Other exercises

I found the following to be very useful, but slightly out of date.  I have linked to my own copies of the questions ad answers, and to the original blog posts.  All kudos to them. :)

- The [George Bridgeman](georgebridgeman.md) Exercises at https://georgebridgeman.com/exercises/
- The [Kreuzwerker](Kreuzwerker_Aggregations.md) at https://medium.com/kreuzwerker-gmbh/exercises-for-the-elastic-certified-engineer-exam-search-and-aggregations-1eefcfb6e992

Also.
- Linux Academy (now owned by A Cloud Guru) has an out of date course, but it is mainly very good.  Here https://acloudguru.com/course/the-linux-academy-elastic-certification-preparation-course (I'd not pay for this until they have updated it to the v7.13 requirements, but if you get it for free from your Company, then it's worth the time)

# Using the elastic.co docs

:warning: It is very important you start to use the elastic.co website instead of google to search for solutions.  The website is available to you in the exam (google is not :blush:).

Use the Search icon at the right hand top of the site (highlighted in Yellow)
![search engine](images/WebSiteSearchIcon.jpg)

## Elastic webinairs

Elastic have a lot of free webinars, you should check these out.   [Youtube.com](https://www.youtube.com/elastic) holds a lot and are easier to look through/search than the elastic.co site.

# Topics (post-July 2021 for v7.13)
:question: Based on https://www.elastic.co/training/elastic-certified-engineer-exam

I have also added the previous (pre-July 2021) exam topics in the headers and marked the new topics.  As you can see most have been renamed or shuffled around.  There are only a few new items and knowing how to build an ELK stack is now removed!?

Each topic below will have example questions and solutions. Click on the Link to follow to that topic.

:bulb: I should point out that if you are learing ElasticSearch then doing both the pre v7.2 and post 7.13 work is recommended, as it contains many things that are still very useful and probably required in daya to day support of ElasticSearch.
But, if you are just looking to pass the exam, then you can do the post v7.13 work.
Personally. i'd do both anyway.

## What is not in here, that is in the exam?

:warning: I didn't do any revision on boosting and scoring.  But encountered a question on it in the exam.  You have been warned. 
At present we are not doing this at work, thus i missed it.

PRs on this or anything else will be greatly accepted. :smile:
## Data Management (previously Indexing Data)
:point_right: [Link (pre July 2021)](v7.2/Indexing_Data.md) v7.2

:point_right: [Link (post July 2021)](Data_Management.md) v7.13

- Define an index that satisfies a given set of requirements
- :new: Use the Data Visualizer to upload a text file into Elasticsearch
- Define and use an index template for a given pattern that satisfies a given set of requirements
- Define and use a dynamic template that satisfies a given set of requirements
- :new: Define an Index Lifecycle Management policy for a time-series index
- :new: Define an index template that creates a new data stream

## Searching Data (Previously Queries/Aggregations)
:point_right: [Link (pre July 2021)](v7.2/Queries.md) :point_right: [Link (pre july 2021)](v7.2/Aggregations.md)  v7.2

:point_right: [Link (post July 2021)](Searching_Data.md) v7.13

- Write and execute a search query for terms and/or phrases in one or more fields of an index
- Write and execute a search query that is a Boolean combination of multiple queries and filters
- :new: Write an asynchronous search
- Write and execute metric and bucket aggregations
- Write and execute aggregations that contain sub-aggregations
- Write and execute a query that searches across multiple clusters

## Developing Search Applications (previously part of Queries/Indexing Data)
:point_right: [Link (pre July 2021)](v7.2/Queries.md) v7.2
:point_right: [Link (pre July 2021)](v7.2/Indexing_Data.md) v7.2

:point_right: [Link (post July 2021)](Developing_Search_Applications.md) v7.13

- Highlight the search terms in the response of a query
- Sort the results of a query by a given set of requirements
- Implement pagination of the results of a search query
- Define and use index aliases
- Define and use a search template

## Data Processing (previously Mappings and Text Analysis)
:point_right: [Link (pre July 2021)](v7.2/Mappings_and_Text_Analysis.md) v7.2

:point_right: [Link (post July 2021)](Data_Processing.md) v7.13

- Define a mapping that satisfies a given set of requirements
- Define and use a custom analyzer that satisfies a given set of requirements
- Define and use multi-fields with different data types and/or analyzers
- Configure an index so that it properly maintains the relationships of nested arrays of objects
(the following were previously part of the Indexing Data topic)
- Use the Reindex API and Update By Query API to reindex and/or update documents
- Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents
- Configure an index so that it properly maintains the relationships of nested arrays of objects

## Cluster Management
:point_right: [Link (pre July 2021)](v7.2/Cluster_Administration.md) v7.2

:point_right: [Link (post July 2021)](Cluster_Management.md) v7.13

- Diagnose shard issues and repair a cluster's health
- Backup and restore a cluster and/or specific indices
- :new: Configure a snapshot to be searchable
- Configure a cluster for cross cluster search
- :new: Implement cross-cluster replication
