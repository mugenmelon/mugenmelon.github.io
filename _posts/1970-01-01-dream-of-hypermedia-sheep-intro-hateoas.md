---
title: 🐑 Dream of Hypermedia Sheep - What is HATEOAS?
tags: essay programming web-development hateoas
diagrams: true
series: 🐑 Dream of Hypermedia Sheep
series_part: 2
published: false
---

TODO

{% include toc.md %}

# What is HATEOAS?

When you ask a developer what "Representational State Transfer" (REST) means, you will most likely hear some variation of:

- HTTP APIs
- JSON payloads
- GET/POST/PUT/DELETE as CRUD operations
- Using semantically correct HTTP status codes

This is a widespread understanding that describes HTTP but has little to do with REST.
REST is the umbrella term of an architectural style for distributed systems that describes 4 pillars:

- **Uniform Resource Identifier (URI)**  
Every addressable piece of information (resource) in a system is identified by a unique URI
- **Representational manipulation of resources**  
Resources can be changed via *representations* of that resource (JSON, XML, HTML etc.)
- **Self-descriptive messages (stateless)**  
Every message must be understandable for a server/client without any knowledge outside of that message

In fact, REST does not constrain itself to any protocol, but for the sake of simplicity let's assume HTTP from now on. The 4th and most easily overlooked pillar is:

- **Hypermedia As The Engine Of Application State (HATEOAS)**

Which is about as vague a term as "Representational State Transfer". So let's outline the individual components of the phrase "HATEOAS" in an order that I think makes the most sense.

## Application State

This part seems obvious at first. This is your server's state i.e. the *resources* in your database, right? Well, yes and no. At least that's not the full picture, because the qualifier *application* bears its own meaning.

*Application* state not only implies the current data in your database, but also *its available transformations*. It tells us two pieces of information for any given resource:

- What is the *current state* of this resource?
- What can I do with the *current state* of this resource to get to a valid *next state*?

This terminology may sound very familiar to some, especially game developers. Yes, it's the classic *state machine* model. In fact, a web server is nothing but a collection of hierarchical finite state machines. Each resource can be in exactly one valid state at a time and transitions to another state in response to certain events. We write application logic (or "business logic") to constrain both the valid state space and available transitions in any given state. This interpretation of web servers as state machines lies at the core of HATEOAS.

Let's look at a simple example. Say we have a web app where users can write and publish articles.

<pre class="mermaid">
flowchart TB
    draft --submit--> review
    review --approve--> published
    published --archive--> archived
    review --reject--> draft
    published --unpublish--> draft
</pre>

An article starts as `draft`. From here we can submit the article to transition to the `review` state. Once approved, the article becomes `published` and we gain the options to `archive` or `unpublish` it. It can be useful to include a `state` property in your data model to make the current state explicit, but states can also be implicitly modeled: We could differentiate `published` articles from others by the existence of a `published_at` datetime property. What I mean to say is: you are always modeling a state machine, regardless of the means you choose to do so. 

## Hypermedia

## The Engine

# How did we get here?

# Next steps
