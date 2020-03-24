---
layout: post
title:  "TrueStreet"
date:   2020-03-14
excerpt: "The project that I worked on during my internship at Google 
for Summer 2019 was released to open source."
project: true

comments: false
---

The Github link for TrueStreet is [here](https://github.com/google/truestreet)

TrueStreet is a read/write storage endpoint to allow Prometheus to use Google Cloud Spanner as a 
timeseries data store. Truestreet leverages Google Cloud Spanner database to store timeseries data, 
resulting in a fast and scalable monitoring system.

If you do not know what [Prometheus](https://prometheus.io/docs/introduction/overview/) is, it
is an open source systems monitoring and alert system. It's practical uses includes giving 
different microservices a Prometheus endpoint, and allowing Prometheus to periodically 
scrape these endpoints for timeseries data. Prometheus can store vast amounts of data locally
on disk, but it also has the capability to write to remote locations. These remote locations
require the use of an adapter, as Prometheus writes to these remote locations in its own defined 
format, and the adapter needs to translate these messages. If the remote location is a database,
like PostgreSQL or Cloud Spanner, these messages get adapted into table entries.

Several advantages exist for having a remote location for timeseries storage. One is that
remote storage can be more reliable as a place to keep data for the long term. Admittedly though,
this advantage is somewhat limited as with large disk size one can store data that stretches
far into the past, with relatively similar cost. If this disk fails however, it is very nice to have
this saved elsewhere. The larger advantage is that multiple instances of Prometheus can share
the same remote write and read endpoint. It is relatively common for larger scale systems
to have multiple Prometheus monitoring systems running and it is not very straightforward 
to have them share their data with each other. Having a remote endpoint that all the systems
have access to solves this problem. 

The code for this project is actually pretty brief, a couple files, but the single commit on
Github hides many iterations that I had to go through before this "final" version. Prometheus,
especially when used by Google, can potentially scrape across millions of timeseries metrics, and 
these each producing millions of data samples. Prometheus locally is capable of writing these
to disk at such a rate, but figuring out how to get these samples written remotely at
such an intense sample rate proved to be an interesting challenge. Additionally, while
Prometheus also locally was able to perform special compression techniques to make each data
point extremely small (around a couple bytes per sample), these same techniques do
not carry over when writing to a table. So, I had to employ other methods to keep space
usage as small as possible. Prometheus already lists many different adapters on their website,
but none for Google's Cloud Spanner. Additionally, some of these adapters do not do
simple optimizations such as writing to the database in batches. The rate at which they
can write to their database is thus serverly limited and cannot take the full advantage of
the database. By simply writing in batches, I was able to achieve greater than 5x the maximum
rate of writing of samples from just one instance of TrueStreet. Additionally, some
adapters wrote a great amount of redundant data for each sample. Specifically, each sample belongs
to a timeseries/metric. These timeseries contain multiple key value labels that are used
to identify the timeseries. Rather than have each sample contain this information, I used a 
unique identier for a timeseries for sample entries. Actual details about timeseries were kept
in a separate table. Furthermore, from experimentation, I found that millions of timeseries can 
share a few thousand unique labels. So, labels can be kept in a separate table as well with 
mappings from timeseries to their appropriate labels. Overall, a lot of the time spent on
TrueStreet was optimizing the initial implementation. Eventually, with a single kubernetes 
instance of TrueStreet and the recommended three CloudSpanner instances, my implementation
was able to cap out at around 50,000 written samples per second. With more instances of
TrueStreet with Kubernetes load balancing, I believe that this number could be higher. In 
my original attempt without any optimizations, I had only managed to write a couple thousand
samples per second.

Some Prometheus adapters only provided write capability to a database, but TrueStreet was able
to provide both read and write capabilities. It's this reason that my Google mentor originally
gave the name "TrueStreet," as we are able to go both ways on this endpoint (Unlike a one way
street). Similar to writing, Prometheus sends its request in its own query language, PromQL,
so this has to be converted through the adapter to and SQL query. The data within Cloud Spanner
as described before is a bit different from how Prometheus asks for it, so we have to request
the data through a series of filters and joins. Glazing over a lot of thinking that I had to do, 
TrueStreetis able to support any PromQL query that gets thrown at it. Prometheus usually
requests data first from its own local disk which of course is much faster, but for older
historical data that TrueStreet has, it can deliver quickly. This is important when the user
is requesting to view this data from some visualization interface like Grafana. 

This provides a somewhat brief overview of what the project was about. Hopefully this work
can be of use to some people now that it is available on Github.
