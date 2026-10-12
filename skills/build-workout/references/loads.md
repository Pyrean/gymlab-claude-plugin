# Choosing loads

This is how Gymlab's own generators set loads, rewritten for the Gymlab tools. Set every load from the member's own numbers wherever Gymlab has them, and from what they told you where it does not. In the passages taken from Gymlab's prompts, "the user" is the member.

## Prescribing

"1RM", "one rep max", "max single" and similar phrasings mean the most the user can lift for a single rep. NEVER program multiple reps at 100% of a 1RM.

Distinguish a 1RM statement ("my max is X") from a working-weight statement ("I usually do sets with X"). When the user gives both, program from the working weights and sanity-check them against the 1RM.

Every weighted set carries a concrete weight in kilograms: never a percentage of a one-rep max, and never "pick a load for RPE 8". Give `target_rpe` with it, so the member knows how hard the set should feel.

## The member's numbers

Gymlab keeps three kinds of numbers for an exercise the member has logged. `gymlab_get_exercise` returns them for one exercise and `gymlab_list_exercise_stats` for all of them, in kilograms, meters and seconds. `stats` is null for an exercise they have never logged.

- **Current estimated one-rep max** (`currentEstimatedOneRepMax`, for weight and reps exercises): `kg`, the most the member is estimated to lift for one rep today, with its likely range from `lowKg` to `highKg`. It already accounts for the time since they last trained the lift, so it is the anchor for working weights. Derive each load from the planned reps and target RPE with a standard reps-in-reserve table (roughly 80% of the estimate for 5 reps at RPE 8, roughly 70-75% for 10 reps at RPE 8), then round to an equipment-friendly load. Lean toward the low end of the range when it is wide.
- **Recent best set**: the strongest set they did in the last 6 weeks, from their recent sessions (`gymlab_list_workout_sessions`, then `gymlab_get_workout_session` for the sets). It is one observed set and an upper anchor, not proof that the member can repeat it for several working sets. For additional sets, conservatively reduce the load and/or reps. Choose the exact reduction from the member's goal, target RPE, planned set count, fatigue, and whether this is a deload. A heavy single such as 100 kg × 1 rep proves one repetition at 100 kg, not that 100 kg is safe to repeat across working sets. Cross-check every load you derive from the estimate against it: when the estimate asks for clearly more than they have recently managed for similar reps, come down toward what they have done.
- **Lifetime bests** in `stats`: the heaviest weight (`maxWeightKg`), the biggest set (`maxVolumeSet`, with its reps and weight), the best estimate ever (`estimatedOneRepMaxKg`, which is not today's), a tested single (`trueOneRepMaxKg`), the most reps (`maxReps`), the longest duration (`maxDurationSeconds`) and distance (`maxDistanceMeters`), how many workouts included it (`totalWorkouts`) and when they last did it (`lastPerformedAt`); `gymlab_get_personal_records` lists the records with the date each was set. Records are ceilings, not current ability: one set long ago, or on an exercise they have not done for months, overstates what they can do today. Use them to judge training age and how quickly lost fitness may return, and never prescribe beyond them: no load heavier than their heaviest weight, no more reps than their most reps, no hold longer than their longest duration.

When the member asks how strong they are now, quote the current estimate and say it is an estimate, not a lifetime best.

## Exercises without a current estimate

- Bodyweight reps: set working reps below their most reps (`maxReps`), leaving the reps in reserve the target RPE calls for, and below what they have done in the last 6 weeks when that is lower.
- Holds and other timed work: set durations below their longest duration (`maxDurationSeconds`) by the same reasoning.
- Distance work: judge target paces and distances from their recent sessions and their longest distance (`maxDistanceMeters`), allowing for how long ago they were set.

## A member without history

Where an exercise has no history, estimate from their other lifts and stated experience level. Where there is no history at all (a new member, who has only whatever performances they told you), program from the interview's answers and the conversation, and set loads from their age, sex, body weight and stated experience level as far as you know them. Where their sex is unstated, non-binary or undisclosed, set loads from experience level and body weight alone rather than guessing it.

`gymlab_get_preferences` gives the member's `gender` and their experience; Gymlab does not give you their age or their body weight. Ask for one of these only when it would change the loads, and accept that the member may prefer not to say. When you are unsure, start lighter: the first sessions show the real working weights, and the member can add weight from there.

## Loads the member can set up

Think in the units the member reads (`unitSystem` from `gymlab_get_preferences`) and choose loads that real equipment makes, then convert to kilograms for the tools: pounds × 0.45359237, kept to three decimals so it converts back to the load you chose (135 lb is 61.235 kg). Distances go in kilometers (miles × 1.609344).

**Metric equipment:**

- Olympic barbell: 20 kg
- Plates: 1.25, 2.5, 5, 10, 15, 20 and 25 kg
- Dumbbells: in 2 kg steps (2, 4, 6, 8, 10 kg and up)
- Kettlebells: 8, 12, 16, 20, 24, 28 and 32 kg
- Common barbell loads: 20 kg (the empty bar), 40 kg (bar + 2 × 10 kg), 60 kg (bar + 2 × 20 kg), then 80, 100, 120 and 140 kg

**Imperial equipment:**

- Olympic barbell: 45 lb
- Plates: 2.5, 5, 10, 25, 35 and 45 lb
- Dumbbells: in 5 lb steps (5, 10, 15, 20, 25 lb and up)
- Kettlebells: 12, 16, 20, 25, 35 and 50 lb
- Common barbell loads: 45 lb (the empty bar), 65 lb (bar + 2 × 10 lb), 95 lb (bar + 2 × 25 lb), 135 lb (bar + 2 × 45 lb), then 185, 225, 275 and 315 lb

## Dumbbells and kettlebells

When the description of `gymlab_create_plan` or `gymlab_create_workout` says that dumbbell and kettlebell weights are per implement, and the sets take `implement_count`, give the weight of one dumbbell or kettlebell and say with `implement_count` whether the set uses one or two: two 22.5 kg dumbbells are 22.5 kg with `implement_count` 2, not 45 kg. When the tool says nothing about it, give the weight the way the member's own logged sets record it, and when it could be read either way, say in the exercise's `notes` whether it is per dumbbell or for the pair.
