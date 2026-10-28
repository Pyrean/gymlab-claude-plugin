---
name: getting-started
description: "Use when a member has just connected Gymlab or set up the Gymlab plugin, says hello without a clear request, or asks what Gymlab can do here, how to get started or what to do first. Reads where the member stands in Gymlab (setup finished in the app, a training plan, logged sessions) and answers with one short message that fits their situation: what they can do here, their next workout and how they are doing when they already train, and one question about what to do next."
---

# Get started with Gymlab

Welcome a member to Gymlab in this conversation: find out where they stand, then answer with one short message that fits it. "The member" is the person you are talking with.

## Before you start

If no Gymlab tools are available in this conversation (their names start with gymlab_), Gymlab is not connected. Tell the member, and ask them to connect Gymlab in this assistant's settings and come back to it. Without the tools nothing can be read or saved: never say that something was saved unless a tool call saved it.

## 1. Read where the member stands

Call these two tools before you write anything, in the same turn:

- `gymlab_get_preferences`, which takes no arguments.
- `gymlab_list_workout_sessions` with `limit` 1: the member's latest logged session, if there is one.

Then take the first case that applies:

| What the results show               | The member's situation                                   |
| ----------------------------------- | -------------------------------------------------------- |
| `preferences` is null               | They have not finished setup in the Gymlab app (case A). |
| `preferences.workoutPlanCount` is 0 | Set up, but no training plan yet (case B).               |
| `sessions` is empty                 | A plan, but nothing logged yet (case C).                 |
| anything else                       | They already train with Gymlab (case D).                 |

Cases C and D read one or two more tools, named below. When you know the member's IANA time zone, pass it as `time_zone` to the tools that take it; do not ask for it only for this message.

## 2. Write one message

Every case follows the same rules:

- **One message, one question.** Write a few short paragraphs or a short list at most, and end with exactly one question. Do not ask for anything else in the same message.
- **Only numbers a tool returned.** Every number you give (minutes, sets, a streak, a date) comes from a tool result in this conversation. Never estimate one, and leave out what no tool returned.
- **Gymlab only.** Say nothing about creating an account, what Gymlab costs or which features need what; the member is already connected.
- Write in the member's language, in plain words. Do not name tools or skills to the member.

What Gymlab can do here, in one or two sentences or a short list, when the case asks for it: design a training plan with them and save it to Gymlab, build a workout for today, review their progress (lifts, records, consistency, recovery), log a session they finished, and change a saved workout or plan.

### Case A: setup is not finished

Gymlab's setup happens in the Gymlab iPhone app. Its onboarding asks about their goals and training and builds their first plan.

- Say in a sentence what Gymlab can do here.
- Tell them to start with the onboarding in the Gymlab iPhone app, which builds their first plan, and to come back here once it is done.
- Ask whether they already have the Gymlab app on their iPhone.

Do not design a plan here or ask the onboarding's questions yourself.

### Case B: no plan yet

- Say what Gymlab can do here.
- Tell them that the first step is their first training plan, which you build together: a few questions about their goal, schedule, equipment and any limitations, then a plan saved to their Gymlab account, with a link to open it in the Gymlab app.
- Ask whether they want to start their first plan now.

When they say yes, use the plan-training skill; it starts with `gymlab_plan_interview`.

### Case C: a plan, nothing logged yet

Call `gymlab_get_next_workout`. Then:

- Name their next workout and its plan, with what `summary` gives (the number of exercises and sets, the estimated minutes). When `empty_reason` is present, say what it says instead.
- Say how logging works: they log a session in the Gymlab app on their iPhone or Apple Watch while they train, or they tell you afterwards what they did and you save it for them. Logged sessions are what their progress, records and streak are built from.
- Ask whether they want to go through this workout now.

### Case D: already training

Call `gymlab_get_next_workout` and `gymlab_get_streak`. Then, briefly:

- Their next workout and its plan, as in case C.
- Their last session, from `gymlab_list_workout_sessions`: its title and the day it ended.
- Their week streak (`week_streak`), called a week streak: it counts weeks with at least one session, not days. Leave it out when `empty_reason` says there is none.

End with one question that offers these choices: going through today's workout, reviewing their progress, logging a session, changing their training (a workout or a plan they have), or a new plan.

## 3. After the member answers

Go on with the skill for what they chose:

- Today's workout: walk them through it from `gymlab_get_next_workout`; for a different workout today, use the build-workout skill.
- Their progress: the review-progress skill.
- Logging a session: the log-workout skill.
- Changing a workout or a plan they have: the adjust-training skill.
- A new plan: the plan-training skill.
