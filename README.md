# Hummingbird Proof of Work

A graph theoretic Proof of Work algorithm using adjacency lists and BAST search tree.

## What is Hummingbird?

Hummingbird is derived from [Cuckoo Cycle](https://github.com/tromp/cuckoo) but with a number of significant differences.

- It uses longer hashes, and they are considered as two halves in an [Adjacency List](https://en.wikipedia.org/wiki/Adjacency_list).

- Similarly, it searches for sets, of a specific size, of vectors that form a head-tail-head-tail chain that close to form a cycle

- The reference solver implementation uses a *dense* [Binary Search Tree](https://en.wikipedia.org/wiki/Binary_search_tree) of my own invention, which minimises the amount of memory required to store the tree as well as linearising the memory access patterns to be a lot more linear than a vector based binary search tree

The Cuckoo Cycle uses a relatively simple bucket sort algorithm repeatedly over a fixed length of hashchain members, and 'edge trims' by removing all of the ones that do not appear twice. Valid candidates for cycles must by definition only appear twice, and only once on the head and once on the tail of two separate vectors.

It seemed like this was an algorithm that could benefit from a faster search/sort process, as it is entirely possible that a solution appears long before all of the hashes are computed.

Furthermore, the proposed implementation of the data format for solutions to be embedded in the block header seemed like it could be shortened significantly by instead of storing the literal hash, only the position in the hashchain. Verification would require the verifier to regenerate the hashchain until the longest member in the list, but this still would be a lot faster to perform (and would stay entirely within a CPU cache) and thus allowing a much larger finite field generated in a hash chain to be explored.

A second feature of Hummingbird solution blocks that is important to note is that they will also encode the limit parameters set as consensus for the block production. These will be encoded in the most compact form possible, as bit counts for the maximum search depth (8 bits), and an 8 bit value encoding the required chain length in consensus at block production time. If a solution is found based on the same nonce in the future, if it is a shorter/equal depth *and* shorter/equal length it is rejected, because massive storage could store viable solutions without depth limit based on the same nonces and advantage miners with such storage. Smaller depths and lengths would not be very likely to indicate better candidate nonces.

By using [BAST](https://github.com/calibrae-project/bast) to track the search for a solution, solver rounds would be of a highly variable time, clustering around the statistical normal instead of a nearly uniform solution time as in Hashcash type PoW algorithms and Cuckoo and similar. They would still have an average time to finding a solution, but a very important difference is that the network can constrain, or expand, the time between solutions for difficulty adjustment with a combination of limiting the maximum verification cost (maximum length of the chain that can be searched) and the number of elements forming a cycle. The longer the maximum length the more likely a longer cycle, or more shorter cycles, and conversely, the shorter the maximum, the less likely.

Thus, the burden for the miner breaks down like this:

- A BAST tree storing the vectors generated, so for 64 bits, only 32 bits is needed to distinctly identify a vector, for keeping track of the position, plus 32 bits for the hashchain position.

- A pair of BAST trees that store as yet non-colliding candidates in their full 64 bit form

- A BAST tree with 32 bit nodes for storing coordinates (halves of the hashes) found to collide

- A two dimensional array storing the 64 bit hash vectors in lists that have not yet been found to close at the required length

- A BAST tree that stores the 32 bit half-hashes that exist within the solutions candidate table, with a 16 bit value storing the index of the solutions table

## Why BAST

BAST (Bifurcation Array Search Tree), due to it's linearity of node position in the progression of rows, acquires some of the advantages of conventional list sort algorithms but at the same time also the benefits of Binary Search Trees, which enable the lowest possible cost during search, and avoiding the inherent fixed delay period required for a linear array sort.

The main reason why this linear memory sort is desirable is because it greatly advantages modern CPUs with their large SRAM based on-die caches, which are the absolute fastest, lowest latency form of memory in production.

## Why not bitwidth for difficulty adjustment?

Because, without the need to store the literal whole solution in the solution segment of the header block, and the use of BAST to keep the hashchains sorted during insert and delete operations, we can allow solvers to produce solutions with a more stochastic time-to-solution time and so long as the floor and ceiling on the time to this is maintained by the network consensus to hit the block time target, solvers can spend more time searching bigger finite fields and this has an important benefit over time, because inevitably the whole field will be explored at some point in the future, forcing a change of strategy.

Meanwhile, network bandwidth and data storage capabilities will continue to increase, and thus Hummingbird will keep its utility for a much longer time, as well as naturally adjustable as CPU caches get bigger, RAM busses get faster and wider, and memory cells get faster and probably also increasingly parallelised.

Furthermore, the cost of these block headers in the storage requirement is not necessarily an issue as eventually Blockchain formats will be superseded for the live network and probably become relegated to an auditable historical record instead. So as Hummingbird solutions slowly grow longer over time, at some point Proof of Work will become obsolete as a means to eliminate collusion on verification, and will shift its function to a part of the business of archiving instead, which will have a different monetisation dynamic to validation and replication of data.

## Notes

Hummingbird has not been implemented yet, and is pending the completion of the development of BAST to begin implementation. Watch this space.