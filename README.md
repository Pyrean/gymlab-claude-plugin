# Gymlab for Claude

Gymlab is a strength training app for iPhone and Apple Watch. This plugin connects Claude to your
Gymlab account and adds six skills that carry the approach Gymlab's own coach takes, so that Claude
plans, builds, reviews, logs and adjusts your training the way the app does.

You need a Gymlab account, set up in the iPhone app. Setup guide:
https://www.gymlab.ai/ai-assistants

## Skills

- `getting-started`: when you have just connected Gymlab or ask what it can do. Claude reads where
  you stand (setup, plan, logged sessions) and answers with your next step and one question.
- `plan-training`: when you want a new training plan or program, want yours rebuilt, or bring a
  program of your own to save, typed, pasted or in a photo. Claude asks Gymlab's plan questions one
  at a time, reads your training before it designs, saves the plan only after you confirm it, and
  offers to make it the plan you follow.
- `build-workout`: when you want a single workout for today or another day. Claude asks what you
  want to train, or picks the focus from how recovered your muscles are, and fits the workout into
  the time you have.
- `review-progress`: when you ask how your training is going: progress on a lift over time,
  records, consistency, strain and recovery, or whether to take a lighter week.
- `log-workout`: when you report a finished session. Claude collects what you did set by set,
  confirms it with you and saves it once.
- `adjust-training`: when you want to change training you already have: swap an exercise for an
  alternative that fits your equipment or an injury, change sets, loads, order or supersets in a
  saved workout, rename a plan, change how many sessions a week it asks for, or choose the plan you
  follow. Claude shows every change and saves it only after you confirm.

## What the plugin sends and fetches

The skills are text instructions. The plugin contains no code, scripts, hooks or executables, and
sends or fetches nothing itself.

Every call goes through the Gymlab connector, the MCP server at https://app.gymlab.ai/mcp, which
this plugin names in `.mcp.json`. Claude calls it only after you sign in with your Gymlab account,
and only for your own account. Through it Claude can read your workout plans and workouts, logged
sessions, personal records, exercise statistics and strength estimates, muscle strain and recovery,
your streak, your exercise notes and your training preferences, which include your sex. The plan and
workout questions also read what you told Gymlab's coach about your equipment, injuries and
limitations, and which exercises you like or avoid, together with a summary of your logged
training. Claude can create plans and workouts, edit a workout's details and its exercises (swap,
remove, reorder, change their sets and supersets) and add exercises to it, rename a plan and change
its sessions per week, choose the plan you follow, delete plans and workouts that were created in a
conversation, log sessions, save exercise notes and update your training preferences; a plan saved after the plan questions can also update what Gymlab's coach
knows about your equipment and injuries where you corrected them. It cannot change or delete logged
sessions. If you connected Hevy to Gymlab, a session you log is also posted to your Hevy account as
a public workout, as a session logged in the app is.

Gymlab handles this data under its privacy policy: https://www.gymlab.ai/privacy-policy

## Install and connect

1. Install the plugin. It adds the Gymlab connector together with the skills. In Claude Code, run
   this in a session:

   ```text
   /plugin install gymlab --marketplace Pyrean/gymlab-claude-plugin
   ```

   or, from your shell, `claude plugin install gymlab --marketplace Pyrean/gymlab-claude-plugin`.

2. The first time Claude uses a Gymlab tool, sign in with your Gymlab account and allow access. In
   Claude Code, run `/mcp` and choose `gymlab` to sign in.
3. Ask, for example, "Help me create a new workout plan", "What should I train today?" or "How is
   my training going?"

To disconnect, remove the plugin or its connector in Claude.

## Support

https://www.gymlab.ai/support

## License

Proprietary, see `LICENSE`. Terms of Service: https://www.gymlab.ai/terms-of-service
