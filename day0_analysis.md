# Day 0 Summary

## The Problem
I am given a hidden word as dashes. I need to guess each letter of that word. Correct guess reveals all positions of that letter. Wrong guess costs 1 life (out of 6). Losing all 6 lives results in losing the game.

## The Baseline Algorithm
The baseline algorithm filters words from the dictionary by matching the current pattern (word length + revealed letters). It counts letter frequency from that filtered set and guesses the letter with highest frequency. If no matches found, it falls back to overall dictionary frequency.

## Data Exploration
I explored the training data:
- Total: 227,300 words in training dictionary
- Most common word length: 9 letters
- Most common letter overall: 'e' (11%)
- Algorithm guesses 'e' first because it's most common

## Practice Games
I ran 27 practice games:
- Win rate: 22% (6 wins, 21 losses)
- Matches expected baseline of ~18-20%

## Failed Game Analysis
I analyzed 3 failed games in detail and discovered two main problems:

### Problem 1: Dictionary Mismatch (~50% of failures)
- Training dictionary: 227,300 words
- Test dictionary: Different 250,000 words
- Many test words aren't in training dictionary
- When word not in dictionary:
  - Algorithm finds 0 matches
  - Falls back to guessing common letters randomly
  - Almost guaranteed to fail
- 2 out of 3 analyzed failures were dictionary mismatch (67%)

### Problem 2: Frequency Strategy Limits (~30% of failures)
- Algorithm always guesses most common letters first (e, i, a, n, o, r, s, t)
- Rare letters (q, x, z, k, j) ranked 20-26
- Only 6 wrong guesses allowed
- Never reaches rare letters before running out of lives
- Even when word IS in dictionary, fails if word needs uncommon letters

### Additional Observation
Dictionary has junk words like "aaa", "aaaaa", "aalii" which may skew frequency counts.

## Ideas for Improvement

### Idea 1: Position-Specific Letter Frequency
- Current: Counts all letters across matching words
- My idea: Count letters separately by position
- Example: For pattern `_a_`, count position 1 separately from position 3
- Expected: More targeted guessing (+2-4%)

### Idea 2: Adaptive Vowel-First Strategy
- Try vowels in frequency order: e, a, i, o, u (based on dictionary analysis)
- Stop after TWO vowel misses
- Switch to frequency-based strategy
- Reasoning:
   - 'e' and 'a' are top 3 most common letters (not just vowels!)
   - Trying them first is statistically sound
   - Two misses suggest word is consonant-heavy or unusual
   - Don't waste more lives on remaining vowels (i, o, u)
- Limitation: Words like "gym", "by" have no standard vowels (rare)
- Expected: +1-3%

### Idea 3: Adaptive Late-Game Strategy
- Early game (≥4 blanks): Use frequency
- Late game (≤3 blanks, ≥2 lives): Try ALL unguessed letters including uncommon
- Don't be conservative when close to winning
- Expected: +1-2%

### Overall Expected Improvement
22% → 26-30% (modest gain)

**Main limitation:** Can't fix dictionary mismatch (~50% of failures)

## What ML Could Help With

### What ML Can Learn:

1. **Letter combinations automatically**
   - "qu" → usually followed by "i" or "u"
   - Common endings: "-ing", "-tion", "-ely", "-ness"
   - Common pairs: "th", "ch", "sh"
   - Would need hundreds of manual rules!

2. **Context-dependent strategy**
   - When word length=8 AND pattern has 'e' at position 6 → next is likely 'y'
   - Combines: word length + pattern + lives + already tried
   - Can't code every combination manually

3. **Learning from experience**
   - ML plays 100,000 games
   - Tracks which decisions led to wins vs losses
   - Automatically adjusts
   - Would have to manually test every change

4. **Adapting to game state**
   - Different strategies for early/late game
   - Different for high/low lives
   - Different for short/long words

### What ML CANNOT Fix:

1. Dictionary mismatch - If word not in training, ML also fails (~50% ceiling)
2. Rare words - ML learns common patterns, struggles with unusual combinations
3. Fundamental difficulty - Only 6 guesses for 26 possible letters

### Expected Performance:
- Baseline: 22%
- With my improvements: 26-30%
- With ML: 40-50%
- Theoretical max: ~70% (limited by dictionary)

## Next Steps
Build local simulator to test improvements without wasting API practice games.