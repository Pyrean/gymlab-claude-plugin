---
name: log-workout
description: "Applies when a Gymlab member has finished a workout and wants it saved to their Gymlab history: they say they did today's or another saved workout, list or paste the sets they did, report a run, ride or other timed or distance work, or ask to log, record or save a session. Covers matching the session to a saved workout or the next workout, collecting what was actually done set by set, converting pounds and miles, effort as RPE, the date and time zone, supersets and exercises outside the workout, confirming with the member, and saving the session exactly once."
---

# Log a finished workout

Saving a session adds it to the member's history and updates their totals, personal records, strength estimates and week streak. A logged session cannot be changed, undone or deleted with these tools; the member can only edit or delete it in the Gymlab app. So collect it carefully, show it to the member, and save it once, after they confirm.

## 1. Find the workout it belongs to

Ask, or take from what the member said, whether they did a workout saved in Gymlab.

- **Their plan's next workout** ("today's workout", "day 3"): `gymlab_get_next_workout` returns the plan, the workout's id and its exercises with `plannedExerciseId`, set targets and `previousSets`. Check the workout's name with the member before you rely on it.
- **Another saved workout:** find it by name in `gymlab_list_workouts` (paged with `next_cursor`), then read it with `gymlab_get_workout`.
- **Nothing saved** (an improvised session, a class, a run): leave `workout_id` out.

Logging against the saved workout (`workout_id`, and each of its exercises' `plannedExerciseId` as `planned_exercise_id`) ties the session to it, so it counts as done in the member's plan.

Then check that it is not logged already: the member may have saved it in the Gymlab app or on their watch. Call `gymlab_list_workout_sessions` with `from` and `to` set to the session's day. When a session on that day looks like this one, ask before you log another.

## 2. Collect what was actually done

Go exercise by exercise, in the order performed, and record every completed set as it was done. Set targets are not results: use them to ask ("the plan had 3 × 8 at 60 kg; how did that go?"), and record missed reps, changed weights and extra sets as they happened. Leave out an exercise the member skipped; every exercise needs at least one completed set. Do not fill in a number the member did not give: ask for it.

Use the exercise's `exerciseType` from the workout, or another of its `allowedExerciseTypes` (from `gymlab_search_exercises` or `gymlab_get_exercise`) when the member measured it differently, for example a treadmill run logged by distance and time. The type decides each set's fields:

| exercise_type     | Every set needs                | It may also carry                                |
| ----------------- | ------------------------------ | ------------------------------------------------ |
| `bodyweight_reps` | `reps`                         | nothing else                                     |
| `bodyweight_time` | `duration_secs`                | nothing else                                     |
| `weighted_reps`   | `reps`                         | `weight_kg`; left out, `weight_kg` is saved as 0 |
| `weighted_time`   | `duration_secs`                | `weight_kg`; left out, `weight_kg` is saved as 0 |
| `distance`        | `distance_km`                  | nothing else                                     |
| `distance_time`   | `distance_km`, `duration_secs` | nothing else                                     |
| `time_only`       | `duration_secs`                | nothing else                                     |

So on a weighted exercise ask for the weight rather than leaving it out, unless it really was bodyweight only. For distance work, give distance and duration; Gymlab works out the pace. When the member gives a pace, the duration is the pace times the distance.

**Effort.** Each set may carry `rpe`, from 1 to 10 in half steps. When the member gives reps in reserve, RPE is 10 minus that number; "to failure" is RPE 10. Ask for it on the main lifts if they know it, because Gymlab's strength estimates read rated sets as firmer evidence than unrated ones, but never invent one: leave it out when unknown. `session_rpe` is the effort of the whole session; left out, it is the average of the sets' RPE.

**Duration.** `duration_secs` is required: how long the whole session took, in seconds, at most 24 hours. Ask for it, or for the start and end times. If the member can only estimate it, use their estimate and say so in the summary.

**Title.** Optional, at most 120 characters; a saved workout's name is the natural title.

## 3. Units

The tool takes kilograms, kilometers and seconds. Convert exactly, and keep at least three decimals instead of rounding to a plate or a whole kilometer, so the session reads back in the member's units as they said it:

- Pounds to kilograms: multiply by 0.45359237 (exact). 135 lb is 61.235 kg.
- Miles to kilometers: multiply by 1.609344 (exact). 3.1 miles is 4.989 km. Meters: divide by 1000.

Durations are seconds: 1:15:30 is 4530. In the summary, show the member's own units next to what you will send.

## 4. Dumbbells and kettlebells

Check the set fields of `gymlab_log_workout_session`. Only when they include `implement_count` does Gymlab count dumbbells and kettlebells, and the tool's description then says:

> Dumbbell and kettlebell weights are per implement: a weight is the weight of one dumbbell or kettlebell, and implement_count says whether the set uses one or two (omit it to use the member's usual count for the exercise).

So "two 20 kg dumbbells" is `weight_kg` 20 with `implement_count` 2. When the tool has no `implement_count`, send no count, and log the weight the way the member's earlier sets of that exercise read in `previousSets`; when "40 kg dumbbells" could mean each or both, ask.

## 5. Exercises that were not in the workout

Find each one with `gymlab_search_exercises` and use the `id` and `name` exactly as it returns them as `exercise_id` and `exercise_name`, with no `planned_exercise_id`. When several results could be it, ask which. When nothing fits after another search term or two, ask the member to pick the closest exercise or leave it out; never log their sets under an exercise they did not do.

## 6. Supersets

For exercises done back to back, one set of each per round, give every set of the superset a `superset_group` (one label for the group, for example "A") and a `superset_round` counted from 1: the sets of the different exercises in the same round share both. A set keeps the pair only when a set of another exercise in the session has the same pair; otherwise it is saved as a straight set. A planned `superset_group` shows what was intended; log what the member actually did.

## 7. When it ended

- `completed_at`: when the session ended, as UTC (for example `2026-06-05T18:00:00Z`). Leave it out when the member has just finished; it defaults to now. It cannot be in the future or before 1 January 2000.
- `time_zone`: the member's IANA time zone (for example `America/Chicago`). It decides the calendar day the session counts on, for the streak and strain. Ask when you do not know it, and convert their local time to UTC with it: a session that ended at 7:15 pm on 4 June in Chicago is `2026-06-05T00:15:00Z`, and with `time_zone` it still counts on 4 June.
- `completed_local_date`: only when the member's calendar day differs from what `completed_at` and `time_zone` give; it must be within one day of `completed_at`. Usually leave it out.

## 8. Confirm, then save once

Show the member what you will save: the workout (or that it is not a saved workout), the title, the day and end time in their time zone, the duration, and every exercise with its sets in their units, RPE and supersets. Tell them two things before they confirm:

- If they have connected Hevy to Gymlab, the session is also posted to their Hevy account as a public workout, as a session logged in the app is. It is not posted to any Gymlab community.
- Once saved, it can be changed or deleted only in the Gymlab app.

Save only after a clear yes. When they change something, show the corrected session again.

Before the first call, generate one UUID for this session and send it as `client_session_id`. Keep it for this session only:

- After an error or a timeout, call again with the same `client_session_id` (after fixing the field a validation error names). If the first call did save, the result has `already_logged` true and nothing is saved twice; report the session as saved and do not call again.
- If the error says the id belongs to a session started in the Gymlab app that is not finished, nothing was saved: generate a new id and call again.
- Every other session gets its own new id. Never reuse one for a different session, and never call again with a new id for a session that may already be saved: check `gymlab_list_workout_sessions` first.

One session per call, at most 50 exercises with at most 50 sets each.

## 9. After saving

Report what the result's `session` shows: its day (`completedLocalDate`), `setsCompleted` and `volume` (the total weight lifted, in kilograms; convert it). If you sent supersets, `gymlab_get_workout_session` shows which pairs were kept. Do not claim a new personal record unless `gymlab_get_personal_records` shows one set in this session. When the member asks how the session fits their progress, use the review-progress skill.
