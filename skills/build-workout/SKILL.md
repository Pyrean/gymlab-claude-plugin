---
name: build-workout
description: "Use when a Gymlab member wants a single workout for today or another day: a specific session (legs, upper body, a 30-minute conditioning workout) or the best workout for them right now, chosen from their muscle recovery and recent training. Also use when the member has a workout of their own they want saved in Gymlab. Runs the Gymlab workout interview, reads how the member's muscles have recovered and their numbers on each exercise, designs a session that fits their time and equipment, and saves it to their Gymlab account once they confirm it."
---

# Build a workout with Gymlab

Design one workout with a Gymlab member and save it to their Gymlab account, or save a workout they bring. "The member" is the person you are talking with.

## Before you start

If no Gymlab tools are available in this conversation (their names start with gymlab_), Gymlab is not connected. Tell the member, and ask them to connect Gymlab in this assistant's settings and come back to it. Without the tools nothing can be read or saved: never say that something was saved unless a tool call saved it.

Then choose the path:

- **The member gives you a complete workout to save as it is** (their own routine, or exercises they list with their sets, typed, pasted or in a photo or screenshot): follow "Saving a workout the member supplies" below. It needs no interview.
- **They want a plan of several workouts to repeat**: that is a training plan; use the plan-training skill instead.
- **They want a workout they already have changed** (another exercise, other sets or loads, a different order): use the adjust-training skill.
- **Otherwise you design the workout**, following steps 1 to 7.

When you know the member's time zone, pass it as an IANA name (for example Europe/Amsterdam) in `time_zone` wherever a tool takes it, so that days and recovery match their calendar.

## 1. Ask what to train

Every workout you design starts with `gymlab_workout_interview`: `gymlab_create_workout` saves a designed workout only with the `interview_id` of a complete interview.

Call `gymlab_workout_interview` without `interview_id`, with whatever the member has already told you in `answers` ("a 45 minute leg workout" is `focus` "legs" and `workout_length` "45 minutes"). Pass the `interview_id` it returns in every later call, and write `answers` in English with digits for numbers, whatever language you speak with the member.

When `open` starts with `focus`, ask one question: is there anything specific they want to train, or should you build an optimal workout for them based on their current strain and recent workouts? Pass what they name as `answers.focus`, or `optimal_focus` true when they leave it to you.

## 2. Confirm the rest

Once the focus is known, `open` lists what is still to be asked. Ask it in one message together with `to_confirm`: what Gymlab already knows that bears on this workout (saved preferences, and what the member told Gymlab's coach). Ask whether each item still holds and should apply today, and pass the reply as `confirmation` with its `confirmation_id`: `confirmed`, or `changed` with the corrected values in `answers`. Repeat until `status` is `complete`.

Only the member can confirm what Gymlab knows: show them every item of `to_confirm` before you send `confirmation`, and never send `confirmed` for something they have not seen. A confirmation counts only for a question an earlier call returned.

### How to ask

- Use everyday language and short sentences. Explain a training term the first time you need it.
- When the member is unsure, offer a concrete suggestion instead of leaving the choice open.
- Accept "not sure", "no preference" and "you choose" as answers, and do not ask again for the same thing.
- Keep unknowns apart from negative answers. Pass `limitations` as "none" only when the member said they have none: an unanswered injury question is not "none".
- If the member asks you something, answer briefly, then continue with the next question.
- Ask only what this plan or workout needs. Do not ask about their job, daily routine, body measurements or health history unless they raise it and it changes what you build.
- Do not give medical advice or diagnose an injury or pain. Plan around what the member tells you, and when pain is new, getting worse or has not been assessed, suggest they have it checked by a doctor or physiotherapist.
- Do not design the workout or suggest exercises before the interview is complete.

## 3. Read the recovery

Once the focus is known, every result carries `recovery`: how the member's muscles have recovered from their logged training.

- `status`: `ok`; `caution` when some muscles of the focus need easing off (they are in `under_recovered`); `low_confidence` when there is too little logged training for the numbers to be reliable (treat them as a weak signal and build mainly from the request); `unavailable` when there is no strain data, usually because nothing was logged recently (program normally).
- `focus_muscles`: the focus's muscles, each with `strain` from 0 (rested) to 100, `recommendation` and `last_trained`.
- `under_recovered`: the focus muscles that need easing off.
- `fresh`: up to four muscles outside the focus that are ready to train.
- `elevated`: the muscles outside the focus that need easing off.
- `blocks_focus`: every muscle of the focus needs rest. The interview's own `status` is then `needs_decision`.

How each `recommendation` shapes the workout is in "Recovery" in `references/workout-design.md`. For a focus the member named, recovery decides how much and how hard, never what they train. When some focus muscles need easing, tell the member once, in one or two sentences, what you will adjust.

**When the interview's `status` is `needs_decision`**, tell the member plainly that every muscle of this focus still needs rest, and ask whether to train something else (pass it as a new `answers.focus`) or keep the focus (pass it in `answers.notes` with how, for example "keep it light"). Do not decide for them.

**With `optimal_focus`** the member asked you to choose: build on the muscles in `fresh`, keep the `elevated` ones light or out, and avoid repeating what they trained most recently. Tell them in a sentence what you chose and why.

## 4. Read what the workout is built on

Read what this workout needs:

- `gymlab_get_preferences`: `unitSystem` (the units they read weights in), `supersetPreference`, their favourite and avoided exercises, rest and set style, and `text_preferences` (when two contradict each other, the newer one holds). Where the interview's answers differ, the answers win: the member confirmed them for today.
- `gymlab_list_workout_sessions`, then `gymlab_get_workout_session` for the latest two or three: what they trained recently and with what weights, for variety, continuity and loads.
- `gymlab_get_exercise` for each main lift you will load: `currentEstimatedOneRepMax`, the member's own rest on it (`userRestSeconds`), their note on it and its `allowedExerciseTypes`. `gymlab_list_exercise_stats` gives the same numbers for every exercise they have logged.

Never invent facts about the member to fill a gap: where something is unknown, use coaching judgement and start conservatively.

## 5. Design the workout

Design with `references/workout-design.md`, set loads with `references/loads.md`, and choose and measure exercises with `references/exercises.md`. The essentials:

- Match the focus, the member's ability and today's recovery; open with compound movements and work toward isolation.
- Use only the equipment they have today; leave out every exercise they excluded and anything that could aggravate an injury they described.
- Warm up each muscle group before its working sets.
- Every exercise comes from `gymlab_search_exercises` by its `exercise_id` and is measured by one of its `allowedExerciseTypes`. When the exercise you would choose does not fit today's equipment, an injury or what they avoid, find a substitute with `gymlab_find_exercise_alternatives` (`references/exercises.md`).
- Every weighted set has a concrete weight in kilograms, set from the member's own numbers wherever Gymlab has them.

Compute each workout's length yourself and keep it within 5 minutes of the length the member gave:

- **Exercise time:** for each set, `target_reps` × 3 seconds, or its `target_duration_secs`; for a set of a `distance` exercise, your own estimate of how long this member needs for that distance.
- **Rest:** (sets − 1) × `rest_seconds_between_sets` for each exercise in straight sets. A superset instead rests once after each round except the last, for the longest rest among its members.
- **Switching:** (number of units − 1) × 60 seconds, where a unit is one exercise in straight sets or one superset.

When you leave out `rest_seconds_between_sets`, Gymlab uses the member's own rest for the exercise, else the exercise's default, else 90 seconds: count with the same value. Warm-up and cool-down exercises count too.

Before you show it, check the draft: the length lands within 5 minutes of the target, every exercise works with today's equipment, nothing excluded or risky for an injury is in it, every muscle is warmed up, and every load comes from the member's numbers or a conservative estimate.

## 6. Present the workout and wait for confirmation

Show the complete workout before you save anything: its name and length, and its exercises in order, each with sets × reps (or time or distance), the load in the member's units, the target RPE and the rest, with supersets marked. Add a few plain sentences on why it fits them today, including any adjustment for recovery.

Ask the member to confirm it or say what to change, and change it with them as often as they like. When they change the focus, the length, their equipment or their limitations, record that with `gymlab_workout_interview` first: the workout is saved against the interview's answers.

## 7. Save the workout

After the member has confirmed, make `gymlab_create_workout` the only tool call of its turn. Pass:

- `interview_id`: the complete interview's, before its `valid_until`.
- `name` (at most 30 characters in English), `description` (what the workout is for and any adjustment you made for recovery), and `unit_system` from their `unitSystem`.
- `exercises` in order: `exercise_id`, `exercise_type` when it is not the exercise's default, `rest_seconds_between_sets`, `superset_group` for superset members, `instructions` or `notes` where useful, and `sets` with the targets the type needs (`target_reps`, `target_weight_kg`, `target_duration_secs`, `target_distance_km`) and `target_rpe`.

Write the name, description and notes in the member's language; exercise names stay English.

When the save is refused:

- **An exercise was refused**: no workout was saved, but exercises in the call may already have been added to the member's exercise library; tell them. Fix the exercise (`references/exercises.md`) and save again.
- **The interview was already used to save a workout**: that save most likely went through and only its reply was lost. Look for the workout with `gymlab_list_workouts` before you do anything else, and never save it a second time. Another workout needs a new interview.
- **The interview was refused as not complete, expired or changed**: call `gymlab_workout_interview` with the same `interview_id`: the call brings it up to date. It keeps the member's own answers for seven days after its last call, so a member who comes back to this conversation within that time goes on where they left off; after more than a day it puts what Gymlab has saved to them again, in `to_confirm`. After seven days it starts over, or is not found once deleted: then start a new interview without `interview_id`, and the answers have to be given again.

After saving, tell the member the workout is with their saved workouts in Gymlab, on its own and outside any plan, and give them `deep_link` to open it. It does not change their plan or which plan they follow. When they want a workout of their plan changed instead, use the adjust-training skill.

## Saving a workout the member supplies

1. Work from the workout the member gave you, whether they typed it, pasted it or showed it in a photo or screenshot; read it back to them when any part of it is unclear. Keep its exercises, sets, reps, loads and order, and do not improve it unless they ask. When they want it changed, it is a workout you design: start at step 1.
2. Find each exercise with `gymlab_search_exercises`. When several results could be the movement, show the member the candidates and use the one they pick.
3. When an exercise's `allowedExerciseTypes` do not include the way they measure it, tell the member and let them choose another variant or one of the allowed types.
4. Tell the member what you convert before you do: pounds to kilograms, a rep range to its lower bound with the range in `notes`, a circuit of 2 to 10 different exercises to one `superset_group`. An exercise without a rest gets the member's own rest for it, else the exercise's default, else 90 seconds.
5. Show the workout as it will be saved. After they confirm, save it with `gymlab_create_workout`, `member_supplied` true and no `interview_id`, as the only tool call of its turn.
