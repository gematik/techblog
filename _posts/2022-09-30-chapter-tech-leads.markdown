---
layout: post
title:  "We open a new chapter"
date:   2022-09-30 08:00:00 +0200
author: Axel Schulz
categories: culture
tags: learning culture chapter 
excerpt: "<br/>We recently switched the organizational structure of the software development department from a line reporting focus to a learning focus. This is how we did it and what we learned.<br/><br/>"
---

At gematik, we work in a matrix organization like many medium or large companies. The idea of a matrix organization is to group different skills in vertical cross-functional product teams where the work is done (the What) while departments, like Software Development, are taking care of the technical education of their employees on a horizontal level across product teams (the How). 

[![Sketch of the matrix organization @gematik]({{site.baseurl}}/assets/img/220930-chapter/matrix_gematik.png)]({{site.baseurl}}/assets/img/220930-chapter/matrix_gematik.png)
*Sketch of the matrix organization @gematik (gematik GmbH)*

While this model provides an undeniable scaling effect, it bears some challenges that should be taken into account:
- While the teams may work on different products, they often face the same issues during development. We need to make sure to have a cross-product exchange of solutions
- Our teams mostly use the same essential technologies, Java, SpingBoot, Maven, and REST APIs - only very few teams are really unique in their tech stack, like the mobile app teams. We need to ensure that people are skilled and trained in the required techs and that we can help across teams.
- Teams have different sizes depending on their needs. We have teams with only 1 developer, so we have to provide a home base for these people.
- The future need for skills in a team depends on its product evolvement In a matrix organization, there is a natural disconnect between the product team and the department. So, we more often reacted to a new demand instead of being prepared.
This is why the Software Development department recently switched its structure.
Previously, we had 2 teams covering 20 people each that supposedly focussed on different topics trying to avoid the caveats described above.
This resulted in 2 teams giving their best in keeping the balls in the air but eventually losing focus on what the company needed.

Therefore, we decided to give up the team structure and form a chapter structure (as explained in the (in-)famous Spotify model) around Technical Leads as Chapter Leads to structure the How for the product teams more efficiently.

[![Differences between Chapter Leads and Product Owners]({{site.baseurl}}/assets/img/220930-chapter/how_what.png)]({{site.baseurl}}/assets/img/220930-chapter/how_what.png)
*Difference between Chapter Lead and Product Owner ([Henrik Kniberg & Anders Ivarsson, Oct 2012](https://blog.crisp.se/wp-content/uploads/2012/11/SpotifyScaling.pdf))*

## What's a Chapter & Chapter Lead for us?
"The Chapter's more of a regular's table for its members, a place where they can touch base with like-minded and can discuss technical things. The Chapter also defines its guidelines regarding work processes (documentation, coding, etc.)."

We define a chapter as a group of people that
- shares similar expertise (or is interested in it)
- shares knowledge, experience, and best practices internally and with other chapters
- is led by a senior developer (tech lead)
- has a size of max. 10 regular employees

There are many definitions for what a Chapter Lead (CL) is and what is required to become one. For us, we defined it as a senior developer who can 
- create a vision for technical topics
- mentor other developers
- increase the passion for development among the Chapter members
- organize joint efforts for education and training. 

For that, the Chapter Lead is free to organize the Chapter and communication as seems necessary.<br/>
 He (there are only male leads in our case at the moment) is responsible for 
 - creating the vision for the Chapter
 - setting up and maintaining  the Chapter Backlog
 - supporting team members in becoming experts in the Chapter's mission topic.

A Chapter Lead does not decide on salary rises, approve vacation days or hire people. But a CL will always give valuable input and information regarding these topics to the line manager.

## How did we do the setup?

### 1. Cut the elephant into pieces
Once the goal was decided, we had to figure out how to get there, and the first step was to carve out the chapters.
Two approaches were discussed:
- by technology, we would put Java devs in one squad, Kotlin devs in another, and those working on PKI topics in a third
- by value stream, we would look at the recurring topics in most of our product teams, group them into categories (e.g., Cloud Services, Interoperability via FHIR/REST, Web Applications, etc.) and build the chapters around that
We finally decided on the value stream approach as we thought it would help the product teams directly and also make the benefit of the system more transparent.

### 2. Find the leads
The next step was to find potential Chapter Leads. We knew we were looking for experienced developers that already stood out in discussions around technology and development culture and where we would trust that they would take it as a long-awaited chance to get things moving.
I spoke to each candidate personally, and everyone was immediately on fire for the position, shooting ideas of what could be done and achieved in this setting.
That showed we were on the right track! 
Every candidate received a "First 100 days expectation"-email giving scope for the role. It is important to emphasize that the position was created as a tech lead. No disciplinary management tasks were involved. The message was: "That's your topic and your people - you are responsible for growing both with any support from management that it takes!"

### 3. Start the work
To create an environment for the chapter leads and developers where the education and growth of knowledge could bloom, we had to make a few yet fundamental decisions about the freedom of the Chapter. Some issues were easy as they were clearly defined by our company guidelines, e.g., who approves vacation requests. Others required deeper discussion like educational budget or flat rate time for chapter work.
In the end, we had pretty good documentation of the goal, framework, and responsibilities within our supposed chapter structure. 
This month the chapters started their work with kick-offs, defining their respective work and focus topics. We created backlogs and boards in JIRA, so it's transparent to everyone what each Chapter works and what the relevant topics are. Initially, we took the most pressing issues from the Product Teams as input for the backlogs in the respective teams. For instance, product teams may require a common standard for API formats or want to create a tool that would also help other teams. The Chapter would then try to add a holistic expert view and support with knowledge and sometimes even staffing.

## Do we recommend it to everyone else?

The setup part definitely - even if we would not have set up the new structure, it already helped us to understand our work.<br/>
During that phase, we learned a lot about our processes, responsibilities, required roles, and daily work.  

But in the end, it depends on your individual situation. If you're facing some of the issues listed in the first paragraph of this post, it could be an option - but be aware that you need to find your own way to make it work. This post can only give you some things to think about.

## Does it work?

We don't know - yet. We just started our journey and did our initial "leap of faith". Regular reviews of processes and achievements are part of the process and will bring insides on improvements and maybe even metrics for measuring our success. 

I'll keep you all posted!

# About the author
Axel Schulz leads the Software Development unit at gematik since 2022. He has worked in development and engineering management for 20 years. His focus topics include product design, developer experience, and tech leadership. In his previous stations, Axel was the CTO of several healthcare companies. Axel lives with his family 500m behind Berlin city limits.
