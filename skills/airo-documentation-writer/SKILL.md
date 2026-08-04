---
name: airo-documentation-writer
description: Write or revise documentation prose (READMEs, setup guides, design notes, hardware notes, changelog entries, docstring prose) in the AIRO house voice — a knowledgeable labmate writing to a colleague who will actually run the thing. Use whenever drafting or editing a .md file or user-facing prose in an AIRO repository, or when asked to match the airo style, to write docs like airo-mono, or to make docs read consistently with the lab's other repos. Also use when reviewing docs that read as generic or marketing-like.
---

# AIRO documentation voice

The voice is **a knowledgeable labmate writing to a colleague who is about to run the
thing** — not a technical writer, not a product page. The reader is assumed competent but
new to *this* code, short on time, and about to point it at real hardware or real data. So
prose earns its place by telling them something they cannot get from the signature.

Everything below is drawn from `airo-mono`. Quoted lines are real; use them to calibrate
register, not to copy verbatim.

## The pronouns are load-bearing

- **"we"** = the authors' decisions, choices, recommendations. *"We recommend [uv] for fast,
  reproducible environments."* / *"We simply wrap a subset of its features to make it more
  verbose and self-explanatory."* / *"We considered this complexity unjustified."*
- **"you"** = the reader doing things. *"You can now move the robot around to collect
  samples."* / *"you might not want to be busy waiting on that command to complete"*
- The thing itself gets an active verb, not a passive construction. *"The driver ships **no
  arm profile** and requires one."* / *"the core silently clamps them into the profile's
  position limits each tick"*

Never write a whole paragraph in the impersonal passive. If a choice was made, say who made
it and why.

## Rules

**1. Open with one flat sentence saying what this is.** No hook, no benefit statement.

> This package contains code for controlling robot manipulators and grippers, specifically:

> Tools for working with datasets. They fall into two categories:

> python package with common types that are used across multiple other packages.

A bare noun phrase is fine. "This package contains / provides / covers X" is the workhorse.
Then immediately a list or a table — the opener is a label, not an introduction.

**2. Give the reason next to the instruction, and give hard reasons their own section.**
This is the strongest marker of the style. Sections are literally titled `Rationale`,
`Design choices`, `Why not semantic versioning?`, `Why not use ROS for everything?`,
`Notes on the different types of interfaces for hardware interaction`. Tables get a **Why**
column. Every "not supported" carries the reason it isn't:

> | `get_current_width` | the gripper reports no position. Use `last_commanded_width` if the last commanded opening is good enough |

> the regular `pip install bkstools` from PyPI is not preferable [because] in later steps we
> need to adjust one of the project files

**3. Be honest about limits, uncertainty and non-ideal choices.** Do not smooth them into
confidence. This is what makes the docs trusted:

> This is not a provably right choice, it is driven by personal experience.

> Could be necessary (unconfirmed): `sudo apt-get install build-essential linux-source`

> other Zed cameras may also work

> However it is important to note that low residual error is no guarantee that the solution
> is accurate.

> This might actually be the most pythonic way, but it requires all downstream code to also
> be async. [...] Even when it is possible, it is not desirable imo.

First-person asides ("imo"), hedges on genuinely uncertain things, and mild exasperation at
a third-party library (*"which is absolutely ludicrous"*) all belong. What does **not**
belong is hedging a hard rule.

**4. Hard rules are asserted flat and bolded mid-prose — not in admonition blocks.** Bold
the claim as a short sentence, then explain in plain sentences after it:

> **Connecting opens the gripper.** Bring-up probes whether a dispatcher is already running
> by writing a benign open, so the fingers move before your code issues any command. Keep
> hands clear while connecting.

> **This is only guaranteed to work when you explicitly wait on a previous command before
> sending a new command**.

> **The driver does no kinematics: no URDF, no forward kinematics, no inverse kinematics.**

Bold also marks the term being defined at the head of a bullet: `- **Minimalism:** Before
coding, explore existing libraries. Less code means easier maintenance.` Label, colon,
terse sentence — fragments are fine.

**5. Show the wrong way, then the right way.** The clearest teaching device in these docs:

> The following will not work as expected:
> ```python
> robot.move_to_joint_configuration(q1)
> robot.move_to_joint_configuration(q2)
> ```
> but this will:
> ```python
> robot.move_to_joint_configuration(q1).wait()
> ```

**6. Code examples are short, real and runnable, with real values and semantic comments.**
Real IPs (`192.168.1.100`), real class names, no `foo`/`bar`, no ellipsis placeholders where
a real line would do. Comments carry the fact the code cannot:

```python
awaitable_robot = robot.move_to_joint_configuration(q1)  # Not blocking.
gripper.open().wait()  # Will already start executing while the arm is still moving. Blocking.
print(robot.get_joint_configuration())      # radians
```

**7. State what the page does *not* cover, and link outward instead of restating.**

> The [airo-fanuc](...) driver's own documentation is the authoritative source for the
> driver itself, and this page links to it rather than restating it.

> this repo does not offer advanced robotics features such as optimization-based motion
> planning, collision checking... If you need such things, you have to use an existing
> framework such as Moveit, OMPL, Drake,...

Link to **source files**, not only to docs, and tell the reader to read the code when the
code is the best explanation: *"But it is also the place to be to get an idea of how to use
the implementation."* / *"Make sure to take a look in the code so that you understand these
values."*

**8. Address contributors in the direct imperative, including the reminders.**

> Don't forget to add a `__main__` codeblock to the new module that runs the tests for that
> hardware implementation. Also don't forget to add the implementation in the table above.

> So make sure you know what you are doing if you send multiple actions.

> It moves the arm around the configuration it is already in, so park it somewhere clear
> first.

**9. Concrete numbers beat adjectives.** *"takes about 30ms to complete"*, *"~85 mm, fully
open"*, *"a 175 mm offset along tool +Z"*, *"at least 3 samples, however more is usually
better"*. `~` for nominal values is normal. Never "very fast", "robust", "powerful".

**10. Keep the slogans.** Short repeatable principles, reused verbatim across pages:
*keep simple things simple* / *Simple things should be simple.* / *avoid reinventing the
wheel* / *Less code means easier maintenance.* If a project has such a phrase, repeat it in
the same words rather than paraphrasing.

**11. Italics carry the precise distinction; em dashes and colons define inline.**

> it enforces *one driver per host*, so a two-arm cell needs a distinct `lock_path` per arm

> those limits are the *active configuration of one controller*, not a property of the model

> the nominal opening of the last command — a command, not a measurement

The "X, not Y" appositive is the signature move for killing a likely misreading. Use it.

**12. Every hardware- or environment-touching page ends with how to verify it, including
without the hardware.** Sections `Testing`, `Without hardware`, `Troubleshooting`. The
troubleshooting entries are symptom-first: bold the symptom or exception name, then the
cause, then the fix.

> - **`is_steady()` never becomes `True`** — a servo stream is still holding its last
>   target. Call `robot.hold()`.

## Sentence-level habits

| Do | Not |
|---|---|
| "You can simply pip install this package." | "Installation is straightforward and can be accomplished via pip." |
| "We recommend X for Y. X is optional: Z remains fully supported." | "X is the best choice for Y." |
| "This is a design choice that should be made by the user of the interface." | (silently choosing and not saying so) |
| "Best to look at the existing interfaces for inspiration." | "Please refer to the existing interfaces." |
| "so only a handful of openings and force classes exist" | "granularity is limited" |
| "Read your own arm's limits off its controller rather than typing them by hand" | "It is recommended that limits be obtained programmatically." |
| "which is absolutely ludicrous" (about a real third-party wart) | pretending the wart is reasonable |

Contractions are used sparingly ("don't", "dont" even slips through — do not imitate typos,
but do not sanitize the register either). Sentences are medium-length and can run long when
carrying a chain of reasoning; they never pad. Cut every "in order to", "it should be noted
that", "leverage", "seamlessly", "comprehensive solution", and every sentence that only
restates the heading.

## Structure conventions (secondary, but consistent)

- **Emoji on headings only at the top level.** The root `README.md` and
  `docs/getting_started.md` put one trailing emoji on most `##`/`###` headings
  (`## Installation 🔧`, `### Testing 🧪`). Package READMEs and hardware/setup notes use
  **none**. Match the level you are writing at.
- **Tables for anything enumerable**, with the reason as a column: `Hardware | Communication
  | Implementation`, `Method | Available`, `Method | Why`, `Workflow | Runs When`,
  `Package | Description | Owner`.
- **A `Structure` section as an annotated tree** when a package has more than a few modules:
  ```
  airo_robots/
      manipulators/
          position_manipulator.py    # base classes for position-controlled manipulators
          hardware/                  # contains the implementations of the interfaces
  ```
- **Numbered steps only for genuinely ordered procedures** (installation, calibration
  bring-up); bullets everywhere else.
- **A table of contents only on a long top-level README**, never on a package README.
- **Changelog entries**: under `## Unreleased`, in the right sub-section, prefixed with the
  package and starting with a past-tense verb — `` - `airo-robots`: Added support for
  UR20. ``

## Before finishing

- Does the first sentence say plainly what this is?
- Is every constraint, refusal and "not supported" accompanied by its reason?
- Is anything uncertain marked as uncertain, and anything hard bolded flat?
- Do the code blocks run, with real values?
- Have you said what this page does not cover and where to go instead?
- Can the reader verify it worked — and try it without the hardware?
- Read it aloud: does it sound like a colleague, or like a landing page? If any sentence
  could appear in a press release, rewrite it.

`references/rewrites.md` has fuller before/after passages for calibrating a rewrite.
