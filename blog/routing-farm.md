<!--
Solving the k-Vehicle Routing Problem
By Adam Michael
-->

At the core of Pathfinder is the problem of finding optimal vehicle routes through a cluster. If you strip away some details (capacities, custom objective functions, etc.) and squint a little bit, this is the classical k-Vehicle Routing Problem (kVRP).

In this post, we will describe how Pathfinder solves kVRP.

## The problem
Given a set of vehicles, a set of locations needing service and a distance matrix over the vehicles and locations, find the assignment of locations to vehicles with an ordering per vehicle that minimizes the total vehicle distance traveled.

kVRP is known to be NP-Complete, which is a synonym for really-really-hard. There are a number of algorithms to approximate the solution. Several rely on assumptions about the problem such as the symmetric property of the distance matrix or the triangle inequality.

The Pathfinder problem is slightly more complicated than the traditional kVRP. In the Pathfinder problem, commodities have two locations: pickup and dropoff. Commodities much be picked up and dropped off by the same vehicle and cannot be dropped off before they are picked up.

## Linear Programming
The first approach we tried was linear programming (LP). We used Julia and the CLP solver. Linear programming works by specifying a set of integer variables and constraints on those variables as well as an objective function to minimize or maximize. CLP then uses the Simplex Method to find the optimal value.

The major benefit of linear programming is correctness. LP will always find the optimal route, regardless of how long it takes. The downside is that it will takes as long as necessary. We found out quickly that for any clusters with more than 10 commodities or vehicles, the time needed to compute a route is [unreasonable](https://github.com/CSSE497/pathfinder-routing/issues/1).

Our linear programming code is available on [GitHub](https://github.com/CSSE497/pathfinder-routing/tree/master/linearprogramming).

## Heuristic Search
When we discovered that linear programming could not handle large inputs, we started looking for other options. The search space of kVRP is exponential in the size of the input, so any kind of brute force search is off the table. So we turned to a heuristic-guided search. We used the [Optaplanner](http://www.optaplanner.org/) framework for optimization problems to implement A\* for kVRP. We set termination conditions so that after 7 seconds, the server returns the best-so-far route. A\* is a graph search algorithm that uses a heuristic function to determine how to move between solutions. In the case of kVRP, the heuristic function is the total distance of the cluster route.

The major benefit of heuristic search is that we can terminate the algorithm and return the best-found solution at any time. However, the drawback is that it cannot know if it has found the optimal solution until it has searched all the nodes.

Our heuristic search code is available on [GitHub](https://github.com/CSSE497/pathfinder-routing/tree/master/heuristicsearch).

## Simulated Annealing
Our benchmarks showed that the Optaplanner heuristic search implementation consistently returned less-than-optimal routes for medium to large clusters, so we turned to a statistical approach. Simmulated annealing is similar to A\* from above, however instead of using a deterministic heuristic function to guide the search, simulated annealing uses an "energy level" of the state plus a global temperature (that cools down over time) to probabilistically choose the next state for any iteration of the search.

We could not find a standard simulated annealing framework to use, so we wrote our from scratch based on the description in Russel and Norvig's *Artificial Intelligence: A Modern Approach*.

Our simulated annealing library is available on [GitHub](https://github.com/CSSE497/simulatedannealing). The webservice code is also available on [GitHub](https://github.com/CSSE497/pathfinder-routing/tree/master/simulatedannealing).

## Putting them all together
Each of the three approaches above has pros and cons. Instead of selecting one approach, Pathfinder uses all three and selects the best result returned after 10 seconds. Our webservice to select the best result is available on [GitHub](https://github.com/CSSE497/pathfinder-routing/tree/master/masterrouter).

At Pathfinder, we deploy all of our services to the Google Container Engine (GKE). GKE makes deployment and scaling horizontally a quick and easy process, so we are able to manage a "routing server farm" of containers running the linear programming, heuristic search and simulated annealing servers. The master routing server has a config file of all the worker routers. Adding a new routing server is as simple as starting a new container and adding its IP to the master config file.
