# Chapter 1: Introduction

## Fix the code. Keep the lesson.

A team fixes a restart bug. An identifier was unique while the process was running, but its counter started at zero after a restart. Old records kept their identifiers. New records reused them. In a graphical interface, fresh text could end up in an old conversation card.

The implementation change might be small: resume the counter safely, or choose a different identity scheme. A regression test crosses the restart boundary. The change passes review and lands.

But what did the team learn?

Not merely that a counter needed initialization. The identity must remain unique for as long as the records that use it remain meaningful. A final response must use the same identity as its streaming updates. Repairing future allocation does not necessarily fix already-colliding history. A test that sends two messages through one running process does not exercise the failure.

Those are engineering lessons. If they survive only in the patch author's head, a later implementation can repeat the same mistake with perfectly reasonable-looking code.

Palimpsest is a project for keeping those lessons executable. We want the correction to change what the next builder is taught, not just what the current program does.

The name comes from a manuscript rewritten while traces of earlier writing remain. Our current codebook should be clean and coherent. Its history should preserve how we learned to write it.

## The book is part of the build

An agentic codebook teaches a builder how to construct a system and how to check the result. Its chapters contain build contracts, explanations in plain language, design reasons, exercises, and graders. The builder may be a human, an AI coding agent, or a collaboration between them.

The important word is not *AI*. It is *teaches*.

A requirement can say that tool execution must be interruptible. Teaching explains the distinction between interrupting a wait and killing a job, which component owns the job, what survives when the caller stops waiting, and how a grader can expose an implementation that confuses those lifetimes. The next builder needs the reasons as well as the expected behavior.

An executable codebook is not a deterministic compiler. Two builders can produce different correct programs. The claim is narrower and testable: the instruction should be sufficient to construct an implementation that meets the declared obligations, without private hints or access to the exercise's existing answer.

The idea behind Palimpsest grew out of work on *The Art of Building AI Coding Agents*. An agent used that book and its graders to construct Ensemble. Bill then began using the resulting agent to improve its own working environment. Real use exposed gaps in the implementation, the teaching, and the checks. That experience supplies a motivation and examples, not proof that every part of the workflow proposed here already works.

The question is what happens next. If we repair the implementation and leave the book behind, the next construction can reproduce yesterday's failures. If every small repair forces an immediate reconstruction of the entire project, keeping the teaching current can become prohibitively expensive.

Palimpsest is intended to make the middle path practical: update the knowledge, repair the running design efficiently, and independently test whether the knowledge transfers.

## Knowledge patches

An ordinary changelist says what changed in an implementation. A Knowledge Patch, or KP, says what must change in the teaching and why.

For the identifier bug, a KP could strengthen the lifetime requirement in the chapter that introduces identity, add later restart coverage of that same obligation when persistence becomes available, and record why fixing only future allocation is insufficient for old history. That later coverage verifies the identity obligation; it does not introduce a new persistence-chapter requirement. The source edits are one realization of the lesson. They are not the only durable record of it.

A KP contains the problem, the proposed correction to instruction, the acceptance checks, and the reasons a future builder needs. It also records uncertainty. We must not manufacture rejected alternatives or pretend an investigation was complete before it happened.

For the first version of Palimpsest, one KP intentionally changes one chapter's contract. This is a deliberate restriction. It gives the change a known owner before implementation begins and avoids trying to infer ownership from an arbitrary mixed patch at HEAD.

Consequences can reach later chapters. Renaming a public function in an early chapter requires updating callers introduced later. Those adaptations belong to the same KP. But a new requirement in a later chapter is not merely a merge-conflict resolution. It needs a separate knowledge decision and, under this initial rule, a separate KP.

The engineer need not know every implementation detail before starting. A short draft can state the problem and the suspected obligation. Investigation may correct both. The acceptance requirement is that the final teaching matches what was established and that the result has been exercised. Writing a plausible story after a patch lands is not enough.

## Three records, one accountable change

Palimpsest connects three records rather than replacing version control.

The **KP history** records discoveries, proposed teaching changes, revisions, review, approval, and integration. The **codebook history** records the current chapters, their prerequisites, and the checks a builder must satisfy. The **implementation history** remains the familiar chain of code changelists or pull requests.

One KP can generate multiple implementation CLs. Every CL has exactly one owning KP. Dependencies between CLs do not change that ownership. If a proposed CL mixes the work of two KPs, split it.

From a submitted KP, we should be able to find its approved revision, the chapter edits, the generated CLs, and their evidence. From an implementation CL, we should be able to find the knowledge change that justified it. Approval of teaching, completion of implementation, and successful reconstruction are distinct facts.

The codebook at HEAD contains integrated chapters. A fresh builder should not have to replay a permanent appendix of old repairs. We retain the KPs as history, but the current lesson belongs where the builder needs to encounter it.

Submitted implementation history stays immutable. Enterprise teams do not need to permit an agent to rewrite accepted commits in order to adopt this workflow.

## A second chain for construction

Sequential chapters naturally produce a construction chain. Chapter 1's implementation supplies the base for Chapter 2, which supplies the base for Chapter 3. We can represent each chapter's contribution as a patch or a commit sequence between two source trees.

Applied in order to a declared base, the canonical chapter contributions must reproduce the accepted implementation tree exactly:

```text
Base + chapter contributions = accepted HEAD
```

This equality is a mechanical check. It does not mean a fresh student's independent solution must use identical source text.

The construction chain is separate from enterprise integration history. Its accepted generations are retained, but a new candidate generation can be rewritten while we work out how the corrected system should be built. We are not claiming that the enterprise's historical CLs originally occurred in this new order.

Git is a useful first storage mechanism. It supplies tree identities, diffs, temporary worktrees, merge machinery, and efficient storage. It does not know which chapter owns a requirement or whether a student received sufficient teaching. Palimpsest supplies those meanings and the evidence around them.

We expect to begin with a command-line tool alongside Git. Both a human and an agent should be able to inspect the same plan, run the same checks, and see the same failure. We do not need a new change-control service before testing the workflow.

## Change a chapter where it exists

To modify Chapter K, begin in an isolated candidate workspace at the construction state immediately after K. That state has the chapter's implementation and its prerequisites, but not the later features that could conceal a missing dependency.

The informed builder revises the chapter, its implementation, and its checks. It runs the chapter's obligations and applicable earlier checks, then submits the result for independent review. The review covers correctness and the architecture, not just whether the tests pass.

A separate student begins from immediately before K. That student receives the revised chapter and the permitted prerequisite guidance, but not the edited answer. We initially assume an honest student: this is an instruction boundary, not an attempt to build a secure examination environment. We still avoid handing over the solution accidentally and record any assistance given.

These two executions answer different questions. The informed builder establishes that the correction works. The fresh student tests whether the revised instruction conveys it. If the student needs a missing rule explained privately, that is a teaching finding to reconcile, not a result to conceal.

After the chapter change is accepted in the candidate, the later contributions replay in order. Agents resolve mechanical conflicts and run the relevant graders as they proceed. A clean textual application is not a clean bill of health: a later patch can reintroduce an old assumption without a merge conflict.

The resulting final tree is checked against the accumulated applicable chapter graders and implementation tests. Its difference from enterprise HEAD becomes the new implementation CL or CLs. In this workflow, the patch at HEAD is an output of the knowledge change, not an arbitrary input we must dissect into historical chapter ownership.

Before publishing a new construction generation, its full chain must reproduce the newly accepted implementation tree. Concurrent changes to HEAD require integration and revalidation; results for one tree cannot silently approve another.

## A valid learning order

Chapters have prerequisites. The book's linear execution order is one ordering of those dependencies. They cannot form a cycle.

Suppose an early memory chapter implements compaction before a later chapter introduces sub-agents. At HEAD, migrating the compressor to a managed child may be straightforward because both capabilities exist. A student at the early memory chapter cannot use machinery not yet constructed.

The solution may be to move the memory exercise, split out a foundation, or add a later integration chapter. A possible order is:

```text
Context and history primitives
    -> Sub-agent lifecycle
    -> Background compaction using sub-agents
```

That is a design decision, not a textual rebase trick. A no-cycles rule gives us a condition to satisfy; it does not select the architecture for us.

Nor is a declared graph enough. A grader fixture or an assumed interface can smuggle in an undeclared future dependency. Executing exercises from their actual prerequisite trees is how we challenge the declarations.

Some changes are genuinely expensive because they alter the construction order. Palimpsest should expose that cost and its reason. It should not label a successful HEAD patch as proof that the revised curriculum builds.

## Checks that can disagree with us

A grader is software with assumptions. It can check the wrong boundary, accept a constant answer, or fail for a reason unrelated to the behavior it claims to test.

Tests should exercise real implementation behavior. Faithful fakes can replace external providers or other dependencies, but should not replace the component whose correctness is at issue. Targeted mutations test whether a check detects the intended defect. A mutation that merely prevents compilation does not demonstrate a behavioral oracle.

At HEAD, the accumulated applicable exercises must continue to pass. Requirements deliberately superseded by a later chapter need explicit treatment; we cannot quietly drop an inconvenient check or require an early exercise to implement features it has not yet introduced. Test output records what actually ran, what failed, and what was not tested.

Independent review applies to both implementation and teaching. A builder cannot silently weaken its grader to approve its own answer. Revision attempts are bounded. An unresolved design question or exhausted review budget becomes a visible blocker, not permission to spawn another worker and forget the failed attempts.

A final malware and security inspection is another acceptance gate, tied to the exact candidate being promoted. It is not proof that arbitrary generated code is harmless. Builds and tests execute code before that gate, so restricted execution and appropriate credential handling are necessary throughout the workflow.

Once the authorized gates pass, agents may make accepted local commits under the project's policy. Push, merge, deployment, and release are separate permissions. Candidate checkpoints are useful, but are not accepted results merely because they exist in Git.

## The human must learn too

There are two products: better software and better engineering knowledge. The second product cannot remain hidden inside an agent's work.

Before a consequential action, the agent explains what it intends to do and why. During investigation, it surfaces discoveries that change the plan: a mistaken assumption, a failed approach, an unexpected dependency, a grader gap, or a security concern. At a stopping point, it explains the result, remaining uncertainty, and the next decision.

This is not a request for private model reasoning. It is a requirement for public engineering explanations that a human can understand and challenge, and that a future builder can recover from the record.

Delegation does not excuse silence. A child reports its discoveries to its coordinator; the coordinator brings the material findings to the human. Returning only the final patch can discard the most valuable part of the work.

Unattended execution can proceed within an agreed scope, but it still records those explanations and reports them when the human returns. Permission to complete a task is not blanket permission to change its architecture. The workflow must distinguish decisions already delegated from questions requiring further judgment.

We do not need commentary on every command. We need the reason for the lifetime boundary, the failed alternative, the observation that invalidated the old plan. A concise explanation of one changed invariant can teach more than pages of raw output.

Humans can accumulate understanding across a project. An ordinary coding session does not update an LLM's underlying weights, and its working context is finite. The codebook makes the project's acquired knowledge available again. Keeping the human informed lets that person continue making good decisions about what the book should teach.

## Rebuild without the old answer

Incremental work can still lean on undocumented knowledge in the current implementation. A periodic clean-build train tests for that drift.

On a schedule appropriate to cost and risk—perhaps nightly or weekly—fresh builders construct the system from the integrated codebook. They receive the declared instructions and prerequisites, not the previous implementation or private repair conversations. They execute the chapters and graders in order, then test the resulting system as a whole.

A failure is evidence to investigate. It may expose incomplete teaching, a flawed grader, a model limitation, or an environmental failure. Those are different diagnoses. If the teaching changes, the affected exercises run again under the new revision; we do not quietly help the student and report the original instructions as sufficient.

The accepted source, instructions, toolchain, builder configuration, and results are identified precisely. Existing version-control objects and protected references can retain the evidence without a permanent copied directory for every chapter. Historical outcomes remain historical facts.

The train does not promise identical source from every builder. It tests whether the book remains sufficient to produce implementations satisfying its obligations. The canonical construction stack has the separate, exact obligation to reproduce the accepted source tree.

## Build the method by using it

Palimpsest is a proposal to implement and test, not an announcement of a finished platform. Its first useful experiment should be small enough to inspect: a short construction chain, a change to an early public function, later consumers, and a fresh student rebuilding the changed exercise.

That experiment can test the important boundaries without first implementing enterprise-scale scheduling. Can the tool materialize the right prerequisite state? Can the correction propagate without rewriting accepted history? Do the graders detect the old behavior? Does the student succeed from the teaching? Does the resulting chain match its intended HEAD? Can the human explain why the change was placed where it was?

We will develop this codebook in the same manner. Choose the next useful capability, implement and investigate it, then preserve the validated contract and its reasons. We do not need to decide every future chapter before writing the next one. We do need to leave an honest distinction between a proposal, a tested implementation, and an independently reproduced lesson.

This approach will not eliminate every kind of technical debt. Instruction can become contradictory. Graders can reward the wrong behavior. Dependency changes can make reconstruction expensive. A book can be incomplete while sounding authoritative. Those are reasons to exercise the knowledge, not reasons to declare it trustworthy in advance.

The purpose is to stop losing what the engineering taught us. Code can be replaced. The lesson should survive, and the next builder should be able to use it.
