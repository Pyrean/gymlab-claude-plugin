# Plan design guide

This is how Gymlab's own plan generator designs a plan, rewritten for the Gymlab tools where it named its own inputs. Work from the interview's answers and what the member told you. Where something is unknown, use science-backed coaching judgement, and never invent facts about the member to fill a gap. In the passages taken from Gymlab's prompts, "the user" and "the client" are the member.

## What a plan is

A Gymlab plan is an ordered rotation of workouts, without assigned calendar dates. Match the rotation to the client’s training days: five days means five workout entries unless they explicitly describe a different session pattern. Include runs that belong in that total and honor a separately stated workout count. Do not compress six requested workouts into three templates repeated twice. For an agreed range, choose a count within it and explain the recommendation; choose a suitable rotation when frequency is unspecified. Balance work across all goals and the full sequence.

Order them as the sequence you would have the user train, and assume they cycle through in that order and start over at the top.

Classify the user's goal(s) first, because it shapes the structure:

- **corrective** — a narrow posture, mobility or single-issue goal (fixing anterior pelvic tilt, building ankle stability to return to training after time off). A single-workout rotation may suit a focused request when no different workout count has been specified.
- **metric** — the goal is moving a measurable fitness number (raising VO2 max, running 5 km faster).
- **practice** — everything else: general strength, hypertrophy, skill work, conditioning, overall fitness.

## What a finished plan satisfies

A finished plan satisfies all of these:

- It is the strongest set of workouts you can build for this user's goal, experience level, limitations, sex, age, and available time.
- Every workout is executable with the equipment they have and inside the time they have.
- No exercise the user asked to avoid appears anywhere, and nothing in the plan goes against the limitations they described.
- Every muscle group trained in a workout is warmed up before its working sets.
- Muscle groups get enough recovery between consecutive workouts in the cycle, at the frequency the user intends to train.
- Every exercise earns its place: it is the best available way to advance the goal, not filler.
- Each workout's computed time lands within 5 minutes of the session duration.
- Every weighted set carries a concrete weight in kilograms — never a percentage of 1RM, never "pick a load for RPE 8".
- Balance muscle-group loading, exercise selection, volume and intensity across the ordered rotation, including recovery from its last workout back to its first. Honor requested effort and activity sequencing. Avoid accidental duplicate exercises and redundant loading of the same muscles in adjacent workouts; repeat work only deliberately to serve the goal or an explicit preference, accounting for its total workload and recovery.

Review your draft against this list and fix whatever falls short before you show it to the member.

## Programming

Match exercise difficulty to the user's experience level. Open with compound movements and work toward isolation. Account for fatigue accumulating through the session.

Program from established strength and conditioning scientific evidence rather than novelty. Favor movements with a well-documented effect on the quality you are training and a good stimulus-to-fatigue ratio — loadable compounds that train a lot of muscle per unit of time and recover predictably — and add isolation work only where a specific muscle or weak point needs it.

Session time is the scarce resource, so every exercise you include displaces another. Cut anything inefficient, redundant with a movement already in the workout, or present only for variety. The user's favorites are the one exception (see below).

Weigh each exercise's benefit against the risk it carries, and take fatigue into account: some movements get materially riskier the deeper into a session they sit.

## Session length

Each workout's target is the length the member gave in the interview (`workout_length`). When they gave different lengths for different activities, each workout takes the length of its activity.

Compute a workout's total time as the sum of:

- **Exercise time** — for time-based sets (`bodyweight_time`, `weighted_time`, `distance_time`, `time_only`), the sum of each set's `target_duration_secs`. For rep-based sets (`bodyweight_reps`, `weighted_reps`), the sum of `target_reps` × 3 seconds across sets. For sets of a `distance` exercise, your own estimate of how long this member needs for that distance on that exercise, judged from their age, training level and recent performance — there is no fixed pace (a 500 m row or a 5 km run takes a beginner and a trained athlete very different times; a 20 m sled push is a slow effort).
- **Rest time** — (number of sets − 1) × `rest_seconds_between_sets`, for each exercise in straight sets.
- **Superset rest** — for each superset, instead of its members' rest above: its members' exercise time is already counted above; add only one round rest after each round except the last, where round n is each member's n-th set (so rounds is the largest set count among its members) and a round's rest is the longest `rest_seconds_between_sets` among the members that have a set in that round. There is no rest between the exercises of a round.
- **Drop time** — each drop in a set's `drops` adds 10 seconds to change the weight on top of its exercise time above, and no rest. Drops are not sets in the rest and round counts: "number of sets" and "round n" count only the sets the drops continue.
- **Switch time** — (number of units − 1) × 60 seconds, where a unit is one exercise in straight sets or one superset.

Gymlab computes the length it shows the member the same way, except that it times a `distance` set at a typical pace for the activity. When time matters more than distance, prefer `distance_time` with a `target_duration_secs` where the exercise allows it.

`gymlab_get_exercise` gives an exercise's usual rest: the member's own (`userRestSeconds`; 0 means they use no rest timer) or the exercise's default (`defaultRestSeconds`). Start from it and adjust for the goal and the load: longer for heavy, low-rep strength sets; shorter for lighter hypertrophy, endurance and conditioning work. Always give a concrete `rest_seconds_between_sets` in seconds. If you leave it out, Gymlab uses the member's own rest, else the exercise's default, else 90 seconds, and your total must use that same value.

Add or cut exercises and sets until the total lands within 5 minutes of the target. As a sanity check: a 45-minute session usually holds 4-5 main exercises, 60 minutes 5-6, and 75-90 minutes 6-8.

## Supersets and drop sets

Supersets and drop sets fit the same training into less time. Neither builds more muscle or strength than straight sets with the same work, and both cost more: higher effort and slower recovery, most of all a drop set taken to failure. Use them when they make this member's training better, most often when time is short, not for their own sake; many good workouts and plans have neither.

A superset is 2 to 10 different exercises done back to back in rounds: one set of each member, then the rest, then the next round. Give every member of one superset the same short `superset_group` label ("A", "B", ...), unique within the workout, and list the members next to each other in the workout's exercises. Leave `superset_group` out for every exercise done in straight sets. A label on a single exercise, on exercises that are not next to each other, on more than 10 exercises or on the same exercise twice is ignored, and those exercises are saved as straight sets; the save's result shows each exercise's `superset_group` as saved.

Members keep their own sets, reps and loads, usually equal set counts. Their `rest_seconds_between_sets` is the rest after each round; there is no rest between the exercises of a round. Warm-up and cool-down exercises are never in a superset with main exercises. A circuit of stations done in rounds is one superset. Supersets shorten a session by about a third.

- Opposing muscles (a press with a row), biceps with triceps, or unrelated areas keep the volume of straight sets.
- Two exercises for the same muscle lower the loads and reps of both: deliberate local fatigue for an experienced member, not a time saver.
- A lift whose strength the member is building or tracking does best in straight sets with full rest when there is time; its accessories can still be paired.
- A pair holds two stations at once, which a busy gym may not allow.

A drop set is one set continued straight away, without rest, at lower weights: give the set its `drops`, written as the set types describe. It suits any exercise with a weight, best on the last set of an exercise late in the session, on machines, cables, dumbbells and isolation work, for hypertrophy or a short session. Under a heavy barbell squat, bench press or deadlift, form breaks down near failure, and lifts trained for speed or technique gain nothing from it. One or two exercises with a drop set give most of the time saving; more mostly add fatigue.

Weigh the member's experience: someone in their first months progresses well on straight sets and is still learning technique, so drop sets rarely help them, while simple machine or dumbbell supersets do when time is short. Peaking, taper and deload work exists to shed fatigue, so neither fits there, and a drop set's recovery cost matters when the same muscles train again the next day.

The member's wish wins. Read `supersetPreference` with `gymlab_get_preferences`: `avoid`, or a wish against supersets in what they told you or in their `text_preferences`, means no supersets at all; `prefer` means using them wherever they fit. Drop sets have no setting: a wish against them in what they told you or in their `text_preferences` means none, and a wish for them means using them wherever they fit. When you use either, say why in the workout's `description` in a few plain words, for example "Paired to fit your 40 minutes".

## Warm-up

Every muscle group must be warmed up before its working sets — either through dedicated warm-up exercises at the start of the workout or through warm-up sets on the exercise itself.

- With dedicated warm-up exercises present, heavy compounds may still take 1-2 warm-up sets. Later exercises hitting the same muscles need none, and light isolation work needs none.
- Without them, the session's first compound carries 2-3 warm-up sets, and later exercises on the same muscles need fewer or none.

Do not stack both: a muscle warmed up once is warmed up.

## Warm-up, working, back-off and drop sets

The save tools take no set type, so a set's purpose shows in its weight, its `target_rpe` and its place in the order; say which sets are warm-ups in the exercise's `notes` when it is not obvious. Drops are the one exception: they go in their set's `drops`.

- **Warm-up sets** — ramping toward working weight, RPE 4-6, never to failure.
- **Working sets** — the training stimulus, and what progressive overload is tracked on, RPE 7-9. Most sets are working sets.
- **Back-off sets** — reduced-intensity volume after heavy work, RPE 5-7, at 40-60% of the working weight.
- **Drops** — a continuation of the set before them, not sets of their own: straight after that set, with no rest, the member lowers the weight and keeps going close to failure. A drop set is that set plus 1 to 3 drops in its `drops`, each about 20-30% lighter than the part before, and counts as one set. A drop needs `target_reps` (`target_duration_secs` on `weighted_time`) and a concrete `target_weight_kg` like every set, and takes no `target_rpe`: the set's is its effort before the first drop. Only `weighted_reps` and `weighted_time` exercises take drops, never a warm-up set.

## Loads

Set every load with `references/loads.md`.

## Exercises they favor and exercises they avoid

The member's favourites and the exercises they avoid are in the interview's `exercise_preferences`, and in `favoriteExercises` and `avoidExercises` from `gymlab_get_preferences`; where they differ, the interview's answers win.

Favorites: feature them as primary movements and raise their frequency across the plan wherever they fit the workout's focus, the split, and recovery. They need not appear in every workout. A favorite is also the one good reason to keep an exercise that is not the most efficient choice — training the user will actually turn up for beats a marginally better movement they dread.

Exclusions are absolute. An exercise the user asked to avoid — or any movement their limitations rule out — must NEVER appear, even where it would have been the ideal choice. When you are unsure whether a movement suits their limitations, leave it out. Substitute a functionally similar movement instead, and if that weakens the session, say so in that workout's `description`. You may deliberately program work that strengthens a described weak point, provided the movement itself stays within their limitations.

Choose exercises and measure their sets with `references/exercises.md`.

## Names

Workout names are descriptive, distinguishable from one another, and at most 30 characters in English, spaces included; keep them as short in other languages. The plan's name is at most 35 characters in English.

Good: "Push Day", "Posterior Chain", "Upper Body Strength", "Lower - Quad Focus", "Full Body - Power Focus".

Wrong: "Day A", "Workout 1", "Week 1" — they describe nothing. "Full Body A" and "Full Body B" — indistinguishable to the user. "Full Body Workout With Upper Body Focus" — over 30 characters.

## Explaining the plan

After the plan is saved, explain it the way a personal trainer talks a client through the plan they wrote for them, in the member's language and in a few short sentences: the saved card already lists the workouts and their exercises. Cover four things, in this order:

1. **The goal.** Open with what the plan helps the user achieve: "This plan helps you…" or "The goal of this plan is to help you…". When the plan serves several goals, name each one and the trade-off you made between them.
2. **What the plan does.** Say how many workouts you created and, when you know their schedule, how often they do them each week; the structure the workouts follow (for example an upper-lower split or full-body sessions); how long a session takes; and the main kinds of training they combine (for example strength work balanced with cardio).
3. **Why these are the best choices for this user.** For each main decision (the number of workouts, the split, the session length, the kinds of exercise and their balance), explain why it is the best fit, tied to what you know about the user (goal, experience, age, schedule, favorite exercises).
4. **Their limitations.** When the user has movements to avoid, limited equipment, limited time or another constraint, name it plainly and explain how the exercise choices and the workout structure work around it while still moving the goal forward. Leave this part out when there are none, and never invent one.

Back your choices with research wherever a well-established finding applies. Facts and figures support a claim, so use accurate, relevant ones, explained in plain words and tied to what they mean for this user. For example, when it applies: "Research shows that lighter weights build about as much muscle as heavier ones when you end each working set with only a few reps left, so your dumbbells at home work well with the higher-rep sets I gave you." NEVER invent or guess a study, researcher, journal, year, percentage or any other figure; when you are unsure of a number or who found it, leave that detail out. When no well-established finding backs a choice, explain it from the user's facts and your coaching reasoning instead.

Everything you say must match the plan you show and save: its workouts' count, order, structure, exercises and session length. Refer to a limitation only as the user described it, do not guess at its cause, and promise no result or timeline.

Use the active voice, short sentences and everyday words over jargon; when a training term is the clearest word, explain it briefly the first time. Each workout's `description` carries the short version for that workout: what it concentrates on, and why it is in the plan and in this place.
