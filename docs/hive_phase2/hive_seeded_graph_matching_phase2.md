# Seeded Graph Matching (SGM)

Phase 1 report can be found [here](https://gunrock.github.io/docs/#/hive/hive_sgm).

From [Fishkind et al.](https://arxiv.org/pdf/1209.0367.pdf):

> Given two graphs, the graph matching problem is to align the two vertex sets so as to minimize the number of adjacency disagreements between the two graphs. The seeded graph matching problem is the graph matching problem when we are first given a partial alignment that we are tasked with completing.

That is, given two graphs `A` and `B`, we seek to find the permutation matrix `P` that maximizes the number of adjacency agreements between `A` and `P * B * P.T`, where `*` represents matrix multiplication.  The algorithm Fishkind et al. propose first relaxes the hard 0-1 constraints on `P` to the set of doubly stochastic matrices (each row and column sums to 1), then uses the Frank-Wolfe algorithm to minimize the objective function  `sum((A - P * B * P.T) ** 2)`.  Finally, the relaxed solution is projected back onto the set of permutation matrices to yield a feasible solution.

## Summary of Results

One or two sentences that summarize "if you had one or two sentences to sum up your whole effort, what would you say". I will copy this directly to the high-level executive summary in the first page of the report. Talk to JDO about this. Write it last, probably.

## Summary of Gunrock Implementation

The Phase 1 single-GPU implementation is [here](../hive/hive_yourworkflowname).

We parallelize across GPUs by ...

The multi-GPU implementation differs from the single-GPU implementation in the following way:

- A
- B
- C


Take as long as you need, but this might be short. Don't provide info that is already in the Phase 1 report.

### Differences in implementation from Phase 1

(If any.)

## How To Run This Application on DARPA's DGX-1

### Prereqs/input

(e.g., "build Gunrock's `dev-refactor` branch with hash X", "this particular dataset needs to be in this particular directory")

Include a github hash for the version you're using.

### Partitioning the input dataset

How did you do this? Command line if appropriate.

<code>
include a transcript
</code>

### Running the application

#### Datasets

Provide their names. We will probably make a separate page for them so you can just use their names.

#### Single-GPU (for baseline)

<code>
include a transcript
</code>

#### Multi-GPU

<code>
include a transcript
</code>

### Output

(Only include this if it's different than Phase 1. Otherwise: "No change from Phase 1.")

What is output when you run? Output file? JSON? Anything else? How do you extract relevant statistics from the output?

How do you make sure your output is correct/meaningful? (What are you comparing against?)

## Performance and Analysis

(Only include this if it's different than Phase 1. Otherwise: "No change from Phase 1.")

How do you measure performance? What are the relevant metrics? Runtime? Throughput? Some sort of accuracy/quality metric?

### Implementation limitations

(Only include this if it's different than Phase 1. Otherwise: "No change from Phase 1.")

e.g.:

- Size of dataset that fits into GPU memory (what is the specific limitation?)
- Restrictions on the type/nature of the dataset

### Comparison against existing implementations

(Delete this if there's nothing different from Phase 1.)

- Reference implementation (python? Matlab?)
- OpenMP reference

Comparison is both performance and accuracy/quality.

### Performance limitations

(Only include this if it's different than Phase 1. Otherwise: "No change from Phase 1.")

e.g., random memory access?

## Scalability behavior

**THIS IS REALLY THE ONLY IMPORTANT THING**

| GPUs | Runtime (ms) | Speedup over single-GPU version |
|------|--------------|---------------------------------|
| 1    |              |                                 |
| 2    |              |                                 |
| 3    |              |                                 |
| 4    |              |                                 |
| 5    |              |                                 |
| 6    |              |                                 |
| 7    |              |                                 |
| 8    |              |                                 |

Why is scaling not ideal?

What limits our scalability?