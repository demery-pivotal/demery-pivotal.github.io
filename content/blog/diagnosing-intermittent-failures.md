---
title: Diagnosing Intermittent Failures
date: 2021-08-10
---

To diagnose a frustrating intermittent failure, start by distinguishing three key ideas: failure, fault, and conditions. Then take a systematic approach to exploring the conditions and seeking the fault.
<!--more-->

## Anatomy of an Intermittent Failure
An intermittent failure always includes these three parts: *Failure*, *Fault*, and *Conditions*.
- A **failure** is an incorrect result.
- A **fault** is an incorrect process, step, or data definition.
- **Conditions** are combinations of inputs, states, and events that affect the results.

An **intermittent failure** means that some component has a fault that under some conditions causes a failure.

## Laws of Intermittent Failures
> If there's a failure, there's a fault.

If there's a failure (intermittent or not), some process, step, or data definition is incorrect. The fault may be in the system we're testing. It may be in the test. There may be multiple faults. But if there's a failure, there is at least one fault.

> Even if the failure is intermittent, the fault is not.

The fault is present even when no failure occurs. We may not know what the fault is, or where to look for it, but it's there.

> If the failure is intermittent, success or failure must depend on conditions that can vary.

If the fault is always present, the difference between success and failure must come from *varying conditions.* These varying conditions determine (among other things) whether the fault is executed, at what moment, and with what values.

> If the failure is intermittent, we are not in control of the conditions that determine success or failure.

We call a failure *intermittent* only when we cannot produce it at will. We cannot produce the failure at will because at least one relevant condition is not under our control. And very often we don't even know what the relevant conditions are.

## Diagnosing with Variables
Sometimes you can diagnose an intermittent failure quickly, guided by your intuition and expertise. But if you're frustrated, that's a sign that some important condition is outside of your control, and perhaps not yet in your awareness.

Consider trying this systematic approach to diagnosis, to generate a richer set of ideas to power your intuition and expertise.

### Step 1: Brainstorm Variables
Write down every variable you can think of that could possibly be involved. By *variable,* I don't mean just the things you might declare in some source code. I mean *anything that could vary.*[^variables]
For this step in particular, it's often helpful to team up with a few other people.

During this brainstorming step, do not limit yourself to variables that you think are *likely* to be involved. Accept any variable that could *possibly* be involved… and don't be too quick to conclude that a variable could *not* be involved. If you think of it, write it down. We will assess the utility of these variables in a later step.

Here are some ideas for identifying variables:
- **Start with the result.** If the failure is intermittent, we know the result is variable. Write it down as a starting point.
- **Include any conditions you know or suspect.** If you know of any conditions that seem to be involved, or if you *suspect* any, express them as variables and add them to the list. Even if you don't know how or why a condition is involved, writing it down can help trigger new ideas.
- **Work forward from each variable on the list.** If this variable were to change, what other variables might be affected?
- **Work backward from each variable on the list.** What other variables might affect the value of this one?
- **Look for states, durations, and the order of events.** Intermittent failures very often arise when a specific event occurs while some part of the system is in a specific state.


### Step 2: Categorize the Variables
Organize the variables into categories (and sub-categories, and sub-sub-categories, …), using whatever criteria seem appropriate.

As you categorize, you will very likely think of additional variables. Write them down and include them in your categorization.

You may want to use mind-mapping software or outlining software for this. Any tool that helps you categorize the variables will do.

Instead of categorizing, or in addition, you may want to create a graph of how the variables affect each other. Any kind of model helps you understand the relevant variables will do.

### Step 3: Select a Variable
Assess the categories and pick the one whose seem most likely to affect the results.

Assess the variables in the selected category and pick the one that seems most likely to affect the results.

### Step 4: Experiment by Varying the Variable
Assess the possible values of the selected variable and pick one that seems most likely to increase your understanding of what is happening.

Gain control of the selected variable[^gain-control], and arrange for it to have the selected value.

Run the test, or exercise the system in some other way, and observe the results.

Repeat with different values for the selected variable as long as it seems fruitful.

### Step 5: Repeat as Necessary
Given what you've learned from your experiments so far, return to whatever earlier step is most likely to increase your understanding.

[^gain-control]: Yeah, I know: Easier said than done.
[^variables]: For more ideas about variables and how to identify them, see [Testing With Variables](https://vimeo.com/34356209), a 20-minute video by Elisabeth Hendrickson and me.
