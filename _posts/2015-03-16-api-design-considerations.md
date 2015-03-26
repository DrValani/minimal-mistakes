---
layout: post
title: Api Design Considerations
excerpt: "Based off ideas discussed at a skillsmatter event on microservices"
modified: 2013-05-31
tags: [design, microservices, best practices, 7digital]
comments: true
image:
  feature: sample-image-5.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---
After attending the skills matter talk [https://skillsmatter.com/meetups/6990-api-design-versioning-and-migration#overview](https://skillsmatter.com/meetups/6990-api-design-versioning-and-migration#overview), it got me thinking about our Api's at 7 digital. 

The keynote speaker was a developer from ITV. They created an on-demand streaming service, and have similar features and functionality to us. 
For example: They have a Catalogue of videos, and have to follow Licensing and Scheduling rules.  
 
The speaker discussed their journey to the current architecture, and in general, where they have a design that is quite similar to our architecture. 


The first speaker talked introduced [Poltel's law][1]. Which closely relates to designing our API's to that they are **_loosely coupled with high cohesion_** between them. This means, that if we make a change in an Api then we shouldn't have to change the API's that call them.   


Lets take the current Search-API. 
How would I redesign this.

Split search-api into two components: Search-Hydrator-API and Search-Core-API.

### Search-Core-API:

Should only return's (release/track/artist) Ids in the response. Search-Hydrator-API will take these ids and most probably call the batch endpoints to populate the response to the client.

Pros & Cons

- Means we have decoupled search api from actual responses. 
- Now if we want to change search we have to make sure it returns all fields and is backward compatible, but now we change just give search only the data it needs to search. 
- We can completely change the backend to Elastic search or any other search engine and only have to worry about returning ids.
- Our Solr index becomes a pure index. We no longer have to store data as it won't be used in the response. 

### Catalogue Access Control

Availability API restrict it just to return just bool if a release/track is. We should be careful of what we expose such as release/streaming/download dates in the response. As models change so will the responses, unless it is kept to true/false values. That way we can change 
the availablity api without needing to change its clients.

 
I think from what ITV have experienced we are moving in the right direction. 


### Caching the Availability API

Responses from the availability api can be quite volatile and hence difficult to cache. ITV use data driven events to let systems know of any updates. 

So for example suppose a product becomes unavailable, the cached is now incorrect. That item is removed from the cache the next request will then just fill in the cache. This allows for updates to become quicker. 

Updates are passed as messages on the queue. Only the Ids are populated on the queue. The queue consumer is responsible for consuming that id and ensure the correct caches are cleared and  

### API Design

Isolation - A change in one API should have minimal impact on its clients.


## Versioning

This is to do with internal versioning not exposing it to our external users.

If all our API's are use a hydrator then the response we get should be the same. Any nonbreaking change will ripple through all the endpoints. 

It is probably good to introduce versioning when implementing a breaking change.
And you want to slowly bring in the change to all the other endpoints. 


[1]: http://en.wikipedia.org/wiki/Robustness_principle "Postel's Law"

