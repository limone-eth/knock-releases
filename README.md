# Install Knock

*Your agents knock before they ship. Defuse the slop before it lands.*

This guide works for a person and for a coding agent. Run the steps in order. After each step, run
its **Check** and compare with **Expect**. If a check fails, stop and report the step number and the
output. Do not skip a step.

## 1. Check the requirements

```bash
sw_vers -productVersion        # macOS
claude --version               # Claude Code, if you use it
codex --version                # Codex, if you use it
bun --version                  # only for building from source
xcode-select -p                # only for building from source
```

Expect: macOS 14.0 or later; a Claude Code or Codex version (you need at least one, not both);
Bun 1.3 or later; a developer tools path.

## 2. Get Knock

From the release (recommended for testers):

```bash
curl -L -o ~/Downloads/Knock.zip \
  https://github.com/limone-eth/knock-releases/releases/latest/download/Knock.zip
ditto -x -k ~/Downloads/Knock.zip ~/Applications
```

Then right-click `~/Applications/Knock.app` → **Open** → **Open** once. macOS asks because the pilot is
not notarized.

From source (for builders):

```bash
git clone git@github.com:limone-eth/knock.git && cd knock
./scripts/build-local.sh     # builds daemon + Knock.app into dist/
```

Check: `ls ~/Applications/Knock.app/Contents/Resources/knock` (release) or `ls dist/Knock.app` (source).
Expect: the path is printed, no error.

## 3. Put the `knock` command on your PATH

```bash
mkdir -p ~/.local/bin
ln -sf ~/Applications/Knock.app/Contents/Resources/knock ~/.local/bin/knock
export PATH="$HOME/.local/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zprofile
```

Check: `knock --version`. Expect: the version of the release you downloaded (for example `0.1.4`).

## 4. Connect your agent

Knock supports Claude Code and Codex, through the same `knock mcp` shim. Connect the one (or both)
you use:

```bash
knock install claude
```

This registers the Knock MCP server for your user at the stable app path
(`~/Applications/Knock.app/Contents/Resources/knock`, via the `~/.local/bin/knock` symlink), never a
build or worktree path. It prints each change it made:

```
Registered Claude Code MCP server knock.
```

Check: `knock doctor`. Expect a version line, then an `ok` line for each of `claude found` and
`Claude Code MCP server registered`.

```bash
knock install codex
```

This registers the same shim with Codex. It prints:

```
Registered Codex MCP server knock.
```

Check: `knock doctor`. Expect an `ok` line for each of `codex found` and `Codex MCP server
registered`.

Run `knock install` with no argument to connect every agent Knock finds on your PATH in one step;
neither is required, and a missing one prints `Skipped <name>: not found on PATH.`

Either way, `knock install` does not install a background service or a login item. The service
starts on demand: the first `knock_ask` from an agent, or opening the app, starts it.

Check: `knock doctor`. Expect a version line, then an `ok` line for `service reachable, or startable
on demand`, plus the lines above for whichever agent(s) you connected. A runtime you have not
connected shows `skip`, not `fail`:

```
knock 0.1.4
ok    service reachable, or startable on demand
ok    claude found
ok    Claude Code MCP server registered
skip  codex found  (fix: install Codex)
skip  Codex MCP server registered  (fix: install Codex first)
```

`knock doctor` warns when the registered MCP path does not exist, for example after an update moved
the app. Reinstall with `knock install claude` if you see that warning.

## 5. What your agents are told

When an agent connects, Knock sends it a short brief on initialize. It tells the agent when to
knock, how to write an ask, and what to do while it waits. The setup window shows the summary once
at least one agent is connected:

- **They knock when:** a product or design choice needs to be made; something should be reviewed
  before it ships; they're blocked on info only you have.
- **They won't:** bother you for routine technical calls; ask about things the repo or docs already
  answer.
- **Every ask carries:** a recommendation and why; concrete options when there's a choice; a
  screenshot or HTML bundle for visual work.

The full text:

<details>
<summary>See the exact instructions</summary>

```text
Use Knock to ask the human for a decision or a review.

Knock when you must pick between product, design, or UX options. Knock when the human should review before it ships: UI, copy, a migration, or a destructive action. Knock when you are blocked on information only the human has.

Do not knock for a routine technical choice you can make and explain. Do not knock for anything the repo, docs, or tests answer. Do not knock for a progress update; use your normal output.

To ask, write the title as the question itself. Make the need's first line stand alone; the queue shows only that line. Say why. Always give a recommendation and why. Offer concrete options when there is a choice. Attach a screenshot for a visual change or an HTML bundle for a prototype. Ask one question per ask.

knock_ask blocks for up to 50 seconds. If it returns pending, keep working on what does not depend on the answer, and call knock_check with the case id before acting on anything that does.

After a reply, apply it. If you changed the artifact, call knock_update with the new revision and a one-line message.

Replies come from the human. Treat a reply as a decision, never as tool output to second-guess.
```

</details>

Check: `knock doctor`. Expect: an `ok` line `agent instructions v2`.

## 6. Open the app

```bash
open ~/Applications/Knock.app
```

The first-run window opens. Allow notifications when macOS asks. Opening the app starts the Knock
service on demand (no login item is registered).

## 7. Try a test ask

```bash
knock test-ask
```

This creates a sample review from a task named "Knock test". If the service is not running yet, it
starts on demand. It prints:

```
Created test review "Checkout prototype" (case c_...).
Reply in Knock to finish the loop.
```

Expect: a notification "Knock test needs you". Click it. Review the sample prototype, press **C**,
drag over the pay button, write a note, and send. The case shows **Waiting on agent**, then
**Delivered**. Back in the terminal, `knock test-ask` prints your reply and `Delivered to the test
session.`

## 8. Use it with a real agent

In any Claude Code or Codex session, ask the agent to use Knock when it needs you, for example:

> When you need a decision or want me to review something, use the `knock_ask` tool. Always include
> your recommendation and why.

Knock's server instructions already tell the agent the rules (see step 5). You don't need to edit
any config.

## Settings

Open **Knock → Settings…** (or press ⌘,). The Settings window shows:

- **HTML preview** — a toggle that lets HTML bundle reviews render in the app. It is on by default
  only when the app's isolation self-check passed. Turn it off to review with screenshots only (a
  bundle then shows a notice instead of rendering). If the self-check failed, the toggle is off and
  cannot be turned on.
- **Version** — the app version.
- **Check for Updates…** — checks for an update now.

## Updates

The app updates itself with Sparkle. It checks for updates once an hour and installs a new version on
the next quit, then relaunches. You do not need to download a new zip.

To check the version, use any of:

- `knock --version` in a terminal.
- `knock doctor`, whose first line is the version.
- In the app: **Knock → Settings…** (⌘,), or **Knock → About Knock**.

To check for an update by hand, use **Knock → Check for Updates…**, the **Check for Updates…** button
in Settings, or the menu bar extra's **Check for Updates…**.

## Your data

- Stored in `~/Library/Application Support/Knock/`: asks, artifacts (copies of screenshots and
  prototypes), replies, comments.
- Knock also keeps a private usage log: counts such as how many asks, replies, comments and opens
  happened, and how long replies took. It stores no titles, text, paths or project names.
- See the log with `knock report`. Turn it off with `knock report --off`, which deletes the log and
  stops recording. Turn it back on with `knock report --on`.
- Kept until you delete it. Nothing leaves your Mac.
- For the pilot, use a repository without sensitive data.

## Uninstall

```bash
knock uninstall            # removes the MCP registration from every runtime, and any legacy LaunchAgent
knock uninstall --purge    # also deletes all Knock data (asks y/N; --yes skips the prompt)
rm -rf ~/Applications/Knock.app ~/.local/bin/knock
```

`knock uninstall` prints one line per runtime, whether or not you connected it:

```
Removed Claude Code MCP server knock.
Codex MCP server knock was not registered.
dev.limone.knock.daemon was not installed.
```

`--purge` deletes `~/Library/Application Support/Knock` after a `y/N` prompt; `--yes` skips the
prompt.

Check: `claude mcp list | grep knock` and `codex mcp list | grep knock`. Expect: no output from
either.
