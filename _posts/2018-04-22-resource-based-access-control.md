---
title: "The New RBAC: Resource-Based Access Control"
categories:
  - Java
tags:
  - RBAC
  - Security Framework
  - Access Control
copyright_url: https://stormpath.com/blog/new-rbac-resource-based-access-control
copyright_title: "The New RBAC: Resource-Based Access Control"
author: Les Hazlewood
---

This article discusses how security policies are managed using the concept of Roles and how the predominant role-based mechanism for securing applications is largely insufficient.  I discuss what I believe is a much better way of securing applications.

## What is a Role?

When speaking about application security, most people are comfortable with the existing concept of a _Role_.  A Role is a named entity that typically represents a set of behaviors or responsibilities.  Those behaviors translate to things you can or can’t do with a software application.  Roles are typically assigned to user accounts, so by association, users can ‘do’ the things attributed to various roles.

For example, if a user logs-in to an application, and their account has been assigned the ‘Project Manager’ role, then the assumption is that the user can ‘do’ all of the things a Project Manager should be able to do with the application – list available applications, add or remove employees to a project, generate project reports, etc.

In this sense, a role is primarily a behavioral concept: roles give you an idea of what users can do in the application.

## Role-Based Access Control

As a role is primarily a behavioral concept, the logical step when developing software is to use Roles as a means to control access to application features or data.  As you might expect, most people call this approach Role-Based Access Control, or RBAC (“are-back”) for short.

However, there are two primary ways of of actually implementing and performing Access Control: an _Implicit_ way, and an _Explicit_ way.  

The very large majority of software applications today use Implicit access control.  I posit that _Explicit_ Access Control is much better for securing today’s software applications.

### Implicit Access Control

As mentioned previously, Roles represent behavior or responsibility.  But how do we know **_exactly_** what behavior or responsibility is associated with a role?

The answer is that, for the large majority of applications, you don’t know _exactly_ what that role represents.  You have a good idea of course – you know that someone with an ‘administrator’ role can probably lock user accounts or configure certain parts of the application, and maybe user accounts assigned the ‘customer’ role can put items in a shopping cart or request new products.  But usually there is nothing that concretely defines what those behaviors are.

Take for example the string “Project Manager”.  It is just a string name – there is nothing else about it that a software program can inspect to find out “based on this name, I know users assigned this role can do X, Y and Z”.  Developers typically program their applications using this name alone.  For example, to see if a user is allowed to view project reports, you’ll typically see code like this:

```java
//Listing 1. Example Implicit Role-Based Access Control security check:
if (user.hasRole("Project Manager") ) {
    //show the project report button
} else {
    //don't show the button
<strong>}
```

In this example code block, the software developer made the decision to show a button or not based on the ‘Project Manager’ role (probably a project requirement).  But notice that nothing in the code above actually explicitly says “the Project Manager role is allowed to view project reports”.  No one defined that behavioral statement anywhere in software – it is _implied_ that ‘Project Manager’ users can view project reports, so the developer writes an if/else statement reflecting that assumption.

### Brittle Security Policy

Security access control like the above example is very brittle.  That is, it is likely to break, fail, or cause inefficiencies with even slight changes to security requirements.

To illustrate, let’s assume that the team writing the software is told “oh, by the way, we need a new ‘Department Manager’ role, and they also need to be able to view project reports.  Make it happen”.

Now the software developer needs to go back into code to change it to be like this:

```java
//Listing 2. Example Modified Implicit Role-Based Access Control security check:
if (user.hasRole("Project Manager") || user.hasRole("Department Manager") ) {
    //show the project report button
} else {
    //don't show the button
}
```

Then the developer needs to update his test cases, re-build the software, go through whatever QA processes exist, and schedule re-deployment to production – all because of a relatively trivial new security requirement.

But what happens if management returns and asks for another role to be able to view reports?  Or if they need to remove that ability later on?

Or, what if the software needs to support the ability to _dynamically_ create or remove roles at runtime because they want customers to be able to configure roles themselves?

In any of these scenarios, the common implicit (static string) Role-Based Access Control approach fails to meet security needs.  If security policies change, it would be ideal if you didn’t need to touch source code at all.  It would be ideal that the source code only needs to change when data model requirements deem necessary.

## Explicit Access Control: A Better Way

As we see above, changing security policies with an implicit access control approach can wreak havok on software development.  It would be a lot nicer if security policy changes didn’t force code refactoring. Ideally, it’d be really nice if security policies can be changed _while the application is running_ so you don’t impact your end-users.  This is even better for security too, because if you see something wrong or dangerous (or you made a policy mistake), you can quickly change your policy to the correct configuration, and the software would still function normally.

So how can we enable these benefits?  We can do this by being _explicit_ as to what can actually be done in an application.  But what does that mean exactly?

When you look at the implicit access control checks above, what were they _really_ trying to do?  What was the most fundamental control being performed?  

Fundamentally, those checks are trying to protect **resources** (project reports) as well as what **actions** a user could do to those resources (e.g. view/read them).  When you break it down to this most primitive level, you can start to describe security policies in a much more fine-grained (and change-resilient) manner.

For example, let’s change the above code blocks to effectively do the same thing with resource-based semantics:

```java
//Listing 3. Example Explicit Access Control security check:
if (user.isPermitted("projectReport:view:12345")) {
    //show the project report button
} else {
    //don't show the button
}
```

This example is much more explicit as to what access is being controlled.  The colon-delimited syntax isn’t really important here – it is just an example.  More importantly, we are basically checking “If the current user is permitted to ‘view’ the ‘projectReport’ with ID ‘12345’, show the project report button”.  That is, we’re explicitly attributing a concrete behavioral statement about a _specific resource instance_ with a user account.

## How is this better?

This latest example above has a subtle yet very important distinction: the code is now based on what is being protected, not _who_ might have an ability.  Although apparently simple (and perhaps more obvious) conceptually, this has a major impact on development and production deployment practices:

* **Reduced Code Refactoring:** By basing code on _what_ the application can do, we’re basing our security on things that are core to the application itself and which change much less frequently – the resources that the application interacts with.  Using this approach, software developers can modify security checks as they work on the application’s functionality – not the other way around as implicit RBAC so often requires.

* **Resources and Actions are intuitive:**  Representing what is being protected and how it might be acted upon is a more natural way of thinking about the problem.  The Object Oriented programming paradigm and REST communication models fundamentally reflect this viewpoint and are very successful because of it.

* **Flexible Security Model:** The above code example does not dictate how a user, group, or role is permitted to perform an action on a resource.  This means any security model design can be supported.  For example, maybe behaviors (permissions) can be assigned directly to a user.  Or perhaps they can be assigned to a role, which is in turn assigned to a user.  Perhaps there is the notion of groups which have associations with roles, etc.  The possibilities are open and can be customized based on your application.

* **Externalized Security Policy Management:** Because the source code only reflects resources and behaviors and not combinations of users, groups, and roles, that kind of assocation management can be externalized to a different part of code or to a dedicated tool or administrative console.  This means developers don’t need to spend their time making security policy changes, and instead a business analyst or even end-user can make security policy changes as necessary.

* **Make Changes at Runtime:** Because security code checks do not depend on knowing how behaviors are associated (i.e. how groups, roles and users might be related), you can change those associations in an application’s security policy while the application is running.  There is no need to refactor code to enforce a new security policy change, as is always the case under implicit RBAC.

## The New RBAC: _Resource_-Based Access Control

In addition to the benefits listed above, I should reiterate the notion of the flexible security model that this explicit mechanism affords.

If you wanted to retain or simulate the traditional notion of role-based access control, you could assign behaviors (permissions) directly to a Role if you want.  In this sense, you would still have a Role-Based Access Control security policy – it is just you would have an explicit RBAC policy instead of the traditional implicit strategy.

But that begs the question – why stop at roles?  You can assign behaviors directly to users, or to groups, or to anything else your security policy might allow.

In this sense, and based on the benefits of explicit, resource-based access control above, RBAC could probably take on a new meaning: “Resource-Based Access Control”.  Just a thought…

## Real World Example: Apache Shiro

If you’re curious about how this can be modeled in a real-world framework used in thousands of real-world applications, check out [Apache Shiro](https://shiro.apache.org/), a modern framework for securing applications on the JVM platform.  Shiro supports Resource-Based Access Control out of the box via its [Permission](https://shiro.apache.org/permissions.html) concept.

Of course you don’t need to use Shiro to benefit from the ideas mentioned in this article, but it could be helpful if you’re looking for ideas or concrete examples of what was discussed.

## Reference

* [The New RBAC: Resource-Based Access Control](https://stormpath.com/blog/new-rbac-resource-based-access-control) by Les Hazlewood on January 15, 2012
