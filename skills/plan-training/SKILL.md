---
name: plan-training
description: "Use when a Gymlab member asks for a new training plan or program, wants their current plan rebuilt, replaced or adapted to a new goal, schedule, injury or equipment, or brings a program they already follow (from a coach, a book or their own notes) to save in Gymlab. Interviews the member through the Gymlab plan interview, reads their logged training to set loads, designs the plan to Gymlab's coaching standards, saves it to their Gymlab account once they confirm it, and offers to make it the plan they follow."
---

# Plan training with Gymlab

Design a training plan with a Gymlab member and save it to their Gymlab account, or save a program they already follow. "The member" is the person you are talking with.

## Before you start

If no Gymlab tools are available in this conversation (their names start with gymlab_), Gymlab is not connected. Tell the member, and ask them to connect Gymlab in this assistant's settings and come back to it. Without the tools nothing can be read or saved: never say that something was saved unless a tool call saved it.

Then choose the path:

- **The member gives you a complete program to save as it is** (from their coach, a book, another app or their own notes, typed, pasted or in a photo or screenshot): follow "Saving a program the member supplies" below. It needs no interview.
- **Otherwise you design the plan**, following steps 1 to 7. This includes rebuilding or replacing a plan they have, and changing a program they supply.
- **They want a smaller change to a plan they have** (swap an exercise, other sets or loads, rename it, train more or less often with the same workouts, or follow another plan): use the adjust-training skill.

When you know the member's time zone, pass it as an IANA name (for example Europe/Amsterdam) in `time_zone` wherever a tool takes it, so that days and recovery match their calendar.

## 1. Interview the member

Every plan you design starts with `gymlab_plan_interview`: `gymlab_create_plan` saves a designed plan only with the `interview_id` of a complete interview. The interview holds the checklist; you hold the conversation.

1. Call `gymlab_plan_interview` without `interview_id`, with whatever the member has already told you in `answers`. Pass the `interview_id` it returns in every later call.
2. Ask about what `open` lists, in its order: the goal on its own first, then their current level and the scope, then everything else in one message together with `to_confirm`.
3. After each reply, call the tool again with only what is new or changed. Write `answers` in English with digits for numbers, whatever language you speak with the member, and keep their meaning: Gymlab reads them in English.
4. Show the member the items of `to_confirm` (what Gymlab already knows: saved preferences, what they told Gymlab's coach, a summary of their logged training) and ask whether each still holds. Pass their reply as `confirmation` with its `confirmation_id`: `confirmed`, or `changed` with the corrected values in `answers`.
5. Repeat until `status` is `complete`.

Only the member can confirm what Gymlab knows: show them every item of `to_confirm` before you send `confirmation`, and never send `confirmed` for something they have not seen. A confirmation counts only for a question an earlier call returned.

Read the other fields as they come:

- `remembered_frequency` is how often the member said they train. Ask how often they want to train with this plan all the same.
- `training_history` summarizes their logged training while their current level is open. When it covers what the goal needs, pass `current_level_from_history` true instead of asking; the member then confirms it with the other saved details.
- `rejected` lists what was not recorded and why: `saved_goal` or `remembered_frequency` when you passed a value saved earlier (ask for the member's own answer for this plan), `no_training_history` when there is no logged training to take the level from (ask them).
- `goal_changed` true means the current level, scope and timeline given for the earlier goal were dropped: ask them again for the new goal.

### How to ask

- Ask one short, easy-to-answer question at a time, apart from the one combined message for the remaining topics and `to_confirm`.
- Use everyday language and short sentences. Explain a training term the first time you need it.
- When the member is unsure, offer a concrete suggestion instead of leaving the choice open.
- Accept "not sure", "no preference" and "you choose" as answers, and do not ask again for the same thing.
- Keep unknowns apart from negative answers. Pass `limitations` as "none" only when the member said they have none: an unanswered injury question is not "none".
- If the member asks you something, answer briefly, then continue with the next question.
- Ask only what this plan or workout needs. Do not ask about their job, daily routine, body measurements or health history unless they raise it and it changes what you build.
- Do not give medical advice or diagnose an injury or pain. Plan around what the member tells you, and when pain is new, getting worse or has not been assessed, suggest they have it checked by a doctor or physiotherapist.
- Do not design the plan or suggest exercises during the interview. The plan follows once the interview is complete and the member has agreed the number of workouts and their order.

What to find out, and how to handle unknowns, schedules and experienced members: `references/interview.md`.

## 2. Check the goal against the details

When `status` is `complete`, judge the goal against their current level, the timeline and the confirmed details: how often they will train, how long a workout can be, their equipment and their limitations.

- If the goal is not realistic from where they stand in that time, propose a realistic target and ask whether to use it.
- If the goal and the details conflict, or one concrete change would clearly help (for example a mobility goal with gym access only: one short session that also works at home), propose it once, in one or two sentences, and ask.
- When goals and available time conflict, explain the trade-off briefly and ask what matters most.

Record the member's answer with `gymlab_plan_interview`: a new target as `goal` (which reopens their current level and scope), anything else as `scope` or `notes`. Propose each change once and respect their decision. When nothing needs proposing, say in a line what you will build (the number of workouts, their order and length) and go on.

## 3. Read what the plan is built on

Read what this plan needs, and skip what does not apply (a member with no logged training has no stats):

- `gymlab_get_preferences`: `unitSystem` (the units they read weights in), `supersetPreference`, their favourite and avoided exercises, rest and set style, and `text_preferences` (when two contradict each other, the newer one holds). Where the interview's answers differ, the answers win: the member gave or confirmed them for this plan.
- `gymlab_list_exercise_stats`: the exercises they have logged, with lifetime bests and `currentEstimatedOneRepMax`, today's estimate of their one-rep max.
- `gymlab_get_exercise` for each main lift you will load: its current estimate, the member's own rest on it (`userRestSeconds`), their note on it, and its `allowedExerciseTypes`.
- `gymlab_list_workout_sessions` and `gymlab_get_workout_session`: the weights and reps they actually did lately.
- `gymlab_get_personal_records`, when you need a ceiling the stats do not show.

These numbers set the loads (`references/loads.md`). Never invent facts about the member to fill a gap: where something is unknown, use coaching judgement and start conservatively.

## 4. Design the plan

Design with `references/plan-design.md`, set loads with `references/loads.md`, and choose and measure exercises with `references/exercises.md`. The essentials:

- A plan is an ordered rotation of workouts with no calendar dates: as a rule one workout per training day the member asked for, in the order they train them, repeated from the top.
- Every workout uses only the equipment they have, leaves out every exercise they excluded and anything that could aggravate an injury they described, warms up each muscle group before its working sets, and fits the agreed length.
- Every exercise comes from `gymlab_search_exercises` by its `exercise_id` and is measured by one of its `allowedExerciseTypes`. When the exercise you would choose does not fit their equipment, an injury or what they avoid, find a substitute with `gymlab_find_exercise_alternatives` (`references/exercises.md`).
- Every weighted set has a concrete weight in kilograms, set from the member's own numbers wherever Gymlab has them.

Compute each workout's length yourself and keep it within 5 minutes of the length the member gave:

- **Exercise time:** for each set, `target_reps` × 3 seconds, or its `target_duration_secs`; for a set of a `distance` exercise, your own estimate of how long this member needs for that distance.
- **Rest:** (sets − 1) × `rest_seconds_between_sets` for each exercise in straight sets. A superset instead rests once after each round except the last, for the longest rest among its members.
- **Switching:** (number of units − 1) × 60 seconds, where a unit is one exercise in straight sets or one superset.

When you leave out `rest_seconds_between_sets`, Gymlab uses the member's own rest for the exercise, else the exercise's default, else 90 seconds: count with the same value. Warm-up and cool-down exercises count too.

Review the draft against "What a finished plan satisfies" in `references/plan-design.md` and fix whatever falls short before you show it.

## 5. Present the plan and wait for confirmation

Show the complete plan before you save anything:

- The plan's name, the number of workouts and their order.
- For each workout: its name, its length, and its exercises in order, each with sets × reps (or time or distance), the load in the member's units, the target RPE and the rest, with supersets marked.
- Why the plan fits them (step 7).

Ask the member to confirm it or say what to change. Change it with them as often as they like, recompute the lengths, and show what changed. Save only the version they confirmed. When they change the goal, how often they train, the length, their equipment or their limitations, record that with `gymlab_plan_interview` first: the plan is saved against the interview's answers.

## 6. Save the plan

After the member has confirmed, make `gymlab_create_plan` the only tool call of its turn. Pass:

- `interview_id`: the complete interview's, before its `valid_until`.
- `name` (at most 35 characters in English), `goal` in the member's words, and `unit_system` from their `unitSystem`.
- `workouts` in training order, each with a `name` (at most 30 characters in English), a `description` (what the workout concentrates on, why it is in the plan and in this place, and any adjustment you made), and its `exercises` in order: `exercise_id`, `exercise_type` when it is not the exercise's default, `rest_seconds_between_sets`, `superset_group` for superset members, `instructions` or `notes` where useful, and `sets` with the targets the type needs (`target_reps`, `target_weight_kg`, `target_duration_secs`, `target_distance_km`) and `target_rpe`.

Write names, descriptions and notes in the member's language; exercise names stay English.

When the save is refused:

- **An exercise was refused**: nothing of the plan was saved, but exercises in the call may already have been added to the member's exercise library; tell them. Fix the exercise (`references/exercises.md`) and save again.
- **The interview was already used to save a plan**: that save most likely went through and only its reply was lost. Look for the plan with `gymlab_list_plans` before you do anything else, and never save it a second time. Another plan needs a new interview.
- **The interview was refused as not complete, expired or changed**: call `gymlab_plan_interview` with the same `interview_id`: the call brings it up to date. It keeps the member's own answers for seven days after its last call, so a member who comes back to this conversation within that time goes on where they left off; after more than a day it puts what Gymlab has saved to them again, in `to_confirm`. After seven days it starts over, or is not found once deleted: then start a new interview without `interview_id`, and the answers have to be given again.

After saving, give the member `deep_link` to open the plan. It is saved beside the plans they already have, which stay as they are.

Then ask whether this plan should be the one they follow. On a yes, call `gymlab_set_active_plan` with its `plan_id`, and tell them that the web app and assistants follow it at once and the Gymlab iPhone app from its next version; on an older version they choose the plan in the iPhone app as well. On a no, nothing changes: the plan they follow stays as it is.

## 7. Explain the plan

When you present the plan, explain it in plain words: what it helps them achieve, what it does (how many workouts, how often, the structure, how long a session takes), why each main choice fits them, and how it works around their limitations. Back choices with research only where a well-established finding applies, and never invent or guess a study, a figure or who found it. Everything you say must match the plan. Make no diagnosis or medical claim, and promise no result or timeline. The details are in "Explaining the plan" in `references/plan-design.md`.

## Saving a program the member supplies

1. Work from the program the member gave you, whether they typed it, pasted it or showed it in a photo or screenshot; read it back to them when any part of it is unclear. It is their program: keep its exercises, sets, reps, loads and order, and do not improve it unless they ask. When they want it changed, it is a plan you design: start at step 1.
2. Find each exercise with `gymlab_search_exercises`. When several results could be the movement, show the member the candidates and use the one they pick. Add an exercise of their own only when nothing in the library is the same movement (`references/exercises.md`).
3. When an exercise's `allowedExerciseTypes` do not include the way the program measures it, tell the member and let them choose another variant or one of the allowed types.
4. Before you convert anything the plan cannot hold as written, tell the member what will change and get their agreement:
   - A superset or circuit of 2 to 10 different exercises done back to back becomes adjacent exercises sharing one `superset_group`. A longer circuit is saved as its exercises in order, with the grouping in each exercise's `notes`.
   - A rep range is saved as its lower bound, and "as many as possible" as a number the member names, each with the original wording in `notes`.
   - A percentage of a max becomes kilograms when the member gives the max, or, if they agree, from the current estimate Gymlab has for the lift. Otherwise the set is saved without a weight, with the percentage in `notes`.
   - Pounds become kilograms (× 0.45359237) and miles kilometers (× 1.609344); `unit_system` makes the app show their units again.
   - An exercise the program gives no rest gets the member's own rest for the exercise, else the exercise's default, else 90 seconds.
5. Show the member the program as it will be saved. After they confirm, save it with `gymlab_create_plan`, `member_supplied` true and no `interview_id`, as the only tool call of its turn. Keep the program's own plan and workout names.
6. Give the member `deep_link`, and offer to make the program the plan they follow, as in step 6.
