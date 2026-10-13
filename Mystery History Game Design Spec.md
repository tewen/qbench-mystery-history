# Mystery History: Game Design Spec

Oct 12, 2026 · @Trevor Ewen

## Overview

Mystery History is an async guessing game for the QBench onsite: players read anonymous stories colleagues wrote about their pasts and guess who wrote each one, with Millionaire-style lifelines to help. Results are revealed to the whole team at once, on screen at the onsite.

- **Audience:** QBench employees, mostly on laptops, some on phones. Anyone who submitted a story is also a player.
- **Format:** fully async. Each person plays at their own pace over days; nobody needs to be online at the same time.
- **Goals:** help people learn surprising things about each other, create a fun shared reveal moment, and reward people who know their colleagues.
- **Session length:** about 1 minute per story. A player can stop and resume anytime; progress is saved.

## Core loop

Each round is one story with ten name options; the player picks one and locks it in.

1. Player opens the app and sees the next unanswered story ("Case 3 of 24") with a progress ladder of all cases.
2. The story text is shown in full, with no author information.
3. Ten name options appear, labeled A to J, each with the person's name and avatar.
4. The player may use any remaining lifeline before answering.
5. The player selects a name, then confirms with "Lock it in" (two steps, like "Is that your final answer?").
6. A short suspense beat (about 1.5 seconds, "Final answer…") plays before the result.
7. Feedback shows, depending on the host setting (see Host controls), then "Next case".

**Question order.** Each player gets their own shuffled order, so early players don't all hit the same stories first. The order must be stable across reloads (seed it from player ID + story ID). Stories added later slot into the order.

**Answer options.** The correct author plus nine distractors drawn from other story authors. Rules:

- A player never gets their own story.
- A player never appears as a distractor in their own options (it would be a free elimination).
- Options for a story should be stable for that player across reloads, so a refresh can't be used to reroll.
- Option position (A to J) is shuffled per story so the answer isn't always in the same slot.
- If fewer than ten authors exist, show as many as are available.

**Already-picked names.** Each person wrote exactly one story, so picking the same name twice guarantees at least one wrong answer. Any option the player has already chosen for another story shows a marker, such as "You picked them for case 4" with a small badge. The option stays selectable: picking it again is allowed, it's just a visible risk. The marker shows the player's own past picks only, never whether those picks were right.

**Answers are final.** Once locked in, an answer can't be changed.

## Scoring

A correct guess is worth 100 points; a wrong guess is worth 0, except under Double down.

| Outcome | Points |
| --- | --- |
| Correct | +100 |
| Wrong | 0 |
| Correct with Double down | +200 |
| Wrong with Double down | −100 |

- Using 50:50 or 20:80 costs no points. Their cost is scarcity: a limited number of uses per game.
- **Tiebreakers**, in order: more correct answers, then fewer lifelines used.
- Compute scores from stored answers at display time rather than storing a running total. That way, if the host fixes a story's author, scores correct themselves.

## Lifelines

Each player gets eight lifeline uses per game across three types, usable on any story before locking in.

| Lifeline | Uses per game | What it does |
| --- | --- | --- |
| 50:50 | 5 | Removes 5 wrong names, leaving 5 options |
| 20:80 | 2 | Removes 8 wrong names, leaving 2 options |
| Double down | 1 | This story is worth +200 if right, −100 if wrong |

**Behavior rules**

- A use is spent the moment it's applied, and that's saved immediately. Reloading the page must not refund it or reroll its result.
- At most one 50:50 and one 20:80 per story. They can be combined: 20:80 after 50:50 trims the remaining 5 options down to 2.
- Removals are deterministic per player and story. Rank the wrong names once with a seeded hash and always remove from the top of that ranking, so 20:80 after 50:50 keeps a subset of what 50:50 left.
- Removed names stay visible but greyed out and unselectable, so the layout doesn't jump.
- With fewer than 10 options (small pool of authors), 50:50 removes half the wrong names, rounded down, and 20:80 leaves 2 options. Neither is available once 2 or fewer options remain.
- The lifeline bar shows each type with its remaining count ("50:50 ×3"). Types with no uses left appear crossed out; one applied to the current story appears highlighted.
- Already-picked markers still show on options that survive a removal.

## Stories

Each person has at most one story, keyed to them; most stories are already collected and will be imported by the host.

- **Bulk import (host):** paste one story per line as `Full name | work email | story text`. The work email is the key, so stories can be loaded before their authors ever sign in. The app shows a preview of parsed rows and flags emails outside the QBench domain or duplicated in the batch, before saving. Re-importing an email replaces that story.
- **Single add or edit (host):** enter name, work email and story, then save.
- **Self-service (players):** a "My story" screen where any signed-in player can write or edit their own story, keyed to their Google email. Prompt copy: "A few sentences about something from your past most coworkers wouldn't guess. Leave out your name and anything that gives you away instantly."
- **Delete (host):** two-step confirm. Deleting a story removes it from play; answers already given to it no longer count toward scores.
- **Text:** plain text, line breaks preserved, no length cap enforced but aim for under about 600 characters so it reads on a phone.

## Host controls

The host runs the game with three switches; only hosts (admins) see the Host screen.

| Setting | Default | Effect |
| --- | --- | --- |
| Accept guesses | On | Off freezes the game: players can't answer, and see "Guessing is closed" |
| Show answers after each guess | On | On: players see "Correct!" or "It was X" and a running score. Off: they see "Locked in" only, and scores stay hidden until the reveal |
| Reveal results to everyone | Off | Opens the Results screen to all players |

**Typical run of the game**

1. Import stories, then announce the link. Accept guesses on, reveal off.
2. Players play over several days. The Host screen shows how many stories exist and how many people have played.
3. Shortly before the onsite session, turn Accept guesses off.
4. At the onsite, put the Results screen on the big screen and turn Reveal on.

Recommendation: leave "Show answers after each guess" on for fun, unless the team is likely to spoil answers in Slack; then turn it off for a bigger reveal. Hosts can preview Results at any time, with a banner saying only hosts can see it.

## Results and reveal

The Results screen is built to be projected at the onsite, with three parts top to bottom.

1. **Awards** (cards): Top detective (highest score), Toughest story to crack (lowest % correct, minimum 1 guess), Dead giveaway (highest % correct).
2. **Leaderboard:** rank, avatar, name, points, "X of Y right", lifelines used. Tied players share a rank. Players with zero answers are left out.
3. **Who wrote what:** every story, sorted hardest first, showing the author's name and avatar, the story text, "X of Y guessed right (Z%)", and "Most often mistaken for \[name\]".

Nice to have: a "reveal mode" that steps through stories one at a time (story first, author on click), for the host to present live.

## Access

The app requires Google sign-in and only admits verified accounts on the QBench Google Workspace domain; there are no public pages besides the sign-in screen.

- **Sign-in:** Google OAuth 2.0 / OpenID Connect, no other login methods.
- **Domain lock, enforced server-side:** on every sign-in, verify the ID token and require `hd` to equal the QBench domain and `email_verified` to be true. The `hd` parameter on the auth URL only pre-filters the account picker and is not enforcement on its own.
- **Consent screen:** if QBench admins allow it, set the Google Cloud OAuth consent screen to Internal, which blocks outside accounts at Google's end too.
- **Rejected accounts** see: "Mystery History is only open to QBench accounts. Sign in with your QBench Google account."
- **Sessions:** every API route requires a valid session; nothing, including story text, is served to signed-out visitors.
- **Identity:** a player is their lowercase verified Google email. Name and avatar come from the Google profile.
- **Hosts:** an allowlist of emails in an environment variable. Only hosts see the Host screen, and host-only API routes check it server-side.
- **Config:** the domain, host allowlist and OAuth client ID/secret are environment variables, not hard-coded.

## Data model

Three collections are enough, keyed by verified work email; the stack is up to the implementer.

| Collection | Key | Fields | Who writes |
| --- | --- | --- | --- |
| stories | author's work email | name, text, createdBy, updatedAt | Hosts any; each player their own |
| plays | player's work email | answers: { storyEmail: { choice, answeredAt } }, lifelineUses: \[ { type: fifty, twenty or double, storyEmail } \] | Server only, on the player's behalf |
| config | single record | acceptGuesses, instantFeedback, revealResults | Hosts only |

**Implementation notes for Claude Code**

- Keying a story by its author's email enforces one story per person and makes the correct answer simply the story's key.
- Derive options, removals and question order deterministically from emails and story keys with a seeded hash, rather than storing them. Compute scores from stored answers at display time.
- Keep the game logic on the server. The client receives story text, option names and avatars, never the author key. The server validates each answer and lifeline use (closed game, already answered, no uses left, one 50:50 and one 20:80 per story) and returns the result.
- Players never read other players' plays, since no lifeline needs them. Leaderboard and results come from a server endpoint that returns data only once Reveal is on, or to hosts.
- With answers checked server-side and the domain lock in place, the dev-tools cheating risk is gone; the only remaining spoiler risk is people talking to each other.

## Screens and edge cases

The app has four tabs (Play, My story, Results, and Host for hosts only); Play shows Intro, Play or Done depending on progress. Signed-out visitors see only a "Sign in with Google" screen.

| Screen | Contents |
| --- | --- |
| Intro | Shown before a player's first answer: headline "Whose story is this?", how many stories there are, the three lifelines with their use counts and one-line descriptions, "Open the first case" |
| Play | Case counter and progress ladder, score (if feedback is on), lifeline bar, story, A to J options with already-picked markers, Lock it in, feedback, Next case |
| Done | "Case closed" with score, a note that new stories may still arrive, link to Results if revealed |
| My story | Textarea for the player's own story, save |
| Results | See Results and reveal |
| Host | Settings switches, add or edit story, bulk import, story list with edit and delete |

**Look and feel:** game-show inspired. Deep navy with gold accents, elongated hexagon answer buttons, round lifeline badges. Works in light and dark mode, and on phones: the ten options sit in two columns of five on desktop and stack to one column on phones.

**Edge cases**

- No stories yet: Play says "No cases yet" and points to My story.
- A story is added after a player finishes: it appears as a new case on their next visit.
- A story is deleted after being answered: that answer stops counting.
- Fewer than 10 authors exist: show fewer options and scale removals as in Lifelines.
- Guessing closed mid-question: the lock-in fails gracefully with "Guessing is closed".
- Network failure on save: keep the selection, show an error, allow retry. Never mark a lifeline used unless the save succeeded.

**Open questions**

- Should people who didn't submit a story still play? (Assumed yes.)
- Should someone's story still appear if they never log in? (Assumed yes, they just don't appear on the leaderboard.)
- Any prizes for the top detective, to mention on the Intro screen?
