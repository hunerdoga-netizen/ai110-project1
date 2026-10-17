# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable. 

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Run the broken app: `python -m streamlit run app.py`

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: *"How do I keep a variable from resetting in Streamlit when I click a button?"*
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.** - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

**Purpose of the game.** Glitchy Guesser is a Streamlit number-guessing game. The player picks a difficulty, guesses the secret number within a limited number of attempts, gets a Higher/Lower hint after each guess, and earns or loses points.

**Bugs I found.**
- The secret number was drawn once, using the range of the difficulty selected at first load, so switching to Easy could still give a secret like 81.
- The hints were swapped: a guess that was too high said "Go HIGHER!" and a guess that was too low said "Go LOWER!".
- On every even-numbered attempt the secret was converted to a string before the comparison, so numbers were compared as text (`"9" > "10"`).
- "New Game" did not reset the game status, so after a win or loss the game stayed blocked. It also ignored the difficulty range.

**Fixes I applied.**
- Redraw the secret and reset the game state whenever the difficulty changes (`reset_game()` in `app.py`). The info text now shows the real range.
- Swapped the hint messages in `check_guess` and removed the string cast and its `except TypeError` fallback.
- "New Game" now calls `reset_game()`, which resets status, history, score and attempts and draws a secret in the current range.
- Refactored `check_guess`, `parse_guess`, `get_range_for_difficulty` and `update_score` out of `app.py` into `logic_utils.py`, then updated `tests/test_game_logic.py` and added two regression tests for the hint messages.

**Known issues I left alone (out of scope for this lab).** A wrong "Too High" guess on an even attempt adds 5 points instead of subtracting them, the win bonus formula uses `attempt_number + 1`, and Hard (1 to 50) has a smaller range than Normal (1 to 100).

## 📸 Demo Walkthrough

Sample game on Normal difficulty (range 1 to 100) with the secret set to 55. These steps come from running the real `app.py` headlessly with Streamlit's `AppTest`:

1. The page opens with the sidebar showing "Range: 1 to 100" and "Attempts allowed: 8".
2. The user enters `40` and clicks Submit. The game shows "📈 Go HIGHER!" and the score is -5.
3. The user enters `70`. The game shows "📉 Go LOWER!". The score goes from -5 to 0, because a wrong "Too High" guess on an even attempt adds 5 points (a known quirk I did not fix).
4. The user enters `55`. The game shows "🎉 Correct!", plays balloons, and reports "You won! The secret was 55. Final score: 60".
5. The game status is now "won". Further guesses are blocked until the user clicks "New Game 🔁".
6. Clicking New Game resets attempts, score and history and draws a new secret within the current difficulty's range, so the user can play again.

## 🧪 Test Results

```
============================= test session starts ==============================
platform darwin -- Python 3.9.6, pytest-8.4.2, pluggy-1.6.0
rootdir: /Users/dogahuner/codepath/project-1/ai110-project1
collected 5 items

tests/test_game_logic.py .....                                           [100%]

============================== 5 passed in 0.00s ===============================
```
