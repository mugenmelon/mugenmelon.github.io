---
title: 🐑 Dream of Hypermedia Sheep - Intro
tags: essay programming web-development hateoas
series: 🐑 Dream of Hypermedia Sheep
series_part: 1
fiction_disclaimer: true
---

HTML was abandoned because "it isn't sophisticated enough". We needed to split every web app into JSON APIs and fat JavaScript single-page-apps.
Turns out HTML was all we needed all along. These are the ramblings of a frustrated web developer about *Hypermedia As The Engine Of Application State* (HATEOAS).

{% include toc.md %}

# Denial

Whenever I read or heard the phrase "Hypermedia As The Engine Of Application State", my first thought was:

> It's that HTML stuff. I know that already.

My mind was set. Hypermedia wasn't something I needed to investigate. It's *just* HTML after all; how important could it really be? No, I am a full-stack developer. I know what I'm doing! *Everyone* knows we do microservices, RESTful JSON APIs, async messaging, and \<*your favorite javascript framework*\> nowadays. And for good reason, because the entire web is running on JSON! We call that *decoupled, layered architecture*, and it makes distributed software more maintainable, faster, and cheaper to develop.

# Anger

*Of course* the flow works like this:

1. Client: user's text input → ViewModel → DomainModel → DTO → JSON → HTTP request → BFF
2. BFF: DTO → DomainModel → DTO → JSON → HTTP request → Backend
3. Backend: DTO → DomainModel → SQL → database → DomainModel → DTO → JSON → HTTP response → BFF
4. BFF: DTO → DomainModel → DTO → JSON → HTTP response → Client
5. Client: DTO → DomainModel → ViewModel → update UI

so the user can update their e-mail! And don't forget we need 90% unit test coverage, OpenAPI specifications, logging, input validation, and sturdy error handling across the board.

> It doesn't work? Frontend must have forgotten the feature flag, wouldn't be the first time!
> Oh, we forgot to map the `email` property in the BFF. John, did you not notice this during your code review? Did you even run it?
> And QA says e-mails like "john.doe+1" cause a validation error in the backend. Business should have specified this more clearly!
> 
> I can't believe the architects want us to encrypt user e-mails now. It's too much work. They really over-engineered it!
> 
> What do you mean \<other team\> added a new enum value to their API spec? That's a breaking change! They must ship a new version!

# Bargaining

That can't be right. This wasn't supposed to happen with *highly maintainable, loosely coupled, and independent microservices*! The architecture is sound; we just need a few small additions...

- **JavaScript everywhere!**  
We use Node.js in the backend and share validation & DTO models. Forget about language agnosticism and team autonomy; we prefer shared business concerns.
- **GraphQL to the rescue!**  
Let's add query complexity & just one more layer of abstraction, and I swear our problems will magically disappear.
- **Code generation from API specs!**  
Because nothing says "decoupled, distributed system" quite like "we are so tightly coupled, we use the exact same data models everywhere".
- **HAL will make us RESTful!**  
Business is thrilled we finally replaced that pesky "Edit" button with a contextual collection of RESTful JSON hypermedia affordances.

...

Wait, I've been here before. Whenever I had to pile on more complexity, more layers, more abstractions, more processes, more standards & bureaucracy, I was fixing the wrong problem... no, it cannot be... is this accidental complexity...?! Arrghh...

# Depression

So this is just how it's going to be, huh? We will keep building strange and complex machines that are more expensive than they have any right to be, using frameworks and patterns nobody really understands or needs. But the architects agreed on this, and I cannot gather the spirit to even argue about it anymore.

If the merge request has too many changes for me to reasonably go through, I'll just approve it.
If QA finds bugs, I just have to search for the correlation ID in our distributed logs and find out who's responsible.
If business keeps complaining about features not getting done on time, we'll just ship it next sprint.
If the architects want to introduce yet another unnecessary microservice, then I'll quietly go along with it.

Fine... I guess I'll check out this HATEOAS thing...

# Acceptance

Well, this is where I am. I have spent a few weeks diving into HATEOAS and used [fixi](https://github.com/bigskysoftware/fixi) (the little brother of [htmx](https://htmx.org/)) to build a simple proof-of-concept web app. We will go through both fixi and my proof-of-concept app in an upcoming essay in this series.

The goal is to create something valuable while limiting myself largely to native browser functionality. A select few additional hypermedia affordances from fixi and a minimal set of custom fixi extensions are allowed. This project will mainly serve as an exploration of hypermedia systems & patterns for educational and personal use. But if things go well, I might release something generally useful.

The tech stack looks like this:

- **HTML, raw CSS and fixi**  
No external JavaScript dependencies. No packaging or build step. Prefer declarative/semantic CSS over framework noise. Fixi extensions only when absolutely necessary. Must be 100% functional with JavaScript disabled (progressive enhancement).
- **Python backend (flask, sqlalchemy)**  
Minimal dependencies. Using jinja2 templates for rendering modular HTML components. Prefer control and clarity over enterprise-grade BS. No pointless abstractions over the fact that we have an SQL database.
- **SQLite database**  
For obvious reasons. A few more constraints: No "magic" ORM migrations. Explicit and deliberate SQL migration files with alembic (similar to golang migrate). Database as first-class-citizen and single source of truth rather than an afterthought.

By now I am fully convinced that a large part of my experience in web development was a lot harder and much more expensive than it really needed to be. Having seen the difference between over-engineered web application microservice landscapes and a coherent hypermedia-driven app, I am baffled by how we got here in the first place.

But before we investigate that, let's first try to understand what *Hypermedia As The Engine Of Application State* means...
