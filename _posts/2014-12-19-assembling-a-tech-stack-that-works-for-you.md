---
layout: post
title:  "Assembling a Technology Stack That Works for You"
date:   2014-12-19 22:34:11
categories:
---
Since entering software development in 1998 I've put together technology stacks for many different projects and every time
the solution is different; unique to the project and the team at the time. The success of particular tech stacks is so
interesting that http://stackshare.io/ exists. You can go there and read about lots of different technology stacks that
teams have used to successfully deliver on their promises. What works for one team my not work well for another. What
makes tech stacks unique and what is it about them that boosts a team's success (or aides in failure)?

I believe that the most important thing about a technology stack has nothing to do with the specific pieces of the stack at all. The
most important thing about it is the team's ability to use and evolve the stack to fit the needs of the team and the products that
it is building and supporting. The tech stack that drives success is intimately tied to the team, and how the team operates. It is
driven by what drives the team, what excites the team; what does this team believe in and what is it passionate about?

Therefore there is no magic bullet. This is why there are so many variations of technology stacks on StackShare; they are as
varied as the teams using them.

What are some of the high level things to look at on your team to see if the stack that you're assembling is a good fit?

* Complexity (solutions are complex or simple?)
* Size (Quantity work output)
* Team skillsets (programming, ops, dev ops, and all the associated tools and languages)
* Attitude (Anything goes? Wants rigid interfaces and strong OO principles?)
* Fault tolerance (Can we handle a good percentage of issues? Or must never fail in production?)

What about the evolution of the stack?

My tech stacks (at least for passion projects) invariably start off as:

1. Google App Engine
2. Web App 2
3. A single Python file
4. Jinja2
5. HTML/CSS and/or JSON

This starter stack is tailored for me, as a one or two person team. It is the easiest, and fastest to assemble and enables me to succeed at producing
a good amount of functionality very fast that works most of the time. This stack handles change extremely well as almost no time is spent
on code organization.

From here, one direction this stack can evolve is:

1. Google App Engine
2. Django
3. Multiple django apps (instead of one python file)
4. Jinja2
5. HTML/CSS and/or JSON

From here, maybe we need more control over aspects of our platform than are available on GAE. One such request from a team that is common is
to instrument the python code for performance tuning. This difficult on Google App Engine so let's evolve this tech stack to address this
requirement:

1. Rackspace Cloud Server
2. Ubuntu
3. MySQL
4. Python 2.7
5. Cloudflare
6. Django
7. Jinja2
8. HTML/CSS
9. New Relic Python module (instrumentation)

Here I've made a few choices to keep everything steamlined for a small team, but I hope to highlight here how the tech stack itself
isn't the most important thing to consider for success; rather consider your team and also how you will evolve your tech stack along
with your team, your company, and your products.

