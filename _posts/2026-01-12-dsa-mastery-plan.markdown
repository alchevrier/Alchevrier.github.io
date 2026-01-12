---
layout: post
title:  "DSA Mastery Plan - 6 Month Roadmap to Leetcode Proficiency"
date:   2026-01-12 10:30:00 +0800
categories: learning algorithms dsa leetcode
published: false
---

## Philosophy
**Quality over Quantity.** Focus on understanding patterns deeply rather than rushing through problems. Target: 200-250 problems with deep comprehension.

---

## Month 1: Foundations (Arrays, Strings, Hash Tables)

### Week 1: Arrays & Two Pointers
**Concepts:**
- Array traversal and manipulation
- Two pointers technique (opposite ends, same direction)
- Sliding window basics

**Problems (15 total):**

*Easy (10):*
- [x] Two Sum (LeetCode 1) - Hash table intro
- [x] Best Time to Buy and Sell Stock (121)
- [x] Remove Duplicates from Sorted Array (26)
- [ ] Merge Sorted Array (88)
- [x] Move Zeroes (283)
- [ ] Squares of a Sorted Array (977)
- [ ] Valid Palindrome (125)
- [ ] Intersection of Two Arrays II (350)
- [ ] Plus One (66)
- [ ] Majority Element (169)

*Medium (5):*
- [ ] Container With Most Water (11) - Two pointers
- [ ] 3Sum (15) - Critical pattern
- [ ] Sort Colors (75) - Dutch national flag
- [ ] Product of Array Except Self (238)
- [ ] Subarray Sum Equals K (560) - Prefix sum

**Daily Schedule:** 2 Easy (Mon-Fri), 1 Medium (Sat-Sun)

**Pattern Recognition:**
- When to use two pointers vs hash table
- Prefix sum for subarray problems
- In-place array manipulation

---

### Week 2: Strings & Sliding Window
**Concepts:**
- String manipulation techniques
- Sliding window (fixed & variable size)
- Character frequency patterns

**Problems (15 total):**

*Easy (10):*
- [ ] Valid Anagram (242)
- [ ] First Unique Character (387)
- [ ] Reverse String (344)
- [ ] Reverse Words in a String III (557)
- [ ] Longest Common Prefix (14)
- [ ] Implement strStr() (28)
- [ ] Valid Palindrome II (680)
- [ ] Isomorphic Strings (205)
- [ ] Length of Last Word (58)
- [ ] Excel Sheet Column Number (171)

*Medium (5):*
- [ ] Longest Substring Without Repeating Characters (3) - **KEY**
- [ ] Longest Repeating Character Replacement (424)
- [ ] Minimum Window Substring (76) - **HARD but important**
- [ ] Group Anagrams (49)
- [ ] Longest Palindromic Substring (5)

**Pattern Recognition:**
- Fixed vs variable sliding window
- Character frequency using arrays (size 26 for lowercase)
- When to expand vs shrink window

---

### Week 3-4: Hash Tables & Sets
**Concepts:**
- Hash table operations (O(1) lookup)
- Hash sets for deduplication
- Frequency counting patterns

**Problems (20 total):**

*Easy (12):*
- [ ] Contains Duplicate (217)
- [ ] Single Number (136) - XOR trick
- [ ] Intersection of Two Arrays (349)
- [ ] Happy Number (202)
- [ ] Word Pattern (290)
- [ ] Missing Number (268)
- [ ] Find All Numbers Disappeared (448)
- [ ] Distribute Candies (575)
- [ ] Jewels and Stones (771)
- [ ] Unique Email Addresses (929)
- [ ] Uncommon Words (884)
- [ ] Subdomain Visit Count (811)

*Medium (8):*
- [ ] Top K Frequent Elements (347) - Bucket sort
- [ ] Group Shifted Strings (249)
- [ ] Valid Sudoku (36)
- [ ] Find Duplicate File in System (609)
- [ ] Brick Wall (554)
- [ ] 4Sum II (454)
- [ ] Contiguous Array (525) - Prefix sum + hash
- [ ] Subarray Sum Divisible by K (974)

**Key Patterns:**
- When hash table is better than sorting
- Using hash for O(1) complement lookup
- Frequency counting for optimization

---

## Month 2: Linear Data Structures

### Week 5: Linked Lists
**Concepts:**
- Single/double linked list operations
- Fast & slow pointers (Floyd's cycle detection)
- Dummy node technique
- In-place reversal

**Problems (18 total):**

*Easy (10):*
- [ ] Reverse Linked List (206) - **FUNDAMENTAL**
- [ ] Merge Two Sorted Lists (21)
- [ ] Linked List Cycle (141)
- [ ] Remove Linked List Elements (203)
- [ ] Middle of Linked List (876)
- [ ] Palindrome Linked List (234)
- [ ] Intersection of Two Linked Lists (160)
- [ ] Remove Duplicates from Sorted List (83)
- [ ] Delete Node in a Linked List (237)
- [ ] Convert Binary Number in Linked List (1290)

*Medium (8):*
- [ ] Add Two Numbers (2)
- [ ] Remove Nth Node From End (19) - Two pointers
- [ ] Reorder List (143) - Multiple techniques
- [ ] Linked List Cycle II (142) - Floyd's algorithm
- [ ] Copy List with Random Pointer (138)
- [ ] Sort List (148) - Merge sort on linked list
- [ ] Odd Even Linked List (328)
- [ ] Rotate List (61)

**Master These:**
- Reversing linked list (iterative & recursive)
- Fast/slow pointer for cycle detection
- Dummy node for edge cases

---

### Week 6: Stacks & Queues
**Concepts:**
- Stack (LIFO) applications
- Queue (FIFO) applications
- Monotonic stack/queue
- Deque for optimization

**Problems (18 total):**

*Easy (10):*
- [ ] Valid Parentheses (20) - **CLASSIC**
- [ ] Min Stack (155)
- [ ] Implement Queue using Stacks (232)
- [ ] Implement Stack using Queues (225)
- [ ] Backspace String Compare (844)
- [ ] Baseball Game (682)
- [ ] Next Greater Element I (496)
- [ ] Remove Outermost Parentheses (1021)
- [ ] Remove All Adjacent Duplicates (1047)
- [ ] Build Array With Stack Operations (1441)

*Medium (8):*
- [ ] Daily Temperatures (739) - Monotonic stack
- [ ] Evaluate Reverse Polish Notation (150)
- [ ] Decode String (394)
- [ ] Asteroid Collision (735)
- [ ] Online Stock Span (901)
- [ ] Next Greater Element II (503)
- [ ] Simplify Path (71)
- [ ] Validate Stack Sequences (946)

**Patterns:**
- Monotonic stack for "next greater/smaller"
- Stack for expression evaluation
- Queue for BFS (coming in graphs)

---

## Month 3: Trees & Recursion

### Week 7-8: Binary Trees & BST
**Concepts:**
- Tree traversals (inorder, preorder, postorder)
- Recursion patterns
- BST properties
- Level-order traversal (BFS)

**Problems (25 total):**

*Easy (15):*
- [ ] Maximum Depth of Binary Tree (104)
- [ ] Same Tree (100)
- [ ] Invert Binary Tree (226)
- [ ] Symmetric Tree (101)
- [ ] Path Sum (112)
- [ ] Merge Two Binary Trees (617)
- [ ] Diameter of Binary Tree (543)
- [ ] Balanced Binary Tree (110)
- [ ] Minimum Depth of Binary Tree (111)
- [ ] Binary Tree Paths (257)
- [ ] Sum of Left Leaves (404)
- [ ] Average of Levels (637)
- [ ] Search in BST (700)
- [ ] Two Sum IV - BST (653)
- [ ] Subtree of Another Tree (572)

*Medium (10):*
- [ ] Validate BST (98) - **CRITICAL**
- [ ] Kth Smallest Element in BST (230)
- [ ] Lowest Common Ancestor of BST (235)
- [ ] Lowest Common Ancestor of Binary Tree (236)
- [ ] Binary Tree Level Order Traversal (102) - BFS
- [ ] Binary Tree Zigzag Level Order (103)
- [ ] Binary Tree Right Side View (199)
- [ ] Construct Binary Tree from Preorder and Inorder (105)
- [ ] Path Sum II (113)
- [ ] Flatten Binary Tree to Linked List (114)

**Master These:**
- All three traversals (recursive & iterative)
- BST validation and operations
- Level-order traversal with queue
- Tree recursion patterns

---

## Month 4: Graphs & Advanced Structures

### Week 9: Graph Traversal (DFS/BFS)
**Concepts:**
- Graph representations (adjacency list/matrix)
- DFS (stack/recursion)
- BFS (queue)
- Topological sort
- Union Find basics

**Problems (20 total):**

*Easy (8):*
- [ ] Find Center of Star Graph (1791)
- [ ] Find if Path Exists (1971)
- [ ] Find Town Judge (997)
- [ ] Employee Importance (690)
- [ ] N-ary Tree Preorder (589)
- [ ] N-ary Tree Postorder (590)
- [ ] N-ary Tree Level Order (429)
- [ ] Maximum Depth of N-ary Tree (559)

*Medium (12):*
- [ ] Number of Islands (200) - **FUNDAMENTAL**
- [ ] Clone Graph (133)
- [ ] Course Schedule (207) - Topological sort
- [ ] Course Schedule II (210)
- [ ] Pacific Atlantic Water Flow (417)
- [ ] Number of Connected Components (323)
- [ ] Graph Valid Tree (261)
- [ ] Walls and Gates (286)
- [ ] Rotting Oranges (994) - Multi-source BFS
- [ ] Word Ladder (127)
- [ ] All Paths From Source to Target (797)
- [ ] Keys and Rooms (841)

**Key Patterns:**
- When to use DFS vs BFS
- Cycle detection
- Topological sort for dependencies
- Multi-source BFS

---

### Week 10: Heaps & Priority Queues
**Concepts:**
- Min/max heap operations
- Priority queue applications
- K-way merge pattern
- Top K problems

**Problems (16 total):**

*Easy (6):*
- [ ] Kth Largest Element in Stream (703)
- [ ] Last Stone Weight (1046)
- [ ] Relative Ranks (506)
- [ ] Min Cost to Connect Sticks (1167)
- [ ] Sort Array by Increasing Frequency (1636)
- [ ] Third Maximum Number (414)

*Medium (10):*
- [ ] Kth Largest Element (215) - Quick select too
- [ ] Top K Frequent Elements (347) - Revisit
- [ ] K Closest Points to Origin (973)
- [ ] Reorganize String (767)
- [ ] Task Scheduler (621)
- [ ] Find K Pairs with Smallest Sums (373)
- [ ] Kth Smallest Element in Sorted Matrix (378)
- [ ] Merge K Sorted Lists (23) - **IMPORTANT**
- [ ] Ugly Number II (264)
- [ ] Find Median from Data Stream (295) - Two heaps

**Patterns:**
- Top K problems → heap
- K-way merge → min heap
- Two heaps for running median

---

## Month 5: Dynamic Programming

### Week 11-12: DP Fundamentals
**Concepts:**
- Memoization (top-down)
- Tabulation (bottom-up)
- State definition
- Common patterns: Fibonacci, climbing stairs, min/max path

**Problems (25 total):**

*Easy (10):*
- [ ] Climbing Stairs (70) - **START HERE**
- [ ] Min Cost Climbing Stairs (746)
- [ ] House Robber (198)
- [ ] Best Time to Buy and Sell Stock (121)
- [ ] Divisor Game (1025)
- [ ] N-th Tribonacci Number (1137)
- [ ] Fibonacci Number (509)
- [ ] Pascal's Triangle (118)
- [ ] Pascal's Triangle II (119)
- [ ] Is Subsequence (392)

*Medium (15):*
- [ ] Coin Change (322) - **CLASSIC**
- [ ] House Robber II (213)
- [ ] Longest Increasing Subsequence (300)
- [ ] Longest Common Subsequence (1143)
- [ ] Unique Paths (62)
- [ ] Unique Paths II (63)
- [ ] Minimum Path Sum (64)
- [ ] Decode Ways (91)
- [ ] Word Break (139)
- [ ] Partition Equal Subset Sum (416) - 0/1 Knapsack
- [ ] Target Sum (494)
- [ ] Longest Palindromic Subsequence (516)
- [ ] Maximum Product Subarray (152)
- [ ] Jump Game (55)
- [ ] Jump Game II (45)

**Master Patterns:**
1. **1D DP:** Climbing stairs, house robber
2. **2D DP:** Unique paths, LCS
3. **Knapsack variants:** 0/1, unbounded, subset sum
4. **String DP:** LCS, palindrome, word break

---

## Month 6: Advanced Topics & Hard Problems

### Week 13-14: Advanced DP & Backtracking
**Concepts:**
- Backtracking template
- Pruning strategies
- DP optimization (space, time)
- Bit manipulation DP

**Problems (20 total):**

*Medium (12):*
- [ ] Permutations (46)
- [ ] Subsets (78)
- [ ] Combination Sum (39)
- [ ] Combination Sum II (40)
- [ ] Generate Parentheses (22)
- [ ] Letter Combinations of Phone Number (17)
- [ ] Palindrome Partitioning (131)
- [ ] N-Queens (51)
- [ ] Restore IP Addresses (93)
- [ ] Beautiful Arrangement (526)
- [ ] Partition to K Equal Sum Subsets (698)
- [ ] Longest Increasing Path in Matrix (329) - DP + DFS

*Hard (8):*
- [ ] N-Queens II (52)
- [ ] Sudoku Solver (37)
- [ ] Word Search II (212) - Trie + backtracking
- [ ] Regular Expression Matching (10)
- [ ] Wildcard Matching (44)
- [ ] Edit Distance (72) - **MUST KNOW**
- [ ] Distinct Subsequences (115)
- [ ] Interleaving String (97)

**Backtracking Template:**
```python
def backtrack(path, choices):
    if is_valid_solution(path):
        result.append(path.copy())
        return
    
    for choice in choices:
        if is_valid_choice(choice):
            make_choice(path, choice)
            backtrack(path, remaining_choices)
            undo_choice(path, choice)
```

---

### Week 15-16: Hard Problems & Interview Patterns
**Focus:** Company-specific patterns, system design adjacent problems

**Problems (20 total):**

*Medium (10):*
- [ ] LRU Cache (146) - **CRITICAL**
- [ ] LFU Cache (460)
- [ ] Design Twitter (355)
- [ ] Time Based Key-Value Store (981)
- [ ] Insert Delete GetRandom O(1) (380)
- [ ] Encode and Decode TinyURL (535)
- [ ] Design Search Autocomplete (642)
- [ ] Snapshot Array (1146)
- [ ] Design Underground System (1396)
- [ ] Design File System (1166)

*Hard (10):*
- [ ] Trapping Rain Water (42) - Two pointers/stack
- [ ] Longest Valid Parentheses (32)
- [ ] Merge K Sorted Lists (23) - Revisit
- [ ] Median of Two Sorted Arrays (4)
- [ ] First Missing Positive (41)
- [ ] Largest Rectangle in Histogram (84)
- [ ] Maximal Rectangle (85)
- [ ] Word Ladder II (126)
- [ ] Minimum Window Substring (76) - Revisit
- [ ] Sliding Window Maximum (239)

---

## Week 17-24: Consolidation & Mock Interviews

### Week 17-20: Pattern Mastery
**Daily routine:**
- 1 random Medium from weak areas
- 1 Hard from any topic
- Weekend: Review 3-5 previously solved

**Focus areas (based on gaps):**
- Identify your 3 weakest patterns
- Redo problems without looking at solutions
- Time yourself (35 min for Medium, 50 min for Hard)

### Week 21-24: Mock Interview Prep
**Schedule:**
- Mon/Wed/Fri: 1-2 new problems
- Tue/Thu: LeetCode contests (weekly/biweekly)
- Sat: Full mock interview (2-3 problems, 90 min)
- Sun: Review contest problems

**Mock Interview Platforms:**
- LeetCode Premium mock interviews
- Pramp
- interviewing.io

**Target Performance:**
- Medium: 20-25 minutes
- Hard: 40-45 minutes
- Explain approach before coding
- Handle follow-up questions

---

## Progress Tracking

### Monthly Milestones
- **Month 1:** 50 problems (35 Easy, 15 Medium)
- **Month 2:** 40 problems (20 Easy, 20 Medium)
- **Month 3:** 45 problems (25 Easy, 20 Medium)
- **Month 4:** 40 problems (15 Easy, 25 Medium)
- **Month 5:** 35 problems (10 Easy, 25 Medium)
- **Month 6:** 40 problems (5 Easy, 25 Medium, 10 Hard)

**Total: ~250 problems**

### Weekly Review Template
```markdown
## Week X Review (Date)
### Completed: X/Y problems
### Time spent: X hours
### Strengths this week:
- Pattern 1
- Pattern 2

### Struggles:
- Concept that needs review

### Next week focus:
- Topic to emphasize
```

---

## Resources

### Books
- "Grokking Algorithms" (you're reading) - Continue!
- "Cracking the Coding Interview" by Gayle McDowell
- "Elements of Programming Interviews" in your preferred language

### Online
- [NeetCode](https://neetcode.io/) - 150 curated problems with videos
- [LeetCode Patterns](https://seanprashad.com/leetcode-patterns/)
- [14 Patterns to Ace Any Coding Interview](https://hackernoon.com/14-patterns-to-ace-any-coding-interview-question-c5bb3357f6ed)

### Video Channels
- NeetCode (excellent explanations)
- Back to Back SWE
- Tushar Roy

### Study Groups
- Join LeetCode Discord
- AlgoExpert community
- Reddit r/leetcode

---

## Tips for Success

### 1. Problem-Solving Framework
```
1. Understand (5 min)
   - Read carefully, ask clarifying questions
   - Identify inputs, outputs, constraints
   - Work through examples

2. Plan (5-10 min)
   - Identify pattern/category
   - Consider multiple approaches
   - Analyze time/space complexity
   - Choose best approach

3. Code (15-20 min)
   - Write clean, readable code
   - Handle edge cases
   - Use meaningful variable names

4. Test (5 min)
   - Test with example cases
   - Consider edge cases
   - Dry run through code

5. Optimize (5 min)
   - Can you improve time complexity?
   - Can you reduce space usage?
```

### 2. When You're Stuck (15-minute rule)
- Spend 15 minutes trying to solve
- If stuck, read one hint or approach
- Try again for 15 minutes
- If still stuck, read solution **but understand it fully**
- Implement from scratch without looking
- Revisit in 3 days, then 1 week, then 2 weeks

### 3. Avoid These Mistakes
- ❌ Jumping to code immediately
- ❌ Not testing your solution
- ❌ Memorizing solutions instead of patterns
- ❌ Doing too many problems without understanding
- ❌ Skipping "easy" problems

### 4. Do These Instead
- ✅ Explain your approach out loud
- ✅ Write down the pattern/template
- ✅ Track time per problem
- ✅ Review problems after 1 week
- ✅ Focus on understanding, not speed (initially)

---

## Language-Specific Tips

### Python
- Use collections: `defaultdict`, `Counter`, `deque`
- List comprehensions for concise code
- `bisect` module for binary search
- `heapq` for heap operations

### Go
- Slice operations and capacity
- Map for hash tables
- `container/heap` for priority queues
- Understand pointer semantics

### Java
- `HashMap`, `HashSet` for hashing
- `PriorityQueue` for heaps
- `ArrayDeque` for stack/queue
- StringBuilder for string manipulation

---

## Success Metrics

### After 6 Months You Should:
- [ ] Solve most Easy problems in < 10 minutes
- [ ] Solve most Medium problems in 20-30 minutes
- [ ] Attempt Hard problems with structured approach
- [ ] Recognize patterns within 2-3 minutes
- [ ] Explain solutions clearly
- [ ] Optimize initial solutions
- [ ] Handle follow-up questions confidently

### Interview Readiness Checklist
- [ ] 200+ problems solved
- [ ] All major patterns mastered
- [ ] Can code without IDE autocomplete
- [ ] Comfortable with at least one language
- [ ] Practiced behavioral questions
- [ ] Done 10+ mock interviews
- [ ] Can discuss trade-offs

---

## Integration with Your Other Goals

**Synergy with Podman/DevOps:**
- Many problems have real-world applications
- Graph problems → dependency resolution, service mesh
- DP → resource optimization, caching strategies
- Design problems → system architecture

**Personal Projects:**
- Build LeetCode tracker (use DSA in practice)
- Implement your own data structures
- Contribute to open-source algorithm libraries

---

**Start Date:** 2026-01-13
**Target Completion:** 2026-07-13

Remember: **Consistency beats intensity.** 1-2 hours daily is better than 10 hours on weekends.

Track your progress in a spreadsheet or use LeetCode's built-in tracking. Update your blog with weekly reflections!

Good luck, Alex! You've got this. 🚀
