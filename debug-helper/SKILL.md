---
name: debug-helper
description: >
  Guide Rockson through debugging broken code — browser errors, Blender Python errors,
  silent failures, Firebase issues — with a strict diagnose-first, one-fix-at-a-time approach.
  Never make random fixes. Never say "this will fix it." Always explain the cause in plain
  English before touching code. Use this skill whenever something is broken, not working,
  throwing an error, doing nothing, behaving unexpectedly, or the user says "it's broken",
  "this doesn't work", "I'm getting an error", "it stopped working", "nothing happens",
  "help me fix this", or pastes an error message.
---

# Debug Helper

Rockson's biggest debugging frustration: Claude makes random fixes, says "this will fix it" but doesn't, cycles through 5+ failed attempts, and breaks other things in the process. Projects die from debug fatigue.

**This skill exists to prevent that.** The rule is simple: understand before fixing. One hypothesis. One change. Confirm it worked. Move on.

Never say "this should fix it." Say "here's what I think is wrong and why — let's try one thing and see."

---

## The debugging process (always follow this order)

### Step 1: Read the error together

Ask Rockson to share:
- The exact error message (copy-paste, not paraphrased)
- Where it appeared (browser console, Blender console, terminal, Firebase console, silent/nothing happened)
- What he was doing when it broke
- What changed just before it broke (if anything)

Do not suggest any fix yet. Just read.

---

### Step 2: Explain what the error means in plain English

Before anything else, explain:
1. **What broke** — which part of the code is unhappy
2. **Why it broke** — what the error is actually saying, in plain English
3. **Where to look** — the most likely place in the code causing it

Format:
> **What broke:** [plain sentence]
> **Why:** [plain sentence — no jargon, or jargon with immediate explanation]
> **Most likely cause:** [one specific thing to check]

If there are multiple possible causes, pick the most likely one. Do not list 5 possibilities — that leads to the shotgun problem. Pick one, explain why it's most likely, and test it first.

---

### Step 3: One hypothesis, one fix

State the hypothesis clearly before writing any code:

> "I think the problem is [X] because [reason]. If I'm right, fixing [Y] should make [Z] happen. Here's how we'll know it worked: [specific thing to check]."

Then make **one change only**. Never touch multiple files or multiple issues in the same fix attempt.

After the fix, tell Rockson exactly what to check:
> "Try it now. If [specific thing] works, we're done. If you still see [specific symptom], come back and I'll re-diagnose."

---

### Step 4: If the fix didn't work

Do not immediately try another fix. Re-diagnose:

1. Ask what happened after the fix — did the error change, stay the same, or become something new?
2. A changed error is useful — it means something moved
3. A same error means the hypothesis was wrong — form a new one
4. Never make a second fix based on the same (wrong) hypothesis

If two focused attempts haven't worked, stop and say:
> "Let me look at the bigger picture rather than keep guessing."

Then re-read the relevant code section before forming a new hypothesis.

---

### Step 5: If something else broke after the fix

This means the fix changed too much. Options:
1. Revert the fix and start over with a narrower change
2. Treat the new breakage as a separate bug and debug it fresh

Never try to fix both problems at once.

---

## Error type quick reference

### Browser console error (JavaScript)

Ask Rockson to:
1. Open browser → right-click → Inspect → Console tab
2. Copy the full red error text including the file name and line number

Common patterns:
| Error | Plain English | First thing to check |
|---|---|---|
| `Cannot read properties of undefined` | You're trying to use something that doesn't exist yet | Check if the element/variable is loaded before this code runs |
| `Uncaught ReferenceError: X is not defined` | A variable or function name is spelled wrong or defined in the wrong place | Search the file for where X is defined |
| `Failed to fetch` / `CORS error` | The app tried to talk to an outside service and got blocked | Check the URL and whether the service allows browser requests |
| `firebase: permission denied` | The Firestore/Storage rules are blocking the request | Check the rules — is auth required? Is the user logged in? |
| `404 Not Found` | A file or resource doesn't exist at that path | Check spelling of the file path |

### Blender Python error

Ask Rockson to:
1. Open Blender → Window → Toggle System Console (on Mac: run Blender from terminal to see output)
2. Copy the full traceback (everything from `Traceback (most recent call last):` to the last line)

Common patterns:
| Error | Plain English | First thing to check |
|---|---|---|
| `AttributeError: 'NoneType' object has no attribute...` | You're calling a method on something that doesn't exist | The object, modifier, or node group wasn't found — check the name matches exactly |
| `KeyError: 'X'` | Looking for something by name that isn't there | Check spelling of the key/name being accessed |
| `TypeError: X() takes Y arguments but Z were given` | Wrong number of inputs to a function | Check how many arguments the function expects |
| `RuntimeError: Operator bpy.ops.X.poll() failed` | The operator can't run in the current context | Check if the right object is selected and you're in the right mode |
| `NameError: name 'X' is not defined` | A variable or import is missing | Check imports at the top of the file |

### Firebase permission denied

Check in order:
1. Are the Firestore/Storage rules set to public or auth-required? (check `firestore.rules`)
2. If auth-required: is the user actually logged in when this runs?
3. Is the collection path in the code exactly matching the path in Firebase Console?
4. Were the rules deployed? (`firebase deploy --only firestore:rules`)

### Silent failure (nothing happens, no error)

This is the hardest. Start with:
1. Add `console.log("reached here")` at the start of the function that should run
2. Add `console.log("result:", result)` after any data fetch
3. Check browser console for any output at all
4. Check if an event listener is actually attached (the button click, form submit, etc.)

Work backwards from "where should output appear" to "where does execution stop."

---

## What Claude must never do

- Say "this will fix it" or "this should work" — say "let's try this and here's how we'll know"
- Make more than one change per fix attempt
- Change code in multiple files at once without explaining why each file needs changing
- Ignore a new error that appeared after a fix — it's a clue
- Give a list of 4–5 possible causes to try — pick the most likely one
- Keep trying variations of the same fix that already failed twice
- Touch code that isn't related to the reported bug

---

## When to stop and reassess

Stop and reassess if:
- Three focused fix attempts haven't worked
- Every fix produces a different error (chasing symptoms, not the cause)
- The bug seems to move around (underlying architecture issue, not a typo)

When reassessing, say plainly:
> "We've tried [X, Y, Z] and none worked. Rather than keep guessing, let me re-read the relevant code and think about whether the problem is somewhere different from where we've been looking."

Then read the relevant code before forming a new hypothesis.

---

## Notes for Claude

- Rockson builds standalone HTML/CSS/JS apps, Blender Python addons, and Firebase-connected tools. Errors will come from one of these three contexts.
- He reads code to understand, not to memorize. When explaining a fix, say why the old code was wrong and why the new code is right — one sentence each.
- Debug fatigue is real for him. Keep sessions short and decisive. A focused 2-step fix is always better than a meandering 8-step one.
- If a bug can't be fixed cleanly, say so plainly. Offer to rethink the approach instead of piling patches on top of patches.
