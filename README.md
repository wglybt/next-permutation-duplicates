# next permutation: the O(n) pivot-and-reverse walkthrough, with code and the duplicate case that breaks most solutions

You land on LeetCode 31, read the statement twice, and the four-line solution you find in the discussion tab looks like something someone invented by staring at a wall. Then you try to reproduce it from memory and it falls apart.

The good news: next permutation is not a clever trick. It is a very literal translation of what "next" means in lexicographic order. Once you see the suffix logic, you can derive the whole algorithm instead of memorizing it — and you can spot the one line where most implementations quietly go wrong.

## What the problem is actually asking for

A permutation is any arrangement of the array's members. Sort all of those arrangements in dictionary order and the next permutation is simply the one immediately after the current one. Two consequences fall out of that definition:

- It must be the **smallest** permutation strictly larger than the current one. Not any larger one.
- If you are already at the largest arrangement (the array is sorted descending), there is no next. Then the answer wraps around to the smallest arrangement, which is ascending order.

That second rule is where the wrap-around behavior in the examples comes from:

| Input | Next permutation | Reason |
| --- | --- | --- |
| `[1,2,3]` | `[1,3,2]` | smallest increase available |
| `[2,3,1]` | `[3,1,2]` | 2 is the only element that can grow |
| `[1,1,5]` | `[1,5,1]` | duplicates are allowed, and they don't get deduplicated |
| `[3,2,1]` | `[1,2,3]` | already the largest → wrap to ascending |
| `[4,5,2,6,3,1]` | `[4,5,3,1,2,6]` | swap 2 with 3, then reorder the tail |

The constraints matter too: modify the array in place, constant extra memory. That rules out "generate all permutations, sort them, find the current one, return the next" — that brute force is `O(n! · n)` time and `O(n!)` space, which is a fine way to *understand* the problem and a terrible way to solve it.

## Why the interesting part is always at the right edge

Look at any array and scan it from the right until the numbers stop descending. That suffix is non-increasing, which means it is already the **largest** arrangement of its own elements. Nothing you do inside a decreasing tail can make it bigger.

So any increase has to come from the element just before the tail — the leftmost position where the array still goes up. In `[1,2,5,4,3]`, scanning from the right: `4,3` descends, `5,4` descends, and then `2 < 5` stops you. The prefix `[1,2]` is the longest front you can share with the current arrangement, and position 1 is the only place you're allowed to change.

This gives you the design goal in one sentence: **change as little as possible, as far right as possible.** Keep the prefix, bump exactly one element, then make everything after it as small as it can be.

## The four steps

Using `[0,1,2,5,3,3,0]`:

1. **Find the pivot.** Walk from the right while `nums[i] >= nums[i+1]`. Here you stop at index 2, value `2`, because `2 < 5`.
2. **Find its successor.** Walk from the right again and take the first value greater than the pivot. Scanning `0, 3, 3, 5`, the first one above `2` is the `3` at index 5. Because the suffix is descending, this scan naturally lands on the *smallest* value that still beats the pivot.
3. **Swap them.** `[0,1,3,5,3,2,0]`. The array is now strictly larger, and the change is confined to one digit.
4. **Reverse the suffix.** Positions 3 onward are `5,3,2,0` — still descending, which means it's the biggest arrangement of those four values. Reverse it to get the smallest: `[0,1,3,0,2,3,5]`.

A longer one, if you want to trace it by hand: `8,3,9,6,4,7,5,2,1`. The pivot is `4` (first descent from the right is `4 < 7`). The smallest value in the tail greater than 4 is `5`. Swap → `8,3,9,6,5,7,4,2,1`. Reverse everything after the pivot position → `8,3,9,6,5,1,2,4,7`. That's the next permutation.

Step 4 is why the whole thing is `O(n)` instead of `O(n log n)`. Sorting the suffix would produce the same result, but you don't need to sort: after the swap the suffix is still in non-increasing order, so a two-pointer reversal is enough. If an interviewer asks "why not sort?", that's the answer they're fishing for.

## Working code

The exact same logic in four languages. Watch the comparison operators — they're the point of the next section.

**Python**

python
def next_permutation(nums: list[int]) -> None:
    n = len(nums)

    i = n - 2
    while i >= 0 and nums[i] >= nums[i + 1]:
        i -= 1

    if i >= 0:
        j = n - 1
        while nums[j] <= nums[i]:
            j -= 1
        nums[i], nums[j] = nums[j], nums[i]

    left, right = i + 1, n - 1
    while left < right:
        nums[left], nums[right] = nums[right], nums[left]
        left += 1
        right -= 1


**Java**

java
class Solution {
    public void nextPermutation(int[] nums) {
        int n = nums.length, i = n - 2;
        while (i >= 0 && nums[i] >= nums[i + 1]) i--;

        if (i >= 0) {
            int j = n - 1;
            while (nums[j] <= nums[i]) j--;
            int tmp = nums[i]; nums[i] = nums[j]; nums[j] = tmp;
        }

        for (int l = i + 1, r = n - 1; l < r; l++, r--) {
            int tmp = nums[l]; nums[l] = nums[r]; nums[r] = tmp;
        }
    }
}


**C++**

cpp
void nextPermutation(vector<int>& nums) {
    int n = nums.size(), i = n - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) --i;

    if (i >= 0) {
        int j = n - 1;
        while (nums[j] <= nums[i]) --j;
        swap(nums[i], nums[j]);
    }
    reverse(nums.begin() + i + 1, nums.end());
}


**JavaScript**

js
function nextPermutation(nums) {
  const n = nums.length;
  let i = n - 2;
  while (i >= 0 && nums[i] >= nums[i + 1]) i--;

  if (i >= 0) {
    let j = n - 1;
    while (nums[j] <= nums[i]) j--;
    [nums[i], nums[j]] = [nums[j], nums[i]];
  }

  for (let l = i + 1, r = n - 1; l < r; l++, r--) {
    [nums[l], nums[r]] = [nums[r], nums[l]];
  }
}


Time `O(n)`, extra space `O(1)`.

## Where implementations break

**Using `>` instead of `>=` in the pivot scan.** This is the most common bug and it only shows up with duplicates. Take `[3,1,2,2]`. The correct answer is `[3,2,1,2]`: pivot is `1`, swap with the last `2`, reverse the tail.

Now run the `>` version. The scan stops at index 2, because `2 > 2` is false, so the pivot becomes `2` at index 2. The successor search then walks all the way back to index 0 to find something bigger than 2, swapping `3` into the middle. You end up with `[2,1,2,3]` — a permutation that is *smaller* than the input. If your solution passes `[1,2,3]`, `[3,2,1]` and `[1,1,5]` but fails one hidden duplicate test, this is almost certainly why.

**Skipping the reverse when the pivot doesn't exist.** When the scan runs off the left edge, `i` is `-1` and the array is fully descending. You still need to reverse the whole thing — `[3,2,1]` must become `[1,2,3]`. Code that does `if (i < 0) return;` gets the wrap-around case wrong. Notice that in all four implementations above, `i + 1` becomes `0` when `i` is `-1`, so the reversal handles the edge case for free. That's not a hack, it's the definition of the wrap-around.

**Returning a new array.** The problem says in place. Copying into a fresh list is a correct answer to a different question, and it also spends `O(n)` extra memory.

**Reversing from the wrong index.** The suffix to reverse starts *after* the pivot, not at the pivot. Off-by-one here produces something that looks plausible on `[1,2,3]` and is wrong everywhere else.

## The built-in shortcut, and when it's fine

C++ has shipped this algorithm since forever: `std::next_permutation(first, last)` in `<algorithm>` transforms the range in place and returns `false` when it wrapped around to the smallest permutation, which makes it handy for enumerating every permutation of a range in a `do { } while (next_permutation(...))` loop. `prev_permutation` is the mirror image. The standard-library version handles duplicates correctly — the LeetCode editorial notes the two-scan approach is what the C++ STL uses.

Python's equivalent for *generating* permutations is `itertools.permutations`, though there's no in-place next-permutation helper. Java has neither.

Using the built-in in a real codebase is usually the right call. Using it on LeetCode 31 will get you a polite "let's implement it manually" — writing the four steps is the actual exercise.

## Problems that reuse the same idea

Next permutation rarely appears alone. The pivot-and-reverse move shows up in several relatives:

- **Next Greater Element III** — the same algorithm applied to the digits of an integer, plus an overflow check on the result.
- **Previous permutation** — the mirror: find the first ascent from the right, swap with the largest smaller element, reverse the suffix to descending.
- **K-th permutation** — a different technique (factorial number system), but it rests on the same "count how many permutations start with this prefix" intuition.
- **Permutations I and II** — backtracking, which is what most people reach for first. Worth knowing both, because the follow-up "now give me all of them" is common.

## Getting from "I can read this solution" to "I can produce it under pressure"

Reading the derivation above takes ten minutes. Recalling it in a 45-minute interview while someone watches you type is a different skill, and the gap between those two things is where most preparation time gets wasted. Grinding 300 random array problems does not close it. Recognizing "this is the descending-suffix problem" within the first two minutes does.

That recognition layer is the specific thing [👉 AlgoMonster's pattern catalogue](https://algo.monster?fpr=li69) is built around. The platform organizes its content by pattern rather than by problem number, with each pattern getting a written explanation, a code template, and a handful of mapped practice problems; the company describes the material as crafted by Google engineers and competitive programmers. Solutions are available in six languages including Python, Java, C++ and Go, and the site lists 500+ lessons across 6 courses with 1,000+ illustrations and 3,000+ editorials.

Two honest caveats before you consider paying anything.

First, there is **no video content** — everything is text and interactive diagrams. If you learn by watching someone solve a problem on screen, the format will feel dry, and plenty of reviewers say so.

Second, the problem bank is small by design. Third-party reviews put the catalogue at roughly 48 patterns and 231 problems, against LeetCode's 3,000-plus. AlgoMonster is a framework, not a volume trainer; most people who get value from it pair it with LeetCode rather than replacing it. Its system design and behavioral coverage is also thin, so senior candidates shouldn't expect it to cover a full L5 loop.

There's also a free tier with limited access to sample content. That's enough to check whether the topics you actually care about — array manipulation, greedy reasoning, two pointers — are covered the way you want, before you spend anything.

## AlgoMonster plans and current pricing

There's no monthly option. The choice is a year or a one-time payment:

| Plan | What you get | Price | Billing | Purchase |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of content and problems | $0 | — | [ Create a free account](https://algo.monster?fpr=li69) |
| Pro Yearly | All patterns, lessons and problems, unlimited Speedrun, unlimited AI assistant, company questions updated monthly, member discount on live bootcamps | $99/year (about $8.25/month) | Billed annually, 12 months of access | [ Check the current yearly price](https://algo.monster?fpr=li69) |
| Pro Lifetime | Everything in the annual plan plus permanent access and all future updates | $189 | One-time payment | [ See the lifetime price](https://algo.monster?fpr=li69) |

One thing worth knowing about these numbers: AlgoMonster's pricing moves with whatever promotion is running, and partner campaigns don't all show the same figure. The yearly plan has been displayed at $69.99 and at $99 on different official subscription pages, and the lifetime plan at $89 in one campaign and $189 in another. So treat the price on the checkout page as the real one, and if the number you see is higher than what you expected, it's worth waiting a few days rather than assuming you missed the deal forever.

The annual plan makes sense if you have interviews in the next few months and want a defined endpoint — the curriculum is built to be finished in roughly 4 to 8 weeks at an hour or two a day. The lifetime plan only wins the math if you expect to interview more than twice over the next several years. If you're prepping once and never touching it again, you're paying a premium for permanence you won't use. Several reviews also reference a job-guarantee policy; the terms live in AlgoMonster's own checkout and FAQ, so read those rather than a blog summary.

If your actual gap is "I can solve this once I know which pattern it is, but I don't see the pattern fast enough," a structured catalogue is a reasonable place to spend money. If your gap is volume and speed under a clock, a $99 subscription won't fix that, and you already know where to go.

## Quick answers

**Time and space complexity?** `O(n)` time, `O(1)` extra space — two right-to-left scans and one reversal, all in place.

**Does it work with duplicate values?** Yes, provided you compare with `>=` in the pivot scan and `> ` when hunting the successor. Duplicates don't get merged or removed; `[1,1,5]` really does produce `[1,5,1]`.

**What if the array is already sorted descending?** There is no larger permutation, so the answer is the ascending sort of the same elements, which the reversal produces for you.

**Can I just sort the suffix instead of reversing it?** It produces the correct output and costs `O(n log n)`. Reverse instead and explain why it's valid.

**Does it work on strings?** Same algorithm, character comparison instead of integer comparison. This is how you generate the lexicographically next string of a given character multiset.
