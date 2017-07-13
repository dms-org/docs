Prologue
========

The DMS framework was borne out of frustration from multiple attempts at creating
maintainable applications using the Laravel framework. Although possible, the Laravel
structure does not encourage clear separation of concerns which could cause problems when
building complex applications.

A common requirement of projects is for content and functionality to be manageable through an admin panel.
These can be tedious and time consuming to set up on a per-project basis.

The DMS aims to alleviate these issues by encouraging a clear application structure, 
inspired by concepts from [DDD](https://en.wikipedia.org/wiki/Domain-driven_design)
and [Hexagonal Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html),
to allow complexity to be expressed through robust and maintainable code. 
While also providing an integrated CMS framework to build powerful backends quickly and easily.

Terminology
-----------

Building on concepts of DDD, this documentation uses similar language and jargon.
[See glossary](http://uniknow.github.io/AgileDev/site/0.1.8-SNAPSHOT/parent/ddd/core/glossary.html).


Architecture
------------

Complex applications must define their business logic somewhere in the application.
Every developer has their own opinions on how to structure an application, the DMS
takes an opinionated approach where the application is separated into layers.
The functionality of an application should be contained within service classes which are
able to be reused by each application entry point, whether it be the web UI, CLI or CMS.

![Architecture Diagram](/resources/images/architecture/overview-1.jpg)

[See blog package for example code](https://github.com/dms-org/package.blog)