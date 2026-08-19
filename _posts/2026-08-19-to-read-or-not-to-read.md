---
layout: post
title: "To read or not to read?"
subtitle: "(code)"
date: 2026-08-19
description: When (and when not) to read the code produced by your coding agent.
---

A debate is raging on X: do we need to read the code produced by our favourite coding agent? There are prominent figures representing both viewpoints, and users are at each other's throat in typical X fashion.

The debate is not without merit, and I wanted to share my point of view, because I think there are a few points the contestants are missing. First of all, the debate is often framed as "AI writes code, therefore you must review every line." Engineers have always delegated implementation details though, the interesting question is where understanding and verification are actually required.

My thesis is that the amount of AI-generated code you should read depends on what you're trying to accomplish and how costly a mistake would be.

## When you're learning

Let's start with the obvious: if you're learning a new language, framework, algorithm, or concept, having AI generate the implementation defeats much of the exercise, since the valuable part of learning is constructing the mental model yourself. This does not mean that you cannot use AI at all — using AI as a tutor, debugger, documentation layer, or source of explanations can make learning much more effective, and it is something I highly recommend. The obvious part is that here the implementation itself is what you're trying to learn: don't outsource it.

## In production, move the review boundary up

Let's now move to a work setting: should you review production code? In my opinion we should move the review boundary up. For non-critical parts of a codebase, you don't necessarily need to understand every implementation detail. Instead, spend your attention on design:

- What problem are we solving?
- What are the interfaces?
- What are the invariants?
- What are the inputs and outputs?
- What failure modes matter?

Once the design and interfaces are correct, AI can implement within those constraints. Your verification then moves toward testing the contract, rather than manually inspecting every generated line.

The more expensive an error is, the further you move the review boundary down toward the implementation, so for example security, financial, and safety-critical code could still need implementation review.

This review boundary framework is a consequence of the fact that good design matters much more in the AI era: AI is often surprisingly competent at translating a clear specification into conventional code, and many failures that look like AI mistakes are actually design failures — ambiguous requirements, incorrect assumptions, missing edge cases, or poorly defined interfaces. Instead, if the design is precise, implementation becomes much more mechanical and therefore easier to delegate.

## Reading every line won't scale

Reading every line of code will also be unfeasible (probably already is) very soon. Suppose AI generates 10x as much code as an engineer could write manually: if you want to read every line anyway, you've preserved the old bottleneck while increasing the volume of work, turning the engineer into a code-review processor.

The goal of automation should be to move humans toward higher-leverage activities, not require them to manually inspect an ever-growing amount of machine-generated output.

## So, to read or not to read?

Should we read AI-generated code? Sometimes. But I don't think "read every line" is a sustainable engineering principle. We should understand the problem, own the design, define the contracts, and verify the results, and then decide how deeply to inspect the implementation based on the risk. The important skill in the AI era may not be reading more code, but knowing which code is worth reading.