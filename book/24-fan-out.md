# Fan-out

_IV · Flows · Chapter 24_

For a large migration, the winning move is not a bigger context window — it is many small invocations, each with one file and one job. This is where `claude -p` stops being a convenience and becomes the point.

1. **Generate the task list — in a session, not a script**

   Let Claude produce the list, because it knows what “needs migrating” means better than a glob does.

   ```
   list every Python file under src/ that still imports the legacy
   `db.connect` helper. write the paths, one per line, to files.txt.
   do not modify anything.
   ```

2. **Write the per-file prompt and test it on three files by hand**

   The prompt is the product here. Refine it against real failures before it runs two thousand times.

3. **Loop, with permissions scoped tightly**

   ```
   for file in $(cat files.txt); do
     claude -p "Migrate $file from db.connect to the new pool API.
                Run: pytest \${file/src/tests}. Return OK or FAIL." \
       --allowedTools "Read,Edit,Bash(pytest *)" \
       --max-turns 12 \
       --output-format text
   done | tee migration.log
   ```

   `--allowedTools` is not optional here. An unattended loop with unrestricted Bash is a loop with unrestricted Bash, two thousand times.

4. **Triage the log, then re-run only the failures**

   ```
   grep FAIL migration.log | cut -d: -f1 > retry.txt
   ```

   The second pass usually needs a different prompt — the easy cases are gone and what remains shares a structure worth naming explicitly.

5. **Review the aggregate, not each diff**

   ```
   git diff --stat
   git diff -- src/legacy/ | head -400
   ```

   Then a reviewer subagent on a sample: “review 10 of these migrations at random for correctness and consistency.”

## Piping Claude into a pipeline

```
# structured output for a script to consume
claude -p "List all API endpoints as JSON" --output-format json | jq '.result'

# schema-validated output
claude -p "Extract the config keys" --json-schema ./keys.schema.json

# feed data in
cat error.log | claude -p "group these errors by root cause"
```

Use `--verbose` while developing the script and turn it off in production. `stream-json` emits one JSON object per line starting with an init event, which is what you want for progress reporting.

## The built-in alternative

`/batch <instruction>` decomposes a large change into parallel work units across git worktrees, inside one session. It costs less setup than a shell loop and gives up some control. Try it first for anything under a few dozen units; write the loop when you need the log, the retry pass, and the budget cap.

> **Key — Why fan-out beats one big session**
>
> Each invocation starts with a clean window, so file #1,847 gets the same quality of attention as file #3. A single session attempting the same work accumulates every previous file’s content, hits the lost-in-the-middle problem around the middle of the list, and degrades exactly where you stopped watching.
