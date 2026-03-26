# aitongl.github.io

Initial Proposal

Parallel Maximum Satisfiability Solver
URL: https://aitongl.github.io/
Aitong(Ivy) Li
Ananya Aggarwal

SUMMARY
We will build a parallel Maximum Satisfiability (MaxSAT) solver by starting from a sequential CDCL SAT solver and implementing a core-guided MaxSAT algorithm on top of it. Our system will expose solver internals such as learned clauses and conflicts, and then introduce parallelism at the MaxSAT level by coordinating multiple workers. Workers will explore different parts of the search while selectively sharing useful information such as unsatisfiable cores, bounds, and high-quality learned clauses to reduce redundant computation.

BACKGROUND
​​Maximum Satisfiability (MaxSAT) extends SAT by allowing some clauses to be “soft” and seeking an assignment that maximizes the number of satisfied clauses. 

Maximum satisfiability is a NP-complete problem, which means solving it efficiently could lead to efficient solvers to many real world problems. For instance, the problem of the traveling salesman can be solved using maximum satisfiability. We want to design a parallel MaxSAT using communication protocols such as OpenMP. This will be different from most of the parallel MaxSAT solvers because we want to experiment with various communication protocols.
At a high level, the sequential algorithm proceeds as follows:
Initialize bounds and relaxation variables
Repeat:
Call SAT solver under current assumptions
If satisfiable: update best solution and bounds
If unsatisfiable: extract unsat core and relax it
Until convergence
The computational bottleneck is the repeated SAT solving, which is expensive and often revisits similar parts of the search space. This makes MaxSAT a good candidate for parallelism. However, unlike SAT, the algorithm has structure across iterations, meaning parallel workers must coordinate rather than operate independently.
Parallelism can be introduced by running multiple workers that explore different bounds, partitions of soft clauses, or relaxation strategies. The key opportunity is to share useful information such as UnSAT cores or strong learned clauses so that workers avoid redundant work.

THE CHALLENGE
The main challenge is that MaxSAT has strong dependencies across iterations. Each SAT call depends on previously learned cores and updated constraints, making naive parallelization ineffective. Simply running multiple SAT instances independently leads to redundant work and poor scaling.
Another challenge is deciding what information to share. There is a distinction between learned conflict clauses (internal to the SAT solver) and UnSAT cores (used by the MaxSAT algorithm). Sharing too many clauses can overwhelm the solver and increase communication overhead, while sharing too little reduces the benefit of parallelism. Designing selective sharing policies is therefore critical.
The workload consists of repeated SAT solves with irregular control flow. Memory access patterns are not strictly regular but benefit from locality within each SAT instance. Communication introduces overhead and must be balanced against computation. Additionally, different workers may progress at different rates, leading to load imbalance.
From a systems perspective, mapping this workload efficiently requires managing synchronization, limiting communication overhead, and ensuring that shared information improves convergence rather than degrading performance.

RESOURCES
We will use starter code from open sourced project Glucose, and we will specifically be using their version of a sequential SAT solver and frameworks for computing conflict sets. We will modify the solver to expose internals such as learned clauses, conflict information, and assumption handling, which are necessary for implementing a core-guided MaxSAT algorithm. This website contains a description of the Glucose project and link to their code base. Additionally, we hope to use these industry standard benchmarks to test and develop our approach.
We do not believe we need any more resources that we did for the previous assignments. The PSC nodes will be sufficient for what we intend to do within our project.
We will reference standard literature on core-guided MaxSAT and CDCL solvers to guide our implementation.

GOALS AND DELIVERABLES
Core Goals
Implement a baseline parallel MaxSAT solver by running multiple SAT solver instances in parallel on different parts of the search space.
Design and integrate at least one communication protocol (e.g., clause sharing or conflict sharing) between parallel workers to reduce redundant computation.
Evaluate performance on standard benchmark instances (e.g., SATLIB / MaxSAT benchmarks) using PSC cluster resources.
Measure:
Speedup vs. sequential baseline
Scalability across cores/processors
Impact of communication frequency on performance
Extended Goals
Compare multiple communication strategies, such as:
No communication (independent workers)
Periodic clause sharing
Adaptive or selective clause sharing
Implement simple load balancing or work-stealing to improve utilization.
Explore heuristics for deciding what information to share (e.g., only high-quality learned clauses). 
Minimum Deliverable (if progress is slower)
A correct sequential core-guided MaxSAT solver and a basic parallel version with independent workers and limited communication, along with initial performance evaluation.
What We Aim to Learn
This project is intended to answer several systems and algorithmic questions about parallel MaxSAT. First, we want to understand how much of the MaxSAT workload can be parallelized given its iterative and dependency-heavy structure. Second, we aim to determine which types of shared information are most effective, specifically whether unsat cores provide more benefit than generic learned clauses. Third, we want to characterize the tradeoff between communication frequency and performance, identifying when communication helps versus when it becomes a bottleneck.
We also aim to understand how solver internals influence parallel performance. By exposing and controlling internal artifacts such as learned clauses, we can study how low-level solver behavior interacts with high-level parallel coordination.
System Capabilities and Expected Performance
The final system will be capable of solving MaxSAT instances using a coordinated parallel architecture, where multiple workers contribute to a shared optimization process. It will support configurable communication policies, allowing experimentation with different sharing strategies.
We expect the system to outperform the sequential baseline on moderate to large benchmark instances, particularly when communication is tuned appropriately. However, we do not expect linear scaling due to synchronization and dependency constraints. Instead, the contribution of the project will be demonstrating how careful use of solver internals and selective communication can yield practical speedups in a setting that is not trivially parallelizable.

PLATFORM CHOICE
We will use C++ and build on the Glucose codebase, which provides an efficient sequential CDCL SAT solver. This allows us to access solver internals and implement custom sharing mechanisms. We will use shared-memory parallelism (OpenMP or threads) for intra-node parallelism and run experiments on PSC machines to evaluate scalability.
This platform is appropriate because the workload involves fine-grained control over solver behavior and benefits from low-overhead communication between threads.

SCHEDULE
Week 1:
Project scoping
Explore code base and understand what Glucose does 
Set up a simple parallel version of MaxSAT by running multiple independent solver instances (no communication) 
Week 2:
Design first communication strategy for clause sharing
Benchmark performance and experiment with at least 3 different strategies 
Identify bottlenecks by profiling performance
Week 3:
Tune sharing frequency / clause selection
Evaluate impact on performance
Produce intermediate results + milestone write-up
Week 4:
If enough bandwidth, explore other communication frameworks
Experiment with high thread count on PSC nodes and continue to tune sharing strategy and hyperparameters
Week 5:
Produce final writeup and graphs for performance documenting


