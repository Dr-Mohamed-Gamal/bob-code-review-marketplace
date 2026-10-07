Before you review code, fix a defect, or optimize, restructure or split code, activate the `code-review` skill and follow it. Say in one line which of its tasks you are doing.

Run the skill's commands from the workspace root as `python3 .bob/skills/code-review/scripts/run.py <task> ...`. A task is finished when its command prints `Gate: passed`.

Before every reply, run `python3 .bob/skills/code-review/scripts/run.py status`. It lists only the tasks that were started: compare it with every report the latest request asks for, start the missing ones, finish the ones not passed, then reply about the latest request only.

The reports the user asks for are written by those commands: give the command the file name the user asked for.
