---
name: review-progress
description: "Applies when a Gymlab member asks how their training is going: whether a lift is progressing or has stalled, how strong they are now, what their personal records are or whether they set one recently, how consistent they have been or how long their week streak is, how strained or recovered their muscles are and what to go easy on, or whether they should deload, take a lighter week or change their plan. Covers which Gymlab tool answers which question, what each number means and how certain it is, how to report findings as observations with the numbers, and when to hand over to building a new plan or a workout."
---

# Review the member's progress

Answer from the member's own Gymlab data, read with the tools below during this conversation. Gymlab computes every number; you read them, say what they show and how certain they are, and leave the decision to the member. The definitions are in [references/metrics.md](references/metrics.md): read its section before you explain a strain score, an estimate or a record.

## Before the first call

- Pass the member's IANA time zone (for example `Europe/Amsterdam`) as `time_zone` to every tool that takes it: `gymlab_get_strain`, `gymlab_get_strain_history` (required), `gymlab_get_streak`, `gymlab_get_stats` (`frequency` and `calendar`), `gymlab_get_exercise_progress`, `gymlab_get_exercise` and `gymlab_list_exercise_stats`. Without it "today" is the day in UTC. Ask when you do not know it.
- A question about one lift needs its id: `gymlab_search_exercises` with the name. When several exercises match (back squat, front squat), ask which one. Each result already carries the member's `currentEstimatedOneRepMax` on that exercise.
- The tools return kilograms, kilometers or meters, and seconds. `gymlab_get_preferences` returns `unitSystem`; answer in the member's units.

## Which tool answers which question

| The member asks                                   | Call                                                                                    | Read                                                                                                                                |
| ------------------------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| How strong am I now on a lift?                    | `gymlab_get_exercise`                                                                   | `stats.currentEstimatedOneRepMax`: `kg`, the range `lowKg` to `highKg`, `basedOn`, `daysSinceObservation`                           |
| Where do all my lifts stand?                      | `gymlab_list_exercise_stats` (paged with `next_cursor`)                                 | the current estimate and lifetime bests per exercise                                                                                |
| What are my records? Did I set one lately?        | `gymlab_get_personal_records` (newest first, paged)                                     | `record_type`, `value` with `unit`, `achieved_at`, `workout_session_id`                                                             |
| Is a lift going up? How has it gone since a date? | `gymlab_get_exercise_progress` with `exercise_id`, `start`, `end`, `granularity`        | `estimate_at_range_start`, `estimate_at_range_end`, `estimate_change`, the `buckets`, and `qualifying_sets`, sets the estimate read |
| What were my average sets on a lift week by week? | `gymlab_get_stats`, `metric` `exercise_progress`, `exercise_name`, `days`               | weekly `avgWeight`, `avgReps`, `avgRpe`, `totalVolume`, `sets`                                                                      |
| What did I actually lift?                         | `gymlab_list_workout_sessions` (`from`, `to`), then `gymlab_get_workout_session`        | each set's reps, weight, RPE and superset                                                                                           |
| How much have I trained?                          | `gymlab_get_stats`, `metric` `frequency`                                                | sessions and seconds per day and per week                                                                                           |
| How consistent am I? What is my streak?           | `gymlab_get_streak`                                                                     | `week_streak`, `rest_days`, the month's days                                                                                        |
| How recovered am I? What should I go easy on?     | `gymlab_get_strain`                                                                     | `score`, each muscle's `effective_strain` and `recommendation`                                                                      |
| Have I been training harder lately?               | `gymlab_get_strain_history` with `start` and `end`                                      | daily `score` and `confidence`, `weekly`, `average`, `peak`                                                                         |
| Am I keeping up with my plan?                     | `gymlab_get_next_workout`, `gymlab_list_plans`, `gymlab_get_plan`, `gymlab_get_workout` | the plan in use, its workouts, the next one and its `previousSets`                                                                  |

A broad question ("how am I doing?") takes several reads: the frequency and streak for consistency, the current estimates of the member's main lifts and their recent records for strength, and today's strain with the last weeks of strain history for load. Read what the answer turns on, then answer; do not dump every tool.

## How to read the numbers

- **Now versus ever.** For "how strong am I", quote the current estimate, call it an estimate and give its range. The lifetime best (`estimatedOneRepMaxKg`, the `e1rm` record) never drops and can be years old. When `daysSinceObservation` is large, say the estimate has been carried forward and decays.
- **Certainty.** `basedOn` says what last moved the estimate. `measured` and `max_effort` sets are the firmest evidence; `submax` and `bound` sets (easier or unrated) can raise an estimate but never lower it. A wide range from `lowKg` to `highKg` means little recent evidence. Strain below the confidence threshold in references/metrics.md is a weak signal.
- **Records are standing bests**, updated in place. A date says when the current best was set; beaten records are gone, so "no records in May" does not mean no personal bests in May.
- **The streak counts weeks**, not days.
- **Strain is per muscle.** The whole-body score averages the major muscles, so one heavily loaded muscle can sit under a low score.
- **Trends.** For "is my bench going up", read `gymlab_get_exercise_progress` over the period the member means (the last 26 weeks when they name none): compare the estimate at the range's start and end. `qualifying_sets` are sets the estimate read, not proof of what changed it; leave out one marked `held_as_suspect`. A bucket marked carried had no qualifying set, so a flat stretch of carried buckets means no evidence, not a plateau. When the member trains the lift rarely, use `month` buckets.
- **Averages mix sets.** `exercise_progress` averages warm-ups and working sets of every exercise whose name matches; `strength_gains` compares a first and a last set, whatever they were. Confirm with real sets before you call a lift up or down.

## How to state findings

- Lead with the answer, then the numbers behind it, with dates and units: "Your estimated squat max is 142 kg (likely 136 to 148 kg), last moved by a set 4 days ago."
- Say what the numbers did, not why. Offer possible reasons only as possibilities, apart from the data ("this can be fatigue, a deload week or sets without an RPE").
- A change inside the steady rule in references/metrics.md is steady, not progress or decline.
- Say when data is thin: few sessions, a wide range, a large `daysSinceObservation`, low strain confidence, or an `empty_reason`. Say what would make it clearer (logging RPE, a few more sessions).
- Never invent or extrapolate a number. Every number you give comes from a tool result in this conversation, or is a conversion of one.
- No medical advice. Strain is a training-load signal, not a diagnosis. Pain, an injury, illness or unusual symptoms go to a qualified professional; do not interpret them.
- Never encourage a true one-rep max attempt without qualified supervision.

## When a deload or a plan change is worth raising

Raise it as an option, with the observations and their numbers, and let the member decide. Do not raise it on strain below the confidence threshold alone.

- **An easier day or a rest day:** today's strain shows a muscle at `avoid_direct_work` or `rest_or_deload`, or the day's sentence says strain is high across several muscle groups. Name the muscles, their strain and when they were last trained.
- **A lighter week:** the `weekly` strain averages have stayed well above the member's own earlier weeks for two weeks or more, and the main lifts have stopped moving: their estimates steady in `gymlab_get_exercise_progress`, no new `e1rm` or `weight` records in that time, top sets in recent sessions no heavier, or `avgRpe` rising at the same `avgWeight`. Both together are a stronger case than either alone. If the member also reports poor sleep or feeling run down, that adds to it.
- **A plan change:** the member trains far less often than the plan asks (`frequency` against the plan's `targetPerWeek`) for several weeks, their goal has changed, the main lifts have set no records and their top sets have not moved for a month or more despite regular training, or a major muscle has no sets week after week (`relative_set_index` and `completed_set_count` near zero) when their goal needs it.

State these as what the data shows ("your weekly strain has averaged 58 for three weeks, against about 40 before, and your squat records date from 3 August"), not as a diagnosis.

## Handing over

- The member wants a new plan, or theirs rebuilt or replaced: use the plan-training skill; it starts with `gymlab_plan_interview`. Pass on what you found as context for the member to confirm, not as answers.
- The member wants a workout for today or another day, for example a lighter session: use the build-workout skill; it starts with `gymlab_workout_interview`, whose `recovery` reads the same strain.
- The member wants to change a workout or plan they have, for example fewer sets on a tired muscle, another exercise for one that hurts, or fewer sessions a week: use the adjust-training skill.
- The member wants to save a session they finished: use the log-workout skill.
