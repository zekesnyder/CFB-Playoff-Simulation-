# College Football Playoff Simulator

A Monte Carlo simulator for the 2024 12-team College Football Playoff, written in R. It builds a statistical profile for each playoff team from 2024 play-by-play data, simulates games play by play, and plays out the full bracket thousands of times to estimate each team's chances of advancing and winning the championship.

This project was originally completed as **Case 5** for a sport analytics course.

## What it does

- Builds a **team profile** for each of the 12 playoff teams from real 2024 play-by-play data (run/pass mix, yardage distributions, turnover rates, kicking, punting, and returns).
- Simulates a **single game** drive by drive with a game clock, downs, turnovers, scoring, halftime, and extra periods.
- Tracks **player-level stats** (rushing, passing, and receiving yards, touchdowns, fumbles, interceptions) for every simulated game.
- Simulates the **entire playoff bracket** and, over many runs, reports the probability that each team reaches the second round, semifinals, finals, and wins the title.

## Teams

The bracket uses these 12 teams: Tennessee, Ohio State, Oregon, Clemson, Texas, Arizona State, Indiana, Notre Dame, Georgia, SMU, Penn State, and Boise State.

| Round | Matchups |
|---|---|
| First Round | Tennessee vs. Ohio State, Clemson vs. Texas, Indiana vs. Notre Dame, SMU vs. Penn State |
| Second Round | Winners face Oregon, Arizona State, Georgia, and Boise State respectively |
| Semifinals | Winners of the second-round games |
| Finals | Winners of the semifinals |

## Repository contents

| File | Description |
|---|---|
| `Snyder_Case5_fixed.Rmd` | The full R Markdown analysis and simulator |

> The two CSV files present in the rmd are not included in this repository. They are contracted data sources protected by a non disclosure agreement

## Requirements

- R (4.0 or later recommended)
- [RStudio](https://posit.co/download/rstudio-desktop/) (optional, but makes knitting easy)
- R package: `tidyverse`

```r
install.packages("tidyverse")
```

## Getting started

1. Clone the repository.
2. Add `CFB_pbp_2024.csv` and `field_goal_summary_season.csv` to the project folder.
3. **Update the working directory.** The setup chunk contains a hard-coded path from the author's machine:

   ```r
   knitr::opts_knit$set(root.dir = "/Users/zekesnyder/Losak/Case 5")
   ```

   Change this to the folder on your machine, or remove the line if you open the `.Rmd` from its own folder.
4. Open `Snyder_Case5_fixed.Rmd` in RStudio and run the chunks in order, or knit to HTML.

The data-loading chunk checks that all required columns exist and that every playoff team appears in the data, and stops with a clear error message if something is missing.

### Required data columns

`pos_team`, `def_pos_team`, `play_type`, `yards_gained`, `yards_to_goal`, `yds_kickoff`, `yds_kickoff_return`, `yds_punted`, `yds_punt_return`, `int`, `fumble_vec`, `fg_make_prob`, `rusher_player_name`, `passer_player_name`, `receiver_player_name`

The kicker file needs `team_name`, `player`, and `pat_percent`.

## Usage

### Simulate one game

```r
set.seed(1551)
one_game <- game_simulator("Texas", "Georgia", profiles = profiles)

one_game$game_outcome        # winner, loser, scores, score differential
head(one_game$player_stats_df)
```

Add `verbose = TRUE` to print the game play by play.

### Simulate the bracket once

```r
set.seed(6767)
bracket_results <- simulate_bracket(teams, game_simulator, playdata, fg_pat_data,
                                    num_simulations = 1, profiles = profiles)

bracket_results$score_data
bracket_results$player_stats
```

### Estimate championship odds

```r
set.seed(2024)
many_results <- simulate_bracket(teams, game_simulator, playdata, fg_pat_data,
                                 num_simulations = 1000, profiles = profiles)

many_results$probabilities %>% filter(stage == "Won Championship")
```

Large runs can take a while, so time a small batch (for example, 10 simulations) with `system.time()` before scaling up. The `many-simulations` and `timing` chunks are set to `eval=FALSE` so they do not run when knitting.

`simulate_bracket()` returns a list with:

- `probabilities`: the share of simulations in which each team reached each stage
- `score_data`: scores and winners for every simulated game
- `player_stats`: player stat lines for every simulated game

## How the simulation works

### Team profiles
Each team's probabilities are calculated once and saved, so a game only draws random numbers rather than re-filtering the full dataset on every play. A profile includes:

- Run/pass play mix
- Yardage category probabilities for runs and passes (negative-yardage plays are capped at 7% of plays)
- Kickoff distances and kickoff return distribution
- Punt distance and punt return distributions (league averages are used as a fallback if a team has no data)
- Fumble rate on runs and interception rate on passes
- Field goal make probability by distance bin (defaults to 60% when there is no data)
- Extra point percentage of the team's most-used kicker (defaults to 95%)
- Player usage shares for rushers, passers, and receivers (players under 7.5% usage are grouped as "Player X")

### Play simulation
Each play, the offense chooses run or pass based on its play mix. It then draws a yardage category from its own distribution and picks a yard value within that category. Interceptions and fumbles are checked, and the ball changes hands when they occur.

### Game flow
- A coin toss decides the opening kickoff, and the other team receives to start the second half.
- Touchdowns are worth 6 points plus a simulated extra point.
- On 4th down, a team attempts a field goal if it is 40 yards from the goal or closer, otherwise it punts.
- Tied games continue in 5-minute extra periods (up to 10), and a coin flip decides the game if it is still tied afterward.

## Assumptions and limitations

To keep the model tractable, the simulator makes these simplifying assumptions:

- No 2-point conversions
- No turnover returns
- No timeouts
- No penalties
- A pass takes 35 seconds and a run takes 45 seconds
- Extra periods are 5-minute periods, a simplification of NCAA overtime
- Teams are modeled only from their own 2024 offensive and special teams data; opponent defensive strength is not modeled

Results should be read as an estimate under these assumptions rather than a prediction of real outcomes.

## Author

Zeke Snyder
