# The microservices architecture

Microservices is an architecture style where you create autonomous, independently deployable services that collaborate together to create a software application. Microservice architectures are popular because they allow us to build applications that scale, perform well, and enable us to adapt quickly to changing business requirements.

##

problems they solve?
the challenges associated with them?

How we can architect microservices, making good decisions about service boundaries and data ownership?

How we can ensure that developers are as productive as possible, and we'll look at some options for how microservices can communicate with each other reliably?

How we can apply the "defense in depth" principle to secure our microservices and how to automate their deployment and monitor them in production?

How small should they be?

The name microservices also suggests that these services should be small, but there isn't an official size limit. Some teams try to keep them extremely small, maybe just a couple of hundred lines of code but many of the benefits of microservices still apply even if they're a bit larger than that.

### Monolothis

A monolith is a software application that typically has:

- Single codebase: All developers collaborate together on that same source code repository.
- Single process: The build artifact is usually a single executable.
- Single host: The process runs on a single host server or virtual machine.
- Single database: Persists all of its data into a single database.
- Consistent technology: The development environment typically uses a single consistent technology, such as what programming language or SDK you're using throughout the code base.

The monolith model works well for single developers or small teams of maybe two to three developers who are working together for just a few months maybe to build a small website that handles a modest number of visitors.

Pros:
- All the code is in one place, so it's easy to find things.
- There's only one thing that you have to build and run, so it's straightforward for a developer to work on.
- When you come to deploy it, there's just one application to update.

Cons:
- As a code base grows larger, there's a tendency for it to become more difficult to maintain due to growing complexity and the accumulation of technical debt.
- Even when modularized, often you'll find that these modules end up becoming very entangled and interdependent. - Even a single line code change requires the entire application to be deployed, which is risky and usually involves a period of downtime, something that's becoming less acceptable in the modern era of cloud services that are expected to maintain high availability.
- Scaling a monolith to meet demands of increased users or large amounts of data is also very difficult. Unless great care has been taken to make your monolith stateless, you probably can't scale it out horizontally. That's where you add additional servers. And so your only option is to scale vertically where you provision much more powerful and expensive servers. And monoliths require the entire application to be scaled out together rather than just calling the individual components that require additional processing power.
- You can very easily find yourself wedded to legacy technology. Whatever tech stack you built the original version with is going to be very hard to get away from as you have to upgrade the entire application to move to a newer framework. And this reduces your agility to adopt newer patterns and practices or to take advantage of innovation such as new tools and services that would benefit your application.

#### Distributed monoliths 

An architecture based on services is not the same as using microservices. It's possible to create a system where you have several services, but ...
- they all access data in a shared database,
- and they're all tightly coupled to each other in such a way that you have to deploy them all together,
- and any change you want to make to the system requires modifications to multiple services.

Systems like this are sometimes referred to as distributed monoliths, and these things are big trouble. They combine all the downsides of monoliths with all the challenges of microservices and offer very few of the benefits of either approach.

### Microservices

Pros:
- Easy to maintain for many developers.
- Easy to maintain for bigger applications.
- Better for applications that will have many users.
- Better for applications that will manage massive amounts of data (even for working with big data tools).


## Build

## Comunication

### Sync

### Async

## Security

## Delivery

## Monitoring
