---
title: "How I Use Claude"
tags: ["claude", "ai", "codex", "prompting", "workflow", "productivity", "engineering", "claude-code"]
date: "2026-02-17T10:00:00+00:00"
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---

"What separates people who use AI well from those who don't?"

It's a question I get from colleagues often. It's been three years since ChatGPT launched, and over a year since I started using Claude Code in earnest. In that time, the way I work has fundamentally changed. It's no longer just "asking AI things"—**the way I think alongside AI** has shifted.

In this post, I want to share honestly how I use Claude: how I ask questions, the patterns I follow to solve problems, and what I've discovered recently by combining it with Codex.

## The Evolution of Asking: From "Do This" to "Let's Think Together"

When I first started using AI tools, it was an extension of a search engine. Short-answer questions like "How do I delete from a map while iterating in Go?" I was just asking Claude instead of Stack Overflow.

But over time, the shape of my questions changed. Early on, I'd jump straight to the question. "How do I fix this error?" Now it's different. I start by explaining **what I'm trying to do**, **how far I've gotten**, and **where I'm stuck**. I know that Claude gives more accurate answers when it understands my thought process. For example:

> "The drop-off rate after step 3 of our onboarding flow is 40%. The current screen has 6 input fields, and user interviews frequently say 'I don't know what to fill in.' Rather than reducing fields, I want to improve by splitting into more steps—what structure would work well?"

Compared to just saying "how do I reduce onboarding drop-off," providing this kind of context dramatically changes the quality of the output. **Good answers don't come from good questions—they come from good context.**

As I gained more experience, I started explicitly stating constraints. Things like "This project uses Go 1.22, and we minimize external dependencies as a principle," or "This code needs to handle 100K requests per second. Performance over readability." Without constraints, Claude gives the "safest" answer—generic, textbook, but not right for my situation. Constraints are guardrails that help Claude **narrow down from infinite possibilities to the space I actually need.**

The most effective pattern I've found is assigning Claude a **role**. When I say "You're a senior SRE right now. Analyze the failure scenarios of this architecture," I don't get a simple code explanation—I get operational risk factors. The same code yields completely different insights depending on which role examines it.

## Problem-Solving Patterns: How I Work with Claude

We used to explain code to a rubber duck to find bugs ourselves. Claude is **a rubber duck that talks back**. When I describe a problem, it asks questions and finds the gaps in my thinking. When I start with "I think there's a deadlock in this function, but I'm not sure if the mutex lock order is the problem," Claude examines the code and points out: "This goroutine locks A and waits for B, while that goroutine locks B and waits for A." An analysis that would take me 30 minutes alone finishes in 3. The key is that I'm not asking Claude for answers—**I'm getting my thought process validated.**

For architecture design, Claude becomes "a colleague standing at the whiteboard with me." When I ask about trade-offs between Kafka vs SQS+SNS for a 5-person team handling a million events daily, Claude lays out the pros and cons while adding perspectives I missed—operational complexity, team learning curve, future scaling scenarios. I make the final decision, but **the surface area of that decision expands.**

My code generation workflow has changed too. Instead of starting from a blank file, I describe my intent, get a draft, and iteratively refine it. The important thing is that **I don't expect perfect code on the first try.** The first output is about 80%. From there, I iterate: "Change the error handling like this," "Convert this part to table-driven tests."

For writing—whether blog posts, design docs, or RFCs—Claude is my **structuring partner**. I dump my thoughts in no particular order, and Claude rearranges them into a logical flow. What's especially useful is **counterargument generation**. When Claude preemptively flags "this counterargument could be raised against your point," the completeness of the writing improves. The "anticipated objections" sections in several posts on this blog mostly came from conversations with Claude.

## Prompting Patterns: What Actually Works

Through trial and error, I've settled on a few prompting patterns.

If you just ask for an answer, Claude gives you the conclusion. When you say "show me the reasoning behind your thinking," the inference process becomes visible, and I can catch assumptions I disagree with. For trade-off-heavy problems like architecture decisions, this pattern is essential.

My most-used pattern is "ask me questions first." When I throw a complex problem at Claude, I request: "Don't answer right away—first tell me what you've understood and ask me what else you need to know." Looking at the questions Claude asks reveals what context I've missed. And answering those questions helps organize my own thinking.

When you receive a single answer, it's hard to judge whether it's the best one. Asking "present three different approaches, each with their trade-offs" maps out the decision landscape. Usually, the third option reveals an approach I hadn't considered.

In Claude Code, placing a `CLAUDE.md` file at the project root automatically passes project context without repeating it every session. I keep project conventions, architecture principles, and frequently used commands there. **It eliminates the cost of explaining the same context every time.** In practice, this single file has a bigger impact on productivity than you might expect.

## Claude + Codex: Using Both Tools Together

Recently, I've been using Codex alongside Claude Code. At first I thought, "Why use two similar tools?" But in practice, **their strengths and use cases differ.**

In medicine, important diagnoses call for a second opinion. The same applies to code. For complex algorithms or system design, I run Claude's answer by Codex for verification, or vice versa. When both models reach the same conclusion, confidence goes up. When they disagree, **the point where I need to think more deeply** becomes clear. For example, if Claude says "this approach is safe" for concurrent code but Codex flags "there's a potential race condition," digging into that difference is how I truly understand the problem. Disagreement between models becomes my learning opportunity.

Being able to invoke Codex as a skill within the Claude Code CLI matters too. No switching between separate tools. While writing code, when I think "let me check this with Codex," I can verify it in the same terminal and come right back. When those 30 seconds of tool-switching disappear, the flow of thought stays unbroken.

What I've learned through use is that there's no need to insist on one tool for every task. Claude is strong at long-context conversations, architecture design, writing, and complex refactoring. Codex is strong at code generation accuracy, idiomatic patterns in specific languages/frameworks, and fast implementation. It's like switching between a driver and an iron depending on the situation. The key isn't being attached to a tool—**it's choosing the right tool for the problem.**

## What's Still Lacking: Honest Limitations

I have no intention of only praising AI tools. The limitations I feel while using them are real.

As Claude Code sessions grow longer, early context fades. There are times it forgets constraints I explained 30 minutes ago and goes in the wrong direction. CLAUDE.md partially solves this, but dynamic in-session context—"I told you that approach doesn't work earlier"—still isn't perfect.

Claude answers confidently even when it's wrong. This happens frequently with newer library versions or lesser-known APIs. It confidently says "this function exists," but the API doesn't actually exist. This is especially dangerous for less experienced developers. **The habit of never blindly trusting AI responses** is necessary.

When you delegate code to Claude, it tends to generate more than what you asked for. Adding error handling, logging, tests... The original intent was a simple utility function, but before you know it, you've got 50 lines of production code. **Drawing the line at "this is enough"** is still the human's job. While Claude handles most error messages and stack traces well, it struggles with hard-to-reproduce intermittent bugs or timing issues in distributed systems. Domains that require deep system understanding and intuition still exist.

## Why Leadership Matters More in the AI Era

The most surprising realization from using AI tools was this: **leadership experience directly translates to AI proficiency.**

Anyone who has led a team knows this. The way you communicate requirements differs when delegating to a junior versus a senior. Small tasks get specific instructions; large tasks get the purpose and constraints, with the method left open. When the direction goes wrong mid-course, you have to make the call to stop and revert. It's exactly the same when using Claude.

For small tasks, I give clear, direct instructions: "Add error handling to this function. Wrap errors with fmt.Errorf and return to the caller." Giving excessive context to a small task generates unnecessary extras. For big tasks like "design the authentication module for this system," the approach differs. First, I request a design in plan mode, review it, then move to implementation. I communicate only "why this needs to be done" and "what constraints exist," and receive a proposal for the **how** first.

One of the hardest decisions a leader makes is "this direction is wrong, let's revert." It's the same with AI. Claude is diligently writing code, but looking at the intermediate output, the fundamental approach is off. The judgment to boldly stop and switch to a different approach, rather than thinking "let's push forward a bit more"—that comes not from prompting skill but from leadership experience. The sunk cost fallacy doesn't only apply to human work. You develop attachment to AI-generated output too—"we've come this far..." But just as a good leader knows when to pivot a project, there are times in AI work where you must decisively change direction.

This becomes even clearer when running multiple agents in parallel in Claude Code. You let exploration agents investigate broadly while implementation agents work narrow and deep. The process of aggregating each agent's output and deciding the next direction is essentially the same as synthesizing research from different parts of your team to make a decision. What engineers need in the AI era is not "the ability to write good prompts." **It's the ability to judge what to delegate, decide how to communicate it, and evaluate the results to determine the next action**—that is leadership itself.

---

## What Has Actually Changed

Looking back over a year of experience, the biggest change isn't **the speed of work—it's the density.**

Not doing more in the same time, but going **deeper** in the same time. Time that used to go to boilerplate and repetitive tasks now goes to design and decision-making. Less time typing code, more time **thinking about** code.

And working with AI has actually sharpened my ability to work with people. Communicating context well, clarifying constraints, critically evaluating responses—these are skills needed equally whether you're working with AI or with colleagues. People who handle AI well lead teams well. And people who have led teams well handle AI well. **The two reinforce each other.**

---

## Takeaways

- **Context determines the quality of the answer.** "Do this" produces far worse results than "In this situation, with these constraints, I'm trying to do this."
- **Claude is a rubber duck that talks back.** It's most effective when used to validate your thinking, not to ask for answers.
- **Prompting is a skill.** "Ask me questions first," "give me three options," "show me your reasoning"—these patterns change outcomes.
- **Use tools in combination.** Claude and Codex have different strengths. Cross-verification and purpose-based separation yield better results.
- **You need to know AI's limits to use it well.** Stay aware of the confidence trap, context volatility, and the temptation of over-generation.
- **Leadership is AI proficiency.** Delegating small and large tasks differently, and knowing when to stop—that judgment comes from leadership experience.

AI tools will keep evolving. But no matter how good the tools get, **judging what to delegate, evaluating results, and reversing course when the direction is wrong**—that remains the human domain, the leader's role.

And one more thing to remember: the tools and workflows I'm using right now won't always be the best option. A day may come when Claude is no longer the best choice, or when today's prompting patterns become irrelevant. Given the pace of change we've witnessed over the past three years, the next wave is likely to arrive even faster.

What matters is not tying your identity to a specific tool. Organize what you've learned today, pick up tomorrow's new methods, and embrace that process itself as growth. Tools change, but **people who know how to learn** adapt every time.

I use Claude as a tool that makes me a better version of myself. Not faster—**deeper.** And when a better tool comes along someday, I intend to learn it, building on this experience.

One last thing. Don't be cheap about it. The cost of an AI tool subscription is less than any course, less than most hobbies. And what you get back is incomparably greater.
