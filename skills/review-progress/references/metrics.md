# Gymlab's training numbers

What each number the read tools return means, how Gymlab computes it and how certain it is. Every definition here comes from the code that computes the number.

## Strain

Gymlab strain is an estimated training-load and recovery signal, not a medical diagnosis or proof of tissue damage. Gymlab looks at completed sets from the last 30 days, maps each exercise to the muscles it trains using each exercise-muscle strain contribution score or primary, secondary, and tertiary defaults, then estimates local muscle impulse from volume, relative load, reps, set RPE/effort, failure, novelty, and movement factors. Recent work follows fatigue and damage curves: fatigue decays steadily, while damage first rises toward a delayed peak before decaying, so strain can still climb the day after a hard session even without new training. Smaller muscles are also aggregated into major muscle groups, and effective strain is adjusted by systemic recovery inputs such as sleep, HRV, resting heart rate, and session-RPE load from workout duration times whole-session RPE; missing inputs use neutral defaults and reduce confidence.

Each muscle's `effective_strain` runs from 0 (rested) to 100. The whole-body `score` of `gymlab_get_strain` is the mean over the trainable major muscle groups, with a major that was not trained counted as 0:

chest, back, shoulders, quads, adductors, hamstrings, glutes, biceps, triceps, core, calves.

So a hard leg day can leave the quads at 85 while the score reads 30. Read the muscles, not only the score. The model gives each muscle one of five recommendations from its effective strain:

| effective_strain | recommendation      | What it means for that muscle today                        |
| ---------------- | ------------------- | ---------------------------------------------------------- |
| 0 to under 25    | `normal_training`   | train it as usual                                          |
| 25 to under 45   | `monitor`           | train it as usual                                          |
| 45 to under 65   | `reduce_volume`     | keep direct work for it lighter                            |
| 65 to under 80   | `avoid_direct_work` | skip heavy direct work and do light technique work at most |
| 80 to 100        | `rest_or_deload`    | give it the day off                                        |

`recommendation` at the top of `gymlab_get_strain` is a sentence, not one of the five values: the line the Gymlab app shows for the day. It is one of these:

- No muscle data, or confidence below 0.5: "Still learning your training — estimates improve as you log more workouts."
- No muscle above `monitor`: "Your body seems well rested. You're ready for your next workout."
- Two or more regions at `reduce_volume` or above: "Strain is high across several muscle groups — make today an easy session or a rest day."
- One region (a muscle and its sub-groups) at `reduce_volume` or above, for example: "Your chest is still carrying yesterday's training — keep direct work light today."

**Confidence.** `gymlab_get_strain` does not return the model's confidence; `gymlab_get_strain_history` does, per day, from 0 to 1. Below 0.5 the numbers are a weak signal: the Gymlab app then shows the learning sentence above instead of advice, and Gymlab's own workout generator treats the numbers as a rough guide only. Do the same: quote them if asked, say they are uncertain, and base no deload or rest-day suggestion on them alone. Confidence is lower when sets carry no weight or RPE, when a lift has no strength estimate yet, and when no sleep, resting heart rate or HRV data reaches Gymlab. When there is no score at all, `empty_reason` says why (usually: nothing logged in the days before).

Other fields of `gymlab_get_strain`: `relative_set_index` is the muscle's sets in `training_window` as a percentage of the member's most trained muscle (volume, not weight); `completed_set_count` and `weighted_set_count` are the sets that worked it, the second counting each set by how much its exercise works the muscle; `last_trained_at` is when it was last worked. `gymlab_get_strain_history` gives one score per day over a range of days, the `weekly` averages on the member's week start, the range's `average` and its `peak` day. A day has no score when nothing was logged in the days before it.

## Strength estimates

`gymlab_get_exercise` and `gymlab_list_exercise_stats` return `stats.currentEstimatedOneRepMax`:

Today's estimate of the member's one-rep max, in kilograms, with its 90% range from lowKg to highKg. Use it, and say it is an estimate, when the member asks how strong they are now. null when Gymlab has no estimate for this exercise.

- `kg`: Estimated one-rep max in kilograms, as of `date`.
- `lowKg`: Lower edge of the 90% range around `kg`.
- `highKg`: Upper edge of the 90% range around `kg`.
- `date`: Day this estimate is for (YYYY-MM-DD); normally today.
- `basedOn`: Evidence the estimate last moved on: 'measured' a true 1RM attempt, 'max_effort' a set at or near failure, 'submax' an easier working set, 'bound' a set that only proves a floor. Null when that is not known.
- `daysSinceObservation`: Days since the last set that moved this estimate. The estimate decays the longer this grows, so mention it when it is large.
- `kind`: 'observed' the user trained this exercise on `date`; 'carried' the estimate was carried forward from an earlier day.

The same `stats` carry lifetime bests, which are different numbers. `estimatedOneRepMaxKg`:

Best estimated 1RM ever recorded for this exercise (lifetime maximum, may be stale). Not the user's current strength.

`trueOneRepMaxKg` (with `trueOneRepMaxRpe`):

A measured single, not an estimate: one rep at RPE 9.5 or higher (or logged to failure) as one of the first 4 sets of the exercise in that session, warm-ups counted in the order. The heaviest such single ever logged.

How the current estimate is built, step by step (the Gymlab app's Analytics tab charts the same estimate as Est. 1RM; its records card shows the lifetime Best e1RM):

- Each completed working set of 10 reps or fewer is one observation: weight × effective reps (reps + reps in reserve from the RPE; below RPE 7 or unrated claims no reserve) through Lombardi (weight × reps^0.10). Sets over 10 reps or over 12 effective reps, and warm-ups, are excluded.
- Tiers: measured (a single at RPE 9.5+ or to failure, taken as the true maximum, with at most 30 prior reps in the session and 12 of them heavy; a later single is max_effort); max_effort (RPE 9+, failure or AMRAP: two-sided, it raises or lowers the estimate, and a set far below it lowers it only a little, with bounded influence); submax (RPE 7 to below 9: raises or confirms, never lowers); bound (no RPE, or RPE below 7: a floor only, raises but never lowers).
- Sets done in a superset can raise the estimated 1RM, never lower it, because an estimate describes a fresh lift.
- Fatigue allowance: after more than 12 earlier reps at 70%+ of the set's load its estimate is raised 0.25% per extra heavy rep (cap 6%), half of it added to the uncertainty.
- One belief per exercise, carried day by day; each day with observations moves it once: a measured single resets it to the bar (and clears any decline run), otherwise the best set not held back moves it by its tier. A set over 4 standard deviations from the estimate is held as a possible typo unless another set that day agrees within 10%, until a set within 14 days agrees within 10%; edit the set to fix a real typo.
- Sustained decline is a separate rule: a best rated set at least 4% below the estimate on 4 days in a row pulls the estimate down towards the best of those sets; one low day or a deload does not.
- Carry-forward: a day without a qualifying set carries the estimate and the range widens. The grace clock restarts on any day that trains the exercise or fully covers its muscles with other training. For 21 days after that (14 from age 65) nothing decays; then the estimate converges towards a floor of 75% of its value at decay onset, faster the less the muscles are trained elsewhere: with no maintenance a 100 kg estimate reads 99.5 kg after the first week past grace (99.3 kg from age 50, 99.1 kg from 65) and nears 75 kg over the following years.
- The likely range is the 90% band (±1.645 standard deviations in log space); a measured single collapses it for that day and readiness noise reopens it the next. Its lower edge is what rankings compare.
- Best e1RM on the records card is a lifetime record (the best per-session value, a measured single winning its session; it never decays). Current e1RM is today's filtered belief (the Est. 1RM tile of the current period), moved by evidence and detraining; the two usually differ.

`gymlab_get_exercise_progress` gives the same estimate over time for one exercise: per day, week (from Monday) or month, each bucket with its `start` and `end` dates; the estimate at the range's start and end and the change between them (`estimate_at_range_start`, `estimate_at_range_end`, `estimate_change`); today's estimate (`current_estimate`); and up to 16 `qualifying_sets`, sets the estimate read (those around its largest change first), each with its `workout_session_id`; one marked `held_as_suspect` looked implausible and was held back from the estimate. A bucket's estimate is the one on its last day, with its likely range. When it is marked carried, the bucket had no qualifying set: the number is the earlier estimate carried forward, not a new measurement. When it is marked as not maintained, the estimate was lowered for the time without training the lift. Without `start` and `end` the tool covers the last 26 weeks, and `truncated` says when the range was cut. When `currentEstimatedOneRepMax` is null, Gymlab has no estimate for the exercise: say so, and do not compute one yourself and present it as Gymlab's.

## Personal records

`gymlab_get_personal_records` holds standing records only: when a stronger performance is saved, the record's value, session and date are updated in place. A record's `achieved_at` therefore says when the member's CURRENT best was set; records that were later beaten are not kept, so a month with no records listed may still have had personal bests at the time. It is not a history of personal records.

One record per exercise and kind:

| record_type | The record                                                                                         | unit         |
| ----------- | -------------------------------------------------------------------------------------------------- | ------------ |
| `weight`    | heaviest weight on one completed working set                                                       | kg           |
| `reps`      | most reps in one completed working set                                                             | reps         |
| `volume`    | most weight × reps in one completed working set                                                    | kg*reps      |
| `duration`  | longest (or, for distance exercises, fastest) completed working set in seconds                     | seconds      |
| `distance`  | longest distance in one completed working set                                                      | km or meters |
| `e1rm`      | best per-session estimated 1RM (Lombardi (weight × reps^0.10); a measured single wins its session) | kg           |

Sets on dumbbell and kettlebell exercises may carry implement_count (1 or 2): weightKg is then one implement's weight, unless a logged set's weight_basis is 'total' (the combined weight of an older set).

## Consistency

`week_streak` from `gymlab_get_streak` counts weeks, not days: going back from today in periods of seven days, the number of periods in a row that each have at least one logged session. Call it a week streak. `rest_days` is the days since the latest session (0 when it was today or there is none). The month calendar gives the sessions per day. `gymlab_get_stats` with `frequency` counts completed sessions and their seconds per day and per week; the `calendar` metric lists one month's sessions. Workouts that were only saved or planned do not count anywhere.

## Summaries in `gymlab_get_stats`

- `exercise_progress` averages every completed set with a weight, warm-ups included, of every exercise whose name contains `exercise_name` ("bench" also matches incline and dumbbell bench presses). A change in the average can be a change in the mix of sets. Pass the exercise's full name, and confirm a trend with the sets in `gymlab_get_workout_session`.
- `strength_gains` compares the weight of the first and of the last set in the period, ignoring reps and set type: the last set can be a warm-up or a back-off set. Treat it as a rough pointer, never as a change in strength by itself.
- `avgRpe` is null in a week without rated sets, not zero.

## Saying how a number moved

Gymlab's own analytics copy is factual: a sentence says what the numbers did, never why, gives no praise, no advice and no judgement beyond the direction (up, down, steady, faster, slower), names no coach and never says "I". Use the same steady rule: a change in the estimated one-rep max of less than 1 kg, or of less than 1%, is steady; for any other number a change of less than one unit (kg, rep, set, second, km), or of less than 2%, is steady.

Separate what the data shows from possible explanations (fatigue, a deload, a typo, unrated sets); the estimate reads each training day's sets as one piece of evidence, so never state that a single set caused a change.

Estimates, likely ranges and estimated max reps are always called estimates. Measured means a maximal set the model reads as the maximum itself: a measured single (a true 1RM), or a measured rep maximum (a set to failure, an AMRAP or RPE 9.5+ among a session's first three). Never encourage the member to attempt a true 1RM without qualified supervision.

## Units

The tools return kilograms, kilometers or meters, and seconds, with RPE from 1 to 10. `unitSystem` from `gymlab_get_preferences` is the units the member reads; convert for them (1 lb = 0.45359237 kg, 1 mile = 1.609344 km) and round only what you show.
