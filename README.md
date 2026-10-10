# Big O Notation: How to Calculate Any Algorithm's Runtime and Explain It Out Loud in an Interview

Ask ten developers to write a function that finds duplicates in an array, and most will hand you something that works. Ask them what its time complexity is, and a fair number will start hedging. They know it's "sort of n squared," or they remember a chart from a course three years ago, but the connection between the code on screen and the O(...) they're supposed to say out loud has gone fuzzy.

That gap is what Big O notation is really about. Not the Greek letters, not the asymptote proof — the ability to look at a loop and say, with reasons, how the work grows as the input grows. It's the vocabulary interviewers use to check whether you understand what you wrote, and it's the reason two solutions that pass every test case can be graded completely differently.

Here's the whole idea, from the ground up, plus the specific places where people get it wrong under pressure.

## What Big O actually measures

Big O describes growth, not speed. A function `f(n)` is in `O(g(n))` if there exist positive constants `c` and `n₀` such that `f(n) ≤ c·g(n)` for every `n ≥ n₀`. In plain terms: past some input size, your runtime will never exceed some fixed multiple of `g(n)`. [1][2]

Two consequences fall straight out of that definition:

- **Constants don't matter.** If the inequality holds with `c = 9`, it holds with `c = 500`. So `3n² + 4n + 2` is `O(n²)`, and so is `0.0000001n²`. [2][3]
- **Lower-order terms don't matter.** As `n` grows, `n²` dwarfs `n` and any constant. Keep the fastest-growing term and throw the rest away. [3][4]

Big O is technically an **upper bound** — it says your algorithm is *no worse than* this. The matching lower bound is `Ω`, and `Θ` means the bound is tight in both directions. [4] Most of the time when someone says "this is O(n log n)," they actually mean `Θ(n log n)`, and nobody in a coding interview will object. Knowing the distinction costs you one sentence and occasionally saves you an argument when a problem explicitly asks for the tightest bound.

## The complexity classes worth knowing

You will see these eight for the rest of your career. The last column is what happens to your runtime when the input size doubles — the most useful intuition shortcut in the whole subject. [5]

| Class | Name | What usually produces it | Example | When n doubles |
| --- | --- | --- | --- | --- |
| `O(1)` | Constant | Direct index, hash lookup, arithmetic | Array access | No change |
| `O(log n)` | Logarithmic | Cutting the search space in half each step | Binary search | Adds a constant |
| `O(n)` | Linear | One pass over the input | Linear search, max of an array | ~2× |
| `O(n log n)` | Log-linear | Sort, or divide-and-conquer that recombines | Merge sort | ~2× |
| `O(n²)` | Quadratic | Two nested loops over the input | Checking every pair | ~4× |
| `O(nᵏ)` | Polynomial | `k` nested loops over the input | Triple nested loop | `2ᵏ`× |
| `O(2ⁿ)` | Exponential | Include/exclude decisions, naive backtracking | Recursive Fibonacci, subsets | Squared |
| `O(n!)` | Factorial | Every ordering of the input | Brute-force permutations | Multiplied by `n` |

The reason interviewers care about this ordering and not the exact step count: on a modern machine, almost any algorithm is fast enough for tiny inputs. The classes only start to separate from each other as `n` gets large, which is exactly the regime Big O describes. As a computer science course at Princeton puts it, when `n` is large the lower-order terms are negligible, and when `n` is small, nobody cares. [5]

## How to read Big O out of code

Four rules cover the overwhelming majority of code you'll be asked about.

1. **Sequential blocks add.** Two loops one after the other cost `O(n) + O(m)`, not `O(n × m)`.
2. **Nested blocks multiply.** A loop of `n` iterations wrapping another loop of `n` iterations is `O(n²)`. Three levels deep is `O(n³)`, and there's nothing special about three — `k` nested levels gives `O(nᵏ)`. [6]
3. **Drop constants and lower-order terms** once you've added and multiplied. `O(n + n²)` becomes `O(n²)`.
4. **Spot the halving or doubling.** A loop written as `for (int i = n; i >= 1; i /= 2)` runs about `log₂n` times, because you're undoing a series of doublings. [5]

Two examples where rule 4 hides behind innocent-looking code:

python
# O(N): the outer loop runs N times, the inner loop runs about N/2 on average
# — but worst case is the whole N, so the inner loop is still a factor of N.
for a in range(1, N + 1):
    for j in range(a, N + 1):
        ...

# O(log N): i is multiplied by 2 each pass, so the loop exits after log N steps.
i = 1
while i < N:
    i *= 2


That first one trips people up constantly. The inner loop doesn't always run `N` times, and it feels unfair to call the whole thing `O(N²)`. But Big O is computed on the worst case, so the inner loop counts as a full factor of `N`. [7]

## Three traps that show up in real interviews

### The logarithm base is irrelevant

`log₂n`, `log₁₀n`, and `ln n` differ only by a constant factor, and constant factors disappear in Big O. If you see `log n` written without a base, that's why. A related identity worth having at hand: `O(2^log N)` collapses to `O(N)`, since exponentiation and logarithms undo each other. [7]

### Amortized time is not a shortcut

A dynamic array's `append` occasionally copies the entire array into a bigger one — a genuinely `O(N)` operation. It still gets called `O(1)` amortized, because that expensive copy happens so rarely relative to the number of cheap appends that its cost is diluted across all of them. Amortized time is the average cost per operation over a long sequence, and it's the honest answer when an interviewer asks why `list.append()` isn't slow. [7]

### Constraints tell you the intended complexity

This is the reverse direction, and it's the most practically useful trick in interviews. Instead of computing complexity from finished code, read the constraints in the problem statement and work backwards to the algorithm the problem author expects you to write. AlgoMonster's runtime overview gives the standard calibration points: `O(2ⁿ)` solutions are typically viable for `n ≤ 20`, and `O(n!)` for `n ≤ 12`. [7]

So if a problem says `n ≤ 18`, a backtracking search over subsets is almost certainly the intended solution. If it says `n ≤ 10⁹`, you need `O(log n)` or `O(1)`, and any nested loop is instantly wrong. That single read of the constraints can eliminate three bad approaches before you write a line of code — and it's a much better use of the first two minutes than staring at the problem.

## Space complexity uses the same vocabulary

Nothing about the notation changes. You count the extra memory your algorithm allocates as a function of input size: a copy of the input is `O(n)`, a few pointer variables are `O(1)`, and a recursion stack is `O(depth)` — which is why an iterative solution to a tree traversal can be `O(1)` space where the recursive version is `O(log n)` for a balanced tree. Reviewers on AlgoMonster's own discussion forum raise this tradeoff regularly, and it's a standard follow-up question after you give the time complexity. [8]

## Why this comes up in every interview loop

An interviewer asking "what's the complexity?" isn't testing whether you memorized a chart. They're checking three things: can you reason about code you just wrote, do you know where the cost is, and can you improve it if pushed. The follow-ups are predictable — "can you do better?", "what's the space tradeoff?", "what if the array doesn't fit in memory?". Big O gives you language for all of them.

The failure mode third-party reviewers describe most often isn't a candidate who can't write the solution. It's a candidate who spends fifteen of their forty-five minutes trying approaches that couldn't possibly fit the problem's constraints. Pattern recognition and complexity analysis solve the same problem from two directions, which is why platforms that teach patterns treat runtime analysis as the first lesson rather than an appendix.

## Where to actually practice this specific skill

Reading about Big O is a fifteen-minute job. Getting fast at it takes repetition on problems where you have to commit to a complexity before checking the answer.

AlgoMonster's free runtime overview page is worth a look on its own: it walks through `O(1)` up to `O(n!)` with worst-case analysis, code samples in Python, Java, C++, and JavaScript, and a set of practice questions where the point is to name the complexity of a snippet rather than solve a problem. [7] It also covers amortized analysis with a hand-written dynamic array implementation, which is a better explanation of that idea than most textbooks manage.

The paid side is a pattern-based curriculum — 48 patterns with code templates and practice problems, according to a third-party pricing breakdown, which also notes the platform is entirely text and diagrams with no video walkthroughs at all. [9] AlgoMonster's own subscribe page advertises 500+ lessons across 6 courses, 1,000+ illustrations, and 3,000+ editorials, plus bonus access to a CS foundation course and a "build an LLM from scratch" course. [10]

Where it's thin, per the same outside reviews: system design coverage is introductory, so it won't carry you through a senior-level loop on its own. [9]

## What it costs right now

Prices on the site move with seasonal promotions, so treat the numbers below as the current listed rates rather than permanent ones. The yearly plan has shown a list price of $198 and the lifetime plan $378, with both discounted on the subscribe page. [10]

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems — enough to judge whether the text-and-diagram format works for you [9] | $0 | — | [Create a free AlgoMonster account](https://algo.monster?fpr=li69) |
| Pro — Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; interview questions updated monthly; 500+ lessons across 6 courses; 1,000+ illustrations; 3,000+ editorials; 12 months of access [10] | $99/year (~$8.25/month billed annually) | Annual, renews | [Start the Pro yearly plan](https://algo.monster/subscribe?fpr=li69) |
| Pro — Lifetime | Everything in the annual plan, plus lifetime updates and permanent access to future content [10] | $189 | One-time | [Get AlgoMonster Pro lifetime access](https://algo.monster/subscribe?fpr=li69) |

There's no monthly tier — it's annual or lifetime. [10][11] At $189 one-time versus $99 per year, lifetime pays for itself at the start of the second year, which matters if you expect to interview again in two or three years. If your loop is six weeks out and you'll never open the site afterward, the yearly plan is the cheaper honest choice.

## Questions people ask before they subscribe

**Is there a free trial?** The subscribe page has at times displayed a 3-day free trial on the yearly plan, with billing starting after. [12] The free tier exists year-round, though it's a limited slice rather than a full trial.

**Does it include video explanations?** No. The platform is text and interactive diagrams throughout — that's a deliberate design choice, and it's the most common complaint from people who learn better by watching someone code. [9]

**Is there a job guarantee?** Several third-party review sites repeat a claim that AlgoMonster guarantees at least one offer or your money back. [13] AlgoMonster's own subscribe page doesn't state that term in the versions I could check, so treat the guarantee as unconfirmed and read the current checkout terms yourself before buying on that basis.

**How long does it take?** Reviewer estimates put the curriculum at 4 to 8 weeks at one to two hours a day. [9] Whether that's accurate for you depends mostly on how much of the pattern-identification work you do before looking at the solution.

## The short version

Big O is a growth rate, not a stopwatch. Learn the eight classes and their shape, apply the four rules — add sequential blocks, multiply nested ones, drop constants, drop lower-order terms — and then use the constraints in a problem to sanity-check the complexity you're aiming for. The halving-loop, base-of-log, and amortized-array cases are what separate people who can recite the definition from people who can analyze code they've never seen. That last skill is the one that gets tested.

If the reading is enough to make the concepts click, you're done. If you want the parts that require reps — reading a loop and knowing instantly that it's `O(n log n)` — that's practice, and 👉 [AlgoMonster's lesson library](https://algo.monster?fpr=li69) is built around exactly that kind of deliberate repetition rather than volume for its own sake.
