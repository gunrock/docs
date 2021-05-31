# Application Classification

The [Phase 1 writeup]((../hive/hive_application_classification.md)) contains a detailed description of the application.

From the Phase 1 writeup:

> The application classification (AC) workflow is an implementation of probabalistic graph matching via belief propagation.  The workflow takes two node- and edge-attributed graphs as input -- a data graph `G = (U_G, E_G)` and a pattern graph `P = (U_P, E_P)`.  The goal is to find a subgraph `S` of `G` such that the dissimilarity between the node/edge features of `P` and `S` is minimized. The matching is optimized via loopy belief propagation, which consists of iteratively passing messages between nodes then updating beliefs about the optimal match.

## Summary of Results

We re-forumlate the `application_classification` workload to improve memory locality and admit a natural multi-GPU implementation.  We then parallelized the core computational region of `application_classification` across GPUs.  For the kernels in that region that do not require communication between GPUs, we attain near-perfect scaling.  Runtime of the entire application remains bottlenecked by network bandwidth between GPUs.  However, mitigating this bottleneck should be possible further optimization of the memory layout.

## Summary of Implementation

The Phase 1 single-GPU implementation is [here](../hive/hive_application_classification.md).

`application_classification` consists of two regions:
  - Region 1: initialization of distance and feature matrices
  - Region 2: iterative loop consisting of of message passing operations and matrix normalization operations

Region 2 accounts for the majority of runtime.  For example, in our single-GPU implementation running on the `rmat18` `application_classification` benchmark dataset, Region 1 takes 37ms (20% of runtime) and Region 2 takes 157ms (80% of runtime).  As such, we focused on parallelizing Region 2 across GPUs.  A multi-GPU implementation of Region 1 would also be possible, but with diminishing returns.

Upon examination of the Phase 1 `application_classification` implementation, we determined that most of the matrices could be transposed to attain better memory locality.  In the original implementation, there were a number of column-wise operations (max reduce on columns; softmax normalization of columns).  Transposing these matrices converts these into row-wise operations, and yields a substantial speedup.  For example, on the `rmat18` benchmark dataset, this reformulation yields a 6.44x speedup on a single GPU.

"Transposing" the problem also makes it more suitable for multi-GPU parallelism, via row-wise chunking of the data matrices.  Chunks are manually scattered across GPUs using `cudaMemcpy`.  Most of the kernels in Region 2 require _no_ communication between GPUs, which leads to good scaling.  The small amount of communication that is required is done by enabling peer access, with remote memory loads / stores happening over NVLink.

Because it is not a canonical graph workload, `application_classification` is written outside of Gunrock using the `thrust` and `cub` libraries (as in HIVE Phase 1).

## How To Run This Application on DARPA's DGX-1

### Prereqs/input

__Note:__ Serban to edit.

```
# get code
git clone https://github.com/bkj/application_classification
cd application_classification
git checkout 40c38a343ac1662d3abf82fcfa86b6c720368c62

# prep binary input data
python prob2bin.py 

# build
mkdir -p results
make clean
make -j12

# run
CUDA_VISIBLE_DEVICES=0       ./main data/data.bin data/pattern.bin > results/cuda_result1
CUDA_VISIBLE_DEVICES=0,1     ./main data/data.bin data/pattern.bin > results/cuda_result2
CUDA_VISIBLE_DEVICES=0,1,2   ./main data/data.bin data/pattern.bin > results/cuda_result3
CUDA_VISIBLE_DEVICES=0,1,2,3 ./main data/data.bin data/pattern.bin > results/cuda_result4
```

### Partitioning the input dataset

Partitioning is done automatically inside the code.

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

No change from Phase 1.

## Performance and Analysis

No change from Phase 1.

### Implementation limitations

Performance limitations regarding the size of the data matrices are mitigated by the multi-GPU approach -- with this implementation, the maximum size of a problem instance should theoretically scale linearly with the number of GPUs.  Practically, the current implementation still does Region 1 on a single GPU, which would create a bottleneck in terms of available memory.

Other performance limitations remain the same as in Phase 1.

### Performance limitations

From the perspective of a single GPU, there is no change from Phase 1.

From the perspective of the multi-GPU system, we are primarily bottlenecked by bandwidth across the NVLink network, which impacts both the runtime of the row scatter operation and the Region 2 kernels that require communication.  This could be (partially) mitigated by additional optimizations -- more details below.

## Scalability behavior

| GPUs | Runtime/Region 1 (ms) | Runtime/Scatter (ms) | Runtime/Region 2 (ms) |
|------|-----------------------|----------------------| ----------------------| 
| 1    |                       |                      |                       |
| 2    |                       |                      |                       |
| 3    |                       |                      |                       |
| 4    |                       |                      |                       |
| 5    |                       |                      |                       |
| 6    |                       |                      |                       |
| 7    |                       |                      |                       |
| 8    |                       |                      |                       |

Scaling of the whole workload's runtime is not ideal, primarily because:
  a) because Region 1 is not parallelized across GPUs
  b) because scattering the rows of the matrices across GPUs takes time.

Region 1 would be relatively straightforward to distribute across GPUs.  The runtime of the scatter could also be reduced via asynchronous memory copies (possibly launched from multiple CPU threads).

The scalability of Region 2 is limited by the couple of kernels that require communication between GPUs, which take ~5x longer to run w/ 4 GPUs than on a single GPU.  Currently, we're bottlenecked by the bandwidth into GPU0 -- scattering an additional datastructure across GPUs would reduce this load by a factor of `num_gpus`, and provide further speedup.  However, this is slightly more complex than the current method, and has not yet been implemented.