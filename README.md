# Knight's Tour Solver Using Genetic Algorithm

I forked this repo to understand how genetic algorithms actually work. After trying to code the knight's tour manually myself, this seemed like the natural playground to really get it. Big shout out to the original owner of this repo.

## How It Works

### The Chromosome Class

The `Chromosome` class defines the knight's inheritable properties — its genes.

- The chromosome has a length of 63, representing the number of moves the knight needs to take to reach every square on the board (64 squares, 63 moves between them).
- **Crossover** is controlled by a `crossover_prob` between 0.0 and 0.9. If crossover doesn't occur, the offspring are direct copies of the parents (no genetic mixing). If it does occur, a random crossover point is chosen and the parents' genes are swapped past that point to produce the two offspring.
- **Mutation** checks each of the 63 genes independently, giving each one a 1% chance of being replaced with a random value from 1–8. Across a 63-gene chromosome, that works out to roughly a 63% chance that *at least one* gene mutates somewhere in the chromosome.

### The Knight Class

The `Knight` class defines the actual organism — the knight itself.

A knight moves in an L-shape: two squares in one direction, one square in the other. The `MOVES` dict holds all 8 possible move vectors a knight can make. These moves are what the genes actually encode so a knight's gene sequence is really a list of 63 moves it intends to make around the board.

## Selection Method: Tournament Selection

This implementation uses **tournament selection** instead of the classic roulette wheel.

Instead of giving every organism in the population a slice of a wheel sized by fitness (roulette wheel), tournament selection randomly pulls a small handful of knights out of the population — a "tournament" — and simply picks the two of that small group with the best fitness. 

The nice thing about it is that selection pressure is easy to tune just by changing the tournament size: a tournament of 1 is pure randomness (no fitness pressure at all), while a huge tournament (close to the full population) almost always picks the single best knight every time, which pushes the population to converge fast — sometimes too fast, at the cost of diversity. It also doesn't need fitness values to be positive or normalized the way roulette wheel does, which made it a simpler fit here.

## My Tweaks: Building a Closed Tour

The original version of this project solves the **open** knight's tour — any path that visits all 64 squares, ending wherever it happens to end. I tweaked the code to instead search for a **closed** tour, where the last square visited also has to be a legal knight's move away from the first square, so the knight could loop right back to where it started.

### What I Noticed

- The open tour only needed around 500 generations, at most, to find a solution with fitness 64.
- Once I switched to searching for a closed tour, it regularly took well over 500 generations, and sometimes close to 1500.
- Runs were surprisingly inconsistent even with identical parameters and nothing else changed — some runs solved it in under 500 generations, others took over 1000, and some didn't find a solution at all.
- Oddly, if one run found a closed tour successfully, the very next run (same parameters, same starting square) sometimes wouldn't — it seemed to alternate back and forth between finding one and not.
- I originally guessed that starting from the origin only leaves the knight one valid closed tour to find in the whole solution space, which would explain why results felt so binary. Since then I've spotted at least two distinct closed tours from the origin, so that theory doesn't fully hold — there's more than one target hiding in there, just still a very small number relative to the size of the search space.
- I suspect the crossover rate of 0.9 plays a role in that inconsistency too, though I haven't fully pinned down how.
- When I reduced the crossover rate, the GA consistently took fewer generations to find a solution, which backs up that hunch.

Basically, the GA is searching for one exact solution somewhere in an enormous solution space — so whether or not it finds it in a reasonable number of generations comes down to a lot of luck, run to run.

## Final Thoughts

All in all, it was a fun project. I had a lot of fun exploring it and running it different times, watching it succeed, fail, and succeed again for reasons I'm still not 100% sure of.