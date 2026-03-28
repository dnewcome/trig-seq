Run a devlog snapshot for the Trig Seq project.

Steps:
1. Run `bash scripts/devsnap.sh` passing any note the user provided as the first argument. If the user gave a note or philosophical musing, pass it with `--note "<their text>"`. On macOS the `--window` flag is not yet supported, so omit it — the script will use Chrome headless against the local server. If no server is running, pass `--no-screenshot` and note that the user should open the app and re-run if they want a screenshot.
2. Read the generated markdown file (it will be the most recently modified file in devlog/).
3. Read `git log --oneline -8` and `git diff HEAD~1 HEAD --stat` for context.
4. Rewrite the "What happened" section of the entry with a real narrative paragraph — written in first person from the builder's perspective, genuine tone, no clickbait. Focus on: what was hard, what was surprising, what it unlocks. 2-4 sentences.
5. Write a tweet draft (≤280 chars) in the "Tweet draft" section. Tone: narrative, genuine enthusiasm, no buzzwords or hype tricks. Should feel like a builder talking to other builders. Include what it is, what changed, and why it's interesting. Can include a short URL placeholder like [link].
6. Save the updated file.
7. Show the user the final tweet draft and the path to the entry file.
8. Ask if they want to publish it to dnuke.com now. If yes, run `bash scripts/devpublish.sh <entry file>`.

If the user's note sounds philosophical or decision-oriented (e.g. "I'm trying to decide...", "thinking about..."), treat the entry as a design journal post instead — focus on the question being wrestled with, the tradeoffs, and what's drawing them in different directions. Still write a tweet that invites conversation rather than announces a ship.
