---
layout: post
title: "Java Elasticsearch Aggregations"
date: 2015-09-19 09:32:00 -0600
comments: true
categories: [programming]
tags: [elasticsearch, java]
---
The [Elasticsearch Java API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/_metrics_aggregations.html) lets you get at aggregations in a similar way to using HTTP requests. Here is an example terms aggregation:
{% highlight groovy %}
        AggregationBuilder agg = AggregationBuilders.terms("agg_name").field("my_field")
        SearchResponse response = client.prepareSearch().setSize(0)
                .addAggregation(agg)
                .execute().actionGet()
        Terms aggTerms = response.getAggregations().get("agg_name")
        for (bucket in aggTerms.getBuckets()) {
            log.debug("Key: " + bucket.key + " Value: "+ bucket.docCount)
        }
{% endhighlight %}
A couple of things to notice:

1. You have to name the aggregation. This is the "agg_name" field that we send to the terms function. We are doing the actual aggregation on the "my_field" field that is already present in our elasticsearch index.
2. We set the size to 0, because by default there is still a normal query performed which will return the default of 10 results if we don't set it.
3. We can perform multiple aggregations with one query, we would just add more aggregations to the search while giving each a unique name.

This terms aggregation will bucket the contents of the specified field and return the different contents and counts, sorted by count descending. Here, we just loop through the results and print them to our log. An example response might be (if we have a few states in the my_field field):

{% highlight bash %}
Key: Illinois Value: 15
Key: Texas Value: 12
Key: Arizona Value: 6
{% endhighlight %}
