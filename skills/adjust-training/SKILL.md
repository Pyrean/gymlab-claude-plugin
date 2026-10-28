---
name: adjust-training
description: "Use when a Gymlab member wants to change training they already have in Gymlab rather than start something new: swap an exercise in a saved workout for another (because of their equipment or their limitations, or because they dislike it), remove, reorder or re-target exercises, change sets, reps, weights or rest, group exercises into a superset or split one, rename a plan, change how many sessions a week a plan asks for, or make another plan the one they follow. Covers reading the workout first, finding alternatives that fit the member, showing the change and waiting for their confirmation, and saving it once."
---

# Adjust the member's training

Change a workout or a plan the member already has in Gymlab, with their agreement. "The member" is the person you are talking with.

## Before you start

If no Gymlab tools are available in this conversation (their names start with gymlab_), Gymlab is not connected. Tell the member, and ask them to connect Gymlab in this assistant's settings and come back to it. Without the tools nothing can be read or saved: never say that something was saved unless a tool call saved it.

Every change in this skill overwrites what is stored. Show the member exactly what will change and save only after a clear yes. When they change their mind, show the corrected change again.

Then choose the path:

- **Change the exercises of a saved workout** (swap, remove, reorder, new set targets, supersets): section 1.
- **Find another exercise for one that does not suit them**: section 2, then section 1.
- **Rename a plan, or change how many sessions a week it asks for**: section 3.
- **Make another plan the one they follow**: section 4.

What this skill does not do:

- A new plan, or a plan rebuilt around a new goal or schedule: use the plan-training skill. A single new workout: the build-workout skill.
- Adding an exercise to a workout: `gymlab_add_exercise_to_workout`. A workout's name, description, difficulty or equipment list: `gymlab_update_workout`.
- A session they already logged cannot be changed with these tools; the member edits it in the Gymlab app. Marking a plan as achieved, and adding or removing a plan's workouts, are done in the app too.

## 1. Edit a saved workout's exercises

### Read the workout first

Find the workout: `gymlab_get_next_workout` for their next one, `gymlab_get_plan` for the workouts of a plan, or `gymlab_list_workouts` by name (paged with `next_cursor`). Then read it with `gymlab_get_workout`. It returns every exercise with its `plannedExerciseId`, set targets and `superset_group`, and the workout's `revision`. Each edit names exercises by `plannedExerciseId`, and the save needs that `revision`. Read `gymlab_get_preferences` too when the change involves loads or supersets: `unitSystem` and `supersetPreference`.

### Build the operations

`gymlab_edit_workout_exercises` takes `workout_id`, `expected_revision` (the `revision` you read) and `operations`: 1 to 30 changes, applied in the order given. Everything an operation does not name stays as it is.

| Operation        | Takes                                                                                                                                                                                   | What it does                                                                                                                                                                                                                                                                              |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `replace`        | `planned_exercise_id`, `exercise_id` (or a new exercise's details, as the create tools take them), `exercise_type` when not the exercise's default, `sets`, `rest_seconds_between_sets` | Puts another exercise in the old one's place, in its superset if it was in one. Without `sets` it keeps the old one's sets with the targets the new exercise measures and their RPE.                                                                                                      |
| `remove`         | `planned_exercise_id`                                                                                                                                                                   | Takes the exercise out of the workout.                                                                                                                                                                                                                                                    |
| `move`           | `planned_exercise_id`, `to_position`                                                                                                                                                    | Moves it to that position, counted from 1 in the list as the earlier operations left it. An exercise in a superset moved outside it takes the whole superset along, which then starts at that position, or ends the workout when it does not fit; show the member the order that results. |
| `set_targets`    | `planned_exercise_id`, `sets`, `rest_seconds_between_sets`                                                                                                                              | Replaces the exercise's sets with the list you give: give every set it should have, the unchanged ones too.                                                                                                                                                                               |
| `group_superset` | `planned_exercise_ids`                                                                                                                                                                  | Makes 2 to 10 exercises one superset. They must be next to each other after the earlier operations, and have the same number of sets.                                                                                                                                                     |
| `ungroup`        | `planned_exercise_ids`                                                                                                                                                                  | Turns those exercises back into straight sets.                                                                                                                                                                                                                                            |

- `sets` take the targets the exercise's type measures, as in the create tools: `target_reps`, `target_weight_kg`, `target_duration_secs`, `target_distance_km`, and `target_rpe`. Set loads with `references/loads.md` and choose types with `references/exercises.md`.
- A superset whose members have different set counts is refused rather than trimmed. When the member wants such a superset, propose the set counts with `set_targets` first and include them only with their agreement.
- When `supersetPreference` is `avoid`, do not propose a superset. When the member asks for one all the same, say that their saved preference is to avoid supersets and follow what they ask now.
- One call is one change of the workout: everything is saved, or nothing is.

### Show the change and wait for confirmation

Show the member every exercise that changes, before and after, with sets × reps (or time or distance), loads in their units, RPE and rest, and the order of the workout when it changes. Tell them two things before they confirm:

- The change applies to the next sessions of this workout, in every plan it is part of. Sessions they already logged stay as they are, and a session they have already started on their phone keeps the exercises it started with.
- If they have this workout open in the Gymlab app's workout editor, they should close it without saving: saving that editor afterwards can overwrite this change.

### Save it

After a clear yes, call `gymlab_edit_workout_exercises`. The result is the workout as saved, with its new `revision`; use that revision for a further edit, and tell the member what changed.

When the call fails:

- **The workout changed after you read it** (the refusal says so; for example it was edited in the app): nothing was saved. Read it again with `gymlab_get_workout`, rebuild the operations against what is stored now, and show the member the change again when anything differs. Never resend the old `revision`.
- **A refusal for invalid input**: nothing was saved. Fix the operation the error names and try again; a changed exercise or set needs the member's agreement first.
- **The save may or may not have happened** (an unknown failure): read the workout with `gymlab_get_workout` before anything else. When the change is there, report it as saved; when it is not, save again with the revision you just read.

## 2. Find another exercise

When an exercise does not suit the member (they lack its equipment, it does not feel right, or they dislike it):

1. Take its `exercise_id` from the workout, or find it with `gymlab_search_exercises`.
2. Ask what is wrong with it when they have not said, and read `gymlab_get_preferences` for `availableEquipment` and `avoidExercises`. Their answer today wins over saved preferences.
3. Call `gymlab_find_exercise_alternatives` with `exercise_id`, `available_equipment` (the equipment slugs they have, at most 20; leave it out when equipment does not matter), `exclude_muscles` (muscle slugs to keep out, at most 10, for example a muscle the member wants to spare) and `limit` (1 to 10, 5 by default). It returns catalog exercises that work the same main muscle as `alternatives`, best fit first, each with the muscles it trains, its equipment, `exerciseType` and `allowedExerciseTypes`.
4. Leave out alternatives the member avoids or that their limitations rule out. The muscle filter follows how the catalog tags exercises; it does not judge whether a movement suits the member's limitations. When `empty_reason` says nothing fits, loosen one filter with the member's agreement, or search by name with `gymlab_search_exercises`.
5. Offer two or three, each with a line on why it fits (the muscles it trains, the equipment it needs), and let the member pick.
6. Decide the new exercise's sets. A `replace` without `sets` keeps the old exercise's sets: each keeps the reps, weight, duration and distance targets the new exercise's type measures and its RPE, and drops the others, so a bodyweight exercise takes no weight. The weight carries over as the same number, so check it with `references/loads.md` against the new exercise's `currentEstimatedOneRepMax` (from `gymlab_search_exercises` or `gymlab_get_exercise`, null when Gymlab has none): a movement loaded differently, such as dumbbells for a barbell lift, usually needs another weight, given the way `references/loads.md` says dumbbell weights are written. When the carried targets suit the new exercise, leave `sets` out. Give every set with `sets` when they do not, or when a target the new exercise needs is missing, such as a weight for the loaded variant of a bodyweight exercise or a duration for a hold.
7. Save it as a `replace` operation (section 1), shown and confirmed like any other change, with the sets it will have.

Exclusions are absolute. An exercise the user asked to avoid — or any movement their limitations rule out — must NEVER appear, even where it would have been the ideal choice. When you are unsure whether a movement suits their limitations, leave it out. Substitute a functionally similar movement instead, and if that weakens the session, say so in that workout's `description`. You may deliberately program work that strengthens a described weak point, provided the movement itself stays within their limitations.

Keep to training: plan around the limitations the member describes, and do not guess at their cause. When they say a movement does not feel right, leave it out and do not push it.

## 3. Rename a plan or change its sessions per week

1. Find the plan with `gymlab_list_plans`, and read it with `gymlab_get_plan` when you need its workouts.
2. `gymlab_update_plan` takes `plan_id` and at least one of `name` (1 to 100 characters, in the member's language) and `target_per_week` (1 to 14 sessions a week, or null to clear it).
3. `target_per_week` is how often the member means to train with the plan. It does not add or remove workouts: the plan stays the same rotation, done more or less often. When they want a different structure, such as more workouts in the rotation or another split, that is a new plan: use the plan-training skill.
4. Show the change, wait for a yes, then call. It works on plans made in the app too. A plan with several goals is refused; the member changes it in the Gymlab app. These tools cannot mark a plan as achieved.
5. Give the member the result's `deep_link` to open the plan.

## 4. Choose the plan they follow

1. `gymlab_list_plans` returns `following`: the plan Gymlab counts as the one they follow, as `plan_id` with a `reason`: `selected` (they chose it), `recently_trained` (no choice made; the plan they trained most recently) or `first_active` (neither; their first active plan). It is null when there is none. Tell the member which plan that is before you change it.
2. After a clear yes, call `gymlab_set_active_plan` with the `plan_id`. Only an active plan can be chosen; another plan is refused, and the member reactivates it in the app first.
3. The result names the plan and `previous_plan_id`. Tell the member that the web app and assistants follow the new plan at once, and the Gymlab iPhone app from its next version: on an older version, they choose the plan in the iPhone app as well. The Apple Watch keeps its own choice. `gymlab_get_next_workout` then answers from the new plan.
