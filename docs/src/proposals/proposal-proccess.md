---
title: Solana Proposal Process
---

# PR Desc:

## Current Proposal Process Shortfalls

The current proposal process is as follows:

1. Discuss across core engineering about a proposed feature in Discord and/or
github issue(s).
2. Create a proposal document under
[docs/src/proposals](https://github.com/solana-labs/solana/tree/master/docs/src/proposals).
3. Review is done by core engineering + community members
4. PR Is pulled into `Accepted Design proposals` and live on [Solana
Docs](https://docs.solana.com/proposals/accepted-design-proposals)
5. If required, a `feature-gate` issue is created and kept up-to-date to track
the feature's activation.
6. If the proposal needs to be tweaked, submit a new PR and have the change
reviewed by the same group in step 3.
7. Once implementation is complete, submit a PR that moves the link from
`Accepted Design Proposals` to `Implemented Proposals`.

While this is a good flow to fast iteration, there are a few shortfalls:

- Not all implementation proposals make it to the docs
- Design documents don't have any standard format and miss a lot of information
- Discussion and feature-gating are difficult to keep track of by community

## Proposal for process:

- New site dedicated to proposals
- Have a more formal process for proposing new features
- Consensus is owned by a committee of core contributors. As labs is the
biggest current contributor of the validator, consensus should be similar to
what it is today.

I know process is generally disliked, but I hope this process can help onboard
new core contributors as well as other validator implementation in the future.

The main changes to the current process:

- More formalized documentation
- Formal "committee"

The process proposed was based on both RFC and EIP.

---

## What is a Proposal?

A proposal is a design document describing a new feature for Solana or its
processes or environment. The proposal should document the rationale for the
feature and enough documentation to understand the implementation.

## Rationale

A proposal is intended to be reviewed by core engineering and community members
keeping in mind security concerns, tradeoffs, and backwards compatibility.
Having a proposal process helps identify design issues early, alert the
community on a change, helps scale newer contributors on the architecture, and
acts as a historical record of the design decisions that have gone into Solana.
For implementors, the proposal is a blueprint for the feature and helps track
the development of a feature.

## When you need to follow this process

You need to follow this process if you intend to make "substantial" changes to
the Validator, RPC, consensus, or a change to the proposal process itself. What
constitutes a "substantial" change is evolving based on community norms and
varies depending on what part of the ecosystem you are proposing to change, but
may include the following:

- A change in format of a RPC API method
- Networking interface changes between validators
- Compute requirement changes on the runtime

Some changes do not require a proposal:

- Rephrasing, reorganizing, refactoring, or otherwise "changing shape does not
change meaning".
- Additions that strictly improve objective, numerical quality criteria
(warning removal, speedup, better platform coverage, more parallelism, trap
more errors, etc.)

## Proposal Types

There are two types of proposals:

- A **Standard** Proposal describes any change that affects most or all Solana
implementations, such as a change to the network protocol, consensus, proposed
application standards/conventions, block or transaction validity, or any change
or addition that affects the interoperability of applications using Solana.
Standard proposals can be broken down into the following categories:

  - **Core**: Anything that affects consensus or substantial changes to the
validator.
  - **Networking**: Changes or substantial improvements to network protocol
specifications.
  - **Interfaces**: Improvements around the client JSON RPC API specifications
and standards. 

- A **Meta** Proposal describes a process surrounding Solana or proposes a
change to (or an event in) a process. Process Proposals are like Standard
Proposals but apply to areas other than the Solana protocol itself. They may
propose an implementation, but not to Solana's codebase; they often require
community consensus; unlike Informational Proposals, they are more than
recommendations, and users are typically not free to ignore them. Examples
include procedures, guidelines, changes to the decision-making process, and
changes to the tools or environment used in Solana development. Any meta-EIP is
also considered a Process Proposals.

## Proposal Lifecycle

The stages in a lifecycle of a proposal are as follows:

- Idea
- Draft
- Review
- Accepted
- Stagnant
- Withdrawn

![Proposal Process](proposal-process.png "Proposal Process")

### Idea

At the idea stage, parties involved in the proposal are you, the champion or
proposal author, the reviewers, and the Solana Core Contributors.

Before you begin writing a formal proposal, you should vet your idea. Ask the
Solana core community first if an idea is original to avoid wasting time on
something that will be rejected based on prior research. It is thus recommended
to discuss the proposal on the Solana Tech Discord under the #core-technology
channel.

### Draft

To begin drafting the proposal, do the following:

- Fork the proposal repository
- Copy `proposals/0000-template.md` to `proposals/0000-my-feature.md` (where
"my-feature" is descriptive)
- Fill in the proposal. Put care into the details: proposals that do not
present convincing motivation, demonstrate lack of understanding of the
design's impact, or are disingenuous about the drawbacks or alternatives tend
to be poorly received.
- Submit a pull request.
- Now that your proposal has an open pull request, use the issue number of the
PR to update the `0000-` prefix to the number.

### Review

During review, the owner of the proposal is in charge of gathering and
integrating feedback. The most relevant core contributors to the proposal
should be included in the review process. Review will take place completely
through Github so that all comments are collected and documented. Once
consensus is met by the core contributors, the proposal can either be accepted
or withdrawn.

### Accepted

Some accepted proposals represent vital features that can be implemented right
away. Other accepted proposals can wait until some arbitrary core contributor
feels like doing the work. Every accepted proposal should have an associated
tracking issue in the Solana repository. If the implementation requires the
feature to be activated using the feature activation program, a feature-gate
issue for tracking across clusters should also be created. While it is not
*necessary* for the proposal author to also write the implementation, it is by
far the most effective way to see a proposal through to completion: authors
should not expect that other project developers will take on responsibility for
implementing their accepted feature.

### Stagnant

If a proposal reaches an amount of time without activity, the proposal will be
marked as stale to be closed. A new proposal can be opened if the proposal is
closed and has a chance of reaching consensus.

### Withdrawn

The author has withdrawn the proposal. This state has finality and can no
longer be resurrected. If the idea is pursued at a later date it is considered
a new proposal.

