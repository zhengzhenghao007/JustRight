import random
import time

# === Initialize deck ===
suits = ['♠', '♥', '♦', '♣']
ranks = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A']
deck = [rank + suit for suit in suits for rank in ranks]
random.shuffle(deck)

# Split deck between two players
player1_deck = deck[:26]
player2_deck = deck[26:]
table_pile = []

# Timer and round counter
start_time = time.time()
time_limit = 10 * 60  # 10 minutes
round_num = 1

# === Render card (always 9 lines) ===
def render_card(card):
    rank = card[:-1]
    suit = card[-1]

    # Face cards rendering
    if rank in ['J', 'Q', 'K']:
        return [
            "+-----------+",
            f"|{rank:<2}         |",
            "|           |",
            "|           |",
            f"|     {rank}     |",
            "|           |",
            "|           |",
            f"|         {rank:>2}|",
            "+-----------+"
        ]

    # Number cards layout
    try:
        count = int(rank)
    except:
        count = 1  # A

    layout_templates = {
        1:  ["           ", "           ", "     ♠     ", "           ", "           "],
        2:  ["           ", "     ♠     ", "           ", "     ♠     ", "           "],
        3:  ["     ♠     ", "           ", "     ♠     ", "           ", "     ♠     "],
        4:  ["           ", "  ♠     ♠  ", "           ", "  ♠     ♠  ", "           "],
        5:  ["  ♠     ♠  ", "           ", "     ♠     ", "           ", "  ♠     ♠  "],
        6:  ["  ♠     ♠  ", "           ", "  ♠     ♠  ", "           ", "  ♠     ♠  "],
        7:  ["  ♠     ♠  ", "     ♠     ", "     ♠     ", "  ♠     ♠  "],
        8:  ["  ♠     ♠  ", "  ♠     ♠  ", "  ♠     ♠  ", "  ♠     ♠  "],
        9:  ["  ♠     ♠  ", "  ♠     ♠  ", "     ♠     ", "  ♠     ♠  ", "  ♠     ♠  "],
        10: ["  ♠     ♠  ", "  ♠     ♠  ", "   ♠   ♠   ", "  ♠     ♠  ", "  ♠     ♠  "]
    }

    symbols = layout_templates.get(count, ["   ♠   "])
    symbols = [line.replace("♠", suit) for line in symbols]

    lines = ["+-----------+"]
    lines.append(f"|{rank:<2}         |")

    padding = 5 - len(symbols)
    top_pad = padding // 2
    bottom_pad = padding - top_pad

    lines += ["|           |"] * top_pad
    lines += [f"|{line}|" for line in symbols]
    lines += ["|           |"] * bottom_pad

    lines.append(f"|         {rank:>2}|")
    lines.append("+-----------+")
    lines.append("+-----------+")  # Ensure 9 total lines

    return lines

# === Show cards side by side ===
def show_cards(cards):
    if not cards:
        print("(Table is empty)")
        return
    card_lines = [render_card(card) for card in cards]
    for i in range(9):
        print("  ".join(line[i] for line in card_lines))

# === Input with range validation (1 to N) ===
def get_valid_input(prompt, max_index):
    while True:
        try:
            idx = int(input(prompt))
            if 1 <= idx <= max_index:
                return idx - 1
            else:
                print(f"Enter a number between 1 and {max_index}.")
        except:
            print("Invalid input. Try again.")

# === Main game loop ===
first_player = 1  # First player of the game

while time.time() - start_time < time_limit:
    print(f"\n======= Round {round_num} =======")
    print("Table Pile:")
    show_cards(table_pile)

    # Determine play order based on who won last round
    if first_player == 1:
        turn_order = [(1, player1_deck), (2, player2_deck)]
    else:
        turn_order = [(2, player2_deck), (1, player1_deck)]

    collected_this_round = None  # Track who collects cards

    for player_id, player_deck in turn_order:
        if not player_deck:
            continue

        print(f"\nPlayer {player_id}'s turn. You have {len(player_deck)} hidden cards.")
        print(f"Select a position from 1 to {len(player_deck)} (memory-based):")
        index = get_valid_input(f"Player {player_id}, choose card index: ", len(player_deck))

        selected_card = player_deck.pop(index)
        print(f"\nPlayer {player_id} plays:")
        show_cards([selected_card])

        if not table_pile:
            table_pile.append(selected_card)
        else:
            if selected_card[:-1] == 'J':
                # J collects all cards from the table
                print(f"J played! Player {player_id} collects all table cards.")
                player_deck.extend(table_pile + [selected_card])
                table_pile.clear()
                collected_this_round = player_id
            else:
                # Check if any matching rank exists on the table
                found_index = None
                for i, card in enumerate(table_pile):
                    if card[:-1] == selected_card[:-1]:
                        found_index = i
                        break

                if found_index is not None:
                    # Collect from the matched rank onward
                    collected_cards = table_pile[found_index:] + [selected_card]
                    print(f"Match found at table position {found_index + 1}! Player {player_id} collects {len(collected_cards)} cards.")
                    player_deck.extend(collected_cards)
                    table_pile = table_pile[:found_index]
                    collected_this_round = player_id
                else:
                    table_pile.append(selected_card)

        print("\nCurrent Table Pile after this play:")
        show_cards(table_pile)

        if len(player_deck) == 52:
            print(f"\nPlayer {player_id} wins by collecting all 52 cards!")
            exit()

    # The collector of this round starts next round
    if collected_this_round is not None:
        first_player = collected_this_round

    round_num += 1

    if not player1_deck or not player2_deck:
        break

# === End of Game ===
print("\nTime is up or one player has no cards. Game Over!")
print(f"Player 1 has {len(player1_deck)} cards.")
print(f"Player 2 has {len(player2_deck)} cards.")

if len(player1_deck) > len(player2_deck):
    print("Player 1 wins!")
elif len(player2_deck) > len(player1_deck):
    print("Player 2 wins!")
else:
    print("It's a tie!")

