# Workout design guide

This is how Gymlab's own workout generator designs a single workout, rewritten for the Gymlab tools. Work from the interview's answers, `recovery` and what the member told you. Where something is unknown, use science-backed coaching judgement, and never invent facts about the member to fill a gap. In the passages taken from Gymlab's prompts, "the user" is the member.

## Match the member's ability

For skill exercises (muscle-ups, handstand push-ups, pistol squats, L-sits, front levers and the like):

- When the member can already do the skill, program the skill at the right volume for their goal.
- When they cannot do it yet but want to learn it, program the right progression exercise, starting where their ability is.

Progressions, from easiest to the skill:

- Handstand Push-Up: Pike Push-Up → Elevated Pike Push-Up → Wall Handstand Hold → Wall Handstand Push-Up → Freestanding Handstand Push-Up
- Muscle-Up: Pull-Up + Straight Bar Dip → High Pull-Up → Jumping Muscle-Up → Band-Assisted Muscle-Up → Muscle-Up
- Pistol Squat: Assisted Pistol Squat → Box Pistol Squat → Eccentric Pistol Squat → Pistol Squat
- Pull-Up: Dead Hang → Scapular Pull-Up → Band-Assisted Pull-Up → Negative Pull-Up → Pull-Up
- L-Sit: Tucked L-Sit → One Leg Extended L-Sit → L-Sit
- Front Lever: Dead Hang → Tucked Front Lever Hold → Advanced Tucked Front Lever → Straddle Front Lever → Front Lever

Never assume the member can do an advanced skill just because they want to train it. A member who wants to work toward handstand push-ups but cannot hold a handstand starts with the Pike Push-Up or the Elevated Pike Push-Up, not with handstand attempts.

## Intensity for the goal

Program the intensity that serves the member's goal:

- **Muscle building (hypertrophy):** moderate intensity, higher volume: RPE 6-8, typically 8-12 reps, 3-4 sets.
- **Strength:** higher intensity, lower volume: RPE 7-9, typically 3-6 reps, 4-5 sets, longer rest.
- **Endurance and conditioning:** lower intensity, higher reps: RPE 5-7, typically 15-20 or more reps, or timed sets.
- **Fat loss and general fitness:** moderate intensity with a metabolic emphasis: RPE 6-8, circuit-style, shorter rest.
- **Skill acquisition:** lower intensity, quality reps: RPE 5-7, practice volume without fatigue.

When the goal is unclear, use moderate intensity (RPE 6-7) with sustainable volume. Prescribe weights the member can complete with good form, not near-maximal loads.

## Choosing and ordering exercises

Choose exercises for the focus and the member's goals, their abilities and fitness level, the equipment they have today, a sound balance between muscle groups, and progressive overload.

Give every exercise complete targets (sets with reps, time or distance, and loads), the rest between sets and a target RPE, and order the exercises from fresh to fatigued.

Match exercise difficulty to the user's experience level. Open with compound movements and work toward isolation. Account for fatigue accumulating through the session.

Program from established strength and conditioning scientific evidence rather than novelty. Favor movements with a well-documented effect on the quality you are training and a good stimulus-to-fatigue ratio — loadable compounds that train a lot of muscle per unit of time and recover predictably — and add isolation work only where a specific muscle or weak point needs it.

Session time is the scarce resource, so every exercise you include displaces another. Cut anything inefficient, redundant with a movement already in the workout, or present only for variety. The user's favorites are the one exception (see below).

Weigh each exercise's benefit against the risk it carries, and take fatigue into account: some movements get materially riskier the deeper into a session they sit.

## Exercises they favor and exercises they avoid

The member's favourites and the exercises they avoid are in the interview's `exercise_preferences`, and in `favoriteExercises` and `avoidExercises` from `gymlab_get_preferences`; where they differ, the interview's answers win. Feature a favourite as a main movement when it fits today's focus and recovery: training the member will actually turn up for beats a marginally better movement they dread.

Exclusions are absolute. An exercise the user asked to avoid — or one that could aggravate an injury they described — must NEVER appear, even where it would have been the ideal choice. Where you are unsure whether a movement could aggravate an injury, leave it out. Substitute a functionally similar movement instead, and if that weakens the session, say so in that workout's `description`. You may deliberately program work that strengthens a described weak point or supports recovery, provided the movement itself cannot aggravate the injury.

## Session length

The target is the interview's `workout_length`.

Compute the workout's total time as the sum of:

- **Exercise time** — for time-based sets (`bodyweight_time`, `weighted_time`, `distance_time`, `time_only`), the sum of each set's `target_duration_secs`. For rep-based sets (`bodyweight_reps`, `weighted_reps`), the sum of `target_reps` × 3 seconds across sets. For sets of a `distance` exercise, your own estimate of how long this member needs for that distance on that exercise, judged from their age, training level and recent performance — there is no fixed pace (a 500 m row or a 5 km run takes a beginner and a trained athlete very different times; a 20 m sled push is a slow effort).
- **Rest time** — (number of sets − 1) × `rest_seconds_between_sets`, for each exercise in straight sets.
- **Superset rest** — for each superset, instead of its members' rest above: its members' exercise time is already counted above; add only one round rest after each round except the last, where round n is each member's n-th set (so rounds is the largest set count among its members) and a round's rest is the longest `rest_seconds_between_sets` among the members that have a set in that round. There is no rest between the exercises of a round.
- **Switch time** — (number of units − 1) × 60 seconds, where a unit is one exercise in straight sets or one superset.

Gymlab computes the length it shows the member the same way, except that it times a `distance` set at a typical pace for the activity. When time matters more than distance, prefer `distance_time` with a `target_duration_secs` where the exercise allows it.

`gymlab_get_exercise` gives an exercise's usual rest: the member's own (`userRestSeconds`; 0 means they use no rest timer) or the exercise's default (`defaultRestSeconds`). Start from it and adjust for the goal and the load: longer for heavy, low-rep strength sets; shorter for lighter hypertrophy, endurance and conditioning work. Always give a concrete `rest_seconds_between_sets` in seconds. If you leave it out, Gymlab uses the member's own rest, else the exercise's default, else 90 seconds, and your total must use that same value.

Warm-up, main and cool-down exercises all count.

Add or cut exercises and sets until the total lands within 5 minutes of the target. As a sanity check: a 45-minute session usually holds 4-5 main exercises, 60 minutes 5-6, and 75-90 minutes 6-8.

## Warm-up

Every muscle group the workout trains is warmed up before its working sets, either by warm-up exercises at the start (light cardio, dynamic mobility) or by warm-up sets on the exercise itself.

- With warm-up exercises at the start, the general warm-up is done. Heavy compound lifts (squat, bench press, deadlift) may still take 1-2 warm-up sets; lighter isolation exercises need none.
- Without them, the first compound exercise carries 2-3 warm-up sets, and later exercises for the same muscles need fewer or none.

Do not over-warm: no extensive warm-up sets for muscles the warm-up exercises already prepared, and none on later exercises for the same muscles.

The save tools take no set type, so a set's purpose shows in its weight, its `target_rpe` and its place in the order; say which sets are warm-ups in the exercise's `notes` when it is not obvious.

- **Warm-up sets:** lighter sets ramping up to the working weight, RPE 4-6, never to failure.
- **Working sets:** the main training stimulus at the target intensity, RPE 7-9; progressive overload is tracked on them. Most sets are working sets.
- **Back-off sets:** reduced-intensity sets after heavy work, RPE 5-7, at 40-60% of the working weight.

For example, a squat for an intermediate member: 40 kg × 10 at RPE 5 and 60 kg × 8 at RPE 6 to warm up, then 80 kg × 6 at RPE 8 for the working sets.

## Supersets

A superset is 2 to 10 different exercises done back to back in rounds: one set of each member, then the rest, then the next round. Give every member of one superset the same short `superset_group` label ("A", "B", ...), unique within the workout, and list the members next to each other in the workout's exercises. Leave `superset_group` out for every exercise done in straight sets. A label on a single exercise, on exercises that are not next to each other, on more than 10 exercises or on the same exercise twice is ignored, and those exercises are saved as straight sets; the save's result shows each exercise's `superset_group` as saved.

Choose a superset when it is the better way to train:

- The member is short on time, or asks for a short, quick or all-out session.
- Conditioning, metabolic or finisher work.
- Training naturally done in rounds: parkour, calisthenics circuits, kettlebell and bodyweight conditioning. A circuit of several stations is one superset.
- Pairs whose fatigue costs little: opposing muscles (a press with a row), arms (biceps with triceps), unrelated areas (an upper-body exercise with core).

Keep straight sets when:

- The exercise is a lift whose strength the member is building or tracking and there is time for straight sets. A superset lowers its loads.
- Both exercises are heavy barbell lifts, or they need two stations that cannot both be held in a busy gym, such as two machines in different areas.
- Both exercises train the same muscle, unless the goal is intensity and the member is experienced.
- The member is still learning the exercise's technique.

The member's wish wins. Read `supersetPreference` with `gymlab_get_preferences`: `avoid`, or a wish against supersets in what they told you, means no supersets at all; `prefer` means using them wherever these rules allow. Warm-up and cool-down exercises are never in a superset with main exercises.

Members keep their own sets, reps and loads. Give them equal set counts unless one member deliberately needs fewer. Their `rest_seconds_between_sets` is the rest after each round; there is no rest between the exercises of a round. When you use a superset, say why in the workout's `description` in a few plain words, for example "Paired to fit your 40 minutes".

## Recovery

`recovery` from `gymlab_workout_interview` gives each muscle a `recommendation`.

Program around each muscle's recommendation:

- `reduce_volume`: about a third fewer direct working sets for that muscle, RPE 7 or lower (2-4 reps in reserve), no sets to failure.
- `avoid_direct_work`: no heavy or high-volume direct work for that muscle, light technique work at most; move the emphasis to the fresh muscles while keeping the requested focus recognisable.
- `rest_or_deload`: no direct work for that muscle; substitute.
- `normal_training` and `monitor`: program as usual.

State the adjustment you made in one sentence of the workout's `description`. An explicit wish of the member that overrides this, such as a note to keep the focus, wins.

With `low_confidence`, the numbers rest on little logged training: let them temper the volume at most. With `unavailable`, program normally.

## Recent sessions

Use the member's latest sessions (`gymlab_list_workout_sessions`) for variety and continuity: when a recent session trained the same muscles, build on it rather than repeating it exercise for exercise, unless the member asks for the same exercises. This is context only: the member's request and the recovery rules above win over it.

The interview's answers (focus, length, equipment, limitations) win over any saved preference that says otherwise.

## Loads

- Use the member's numbers in Gymlab, and the working weights and fitness level they told you, to prescribe loads (`references/loads.md`).
- When they mention a one-rep max, prescribe working sets well below it.
- When they mention working weights ("I train with 80 kg on bench"), prescribe similar or slightly lighter weights for sustainable training volume.
- For bodyweight exercises, adjust difficulty through progressions rather than added weight.
- Always put form and sustainable volume before maximal loads, and match the loads to the goal (see "Intensity for the goal").

## Name and description

Name the workout for what it trains, descriptive and at most 30 characters in English ("Lower Body Strength", "Push - Chest Focus", "Full Body Conditioning"), never "Workout 1". Its `description` says in a few plain sentences what the workout is for, how it fits today's recovery, and why any superset is there.
