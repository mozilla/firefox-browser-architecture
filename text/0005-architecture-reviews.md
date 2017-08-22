# Architecture Review Process

## Introduction

Architectural Review decisions within the Firefox organization fall into two categories: "Roadmap" and “Design”.

Product Reviews are not the scope of this process. This process is about architectural change, which should ultimately be in support of product features, but will generally have an effect on multiple product features. Architectural change can be growing new skills or in reducing large-scale technical debt.

The function of a **Roadmap Review** is to decide if a thing should be done. The goal is to bring together a packet of data to inform a management decision to provide resources to make the thing happen. A Roadmap Review should happen early in the process so that build time isn’t wasted on a "No" decision, but so that enough information is available to management.

A **Design Review** is to decide if the thing should be done in a particular way. The goal is to ensure that the problem space is understood well enough that a realistic bug breakdown can be made, that those working on the problem all agree on the challenge, and that work can be prioritized to hit the hardest parts of the problem first.

## Timetable

For both a Roadmap Review and a Design Review the **timetable** is as follows:

1. Lay the stage. The team should document (See "initial meeting" below):
  * Someone to **chair** the review
    - For a Roadmap Review this will be the *least* senior person in the management chain with the ability to provide resources to the problem.
    - For a Design Review this will be the *most* experienced engineer outside of the team willing to put time into the problem. As engineers don’t review their own code they shouldn’t review their own designs.
  * The **question** (or questions) to be resolved, which should be agreed between the team asking for review and the chair. For a Roadmap Review this is likely to be in the form of a Go/No-Go decision. For a Design Review this is likely to be a more open question (e.g. "How should this design be improved")

2. The team produces a **Review Packet** designed to answer the question identified in step 1. The production of the Review Packet is expected to be collaborative and iterative to ensure that the review focuses on the key questions and the team presents its best case. This is particularly true of a Roadmap Review where the consequences of the answer are likely to be more significant.
The Review Packet includes:
  * A lay summary of the problem space and the stakeholders which is focused on defining a shared language and identifying the key forces behind the problem
  * A brief explaining what the team proposes. This should read more like an encyclopedia entry than a marketing document. The audience is the people in the review; i.e. this should attempt to plug organizational documentation flaws, the documentation process should not be more onerous than is required for the review. The brief should identify:
    - the forces on the system
    - the essential architecture behind the proposal
    - scenarios that exercise the relevant forces against the proposed architecture
    - how the proposal handles these scenarios
  * A competitive analysis suggesting alternatives, costs, and opportunities.

3. The chair creates a list of questions to be discussed during the review. These questions should be:
  * Open-ended
  * Made available to the the team prior to the review so they can prepare answers

4. The chair convenes a review meeting. The discussions of this meeting should be carefully minuted. The meeting comprises:
  * The team making the proposal
  * The chair
  * (For Design Reviews) other senior engineers who will provide valuable input.

The review meeting should be a discussion of the issues, but should avoid feeling pressured to make a decision. The goal is to understand the issues raised by the question(s) and the background from the review packet, and to add to it wisdom from the people at the review. The meeting may decide to alter the questions and undergo a subsequent review. For questions that are go/no-go in nature the decision does not need to be taken during the meeting, although we should be very open about conversations that happen after the review meeting.

The ideal review process scales well. The same basic system should work for a quick 2 person decision over the best way to design a certain feature, and for a critical organization wide decision about a path to take. The term “team” is used above although we strongly recommend design reviews for smaller questions. If you feel yourself wondering if some design is best, it should be easy to valuable to perform a review.

For both a Roadmap Review and a Design Review the goal is to hear perspectives that will lead to a better definition of a problem or solution, and better judgment about whether it should be solved or how to solve it.

## Rationale

### Key considerations

* What are the desired **outputs** of the review process?
* Which **audience** is intended to consume the outputs of the review process?
* What committee **structure** is best positioned to deliver those outputs?
* What are the **inputs** that the committee should evaluate?
* What size and **maturity** should projects achieve before undergoing review?
* When should review results be **disseminated**?

### Outputs and audience

The first two considerations define the goals of the review process and are two sides of the same coin.

The audience of a Roadmap Review is those "up the chain" of decision makers and the audience of a Design Review is those running “down the chain” of implementers.

A Roadmap Review should focus on:

* The problem space
* Large forces impacting Mozilla’s choices in the space
* Competitive analysis
* Opportunity cost

The key outcome is senior management investing in the problem space and building consensus around a shared reality.

A Design Review should focus on:

* The solution space
* Technical roadmaps
* Milestones and timelines
* Resource allocation

The key outcome is middle management and individual engineers understanding the value proposition and the path to success.

In [RACI](https://en.wikipedia.org/wiki/Responsibility_assignment_matrix) terms, the team is Responsible, the chair is Accountable, other senior engineers or managers are Consulted and various others in the organization are Informed.

### Structure and inputs

The second two considerations shape how the review process actually happens.

The chair and the team frame the question or questions at the start of the process to ensure agreement on what should be tested. The chair is accountable for answering the question or questions following the review process.

In order to agree on the question(s), the chair and the team/technical lead or principles will generally have an initial meeting. This meeting should be as small possible and is intended to:

* Establish vocabulary
* Help the team "argue the right level" for the review
* Identify stakeholders and domain experts — those Consulted, per RACI — in the organization

Subsequently, the team prepares a review packet detailing the problem space and their approach.

The goal is to have the chair (representing the responsible decision makers) and the team establish some shared reality informally, and then to have the team document this and explain how their work addresses the decision maker’s question within that reality.

The chair then evaluates and refines the team’s document with the assistance of domain experts and the identified stakeholders. Hopefully the consulted stakeholders concur with the framing of the problem and are aligned behind the architectural approach, but perhaps they are not. The chair collects such concerns and presents them. This should be a collaborative process whereby the architectural review functions to document a problem and a proposed solution, in which case the chair can solicit response from the team.

Involving the team and the chair informally and subsequently sharing ownership of a document is designed to seed a collaborative process. A particular benefit is avoiding drift between the proposal and the focus of the review: that is, a "successful review" that results in a “go” or “no go” from responsible decision makers in which the review doesn't actually evaluate what the team proposes.

### What to review and what to say after review

The final two considerations shape what is reviewed and how the review concludes.

A project under review will be in one of the following states:

* The "Roadmap" stage: the team has a proposal, and wants broader buy-in to justify investment in the problem space.
* The "Design" stage: the problem space is considered valuable, and the team wants to commit to one path through the solution space.
* The "Execution" stage: the team is actively pursuing a solution.
* The "Landing" stage: the team is verifying the solution meets expectations.

Generally our review process focuses on projects in the “Roadmap” or “Design” stages. However a project in the “Execution” or “Landing” stage may need subsequent review to decide if the project is being done the correct way, or if it should be continued at all, so it may return to either of the “Roadmap” or “Design” phases.

After the review, the chair and responsible decision makers should provide concrete recommendations and examples for the next teams to undergo the review process. The results should be disseminated as widely as possible, with the package documents and recommendation document archived. Some version of the decision should be publicly linked, so that it can be the decision of record, to which blog posts, mailing list posts, the message saying that the GitHub repository is obsolete, etc, can refer. The goal is to avoid critical technical and resourcing decisions being internally settled and only — later, begrudgingly — being made public on some mailing list. Or, per RACI, the goal is to broaden the set of those Informed.

## Additional Reading

Multiple Perspectives On Technical Problems and Solutions - John Allspaw - 2017

* https://www.kitchensoap.com/2017/08/12/multiple-perspectives-on-technical-problems-and-solutions/

Architecture Reviews - Grady Booch - 2010:

* https://www.researchgate.net/publication/224132839_Architecture_Reviews
* http://media.computer.org/sponsored/podcast/onarchitecture/onarch-025-v.mp3

A Toolbox of Software Architecture Review Techniques - Jason Baragry - 2014, 2015

* https://swarchitectonics.blogspot.co.uk/2014/12/a-toolbox-of-software-architecture.html
* https://swarchitectonics.blogspot.co.uk/2015/02/a-toolbox-pt2.html
* https://swarchitectonics.blogspot.co.uk/2015/09/a-toolbox-pt3.html

Software Architecture Review and Assessment (SARA) Report - Kruchten et al. - 2002

* https://pkruchten.files.wordpress.com/2011/09/sarav1.pdf
