# Choosing and prescribing exercises

This is how Gymlab's own generators choose exercises and measure their sets, rewritten for the Gymlab tools. In the passages taken from Gymlab's prompts, "the user" is the member.

## Finding exercises

Choose every exercise, including warm-ups and cool-downs, from Gymlab's library: find it with `gymlab_search_exercises` and give its `exercise_id`. The id is what links the workout to the exercise the member logs and to their history, so never invent one.

Prefer exercises the member already does whenever they fit the workout's goals: `gymlab_list_exercise_stats` lists them, and `gymlab_search_exercises` finds the member's own exercises as well as the catalog's.

Create a new exercise only when no library exercise can serve the movement. This includes variations: when the library lists the variation you want (a "Diamond Push-Up" beside "Push-Up"), choose it by its id.

For each exercise, choose `exercise_type` only from its `allowedExerciseTypes`, and prescribe sets using that type's measurements. These are the tracking options Gymlab supports for the exercise, including during warm-ups and cool-downs. For a duration-only warm-up or cool-down, select an exercise that lists `time_only` or `bodyweight_time`. Never assume an unlisted variation supports a timed type. If no suitable timed exercise is listed, prescribe a rep-based warm-up or cool-down instead; do not invent a distance or rename an existing exercise to get around its allowed types.

Search by the exercise's plain English name ("romanian deadlift", "lat pulldown"), whatever language you speak with the member. Leave `exercise_type` out when the exercise's default (`exerciseType`) is the one you want.

Each search result also carries the exercise's `muscleGroups` and `equipment`, and `currentEstimatedOneRepMax`, the member's current estimate on it (null when Gymlab has none). So the search itself shows whether an exercise fits the member's equipment and what they lift on it; read `gymlab_get_exercise` when you need more, such as their own rest or note on it.

## An exercise that does not suit the member

When the exercise you would choose needs equipment the member lacks, could aggravate an injury they described, or is one they avoid, find a substitute with `gymlab_find_exercise_alternatives`: pass its `exercise_id`, the equipment slugs they have as `available_equipment`, and the muscles to keep out as `exclude_muscles`. It lists catalog exercises that work the same main muscle, with their equipment and `allowedExerciseTypes`. Its muscle filter follows how the catalog tags exercises and does not judge whether a movement is safe for an injury: the exclusion rules still decide.

## Measurement types

Every exercise is measured in one way, chosen by `exercise_type`. The type is a property of the exercise, not of the workout: for a library exercise, pick a type from its `allowedExerciseTypes`. If the type you want is not listed, you have the wrong exercise: pick or create the variant whose type fits, following these rules.

- A set prescribes only the targets you intend, and every other target is left out. Under `distance_time`, a `target_duration_secs` without a distance is exactly how to program "walk, run or row for N minutes". Never switch to a type the exercise does not list because a target goes unprescribed.
- A differently-loaded movement is a different exercise with its own name: to load a bodyweight movement, use its loaded variant ("Weighted Pull-Up", "Machine Standing Calf Raise"), never a bodyweight exercise with a weighted type.
- Continuous rhythmic conditioning (jump rope, shadowboxing, rope turns, battle ropes) is prescribed by duration, never by counts: nobody counts hundreds of repetitions mid-bout. Distinct counted skill variants ("Double-Under") stay rep-counted.
- Only a genuinely new exercise takes its type from you: choose the type a coach would record it in, applying the rules above.

What each type needs on every set:

- `time_only`, `bodyweight_time`, `weighted_time` and `distance_time`: `target_duration_secs`. The workout's length is computed from it.
- `bodyweight_reps` and `weighted_reps`: `target_reps`.
- `distance_time`: `target_distance_km` is optional. Give it only when the distance itself is a target, and leave it out for duration-only work.
- `distance`: `target_distance_km`.
- `weighted_reps` and `weighted_time`: `target_weight_kg` on at least one set.
- An exercise performed without external load uses a type without weight, never a weighted type with no weight on any set.

Leave out every target the type does not measure: the save tools refuse one, for example reps on a distance exercise. `target_rpe` goes on any set.

## Notes and instructions

An exercise's `instructions` and `notes` are for this workout only: tempo, pauses, range of motion, the side to start on, which sets are warm-ups. Leave them out when there is nothing specific to say. Do not explain how to perform the exercise: Gymlab shows each exercise's own guide.

## An exercise that is not in the library

Give it without `exercise_id`, with `exercise_name`, `exercise_type`, `exercise_description` (what it is and the essential technique, in a few sentences), `muscle_groups` and `equipment` (an empty list for none), and leave `create_new_exercise` out. A name that is in the library uses that exercise. A name close to library exercises is refused with a list of them and their ids: when one is the same movement, send its `exercise_id` instead; only when none is, send the exercise again with `create_new_exercise` true, and it is saved as the member's own exercise.

Use the standard, globally recognized name for an exercise that is not in the catalog. Variation names are part of the standard name and belong in it: "Archer Pull-Up", "Diamond Push-Up", "Cossack Squat".

Never put these in an exercise name:

- Parenthetical qualifiers: (Easy Pace), (Dynamic), (Slow), (Fast), (Controlled), (Explosive)
- Difficulty indicators: Easy, Hard, Difficult, Simple, Basic
- Training qualifiers: Attempts, Attempt, Practice, Session, Work, Training, Drills, Drill, Progression
- Skill levels: Beginner, Intermediate, Advanced
- Modifications in parentheses: (Assisted), [Modified], (Easy), (Attempt), (Full), (Partial)
- Distances: 5km, 1 mile, 5k, 400m

Tempo and pace belong in `instructions` or `notes`; distance belongs in the set's `target_distance_km`. So: "Rowing Machine", not "Rowing Machine (Easy Pace)". "Run", not "5k Run". "Front Lever", not "Front Lever Progression". "Muscle-Up", not "Muscle-Up Practice".

Name a new exercise in English, whatever language you speak with the member, as Gymlab's catalog does.
