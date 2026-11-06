# Lab 07: The Integer-Valued Knapsack Problem

**Name:** Abishek Dhakal

## Introduction
In this lab, you will explore different variations of the Knapsack problem and design a dynamic programming solution for one of them. You may work in small groups or individually, but everyone should write up their own answers to the questions below. 

Suppose we're going hiking. We have items $1, \ldots, n$ that we could pack in multiples with corresponding values $v_1, \ldots, v_n$ and weights $w_1, \ldots, w_n$. For example, we might have a water bottle (weight 1, value 2) and a sandwich (weight 2, value 5). Consider the following three problem variations:

1. Maximize value subject to capacity $W$, where you can pack at most one of each item.
2. Maximize value subject to capacity $W$, where you can pack a positive integer value of each item.
3. Minimize weight subject to minimum value $V$, where you can pack a positive integer value of each item.

---

## Part 1: Dynamic Programming Fundamentals

**Question:** Which of the three problem variations can be solved using the dynamic programming approaches we've studied in lecture? Explain your reasoning.

I would use dynamic programming for all three variations. In each one, I can solve smaller problems first and save the answers in a table. That way, I do not have to calculate the same answer again. I am assuming the weights and values are positive whole numbers.

1. **Variation 1:** For each item, I would compare taking it with leaving it out. I would also keep track of which items I can still choose so I do not take the same item twice.
2. **Variation 2:** I would try each item that fits and use the table to find the best value for the space left over. Here, I can choose the same item again.
3. **Variation 3:** I would make a table showing the lowest weight needed to reach each required value. When I try an item, I would add its weight to the best answer for the value I still need. Then I would keep the choice with the lowest weight. Going above the required value is allowed.

I am reading variations 2 and 3 as allowing me to skip an item or take several copies, since that is how the given algorithm works.

## Part 2: The Naive Recursive Algorithm

Below is a naive recursive algorithm intended to solve one of the variations listed above.

```text
Function MaxValue(W):
    if W == 0:
        return 0
        
    max_val = 0
    for i from 1 to n:
        if w_i <= W:
            current_val = v_i + MaxValue(W - w_i)
            if current_val > max_val:
                max_val = current_val
                
    return max_val
```

**Question:** Which of the three problem variations (1, 2, or 3) does this algorithm solve? How do you know? Explain your reasoning.

This algorithm solves **variation 2**. The part that shows this is the loop over all the items in every call. After choosing an item, the algorithm calls itself with less space, but it can still choose that same item again. It keeps the choice that gives the highest value without going over the weight limit.

It would not work for variation 1 because it does not stop an item from being used again. It also does not solve variation 3 because it is looking for the highest value, while variation 3 asks for the lowest weight.

## Part 3: Recursion Tree

Consider the following simple example:
* **Knapsack Capacity:** $W = 3$
* **Item 1:** Weight $w_1 = 1$, Value $v_1 = 2$
* **Item 2:** Weight $w_2 = 2$, Value $v_2 = 5$
* **Item 3:** Weight $w_3 = 1$, Value $v_3 = 3$

**Question:** Draw the full recursion tree for `MaxValue(3)` using the example above. Label the remaining capacity at each node and indicate which item is chosen along each branch.

I started with capacity 3 and made a branch for each item that fits. After choosing an item, I subtracted its weight and repeated the same steps. In `MaxValue(c) = v`, `c` is the space left and `v` is the best value for that space. I included repeated calls because the recursive algorithm calculates them each time.

```text
MaxValue(3) = 9
├── item 1 (w=1, v=2) → MaxValue(2) = 6
│   ├── item 1 (w=1, v=2) → MaxValue(1) = 3
│   │   ├── item 1 (w=1, v=2) → MaxValue(0) = 0
│   │   └── item 3 (w=1, v=3) → MaxValue(0) = 0
│   ├── item 2 (w=2, v=5) → MaxValue(0) = 0
│   └── item 3 (w=1, v=3) → MaxValue(1) = 3
│       ├── item 1 (w=1, v=2) → MaxValue(0) = 0
│       └── item 3 (w=1, v=3) → MaxValue(0) = 0
├── item 2 (w=2, v=5) → MaxValue(1) = 3
│   ├── item 1 (w=1, v=2) → MaxValue(0) = 0
│   └── item 3 (w=1, v=3) → MaxValue(0) = 0
└── item 3 (w=1, v=3) → MaxValue(2) = 6
    ├── item 1 (w=1, v=2) → MaxValue(1) = 3
    │   ├── item 1 (w=1, v=2) → MaxValue(0) = 0
    │   └── item 3 (w=1, v=3) → MaxValue(0) = 0
    ├── item 2 (w=2, v=5) → MaxValue(0) = 0
    └── item 3 (w=1, v=3) → MaxValue(1) = 3
        ├── item 1 (w=1, v=2) → MaxValue(0) = 0
        └── item 3 (w=1, v=3) → MaxValue(0) = 0
```

At capacity 1, I could only choose item 1 or item 3 because item 2 is too heavy. At capacity 0, the call returns 0. The best choice in this example is three copies of item 3, with total weight 3 and value 9.


## Part 4: Recurrence Relation

**Question:** Derive a mathematical recurrence relation for $OPT(W)$, the maximum value you can pack into a knapsack of capacity $W$, based on the naive recursive algorithm. Be sure to state your base case(s) and explain how you arrived at your formula.

I use $OPT(c)$ for the best value I can get with capacity $c$. My base case is zero because I cannot pack anything when the capacity is zero:

$$
OPT(0)=0,
$$

When $c>0$, I compare the results from choosing each item that fits:

$$
OPT(c)=\max\left(\{0\}\cup\{v_i+OPT(c-w_i):1\leq i\leq n,\ w_i\leq c\}\right).
$$

The zero in the formula covers choosing nothing. If no item fits, the answer is 0.

I got this formula by following what the algorithm does. If I choose item $i$, I get its value $v_i$ and have $c-w_i$ space left and add the best value for that leftover space, which is $OPT(c-w_i)$. Then I take the largest answer from all the choices. Each item uses some space, so the calls eventually stop.


## Part 5: Bottom-Up Dynamic Programming

Instead of using the slow recursive approach, we can build a table bottom-up. 

**Question:** Use your recurrence relation from Part 4 to fill in the following table for the example provided in Part 3 ($w_1=1, v_1=2$ and $w_2=2, v_2=5$). At each step, record the optimal value and the specific item chosen to achieve that value. 

| Capacity ($W$) | Optimal Value ($OPT$) | Item Chosen |
|:---:|:---|:---|
| **0** | 0 | None (base case) |
| **1** | 2 | Item 1 |
| **2** | 5 | Item 2 |
| **3** | 7 | Item 1 (tie with item 2) |
| **4** | 10 | Item 2 |
| **5** | 12 | Item 1 (tie with item 2) |
| **6** | 15 | Item 2 |

Here, I used only items 1 and 2 here because those are the items listed in this question. I filled the table from capacity 0 up to 6, using the earlier rows to calculate each new row. In “Item Chosen,” I wrote the item to take first. When both choices gave the same value, I kept item 1, following the order in the algorithm.

For $W=6$, I checked both choices:

- Item 1: $v_1+OPT(6-w_1)=2+OPT(5)=2+12=14$.
- Item 2: $v_2+OPT(6-w_2)=5+OPT(4)=5+10=15$.

Choosing item 1 gives 14, while choosing item 2 gives 15. So 15 is the best value and item 2 is the choice.


## Part 6: Traceback

**Question:** Using the "Item Chosen" column from the table you completed in Part 5, trace back to find the exact quantities of each item to pack that maximize the value. What is the final optimal packing configuration for $W=6$ and the maximum value?

I started at capacity 6 and looked at the “Item Chosen” column. It says item 2, so then I took that item and subtracted its weight of 2. I repeated this at capacities 4 and 2:

| Remaining capacity | Item selected | Next capacity |
|:---:|:---:|:---:|
| 6 | 2 | 4 |
| 4 | 2 | 2 |
| 2 | 2 | 0 |

I stopped at capacity 0 and chose item 2 three times, so my final packing is **0 copies of item 1 and 3 copies of item 2**. The total weight is $3\cdot2=6$, and the maximum value is $3\cdot5=15$.
