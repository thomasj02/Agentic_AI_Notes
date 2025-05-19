# Some Notes on Using Agentic Tools

*This was written in mid-May 2025\. If you're reading this more than a couple months after this date, the details of this document are probably already obsolete. The broad guidelines are likely valid for at least the next 3 months.*

---

## TL;DR

Agentic coding tools like Claude Code, Codex CLI, and others are powerful accelerants that make developers faster and expand their capabilities, especially in areas outside their expertise. They require active management and new skills like code review. Success comes from:

- Setting up good architecture beforehand  
- Using auto-accept for 5-15 minute tasks with feedback loops  
- Monitoring for when they go off-rails  
- Managing context and knowing when to reset  
- Leveraging tooling (linters, formatters, MCP)  
- Adopting a "micromanaging tech lead" mindset

## Table of Contents

1. [Introduction](#introduction)  
2. [General Properties](#general-properties)  
   - [ATs are an accelerant](#ats-are-an-accelerant)  
   - [ATs are a capability lifter](#ats-are-a-capability-lifter)  
   - [ATs are unaligned](#ats-are-unaligned)  
   - [ATs have spiky skills](#ats-have-spiky-skills)  
3. [Practical Guidelines](#practical-guidelines)  
   - [Learn to read and review code](#learn-to-read-and-review-code)  
   - [ATs work best with more up-front work](#ats-work-best-with-more-up-front-work)  
   - [Auto-accept can be pretty good](#auto-accept-can-be-pretty-good)  
   - [It needs to be managed by tooling](#it-needs-to-be-managed-by-tooling)  
   - [The AT needs to be managed by you](#the-at-needs-to-be-managed-by-you)  
4. [Conclusion](#conclusion)

---

## Introduction

Over the past few months, I've been experimenting with a variety of agentic and non-agentic[^1] coding tools.

My "daily driver" when I use these tools is [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview), a terminal-based coding agent that uses Claude 3.5 and 3.7 to analyze and perform code changes. It's similar to tools like Windsurf.ai, [Cursor.sh](http://Cursor.sh) and Codex CLI, and these notes should apply to most agentic tools.

There are a lot of disparate opinions on the usefulness and quality of tools like these. Some people try them for an hour, get poor results, and then write them off. This is like choosing to ride a bike for the rest of your life because you couldn't easily keep the car in its lane the first time you learned to drive. Using agentic tools is a skill, like learning to use your IDE well, or learning a new programming language. **They are already extremely good at many tasks, and what you are seeing right now is the worst that these tools will ever be.[^2]** Over the next 12 months, knowing how to use agentic tools is going to be as important as knowing your most frequently used programming language.

I'd like to share some of the lessons learned while using agentic tools (hereafter ATs), and Claude Code specifically.

## General Properties

### ATs are an accelerant

From a pure code generation speed, ATs are much, much faster than you. An AT can bang out a reasonable version of a class and corresponding tests in a few tens of seconds. Of course the code is not going to be the same code you would write. Frequently it will not be as good, but sometimes, it will be better\!

ATs are particularly good when the task is well-defined and it is easy to verify correctness. Recall from computer science the concept of NP-complete problems: Hard to solve, easy to verify solutions. This is how you want to structure tasks that you give to ATs. If you give them a task that is hard to solve and hard to verify, then you are much less likely to get good results.

The tasks that ATs are fastest at are ones where there are already good examples or templates to build off of. For instance, one project I’m working on has a few classes that write to various Clickhouse tables. Creating another one of these classes for a new table is going to be a relatively easy task for an AT.

Similarly, if you are using third-party libraries, ATs work best with more popular libraries that have many examples in github, good documentation, and a lot of content on sites like StackOverflow. If you find the AT is having trouble with a particular library, download the library's docs or even its entire source code into a subdirectory and let the AT know that it should look there to understand how to use the library.

### ATs are a capability lifter

Multiple studies have shown that the people who benefit most from AIs are the ones who are the least skilled.[^3]

You are probably thinking, "That might be true, but I am highly skilled at my work." And you are right \- but you are not equally highly skilled across all areas of your work. Do you know how to use matplotlib to create and style graphs to visualize data? Do you know how to write efficient database queries to fetch data from databases? Can you use testing frameworks and mocking libraries to write high quality tests? Can you confidently refactor code in unfamiliar languages when methods get too complex?

We are all good at many of these things, but we each have our specialties. Effective use of ATs gives an extended reach for your existing skill set. ATs can facilitate task completion in areas peripheral to your primary expertise, making work that might otherwise require a substantial learning investment more manageable in terms of the time and effort required to achieve a valuable outcome.

### ATs are unaligned

In AI, "alignment" refers to whether or not an AI does what you want. For example, hallucinations and sycophancy[^4] are common unaligned behaviors in non-agentic AI.

In coding ATs, unalignment comes most frequently in two forms:

#### They forget rules

All ATs have some way to provide guidelines on accomplishing tasks. Claude Code has CLAUDE.md, Cursor has .cursor/rules, etc. These rules help bring the AT up to speed on coding guidelines, how to run builds and test code, and other basic knowledge it needs to help with the codebase.

Unfortunately, these rules are not always followed. ATs have a limited context window and a "way of thinking" that has been created during training by the company that trained them. This means that sometimes the ATs won't correctly follow a rule, even if you clearly specified the desired behavior. For example, you might tell the AT to always compile code that it writes, but then find that in fact it has claimed that a task is completed without attempting to compile. The rules you create help to steer the AT, but you should think of them more like strong suggestions rather than imperatives that the AT is definitely going to follow.

#### They try to cheat

ATs really, really want to complete whatever work you ask them to do. They want to complete the tasks so much that if they run into issues that they can't figure out, they will cheat aggressively. This happens most frequently with test cases that it can't get to pass after trying for a while. Watch out for the AT removing an entire test and replacing it with falsely reassuring comments like "In a real production codebase, we would test…" or "This has been verified manually, so no need to test…" or similar fakery.

If you catch the AT doing this while you're watching it work, you can sometimes interrupt it and try to course-correct what it's doing. More often, it's worth clearing its context by restarting the task, or just acknowledging that it's beyond the AT's capabilities and fixing it yourself.

### ATs have spiky skills

All AIs, including ATs, are "spiky". That is, they are unexpectedly excellent at some things and unexpectedly terrible at other things. It is not always obvious a priori what things it will be good at and what things it will be bad at, and it changes from model to model. The only way to get a feel for what the AT you're working with can do is to use it on a regular basis, and build an intuitive sense of how to work with it.

Do not assume that using an AT successfully on one task means that it will do well on a different task; similarly do not assume that if an AT fails at one task it will also fail on a different task.

## Practical Guidelines

### Learn to read and review code

Most of us have written far more code than we've read. We all have a bias to prefer the code that we've written over code someone else has written. Our own code seems clear and obvious to us, at times even elegant; other people's code seems overengineered, poorly designed, and full of hacks.[^5] Reading code is, at least for me personally, somewhat less pleasant than writing it. I find it harder to get into a flow state, and more mentally taxing compared to working on new code or code that I've recently written; perhaps you feel the same way.

You should expect then that reviewing code written by ATs will feel uncomfortable, especially at first. You are less practiced at it, and initially you won't know where the AT is likely to make mistakes or cheat. I find that going through the code block by block and rewriting bits in my own style (e.g. renaming variables, or making small changes to reduce variable scope, or adding comments) helps keep me engaged and focused enough to understand the code.

All of us, as engineers, need to improve our skill in reading and reviewing code. If you're not reviewing the code, then you're not doing engineering, you're just vibe coding.

### ATs work best with more up-front work

Current ATs are much better at writing code than they are at designing architecture. Additionally, good architecture is important to get the best performance out of ATs. Here's what one of the OpenAI engineers who worked on Codex has to say:[^6]

> I saw this presentation recently by someone here who was — they weren't vibe coding, they were a professional software engineer — but using tools like Codex to build a new system. And they got to build a system from scratch, and there was kind of this graph of their commit velocity \[gestures a line going up\], and then their system had some traction. So then it was like, "Okay, now we're going to port it into, you know, the monolith that is the overall ChatGPT codebase," that, you know, has seen ridiculous hypergrowth and so maybe is not the most architecturally pre-planned. And their commit rate—the same engineers, same tooling (actually the AI tooling continues to improve)—their commit rate just plummets, right? And so I think the other thing is just, yeah, architecture—like good architecture—is even more important than ever. And I guess the fun thing is, for now, that's a thing that humans are really good at. So, kind of good, important for the software engineers to do their job.

While ATs aren't great at design, long-context chat-based AIs can be helpful to review design docs or class architectures. For example, o3 and Gemini are particularly good at evaluating long documents or code outlines spanning many files.

Once you have this architecture, providing it to the AT as either a markdown document or a set of classes with "TODO" comments and having it work to fill in the rest can significantly improve its output.

### Auto-accept can be pretty good

Most ATs by default want you to approve each AT action and change, but they all have some version of auto-accepting proposed code changes.[^7]

State of the art models can finish hour-long tasks with 50% reliability, and the 50%-success task length seems to be doubling about every 7 months.[^8] In my experience, tasks that take 5-15 minutes are currently the sweet spot for using auto-accept.

If you choose to enable auto-accept mode, it's critical that you provide the AT with some kind of fast-feedback loop that is appropriate for the task. For instance, if it's fixing a bug, the AT needs to be able to run a known-good test suite. If it's implementing a new feature, try to have it write an interface and tests first before implementing code. If it's fixing build errors, make sure it knows how to repeatedly recompile the code.

If the task takes more than about 15 minutes, carefully check the AT's progress to see if it's gone down a rabbit hole. If this happens, your best bet is frequently to provide more up-front help and retry, or of course you always have the option of implementing the work yourself.

### It needs to be managed by tooling

Tooling like linters and code formatters can help the ATs significantly both with bug fixing and code style. In Python, try to ensure that tools like pylint and ruff are configured correctly, and that the agent rules have instructions on how to run these tools. In C++, clang-tidy and clang-format can be helpful. Adapt as appropriate for your favorite language.

Think about how strict you want these tools to be. At one point I tried to have Claude use mypy with strict mode for a new project. This caused the AT to spend as much as 30 minutes fixing type annotations, without any significant increase in code quality (and a dramatic decrease in readability when the type annotations got complex).

#### Aside: MCP

Many of you have probably heard about Model Context Protocol (MCP). MCP was originally introduced by Anthropic to provide a standardized way for ATs to interact with external services[^9]. Think things like querying databases, posting and reading Slack messages, reading and editing Linear tickets, etc. Most major vendors have hopped on this trend and provide an MCP interface; there's also a vibrant open-source community that is building MCP servers.[^10] As you can imagine, given that it was introduced by Anthropic, MCP is not natively supported by OpenAI tools.

If a service that you're working with has an MCP interface and you're using Claude or another tool that supports MCP, I would encourage you to enable it. Each AT has a slightly different way of specifying available MCP servers, so you'll have to check your own tool's docs to see how to do this.

### The AT needs to be managed by you

ATs are not yet advanced enough to be fully trusted with most tasks, despite what the Codex developers would have you believe. This may happen soon, but we're not quite there yet. In my experience, the most effective way to work with ATs is to actively monitor their progress if the task is tricky or short, intervening when necessary. If the task is longer, at least check back every couple of minutes to make sure it's still on-task. In essence, you need to change your mindset from an individual contributor to a micromanaging tech lead.

#### Mitigating off-the-rails risk

ATs have a particular failure mode where they can go off the rails, making a series of edits that actually takes them farther away from the correct solution. When this happens, it becomes almost impossible for them to get back on track. You have to monitor for this, particularly on long-running auto-accept tasks.

This can happen even after a series of successful changes. I suggest making frequent small commits in your local branch as the AT completes subtasks. This allows you to easily roll back to the last useful change that the AT did, and either retry with a different prompt or step in to do the work yourself.

One additional tip if you want to re-try a previously failed task with the AT is to do more pre-planning. Either ask the AT to analyze the code and develop a plan (which you should then review), or ask a stronger long-context model like Gemini Pro or o3 to help you design a plan instead. Once you have this plan, you can paste it directly into the AT's prompt, or put it in a text file that you ask the AT to read before retrying the task.

#### When to reset context

As the AT works, it builds up context that acts like a short-term memory of the actions that it's taken. This context influences future output of the AT, and therefore future code edits and tool calls. However, sometimes this context can become unhelpful, causing the AT to act in a misaligned way or to have reduced performance. When this happens, it can be helpful to clear the context and start fresh.

Common reasons why you might clear the context and restart include:

- To stop an off-the-rails task as described above and retry with a new prompt  
- After switching to a fundamentally new task, or a task in a different part of the codebase  
- If you update the agent instructions and you want to agent to reload the instructions and try again  
- After merging in new code that affects the task the AT is working on

Some tools, including Claude Code, will automatically compact the context once it grows too long. The context is compacted by having the AT's model generate a short summary of the existing context, and replacing the full context with that summary. You should pay particular attention to a task that spans compaction, because the summary might not include all the important details of the context, or could in some cases even be incorrect due to hallucinations or a misunderstanding of the work that has been done. If you notice this happening, you should intervene or clear the context.

#### When to disable auto-accept

Auto-accept is the key feature of ATs. In an ideal world, all ATs would work exclusively with auto-accept.[^11] Unfortunately, we're not quite there yet in terms of AT capabilities. Typically I turn on auto-accept once I see that the AT is on the right track and is making progress on a task. On the other hand, you should be quick to turn off auto-accept if you see the AT starting to make mistakes and not recover from them.

The other time I will turn off AT is if I want the AT's help on a task, but I'm not sure exactly what I need yet. For instance, I might ask it for help designing a test to track down a bug, or to prototype a script that needs to be more fully built out before it's usable. In this case when I know that I want to intervene before the AT gets carried away, I'll leave auto-accept off until I review the AT's intermediate work product.

#### Cleaning up after ATs

One behavior of ATs that I appreciate is that they're often willing to write one-off scripts or tests to validate a particular feature or help debug an issue. The AT is much more comfortable writing a helper script to do something like analyze code coverage or generate additional tests to debug an issue versus directly issuing the commands in the shell. You could say that the AT's native language is closer to Python than bash.

However, I've found that the ATs are not great about cleaning up after themselves. I'll frequently work on a hard bug with the AT for 30-45 minutes and after the bug is fixed there will be half a dozen scripts littered throughout my project directory, and I'll tend to forget which were important and which were just one-off scripts to help debug. I've been able to steer this somewhat by encouraging the AT to name helper scripts with a special naming convention like helper\_\*.py. In some cases you can ask the AT directly and it will tell you what code is important and what code is just temporary. The real solution though is just to carefully keep track of what the AT is doing, or to manually review the code after the AT is done.

## Conclusion

In summary, agentic coding tools like Claude Code are already revolutionizing software development, acting as significant accelerants and capability lifters, and what we see today is the "worst they will ever be." Effectively harnessing their power is not automatic; it requires engineers to cultivate new skills, emphasizing critical code review over sheer generation, investing in robust upfront architectural planning, and adopting an active, guiding role akin to a diligent tech lead. While navigating their current limitations \- such as unalignment, the tendency to forget rules, and even attempts to "cheat" on tasks \- can be challenging, the practical guidelines shared here offer a roadmap for productive collaboration. As these tools rapidly advance in the coming months and years, the ability to skillfully partner with them will transition from a niche advantage to a fundamental engineering competency.

---

About Me:

I have been in software engineering for over 20 years, and I think AI is going to have a bigger impact on our industry than the Internet. I am currently a Principal Research Engineer at Kbit, digital asset hedge fund. I am occasionally on X at https://x.com/ThomasJ02


[^1]: Non-agentic AI tools are traditional chat-based tools like ChatGPT, Gemini, or Claude. Agentic tools edit code, interact with external services, and can work in semi-autonomous long-running write/run/debug loops like a human software engineer.

[^2]:  Thanks to [https://thezvi.substack.com/](https://thezvi.substack.com/) for introducing me to this phrase

[^3]: [https://mitsloan.mit.edu/ideas-made-to-matter/workers-less-experience-gain-most-generative-ai](https://mitsloan.mit.edu/ideas-made-to-matter/workers-less-experience-gain-most-generative-ai), [https://mitsloan.mit.edu/ideas-made-to-matter/how-generative-ai-affects-highly-skilled-workers](https://mitsloan.mit.edu/ideas-made-to-matter/how-generative-ai-affects-highly-skilled-workers), [https://www.mckinsey.com/capabilities/mckinsey-digital/our-insights/superagency-in-the-workplace-empowering-people-to-unlock-ais-full-potential-at-work](https://www.mckinsey.com/capabilities/mckinsey-digital/our-insights/superagency-in-the-workplace-empowering-people-to-unlock-ais-full-potential-at-work)

[^4]: [https://openai.com/index/sycophancy-in-gpt-4o/](https://openai.com/index/sycophancy-in-gpt-4o/)

[^5]: But try to read code that you wrote more than a year ago on a different project. Does it still seem as good? More than one engineer has had the experience of asking "Who wrote this horrible nonsense? \<checks git log\> Oh, it was me."

[^6]: Alexander Embiricos on [ChatGPT Codex: The Missing Manual](https://www.youtube.com/watch?v=LIHP4BqwSw0)

[^7]: Each AT names this option slightly differently. My personal favorite is Jetbrains Junie, which calls this "brave mode".

[^8]: [https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/) Note that if this continues, by 2027 the 50% success task length will be 8h (one work day) and by 2029 it will be one work-month.

[^9]: More details are at [https://modelcontextprotocol.io/](https://modelcontextprotocol.io/)

[^10]: See for example [https://github.com/punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)

[^11]: OpenAI's Codex tries to operationalize this, with varying degrees of success
