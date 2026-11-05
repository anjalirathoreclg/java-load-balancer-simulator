# Load Balancer Simulator — Java DSA project

A console simulation with handwritten data structures and commented code.
It simulates request scheduling; it does not distribute real network traffic.

## Run in VS Code

1. Install a JDK (17 recommended) and VS Code's Extension Pack for Java.
2. Extract this ZIP and open the `LoadBalancerProject` folder in VS Code.
3. Open Terminal → New Terminal. Check `java -version` and `javac -version`.
4. Compile and run:

```bash
javac *.java
java Main
```

Run the verification suite with `java ProjectTests` after compiling.
No Maven, Gradle, packages, or external dependencies are needed.

## Files and learning order

1. `Models.java`: Request, Server, statuses, algorithms and immutable event records.
2. `Structures.java`: custom dynamic array, FIFO queue, linked stack, hash table and binary min heap.
3. `Simulator.java`: scheduling, processing ticks, failure/recovery and metrics.
4. `Main.java`: input validation, menu and replay comparison.
5. `ProjectTests.java`: checks for containers and important simulation rules.

The classes are in separate files for readability; compile all files together.
`Scanner` is used only for input. No predefined Java collection classes are used.
The generic `SimpleList<T>` lets the same handwritten dynamic array hold servers,
requests or events. Its internal Object array requires one checked-boundary cast.

## Rules

- Capacity = number of concurrent running requests, not processing speed.
- Every running request processes one unit per tick. Capacity 2 therefore means
  two requests can each process one unit during the same tick.
- Arrivals and recovered capacity trigger assignment automatically. A request
  assigned at time t begins processing in the next tick.
- Round Robin skips offline/full servers and advances past the chosen server.
- Least Load minimizes sum of remaining request times; smaller numeric server ID
  wins ties. The custom heap is rebuilt after every assignment and load change.
- Failed requests preserve arrival time and remaining time. They return to the
  REAR of the normal FIFO queue, after existing waiting requests. Returned requests
  are sorted by original arrival time, then case-sensitive ID for simultaneous arrivals.
- IDs are case-sensitive, trimmed, nonempty and never reused, even after completion.
- Completion stack displays newest first without removing records. Completions in
  the same tick have deterministic order from server/list traversal.
- Turnaround = completion - arrival. Waiting = turnaround - total processing.
- Maximum queue length is sampled after automatic assignment: requests still waiting.
- Average metrics include completed requests only. No completed requests means N/A.
- Each comparison uses fresh objects, the original capacities, and the same valid
  ADD/TIME/FAIL/RECOVER events. It drains remaining work after the last event.
- If all servers remain offline, draining stops and reports incomplete results.
  It does not invent a recovery or declare a winner. The live run stays unchanged.
- Terminal bounds: 1–100 servers, 1–100 capacity, 1–100000 processing time/ticks.
- Data is in memory for one session; there is no save/load feature.

## Demo

Choose 2 servers, both capacity 1, Round Robin. Add A with duration 3, B with
duration 2 and C with duration 1. Display servers and queue. Advance 1 tick,
fail server 1, then view the queue: C stays before returned A. Recover server 1,
advance 5 ticks, display completed history and report. Search A, then compare
both algorithms. Try reusing A's ID and observe rejection.

## DSA explanation and complexity

| Structure | Purpose | Cost |
|---|---|---|
| Dynamic array | Servers, running requests, event history | Access O(1), append amortized O(1), removal O(n) |
| Linked queue | Waiting requests, including returned failures | Enqueue/dequeue O(1) |
| Linked stack | Latest completion first | Push O(1), display O(n) |
| Hash table | Find request by ID; prevent duplicates | Expected short-chain lookup; worst O(n), plus ID hashing |
| Binary min heap | Least-loaded eligible server | Minimum O(1), insertion O(log s) |

The hash table uses 101 fixed buckets with separate chaining. Heavy workloads
can create longer chains. Heap rebuilding by insertion costs O(s log s), and
does not give an overall performance advantage over a simple O(s) server scan.
We deliberately use rebuilding to keep mutable-load handling easy to understand.
Repeated pending assignments repeat this rebuild. This is a learning project,
not a claim of an optimized production load balancer.

## Suggested four-person split

- Person 1: models, dynamic array and normal queue.
- Person 2: linked stack and request hash table.
- Person 3: min heap, scheduling and processing ticks.
- Person 4: failure/recovery, menu, metrics and replay comparison.

Everyone should understand the end-to-end flow and capacity/failure rules.

## New: Auto mode and autoscaling

Choose 3 (Auto) at startup, or choose either manual algorithm as before.
Auto chooses once before each nonempty pending assignment batch. It considers
remaining durations of pending and online running requests. A maximum/minimum
ratio of at least 2:1 selects Least Load. Otherwise a load spread of at least
two units across online servers selects Least Load. Otherwise it selects Round
Robin. These are simple classroom heuristics, not a guarantee of optimality.
The terminal prints the selected algorithm and reason; reports show the last choice.

Menu 11 configures autoscaling (initially OFF). ON requires a maximum total
server count and the capacity for newly added servers. When pending work cannot
fit on any online server, add a server and continue assigning, repeating only
up to the limit. This also permits adding a server when all old servers are
offline. The limit counts offline servers too. OFF stops future additions;
existing added servers stay. There is no scale-down. Server IDs remain unique
and can be used in failure and recovery options.

Comparison replays the same scaling ON/OFF settings and workload events for
Round Robin and Least Load. It reports total and added servers as well as time
and waiting metrics, since using more servers has a resource cost. Auto is a
heuristic mode; this comparison evaluates the two fixed algorithms.
If a server-specific event cannot replay (for example a scaled server has not
been created in one algorithm), or a later limit is below one replay's server
count, comparison is unavailable and explains why. It never invents servers
or changes the live run. For a comparable demo, keep one scaling limit and
fail/recover only initial servers.
