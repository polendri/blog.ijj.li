+++
title = "Notes on Roy Fielding's REST Paper"
date = "2019-03-25T19:42:56-07:00"
categories = ["compsci"]
tags = []
draft = true
+++

This should be the start of a series titled "Paul reads papers he's embarrassed to have never looked at before now". Roy
Fielding is a computer scientist famous for his work on the HTTP specification and for defining the
[REST achitecture](https://en.wikipedia.org/wiki/Representational_state_transfer).

TODO: Complete introduction

Link to the paper: https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf

## Chapter 1: Software Architecture

We define "software architecture" and related terms. Namely:

software architecture
: An abstraction of the run-time elements of a software system during some phase of its operation. A system may be
  composed of many levels of abstraction and many phases of operation, each with its own software architecture.

element
: A software architecture is defined by a configuration of architectural **elements**---components, connectors, and
  data---constrained in their relationships in order to achieve a desired set of architectural properties.

component
: An abstract unit of software instructions and internal state that provides a transformation of data via its interface.

connector
: An abstract mechanism that mediates communication, coordination, or cooperation among components.

datum
: An element of information that is transferred from a component, or received by a component, via a connector.

configuration
: The structure of architectural relationships among components, connectors, and data during a period of system
  run-time.

architectural style
: A coordinated set of architectural constraints that restricts the roles/features of architectural elements and the
  allowed relationships among those elements within any architecture that conforms to that style.

Notable is the distinction that "software architecture" is an abstraction of *run-time* elements of a software system:
this makes explicit that different phases of an program's operation (initialization, processing, shutdown, etc) might
have different architectures, with yet another architecture responsible for transitioning between these phases. It's
architectures all the way down.

Fielding's main motivation for this definition seems to be to keep it well grounded in the actual operation of the
software, particularly with regards to data. "Box and line" architecture diagrams ignore data entirely, and so they
don't adequate capture the behaviour of software. This is apparent later when we start talking about architectures
which blur the lines between data and code (such as sending code over the network).

## Chapter 2: Network-based Application Architectures

Next, we define a list of properties of architectural styles that are valuable or interesting, so that later we can
evaluate a whole host of architectural styles against these properties.

The properties considered are:

* **Performance:**
  * **Network performance:** Does it send a lot of data over the network?
  * **User-perceived performance:** Does it have low latency? Does it render quickly?
  * **Network efficiency:** Can it avoid using the network, e.g. via caching?
* **Scalability:** Can it scale to a large number of components?
* **Simplicity:** Is it easy to understand and implement?
* **Modifiability:**
  * **Evolvability:** Can a component be changed without affecting other components?
  * **Extensibility:** Is it easy to add functionality to the system?
  * **Customizability:** Is it easy to override or specialize its behaviour for unusual requirements?
  * **Configurability:** Can it be easily adapted post-deployment?
  * **Reusability:** Are its components generic enough to be reusable for other applications?
* **Visibility:** Is it easy to view the operation of the system externally?
* **Portability:** Is it portable across many runtime environments?
* **Reliability:** How susceptible is it to failure?

There's one important property which is conspicuously absent from this list though, and it may have occurred to you
already. Something which really wasn't on the radar at the time of publication (2000) in the same way it is today.
What I'm thinking of is...

* **Security:** Will I get pwned by running this system?

It's almost quaint reading a famous CS paper where security gets nothing more than a casual mention, and where the
ability for firewalls to inspect data over the wire is touted as a desirable feature. For REST itself, I think it's
pretty clear that security is left as a detail of the protocols and services which implement the architecture, but I
can't help but think that if this paper were written today it would bring up the security trade-offs while it discusses
*passing code back and forth between clients and servers*.

This is a great list of criteria to keep in mind for *any* evaluation of different architectures, not just for the ones
Fielding evaluates in this dissertation.

## Chapter 3: Network-based Architectural Styles

Now we do the evaluation using the properties defined in Chapter 2. This is a hard one to summarize, so I'd recommend
just reading through the chapter. Here's the final evaluation summary from the chapter: each "Style" is listed as an
acronym, and each property has `+` or `-` characters proportional to a rough measure of how beneficial or detrimental
the style is to that property.

![The evaluation summary, in its full terse glory](/images/notes_on_roy_fielding's_rest_paper/chap3_comparison.png)

We essentially run through every imaginable combination of client-server architecture features:

* stateful vs stateless,
* caching vs no caching,
* layering vs no layering,

along with every imaginable combination of where code runs:

* all on the server with a dumb client,
* all on the client with a dumb database server,
* code getting sent from server to client,
* code getting sent from client to server, and my personal favourite,
* code bouncing bouncing back and forth between client and server.

In the end, we find that "LCODC$SS" gets the most `+` signs across the board; that's short for *Layered*,
*Code-On-Demand*, *Client Cache*, *Stateless Server*. Sound familiar? With "layering" referring to gateways and proxies,
"code on demand" referring to JavaScript, and caching/statelessness being properties of HTTP, what we're describing here
is the Web as we know it.

This is the meatiest chapter in the paper, because it's providing some level of justification for REST over all these
other competing architectural styles. If all you did was skip to Chapter 5, then you missed everything that justifies
why REST might be worthwhile!

As a side note, I couldn't help but notice that one of the mentioned styles sounded familiar...

> The **remote data access** style is a variant of client-server that spreads the application state across both
> client and server. A client sends a database query in a standard format, such as SQL, to a remote server. The server
> allocates a workspace and performs the query, which may result in a very large data set. The client can then make
> further operations upon the result set (such as table joins) or retrieve the result one piece at a time. The client
> must know about the data structure of the service to build structure-dependent queries.

What's this? How did [GraphQL](https://graphql.org/learn/) get into this paper from 2000 (referencing another paper from
1997)? It's good to be reminded that some of these hot new tools are iterations on ideas that are decades old.

## Chapter 4: Designing the Web Architecture: Problems and Insights

TODO

## Chapter 5: Representational State Transfer (REST)

TODO

## Chapter 6: Experience and Evaluation

TODO
