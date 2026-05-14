---
title: "Adopt AI to Augment Your Capabilities"
date: 2026-05-14T14:56:36-04:00
draft: false
---

I'm writing this because I keep meeting smart people, my spouse, my colleagues, the graduating class at UCF, who booed their commencement speaker for mentioning AI, who have concluded that the technology is hype, harm, or both. They have reasons. Sam Altman did stage a regulatory capture performance in front of Congress. CEOs are absolutely using "AI efficiency" as cover for COVID-era overhiring. Microsoft did ship Copilot before it was ready. The press runs scare stories because scare stories convert. Every one of those grievances is legitimate, and none of them is an argument about what the tools can do on your laptop on a Thursday morning.

I'm a Chief Architect for a large organization. I tend to prefer small, lean teams because there's less management overhead. However, we support a much larger user base, which makes it difficult to perform architectural audits at scale. Auditing over a thousand Git repositories is no one's idea of a good time. I'm going to describe a task that was continuously put off because of other higher-priority items, but was still important. This is an example of how I use AI to augment my work, and why your resistance to AI will cost you your job.

The skeptics' strongest case isn't the noise; it's the data. Economist Acemoglu's productivity estimates are modest. METR's developer study found that engineers were slower than they felt. Entry-level roles in software, support, and copywriting are visibly contracting. I take all of that seriously. My claim is narrower than the boosters' and harder than the doomers': at the individual level, being in the top quartile of AI-augmented workers in your function is a defensible position regardless of what the macro numbers do.

## A 1,000 repos, one weekend, one architect

As an enterprise architect, I wanted to audit the architecture of every software project in our GitHub organization, over a thousand repositories, accumulated over 4 years, owned by teams that have since reorganized or dissolved. The conventional version of this audit is a staffing request: two or three engineers, six to eight weeks, a spreadsheet nobody updates, a slide deck nobody reads. I've watched that exact project run multiple times in my career. It never finishes, and the artifact is stale the day it ships. Also, I'm not going to get this staffed or funded.

Instead, I spent a weekend building it with Claude Opus inside Microsoft Copilot.

The architecture is two Rust services. The first service uses Rig to orchestrate an agentic loop: clone a repo, identify architecturally significant files (Dockerfiles, Terraform, Helm charts, CI configs, package manifests), pass them to a local Qwen model for analysis, and write structured output to SQLite. Qwen runs on my MacBook. Local inference cost me electricity and a Saturday of fan noise. The quality tradeoff is real. Qwen misses the nuance Opus would catch, but for "what is this repo, architecturally?" 

The second service is an Axum web app that reads the SQLite database and renders an infrastructure map in the browser, styled with Tailwind. You can filter by AWS service, Databricks service, Kubernetes, language, and other technologies. You can see that 340 services touch S3, that 47 of them reinvent the same Kafka consumer pattern, and that three different teams are running their own Postgres on EC2 instead of using RDS. The map answers questions in seconds that the conventional audit takes months to answer, if it answers them at all. (All specifics have been changed to not reveal actual tech stacks and numbers, but you get the idea)

I wrote far less of the implementation than I normally would have. I described what I wanted, reviewed what came back, redirected when it went off course, and pushed back when it tried to over-engineer. The Rust compiler caught most of the model's mistakes for free, which, separately, is an argument for choosing Rust as your AI-augmented language of choice. The type system is a second reviewer that never gets tired. Also, running Rust's linter, Clippy, keeps token usage down.

That's the build. The interesting part is what it implies. I bet your org has tons of teams with wish lists of software that would not get funded because it was too expensive. AI has changed that!

## Four questions before you build anything
If you want to do what I did, you don't need a course. You need a diagnostic. Before you write a prompt or pay for a subscription, ask four questions about a piece of your job:

### Is the work repetitive enough that you've stopped thinking about it?
The architecture dashboard was. The 1,200-repo audit was at the per-repo level. Every clone-walk-describe loop is the same. Repetition is the precondition. If every instance of the task requires fresh judgment, AI augmentation buys you less than you'd hope. If the instances are structurally identical and only the data changes, augmentation is straightforward.

### Are the inputs and outputs well-defined?
The audit's input was "a Git URL." Its output was "a few rows in SQLite with these columns." Tight contracts at both ends make the middle tractable for an agent. Vague contracts make the middle a hallucination factory. Be specific!

### Are you the bottleneck, or is something else?
If your bottleneck is approvals, vendor negotiations, or waiting on another team, AI doesn't help. If your bottleneck is your own throughput on a defined task, it helps enormously. The 1,000-repo audit was bottlenecked on me. So is most knowledge work, if you're honest about it. Also, consider if the bottleneck can be automated away.

### Can you verify the output cheaply? 
This is the one most people skip. The audit was verifiable because I could spot-check 20 random repos against their actual architecture and confirm Qwen got the major components right. The dashboard was verifiable because I reviewed it. If you can't cheaply check the work, you've built a liability, not an asset. The Air Canada chatbot ruling is a cautionary tale: the airline was held to a refund policy its chatbot had invented because no one verified the output before letting it talk to customers.

This is the actual content of "AI fluency". Not Delegation, Description, Discernment, Diligence as a curriculum, but those four diagnostics applied to your specific work. The 4 Ds are fine as a memory aid. They're worthless as a substitute for sitting down with your actual job and asking which parts of it pass the four-question filter.

Once you've found a piece that passes, build the smallest possible version. The 1,000-repo audit dashboard started as a well-formulated prompt running in CoPilot CLI in plan mode. Initially, I  ran it against five repos. Most AI projects in large companies fail because someone tried to boil the ocean, "transform the org with AI", instead of automating one thing and seeing what happened. Start small.
What I'm actually claiming

I don't know what the labor market looks like in five years. Neither does anyone telling you they do. The honest answer is that the people who claim to know are selling something. Usually a book, a course, or a regulatory framework that benefits their incumbent position.

What I know is narrower and harder to dismiss.

In my own work, I am visibly faster than I was eighteen months ago. I prototype faster because Rust + Opus together produce compilable first drafts of services I'd otherwise spend a day scaffolding. The gap between me and the version of me from two years ago is large enough that I can feel it when I switch contexts to a tool that doesn't have AI assistance.

The colleague who can compress a six-week audit into a weekend is harder to lay off than the colleague who can't. That is the entire claim. Not "augmented or unemployed." Not "you have been warned." Just: there is a widening gap between the two versions of the same role, and the gap is currently bridgeable for the price of a $20/month subscription and a few weekends of attention.

If the macro picture turns out grim, if AI does compress labor demand in your function, being in the top quartile is still the defensible position. If the macro picture turns out benign and the productivity gains diffuse evenly across the workforce, you've still spent a few weekends learning tools that made your current job easier. The downside of trying is small. The downside of not trying, if the skeptics are wrong even partially, is large. That asymmetry is the entire opportunity, and it's the only argument I'm making.

The boos at UCF were not stupid. They were a rational response to a discourse that has spent three years telling that audience their futures are gone. The discourse is wrong about the inevitability of that outcome, but it's right that nobody is coming to save them. The save, if there is one, looks like the four questions above applied to one workflow at a time, starting this weekend.
