---
title: 🐑 Dream of Hypermedia Sheep
tags: essay programming web-development hateoas
---

{% include toc.md %}

# Intro

## Denial

Whenever I read or heard the term "Hypermedia As The Engine Of Application State" (HATEOAS) my first thought was:

> It's that HTML stuff. I know that already.

My mind was set. Hypermedia wasn't something I needed to investigate. It's *just* HTML after all, how important could it really be?

No, I am a full stack developer. I know my "-ends" from the front and the back! I don't need to read about it. *Everyone knows* we do microservices, RESTful JSON APIs, async messaging and *\<your favorite javascript framework\>* for good reason. We call that a *decoupled, layered architecture*. It's the way enterprise software works!

## Anger

*Of course* we have to map the user's text input to the client view model, map that to the client domain model, map that to the outgoing DTO model, serialize that to JSON, send that via HTTP request to the BFF, deserialize that to the incoming DTO model, map that to the BFF domain model, map that to the outgoing DTO model, serialize that to JSON, send that via HTTP request to the backend, deserialize that to the incoming DTO model, map that to the backend domain model, map that to SQL database columns, store the data, get the data from SQL database columns, map that to the backend domain model, map that to the response DTO model, serialize that to JSON, send that via HTTP response to the BFF, deserialize that to the backend response DTO model, map that to the BFF response DTO model, serialize that to JSON, send that via HTTP response to the client, deserialize that to the BFF response DTO model, map that to the client domain model and map that to the client view model so the user can update their e-mail! We also need 90% unit test coverage, OpenAPI specifications, logging, input validation and sturdy error-handling across the board.

> It doesn't work? Frontend must have forgotten the feature flag, wouldn't be the first time!
> Oh, we forgot to map the `email` property in the BFF. John, did you not notice this during your code review? Did you even run it?
> And QA says e-mails like "john.doe+1" cause a validation error in the backend. Business should have specified this more clearly!
> 
> I can't believe the architects want us to encrypt user e-mails now. It's too much work. They really over-engineered it!
> 
> What do you mean \<other team\> added a new enum value to their API spec? That's a breaking change! They must ship a new version!

## Bargaining

That can't be right. This wasn't supposed to happen with *highly maintainable, loosely coupled and independent microservices*! The architecture is sound, we just need a few small additions...

- **JavaScript everywhere!**  
We use Node.js in the backend and share validation & DTO models. Forget about language agnosticism and team autonomy, we prefer shared business concerns.
- **GraphQL to the rescue!**  
Just one more layer of abstraction and I swear our problems will magically disappear.
- **Code generation from API specs!**  
Because nothing says "decoupled, distributed system" quite like "we use the exact same data models everywhere".
- **HAL will make us RESTful!**  
Business is thrilled we finally replaced that pesky "Edit" button with a contextual collection of RESTful hypermedia affordances.

...

Wait, I've been here before. Whenever I had to pile on more complexity, more layers, more abstractions, more processes, more standards & bureaucracy I was fixing the wrong problem... no, it cannot be... is this accidental complexity...?! Arrghh...

## Depression

## Acceptance
