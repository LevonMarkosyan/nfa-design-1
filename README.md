# NFA Design 1

NFAs built for JFLAP 7.1 for 6 problems from the N/DFA/RE design exercise list, all over Σ = {0, 1}.

| # | Language | NFA | Tests | Report |
|---|---|---|---|---|
| 11 | 2nd to last bit is 1 | [n11.jff](n11.jff) | [n11t.txt](n11t.txt) | [n11r.md](n11r.md) |
| 13 | at least two 1's | [n13.jff](n13.jff) | [n13t.txt](n13t.txt) | [n13r.md](n13r.md) |
| 16 | every odd position is 1 | [n16.jff](n16.jff) | [n16t.txt](n16t.txt) | [n16r.md](n16r.md) |
| 20 | 3k+1 1's (k ≥ 0) or an odd number of 0's | [n20.jff](n20.jff) | [n20t.txt](n20t.txt) | [n20r.md](n20r.md) |
| 23 | odd length or s = 01 | [n23.jff](n23.jff) | [n23t.txt](n23t.txt) | [n23r.md](n23r.md) |
| 24 | odd number of 0's or s = 001 | [n24.jff](n24.jff) | [n24t.txt](n24t.txt) | [n24r.md](n24r.md) |

## Summary of learning

### 1. Which problems gave the most trouble?

**Problems 20 and 24**, and the trouble was in the test strings, not in the machines. With an "or" language it is tempting to judge a string by one condition only. Two strings were first sorted into the wrong group while the tests were being drafted:

- `110011` (problem 20) looks like a reject because its number of 0's is even, but it has four 1's and 4 = 3·1 + 1, so it is accepted.
- `0010` (problem 24) looks like a near miss of `001`, but it has three 0's, so the parity branch accepts it.

Both were caught by checking every label against the language definition before the files were saved.

**Problem 11** needed the biggest change of thinking compared with a DFA. A DFA has to remember the last two bits (4 states). The NFA just loops in `q0` and guesses, on any 1, that this is the 2nd to last bit (3 states, no bookkeeping).

**Skipped problems.** Problem 21 has the same two conditions as problem 20 joined by "and". A λ-union cannot express "and"; it needs the product of the two counters, which is a DFA construction, so nondeterminism buys nothing there. Problems 12, 14, 15 and 17 to 19 were left out for a similar reason: their natural machine is already deterministic. Problem 22 repeats the λ-union idea of problem 20. Problems 6 to 10 were never considered, because of the numbering mistake described in section 3.

**AI use.** Claude (Anthropic's AI assistant, through Claude Code) was used throughout: to design the NFAs, write the test strings, explain Step by State versus Step with Closure, and check each machine against its language definition for every binary string up to length 12. The batch-run and step pictures were produced by a script that drives JFLAP 7.1's own Multiple Run and Step with Closure code and saves the JFLAP window, instead of clicking through it by hand. The computation trees in the reports are typed, not hand-drawn.

### 2. Gold-st-rings

Three strings were traced step by step: [`0110` on problem 11](n11r.md), [`10110` on problem 16](n16r.md) and [`0011` on problem 24](n24r.md). In each one there is a next state that is easy to leave out of the set:

- **`0110`, problem 11.** After the second 1 the active set is {q0, q1, q2}, not {q1, q2}. The state that gets forgotten is `q0`: it loops on every symbol, so it is always still active, and it is what allows the next guess. The first guess reaches `q2` one symbol too early and dies on the last 0; the second guess reaches `q2` exactly at the end.
- **`10110`, problem 16.** Both states are final, so this string cannot be rejected by "ending in a non-final state". It is rejected because `q0` has no move on 0, which leaves the set of next states empty. The empty set is a legal result and it means reject.
- **`0011`, problem 24.** Before any symbol is read the active set is already {q0, q1, q3}, because of the λ-moves. After `001` the set contains the final state `q6`, but that does not matter, because acceptance is only decided when the input ends. The last 1 kills the chain and leaves `q1`, which is not final.

How to avoid these errors in a state controller, a project or an exam:

1. Compute the next set mechanically. For every active state follow every arrow labelled with the symbol, take the union, then add everything reachable by λ. Do not follow only the path that looks promising.
2. Treat "no transition" as an explicit outcome (the branch dies), and decide on purpose whether that is what the design wants.
3. Write the expected accept/reject labels from the language definition, never from the machine, then run them.
4. Where the input space is small, check it exhaustively. Here every string up to length 12 was compared with the definition; for a controller the same idea is to enumerate every state and input pair.

### 3. Other insights, comments, questions

- **The files were first committed under the wrong numbers.** The exercise list was copied out of the notebook as a list numbered from 1, so problems 11, 13, 16, 20, 23 and 24 were first saved as n06, n08, n11, n15, n18 and n19. The mistake was found by comparing with classmates' repos and fixed by renaming; the commit history shows it.
- **λ cannot be loaded from a test file.** JFLAP's Load Inputs splits the file on whitespace, so the empty string has to be added in the Multiple Run table with Enter Lambda. It is the last row of every batch run here.
- **JFLAP keeps one input table per session.** Opening Multiple Run for a second automaton still shows the first automaton's strings, so press Clear before loading the next file.
- **Step by State versus Step with Closure** only differ on problems 20, 23 and 24, the ones with λ-transitions. With Closure each click consumes one symbol, which matches the levels of the computation tree.
- **Problem 23's question** (why 5 states instead of 2 × 4 = 8 in the union DFA): three of the four states of the `s = 01` machine fix the length of the input (nothing read, `0` read, `01` read), so each can pair with only one parity state. Only the dead state can pair with both. That gives 3 + 2 = 5 reachable pairs. The NFA avoids the product entirely: 1 start state + 2 parity states + 3 chain states.
