---
title: Architecture Review Process
layout: text
---

**Version**: 0.1

**Warning**: This document is incomplete. We've added it to the repo before it's finished because we would like a couple of trial runs before we say we're happy, and we would like something to refer to during those reviews.

# Architecture Review Process

## Introduction

This process targets Architectural Reviews in two categories: “Roadmap” and “Design”. It doesn't tackle how to review in-progress projects to see if they should continue.

The function of a **Roadmap Review** is to decide if a thing should be done. The goal is to bring together a packet of data to inform a management decision to provide resources to make the thing happen. A Roadmap Review should happen early in the process so that build time isn’t wasted on a “No” decision, but so that enough information is available to management.

A **Design Review** is to decide if the thing should be done in a particular way. The goal is to ensure that the problem space is understood well enough that a realistic bug breakdown can be made, that those working on the problem all agree on the challenge, and that work can be prioritized to hit the hardest parts of the problem first (in order to get early warning if the hard parts prove impossible or to inform schedule projections early).

## Timetable

For both a Roadmap Review and a Design Review the **timetable** is as follows:

1. The team should set the stage:
    * Summarize in one or two sentences the proposal to be reviewed. This should be agreed between the team asking for review and the reviewer.
    * Find a **reviewer** (See “Outputs and Audience” below for the rationale):
      - For a Roadmap Review this will be the least senior manager with the ability to address the problem in its entirety.
      - For a Design Review this will be an experienced engineer in the problem domain outside of the team willing to put time into the problem. As engineers don’t review their own code they shouldn’t review their own designs. The principle is to get the most informed and least biased feedback possible.
    * Optionally, for larger reviews, find someone to **chair** the review. This can facilitate the process (i.e. scheduling the meeting, managing the clock, ensuring minutes are taken, etc.) and enables the reviewer to concentrate on the review itself. The rest of this document uses 'chair' for the administrative role. For smaller reviews the reviewer also does the tasks of the chair.

2. The team produces a **Review Packet** designed to document the proposal identified in step 1. The Review Packet includes:
    * A lay summary of the problem space that defines a shared language and identifies the key forces behind the problem.
    * A list of the groups directly affected by the proposal to ensure that the review meeting includes representatives from those groups.
    * A vision of what the project will achieve on completion.
    * A brief that explains what the team is proposing. This should read more like an encyclopedia entry than a marketing document. The audience is the people in the review; i.e. this should attempt to plug organizational documentation holes. The documentation process should not be more onerous than is required for the review. The brief should identify:
      - the architecture being proposed
      - the non-functional requirements, like accessibility, performance, security, and stability
      - relevant business goals, such as improving user satisfaction and retention
      - scenarios that exercise the requirements/goals against the proposed architecture
      - how the proposal handles these scenarios
      - what review and discussion of the proposed architecture has already happened (and with whom)
    * A competitive analysis suggesting alternatives, costs, and opportunities
    * Input by representatives from the affected groups. The team may update the review packet based on this input but is not required to answer all questions and address all comments before the review.

3. The reviewer creates a list of questions to be discussed during the review, possibly from the comments made in step 2. These questions should be:
    * Designed to foster productive analysis and discussion
    * Made available to the the team prior to the review so they can prepare answers

4. The chair convenes a review meeting. The discussions of this meeting should be carefully minuted.
    * The meeting should have a limited attendance to avoid a sprawling unfocussed meeting, but it should include:
      - The team making the proposal
      - Representatives from the affected groups, taking care to:
        + Select a full range of perspectives
        + Avoid “stacking the deck” with too many people that think the same way
    * A suggested agenda for the review meeting:
      - Short opening statement by the team on the current state of play
      - Discussion of Questions and Answers (see 3. above)
      - A step back to discuss if the proposal is right in the wider context
    * A lightweight review might not need the input of others, and ultimately, might not even need a specific review meeting.

The review meeting should be a discussion of the issues, but should avoid feeling pressured to make a decision. The goal is to understand the issues raised by the question(s) and the background from the review packet, and to add to it wisdom from the people at the review.

A generally positive review is likely to have a number of action items which the team should respond to before the review is complete. The reviewer is responsible for enumerating the action items, validating the answers and ultimately deciding the outcome of the review.

A more negative review may decide to alter the questions, undergo a subsequent review or drop the subject entirely. It is a goal of the process that we discover fatal flaws as early on in the process as possible.

The ideal review process scales well. The same basic system should work for a quick 2 person decision over the best way to design a certain feature, and for a critical organization-wide decision about a path to take. The term “team” is used above although we strongly recommend design reviews for smaller questions. If you feel yourself wondering if some design is best, it should be easy to perform a review.

For particularly small reviews it is possible to merge a roadmap review and design review into a single review, in which case the question is both 'should we do this', and 'should we do it this way'. It is possible to conduct combined reviews where the total time for the review for all participants is 1 working day.

The results of the review (including review packet and reviewer summary) should be published and communicated widely, with the review packet and results archived. Some version of the decision should be publicly linkable, so that it can be the decision of record, to which blog posts, mailing list posts, can refer.


## Rationale: Outputs and Audience

The first two considerations define the goals of the review process and are two sides of the same coin.

The audience of a Roadmap Review is those “up the chain” of decision makers and the audience of a Design Review is those running “down the chain” of implementors.

A Roadmap Review should focus on:

* The problem space
* Large forces impacting Mozilla’s choices in the space
* Competitive analysis
* Opportunity cost

The key outcome is firstly, senior management investing in the problem space and building consensus around a shared reality. Secondly, to provide documentation of both the goals and assumptions of the project to aid later evaluation.

A Design Review should focus on:

* The solution space
* Technical roadmaps
* Milestones and timelines
* Resource allocation

The key outcome for middle management is to understand the value proposition and the path to success. The key outcome for engineers is firstly to understand the design constraints and the tradeoffs between them, which is important to avoid unnecessary re-work when implementation reality causes a re-think. Secondly, to provide a proposal detailed enough to begin an initial breakdown of engineering work for the proposal (although the work implied by the proposal should not be filed prior to the completion of the review).


## Examples

A number of reviews have been completed and can be used as a template for a successful architectural review. [XBL Removal](0007-xbl-design-review-packet.md) was the first example of a design review, [Sync and Storage](0008-sync-and-storage-review-packet.md) was the first roadmap review, however any of the documents on this website with a title that includes "review" could be used.

## Additional Reading

### Multiple Perspectives On Technical Problems and Solutions - John Allspaw - 2017

Describes how Etsy's architecture review process evolved. A solid write-up of the social side of an Architecture Review process in an environment that is likely closer to Mozilla's than many of the other write-ups.

> Fundamental: engineering decision-making is a socially constructed activity

> We called these “something new” things departures. ... the basic idea is that there are costs ... for making departures that attempt to solve a local problem without thinking about the global effects it can have on an organization.

* https://www.kitchensoap.com/2017/08/12/multiple-perspectives-on-technical-problems-and-solutions/

### Architecture Reviews - Grady Booch - 2010

A formal write-up from someone who spends a significant time in Architecture Reviews. I've not found the paper online, but the MP3 is an audio version of the paper (with bonus guitar-riff intro). This IBM process suffers from lack of attention to the social aspects, but is helpful in digging into the mechanics of the review itself.

* https://www.researchgate.net/publication/224132839_Architecture_Reviews
* http://media.computer.org/sponsored/podcast/onarchitecture/onarch-025-v.mp3

### A Toolbox of Software Architecture Review Techniques - Jason Baragry - 2014, 2015

Jason Baragry reviews approaches to architecture review and notes some potential pitfalls.

> Architecturally significant requirements are often hard to identify, the architecturally significant decisions are often not documented, and the way the decisions interrelate is often not easily understood. A significant part of the review process is often teasing these out.

* https://swarchitectonics.blogspot.co.uk/2014/12/a-toolbox-of-software-architecture.html
* https://swarchitectonics.blogspot.co.uk/2015/02/a-toolbox-pt2.html
* https://swarchitectonics.blogspot.co.uk/2015/09/a-toolbox-pt3.html

### Software Architecture Review and Assessment (SARA) Report - Kruchten et al. - 2002

Formal process with input from a number of companies which targets a more enterprise environment than exists at Mozilla, however the process is clearly defined.

> A software architecture is a set of concepts and design decisions about structure and texture of software that must be made prior to concurrent engineering (i.e., implementation) to enable effective satisfaction of architecturally significant, explicit functional and quality requirements, along with implicit requirements of the problem and the solution domains.

* https://pkruchten.files.wordpress.com/2011/09/sarav1.pdf
