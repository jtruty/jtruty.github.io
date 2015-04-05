---
layout: post
title: "Elasticsearch HTTP Queries with Jest"
date: 2015-04-03 12:35:00 -0600
comments: true
categories: [elasticsearch, solr, java, jest, http]
---
Elasticsearch has a robust and expansive java api with one large issue - you cannot use the Elasticsearch HTTP Rest protocol to talk to your cluster. Thankfully, HTTP elasticsearch clients exist to work around this issue. [Jest](https://github.com/searchbox-io/Jest) acts as an HTTP wrapper on top of the native elasticsearch api. This is fantastic, because it makes using POJOs and marshalling fairly simple when you cannot use the native api. There are a plethora of elasticsearch features, and by stepping through the official docs and perusing the Jest examples and unit tests, you can more or less figure out how to accomplish what you need.

One issue I came across deals with dates: the bane of every programmer's existence. Elasticsearch by default uses Joda date formats while Jest by default uses Java.util Dates (it uses Gson to parse). When marshalling out of a document, Jest is able to see the class (Date) and the field (long - date timestamp). Marshalling this will give parsing errors, as the Date class has no long constructor. Custom parsing to the rescue!

Jest supports using a custom Gson parser to marshall out of Elasticsearch, but it is not exactly straightforward to setup. There are 2 steps:

1- Create a new class to implement JsonDeserializer (JsonSerializer if you want to index). All that is required is implementing the deserialize function, which takes in a JsonElement that you can convert to a long and create a Date from. See below:
{% highlight java %}
class DateTimeTypeConverter implements JsonSerializer<Date>, JsonDeserializer<Date> {
    // No need for an InstanceCreator since DateTime provides a no-args constructor
    @Override
    public JsonElement serialize(Date src, Type srcType, JsonSerializationContext context) {
        return new JsonPrimitive(src.toString())
    }

    @Override
    public Date deserialize(JsonElement json, Type type, JsonDeserializationContext context)
            throws JsonParseException {
        return new DateTime(json.getAsLong()).toDate()

    }
}
{% endhighlight %}
2- Register this type adapter with Gson, and give this parser to Jest:
{% highlight java %}
JestClientFactory factory = new JestClientFactory()
Gson gson = new GsonBuilder()
        .registerTypeAdapter(Date.class, new DateTimeTypeConverter()).create()
factory.setHttpClientConfig(new HttpClientConfig
        .Builder(endpoint)
        .gson(gson)
        .multiThreaded(true)
        .build())
client = factory.getObject()
{% endhighlight %}
Now, you can parse your Pojo containing Dates directly out of Elasticsearch! This should work now (at least without Date errors):
{% highlight java %}
SearchResult result = client.execute(search)
List<SearchResult.Hit<Article, Void>> hits = searchResult.getHits(Article.class)
{% endhighlight %}
