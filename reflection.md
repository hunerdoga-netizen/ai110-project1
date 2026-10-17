# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
- List at least two concrete bugs you noticed at the start  
  (for example: "the hints were backwards").

**Bug Reproduction Log**

Document at least 3 bugs you found. Add rows as needed.

| Input | Expected Behavior | Actual Behavior | Console Output / Error |
|-------|-------------------|-----------------|------------------------|
| Difficulty set to Easy (range 1 to 20), then guess `6` | Secret is within 1 to 20, and the hint for 6 is consistent with the secret | The secret was 81, outside the Easy range. Suspected cause: `st.session_state.secret` is only set once (app.py lines 92-93) using the range of the difficulty selected at first load (Normal, 1 to 100), so it is never redrawn when the difficulty changes. `get_range_for_difficulty` itself returns the right range for Easy. | None (visible in the Developer Debug Info expander: `Secret: 81`) |
| Guess `6` when the secret is 81 | "Too Low" with a "Go HIGHER!" hint | The game said to go lower. The hint messages are swapped in `check_guess` (app.py lines 37-40): the "Too High" branch returns "Go HIGHER!" and the "Too Low" branch returns "Go LOWER!". | None (silent logic bug) |
| Click "New Game 🔁" | A fresh game starts and I can guess again | The button does nothing useful, especially after a win or loss. Suspected cause: the `new_game` block (app.py lines 134-138) resets `attempts` and `secret` but not `status`, so `status` stays "won"/"lost" and `st.stop()` (line 145) blocks the game. It also picks the new secret with `random.randint(1, 100)` regardless of difficulty. | None |

---

## 2. How did you use AI as a teammate?

**AI tools used:** Claude Code inside VS Code to read and edit the code, plus a separate chat window where I asked follow-up questions one at a time to understand each step before moving on.

**How I worked with it:** Every time the AI changed code, I compared the old version with the new version and read the diff critically before accepting it. I did not let it run ahead. I kept control of the logical flow and did one step at a time.

**A correct suggestion:** The AI said the swapped hints in `check_guess` were not the only problem. On even-numbered attempts `app.py` also converted the secret to a string (`str(st.session_state.secret)`) before calling `check_guess`, so the comparison became a string comparison (`"9" > "10"` is true). It suggested fixing the swapped messages, always passing the integer secret, and removing the `except TypeError` fallback that only existed to handle that cast. This was correct because I could point to the exact lines causing the symptoms. I verified it by reading the old/new diff, running pytest (the hint tests pass), and driving the real app with Streamlit's `AppTest`: a guess of 2 against a secret of 1 returned "Go LOWER!".

**A suggestion I did not accept as written:** After the logic was moved into `logic_utils.py`, the AI wanted to go ahead and update the test file as well, and it also offered extra cleanups (the Hard range, the scoring formula, the attempts off-by-one). I stopped it before it got to the tests. I wanted to understand the refactor first and keep each change small enough to review. I had it do the refactor, then the tests, in separate steps, each with its own diff. I left the Hard range and scoring alone because they were outside the bugs I was fixing. I checked that the old tests would break (they expected a plain string, but `check_guess` returns `(outcome, message)`), and after updating them all 5 tests passed.

---

## 3. Debugging and testing your fixes

**How I decided a bug was fixed:** I re-ran the exact steps from my Bug Reproduction Log and checked that the expected behavior now happened, rather than trusting that the code looked right.

**Tests I ran:**
- `pytest` on `tests/test_game_logic.py`: 5 passed. Three are the starter tests (now checking the outcome from the `(outcome, message)` tuple). Two are new regression tests for the swapped hints: a guess of 60 against a secret of 50 must say "LOWER", and a guess of 40 against 50 must say "HIGHER". The starter tests only check the outcome label, which was never wrong, so only the two new tests would have caught the swapped messages.
- A headless run of the real `app.py` with Streamlit's `AppTest`. After switching to Easy the secret was inside 1 to 20. A guess above the secret said "Go LOWER!". After a win, New Game set the status back to "playing", reset attempts to 0 and drew a new secret in the Easy range, with no exceptions. The secret happened to be 1, so the "Go HIGHER!" case was only covered by pytest.
- I also ran the live app with `streamlit run app.py` and checked the secret in the Developer Debug Info expander.

**AI's role in the tests:** The AI proposed the test cases and the regression idea. I held it back until the refactor was done, then reviewed the old/new test diff myself. I asked my follow-up questions about why the old tests failed in a separate chat window.

---

## 4. What did you learn about Streamlit and state?

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
- What is one thing you would do differently next time you work with AI on a coding task?
- In one or two sentences, describe how this project changed the way you think about AI generated code.
