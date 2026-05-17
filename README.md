# Split or Steal Coding Competition

My submission for the SVC Computing Club prisoner's dilemma tournament (February 2026).

## Competition Results

**Tested Performance:**
- **1st place** with 957 points in diverse competition
- Beat pure STEAL strategy by 91 points
- Beat Grudger, Tit-for-Tat, and all other test strategies

## Strategy: Adaptive Grudger with Endgame Exploitation

### Core Principles

1. **Start Cooperative** - Signal willingness to cooperate (Round 0: SPLIT)
2. **2-Strike Grudge** - Hold permanent grudge after opponent steals twice
3. **Endgame Exploitation** - Steal final 3 rounds from cooperators (< 30% steal rate)
4. **Tit-for-Tat Default** - Mirror opponent's last move otherwise

### How It Works

```python
# Round 0: Always cooperate
if round == 0:
    return SPLIT

# Track opponent aggression
steal_count = count steals in opponent_history
steal_rate = steal_count / total_rounds

# Permanent retaliation after 2 steals
if steal_count >= 2:
    return STEAL

# Endgame (rounds 47-49): exploit cooperators
if round >= 47 and steal_rate < 0.3:
    return STEAL

# Default: mirror opponent (Tit-for-Tat)
return opponent_history[-1]
```

### Performance vs Different Strategies

**vs Always Cooperate:**
- Rounds 0-46: 3 pts/round (both SPLIT) = 141 pts
- Rounds 47-49: 5 pts/round (I STEAL) = 15 pts
- **Total: 156 points**

**vs Always Steal:**
- Rounds 0-1: 0 pts (I cooperate, they steal)
- Rounds 2-49: 1 pt/round (both STEAL) = 48 pts
- **Total: 48 points** (damage minimized)

**vs Tit-for-Tat:**
- Rounds 0-46: 141 pts (mutual cooperation)
- Round 47+: Endgame advantage
- **Total: ~148 points** (beats standard TFT)

**vs Other Adaptive Grudgers:**
- Mutual cooperation until endgame
- **Total: 140-150 points** (competitive)

## Game Rules

- **50 rounds** per game
- **Round-robin** tournament (play everyone once)
- **Scoring:**
  - Both SPLIT: 3-3
  - Both STEAL: 1-1
  - SPLIT vs STEAL: 0-5
- **Winner:** Highest total points across all games

## Why This Strategy Wins

1. **High scores with cooperators** (majority of opponents)
2. **Protected from exploitation** (2-strike grudge)
3. **Endgame edge** over naive strategies
4. **Robust against diverse field** (handles all opponent types well)

## Code Walkthrough

### Complete Implementation

```python
def strategy(round: int, my_history: list[game.choice], opponent_history: list[game.choice]) -> game.choice:
    # First round: cooperate
    if round == 0:
        return game.choice.SPLIT
```
**Line 1-3:** Always cooperate on first move. Signals willingness to collaborate and establishes trust.

```python
    # Calculate opponent's steal count
    steal_count = sum(1 for choice in opponent_history if choice == game.choice.STEAL)
    steal_rate = steal_count / len(opponent_history)
```
**Lines 4-6:** Track opponent behavior. `steal_count` = total steals, `steal_rate` = percentage (0.0-1.0).

```python
    # Permanent grudge after 2 steals total
    if steal_count >= 2:
        return game.choice.STEAL
```
**Lines 7-9:** Core defense mechanism. After opponent steals twice (ever), always retaliate. This:
- Prevents repeated exploitation
- Minimizes losses to defectors (1 pt vs 0 pt per round)
- Optimal threshold: 2 strikes balances forgiveness with protection

```python
    # Endgame: last 3 rounds, steal from anyone with steal_rate < 30%
    if round >= 47:
        if steal_rate < 0.3:
            return game.choice.STEAL
        return opponent_history[-1]
```
**Lines 10-14:** Endgame exploitation. In rounds 47-49:
- If opponent is highly cooperative (< 30% steals), defect for bonus points
- Otherwise mirror their last move to avoid endgame retaliation spirals
- Gets 5 pts instead of 3 with minimal risk

```python
    # Otherwise: pure Tit-for-Tat
    return opponent_history[-1]
```
**Lines 15-16:** Default behavior: mirror opponent's last move (Tit-for-Tat). Maintains cooperation with cooperators, retaliates once against single defections.

### Design Decisions

**Why 2-strike grudge instead of 1?**
- Allows recovery from accidental defections
- Prevents punishment loops from single mistakes
- Testing showed 2 > 1 or 3 in diverse fields

**Why 30% steal rate threshold?**
- < 30% = cooperative enough to exploit safely
- ≥ 30% = too aggressive, risk retaliation
- Empirically optimal in testing

**Why start at round 47 for endgame?**
- 3 rounds = maximum exploitation with acceptable risk
- Earlier = too risky (more time for retaliation)
- Later = missed opportunities

**Why Tit-for-Tat as default?**
- Historically proven in Axelrod's tournaments
- Simple, predictable, encourages stable cooperation
- Works well with other cooperative strategies

## Testing & Replication

### Test Results (8 Opponent Field)

| Rank | Strategy      | Total Points |
|------|---------------|-------------|
| 1st  | **James No**  | **957**     |
| 2nd  | James No V1   | 936         |
| 3rd  | Grudger       | 916         |
| 4th  | Tit-for-Tat   | 900         |
| 5th  | Pure STEAL    | 866         |
| 6th  | Pure SPLIT    | 804         |
| 7th  | Pure SPLIT    | 777         |
| 8th  | Random        | 684         |

### How to Test

To replicate results, you need:
1. **game.py** - Defines `game.choice.SPLIT` and `game.choice.STEAL` enums
2. **competition.py** - Tournament framework
3. **Test opponents** - Various strategy implementations

```python
# Example test setup
import competition
import submissions.james_no
import submissions.tit_for_tat
import submissions.always_steal
# ... more opponents

players = (
    competition.Player("James No", submissions.james_no.strategy),
    competition.Player("Tit-for-Tat", submissions.tit_for_tat.strategy),
    # ... more players
)

competition.run(players)  # Runs round-robin, displays results
```

### Creating Test Opponents

**Tit-for-Tat:**
```python
def strategy(round, my_history, opponent_history):
    if round == 0:
        return game.choice.SPLIT
    return opponent_history[-1]
```

**Always STEAL:**
```python
def strategy(round, my_history, opponent_history):
    return game.choice.STEAL
```

**Grudger:**
```python
def strategy(round, my_history, opponent_history):
    if round == 0:
        return game.choice.SPLIT
    if game.choice.STEAL in opponent_history:
        return game.choice.STEAL
    return game.choice.SPLIT
```

## Files

- `james_no.py` - Main strategy implementation
- `james_no.txt` - Same file with .txt extension (for email submission)
- `README.md` - This documentation

## Competition Details

- **Event:** SVC Computing Club Split or Steal Competition
- **Date:** February 2026
- **Submission:** james_no.txt via email
- **Status:** Submitted

---

*Strategy developed with game theory analysis and tested against multiple opponent archetypes.*
