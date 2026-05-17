"""
James No's submission for the SVC Computing Club
Split or Steal Coding Competition.

This file defines a strategy function that determines
this player's decision in each round of a game.
"""

import game


def strategy(round: int, my_history: list[game.choice], opponent_history: list[game.choice]) -> game.choice:
    """
    Adaptive Grudger with aggressive endgame.
    
    Strategy:
    - Cooperate initially
    - Hold permanent grudges after 2 steals
    - Steal last 3 rounds from cooperators
    
    :param round: Current round number (0-49)
    :param my_history: List of your previous decisions (length == round)
    :param opponent_history: List of your opponent's previous decisions (length == round)
    :return: Your decision for the current round
    """
    
    # First round: cooperate
    if round == 0:
        return game.choice.SPLIT
    
    # Calculate opponent's steal count
    steal_count = sum(1 for choice in opponent_history if choice == game.choice.STEAL)
    steal_rate = steal_count / len(opponent_history)
    
    # Permanent grudge after 2 steals total
    if steal_count >= 2:
        return game.choice.STEAL
    
    # Endgame: last 3 rounds, steal from anyone with steal_rate < 30%
    if round >= 47:
        if steal_rate < 0.3:
            return game.choice.STEAL
        # Mirror if they're aggressive
        return opponent_history[-1]
    
    # Otherwise: pure Tit-for-Tat
    return opponent_history[-1]
