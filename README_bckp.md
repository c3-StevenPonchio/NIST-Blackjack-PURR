# NIST-Blackjack-Testing

## Overview

NIST-Blackjack-Testing is a simulation framework for the classic card game Blackjack, implemented in modern Fortran. The repository provides tools to shuffle and manage decks of cards, handle gameplay logic for player and dealer interactions, and determine game outcomes based on Blackjack rules. The project utilizes the Knuth Shuffle algorithm for randomization and emphasizes modular design with separate components for game logic, shuffle utilities, and auxiliary programs. This implementation is robust, modular, and suitable for testing and expanding upon Blackjack simulations.

## Key Features

- **Blackjack Game Logic**  
  Simulates simplified Blackjack gameplay, including shuffling cards, dealing, tracking scores, and determining winners.

- **Knuth Shuffle Algorithm**  
  Efficiently shuffles arrays, ensuring random permutations for reliable card randomization.

- **Debug Mode**  
  A feature toggle for manual testing and controlled input during development.

- **Modular Implementation**  
  Separate modules for card shuffling, gameplay logic, and standalone utilities, improving code maintainability and extensibility.

- **Flexible Gameplay**  
  Designed for simulations of a single Blackjack hand or standalone shuffling operations.

- **External Shuffle Utility**  
  Independent programs to shuffle integer lists, enabling reuse in broader applications or testing scenarios.

# Layout and Architecture
```
# File Tree with Brief Comments

└── 672d8a68-978f-4850-a1ea-82d7e1e8003c
    └── NIST-Blackjack-Testing
        ├── .github
        │   └── workflows
        │       └── ci.yml           # CI configuration file
        ├── CMakeLists.txt           # Build system settings for CMake
        ├── CMakePresets.json        # CMake preset configurations
        ├── LICENSE                  # Project license file
        ├── app                      # Application-level code
        │   ├── main.f90             # Main program for Blackjack game
        │   └── rand_order.f90       # Auxiliary program for generating/shuffling integer lists
        ├── fpm.toml                 # Fortran package manager configuration
        ├── meson.build              # Build system settings for Meson
        ├── src                      # Source code for core modules
        │   ├── blackjack.c          # Placeholder for C code (possibly unused/experimental)
        │   ├── blackjack.f90        # Module implementing the Blackjack game logic
        │   └── shuffler.f90         # Module for shuffling utilities (Knuth shuffle implementation)
        └── tests                    # Test scripts and files
            ├── test_hit.cmake       # CMake test script for "hit" functionality
            ├── test_hit.py          # Python-based test for "hit" functionality
            └── y.asc                # Input/test data file
```

```mermaid
graph TD
    subgraph NIST-Blackjack-Testing
        A["blackjack (Entry Point)"] --> B["Game Logic (blackjack.f90)"]
        A --> C["Shuffle Utility (rand_order.f90)"]

        subgraph "Core Modules"
            B --> D["game (Module)"]
            D --> E["hand()"]
            D --> F["hit()"]
            D --> G["mix()"]
            C --> H["shuffler (Module)"]
            H --> I["knuth_shuffle()"]
        end

        subgraph "Utilities"
            H --> J["random_init()"]
        end
    end

    A -.-> ["main.f90 (Program)"]
    C -.-> ["rand_order.f90 (Program)"]
    D -.-> ["blackjack.f90 (Module)"]
    H -.-> ["shuffler.f90 (Module)"]
    J -.-> lib_utils["Utility Functions"]
```


## Usage Examples

### Build

To build the Blackjack game:
```bash
mkdir build
cd build
cmake ..
make
```

### Test

Run the full set of tests:
```bash
ctest
```

Alternatively, a specific Python-based test for the "hit" functionality can be executed:
```bash
python3 tests/test_hit.py path/to/executable
```

### Run

Execute the Blackjack game program:
```bash
./game
```

### Gameplay

Initialize and shuffle the deck:
```fortran
call mix(cards)
```

Simulate a Blackjack hand:
```fortran
win = hand(cards)
if (win == 1) then
    print *, "Player Wins!"
else if (win == 0) then
    print *, "Dealer Wins!"
else
    print *, "It's a push (tie)!"
endif
```

### Shuffle Utilities

Perform a Knuth shuffle on an array:
```fortran
call knuth_shuffle(A)
```




# Key Feature Implementation Deep Dive

## 1. Deck Shuffling (`game::mix` using `shuffler::knuth_shuffle`)
The `mix` subroutine in the `game` module creates and initializes a standard Blackjack deck (including values from 2 through Ace). The `shuffler::knuth_shuffle` subroutine is then invoked to randomize the order of the deck. This shuffle is implemented using the Knuth/Fisher-Yates algorithm for efficient, uniform randomization:
1. **Algorithm Process**:
   - Iterate from the last element to the second.
   - Swap the current element with a randomly chosen one before it.
2. **Implementation Specifics**:
   - Calls `random_number` to generate a random float.
   - Maps this float to an index via scaling.

## 2. Card Handling Logic (`game::hit`)
The `hit` subroutine encapsulates the logic for drawing a new card and maintaining accurate game totals. Key responsibilities include:
1. **Updating Totals**:
   - Adds the drawn card's value (`cards(i)`).
   - Handles aces using special rules (default to 11, reduce to 1 if bust occurs).
2. **Debug Mode**:
   - Provides manual input capability for testing, driven by the `debug` flag.
3. **Coordination**:
   - Adjusts the `total` and `aces` counts with proper boundaries.

## 3. Game Simulation (`game::hand`)
The `hand` function is the centerpiece of the gameplay. It handles interactions for both the player and dealer:
1. **Player Actions**:
   - Initial draw: Two cards for player and dealer.
   - Handles player choices via a "hit" loop until stand or bust.
2. **Dealer Logic**:
   - Continues drawing cards until reaching a certain threshold (e.g., 16).
   - Checks for bust.
3. **Results**:
   - Determines winner based on totals and blackjack/bust conditions.
   - Supports ties (push) as an outcome.
4. **Integration**:
   - Calls `hit` for all card draws, updating the deck state dynamically.

## 4. Main Logic (`blackjack` program)
The `blackjack` program orchestrates the gameplay:
1. **Initialization**:
   - Toggles debug mode based on command-line arguments.
   - Invokes `game::mix` for deck preparation.
2. **Execution**:
   - Handles main gameplay loop by calling `game::hand`.
3. **Debugging**:
   - Outputs deck state for verification in debug mode.

## 5. Shuffler Module (`shuffler::knuth_shuffle`)
The `knuth_shuffle` subroutine is an independent component usable across applications requiring randomization:
1. **Implementation**:
   - Directly modifies array elements using swaps.
2. **Reusability**:
   - Could be extended to other card games or general list-shuffling problems.

Overall, the interplay between these features creates a cohesive and modular implementation of Blackjack, balancing efficiency with clarity. Enhancements could focus on extending the game rules or optimizing debug outputs.



# Implemented User Stories

## Blackjack Game Logic
- [ ] As a game player, I want to play a fully simulated Blackjack game against the dealer, so that I can experience a traditional card game simulation, which requires shuffling cards and handling gameplay logic (draws, blackjacks, busts, and score comparisons).
- [ ] As a developer, I want to simulate the outcome of a Blackjack hand, so that I can determine the winner between player and dealer, which requires handling ace adjustments and score updates.
- [ ] As a game player, I want to draw additional cards during my turn, so that I can try to improve my score without exceeding 21, which requires a method for dealing cards dynamically from the deck.
- [ ] As a game player, I want to automatically end my turn with a blackjack or after drawing five cards without busting, so that I can follow the standard game rules, which requires checking for automatic win criteria.
- [ ] As a developer, I want to observe the dealer's moves being handled automatically, so that the dealer tries to win while following the rules (e.g., standing at 17 or higher), which requires game logic for automated play.

## Debugging Mode
- [ ] As a tester, I want to toggle a debugging mode, so that I can manually input cards for testing scenarios, which requires a debugging flag to enable manual handling of card draws.

## Deck Shuffling
- [ ] As a developer, I want to shuffle a standard deck of 52 playing cards, so that I can ensure fair gameplay, which requires initializing and randomizing card order with the Knuth shuffle algorithm.

## Shuffling Utilities
- [ ] As a developer, I want to apply a Knuth shuffle algorithm to any array, so that I can randomize the order of elements efficiently, which requires implementing the algorithm as a stand-alone utility.
- [ ] As a developer, I want to shuffle a custom array of integers, so that I can test permutations for non-card-related scenarios, which requires the Knuth shuffle utility's generic implementation.

## Test Functionality
- [ ] As a quality assurance engineer, I want to test the logic for handling card draws, so that I can verify correctness in total score calculations, ace handling, and bust scenarios, which requires automated tests for the "hit" functionality.
- [ ] As a quality assurance engineer, I want to test the deck shuffle implementation, so that I can ensure the Knuth shuffle works correctly, which requires automated tests for the shuffle utility.

## Random Integer Operations
- [ ] As a user, I want to generate and shuffle a list of integers up to a custom maximum value, so that I can test sorting or permutation scenarios, which requires an auxiliary program to manage integer shuffling.
- [ ] As a developer, I want to initialize a random seed for shuffle operations, so that I can achieve reproducible randomization during tests, which requires a random-init subroutine.

## Game Outcome Reporting
- [ ] As a player, I want to view the game result after every hand, so that I can understand whether I won, lost, or tied, which requires output functionality after evaluating outcomes.
- [ ] As a developer, I want to ensure the game accurately prints player and dealer scores, so that users can follow the results step-by-step, which requires output formatting for game statistics.


# Dependencies




## Intrinsic

Standard Fortran intrinsic modules and functions.
- **iso_fortran_env**
  - `ALL`
- **iso_c_binding**
  - `c_int`
## Internal

Modules and functions defined within this project that are accessed in a different module or program.
- **shuffler**
  - `knuth_shuffle`
- **game**
  - `debug`
  - `hand`
  - `mix`
## External Functions

External (non-Fortran, bound with the C ABI) functions called by this project.
- `hit`
- `knuth_shuffle`
- `mix`
