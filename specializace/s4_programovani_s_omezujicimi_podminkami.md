# S4. Programování s omezujícími podmínkami

> Algoritmy a konzistence: [hranová](#arc-consistency), [po cestě](#path-consistency), [k-konzistence](#k-consistency), [obecná hranová konzistence](#generalized-arc-consistency-gac), [konzistence mezí](#bound-consitency-bc), [směrové varianty](#directional-arc-consitency-dac), [šířka grafu podmínek](#graph-width). [Stromové prohledávání](#tree-search-and-heuristics), [pohled dopředu](#look-ahead-algorithms-la), [pohled zpět](#look-back-algorithms), [neúplná stromová prohledávání](#incomplete-tree-search). Modelování pomocí omezujících podmínek, [globální podmínky](#global-constraints), [omezující podmínky pro rozvrhování](#11-problém-splňování-podmínek), programování pomocí CPLEX Optimization Programming Language. (PA163)

*poznámky na základě přednášek a materiálů Hany Rudové (PA163,PA167)*

# Úvod

Given a set of (domain) **variables** $Y=\{y_1,\ldots,y_k\}$ and a finite set of **domains** $D=D_1\cup\ldots\cup D_k$ (domain $D_i$ is set of values) the contraint $c$ on $Y$ is a subset of $D_1\times\ldots\times D_k$.

- constraint is relation
- $c$ constrains the values that variables can take simultaneously
- the constraint $c$ defined on variables $y_1,\ldots,y_k$ is **satisfied** if $(d_1,\ldots,d_k)\in c$ holds for the values $d_i\in D_i$

Given a finite set of variables $Y=\{y_1,\ldots,y_k\}$, a finite set of domains $D=D_1\cup\ldots\cup D_k$ and a finite set of constraints $C=\{c_1,\ldots,c_k\}$ the **constraint satisfaction problem** is the tripple $(Y, D, C)$.

- **partial assignment** - some variables have assigned their values
- **complete assignment** - all variables have assigned values

A complete assignment of the variables that satisfies all constraints is the solution of CSP.

- we look for
    - one solution (consistent solution, feasible solution)
    - all solutions
    - optimal solution wrt. some objective function (**constraint optimization problem**)

- CP solution approach
    - formulation of the problem using constraints: modeling
        - we specify which variables we have and what must hold for them
        - declarative programming approach: we specify what must hold
    - solving of a selected representation using
        - domain-specific methods
            - specialized algorithms
            - if there is a domain-specific method, we use it instead of general methods
        - general methods
            - constraint propagation algorithms (consistency algorithms)
                - allow to remove inconsistent values from variable domains
                - simplify the problem
                - maintain equivalence between the original and the simplified problem
                - are used to compute local consistency
                - approximate global consistency
            - search algorithms
                - explore the state search space
                - examples: backtracking, branch & bound
- complexity
    - general CSP NP-complete (with boolean constraints as well)
    - constraint propagation algorithms mostly polynomial incomplete
    - search algorithms complete, but not polynomial (solution for example time-limited search)

### Binary CSP

- only binary constraints
- unary constraints encoded in the variable domain
- CSP can be transformed to binary CSP
- benefits of binarization
    - unified form of CSP
    - many algorithms designed for binary CSP
- disadvantage of binarization - problems are considerably bigger

### CSP equivalence

- CSPs equivalent $\iff$ the same set solution
- extended equivalence - problem solutions can be "syntactically" converted between each other
    - example: convert a generic CSP to a binary CSP and compare these binary CSPs

### CSP graph representation

- representation of constraints
    - intensional (mathematical/logical formula)
    - extensional (list of k-tuples of compatible values, 0-1 matrix)
- CSP representation using hypergraph
    - vertex = variable, hyperedge = constraint
    - for binary CSP simple graph
    - CSP can be transformed to binary CSP
- CSP equivalence
    - two CSPs are equivalent if they have the same set solution

![Genral CSP](https://s3.hedgedoc.org/demo/uploads/0c396c36-f3b2-4f6e-803c-d7b9940f95e5.png)

![Binary CSP](https://www.boristhebrave.com/wp-content/uploads/2021/08/arc_consistency_5var_diagram.svg)

![Graph coloring problem](https://s3.hedgedoc.org/demo/uploads/4c69722a-abb6-46ac-ac30-218b35fea0dd.png)


### Dual problem 

- **variables**: we convert the $k$-ary constraint $c_i$ to dual variable $v_i$ with a domain containing consistent $k$-tuples
- **constraints**: for each pair of constraints $c_i$ and $c_j$ sharing variables, we introduce a binary constraint $R_{ij}$ between $v_i$ and $v_j$ restricting the dual variables to $k$-tuples, in which the shared variables have the same value
    - relation $R_{rs}$ between $v_i$ and $v_j$ means that $r$-th coordinate of $v_i$ must be equal to the $s$-th coordinate of $v_j$

- example:
    - variables: $x_1, \ldots, x_6 \in \{0,1\}$
    - constraints:
        - $c1: x_1 + x_2 + x_6 = 1 \rightarrow$ dual variable $v_1$
        - $c2: x_1 - x_3 + x_4 = 1 \rightarrow$ dual variable $v_2$
        - $c3: x_4 + x_5 - x_6 = 0 \rightarrow$ dual variable $v_3$
        - $c4: x_2 + x_5 + x_6 = 0 \rightarrow$ dual variable $v_4$
![](https://s3.hedgedoc.org/demo/uploads/b89d86db-118c-405e-b10f-86a1b0c318f0.png)

- dual problem construction is one of the possible binarization methods

# CSP consistency

### Constraint propagation

- algorithms for constraint propagation (consistency algorithms)
    - remove inconsistent values from variable domains
    - simplify the problem
    - equivalence between the original and the simplified problem

## Node consistency 

- transforms the unary constraints into variable domains
- node representing variable $V_i$ is node consistent $\iff$ every value from the current domain of the variable $V_i$ satisfies all unary constraints with variable $V_i$
- CSP is node consistent $\iff$ every node is node consistent

## Arc consistency

- *binary CSP only*
    - constraint corresponds to an arc (edge) in the constraint graph
    - multiple constraints on one arc converted into one constraint
- arc $(V_i, V_j)$ is arc consistent $\iff$ for each value $x$ in the domain $D_i$ there exists a value $y$ in $D_j$ such that [$V_i=x, V_j=y$] satisfies all binary constraints over $V_i, V_j$.
- **directional!** - arc consistency $(V_i, V_j)$ does not guarantee arc consistency $(V_j, V_i)$
- CSP problem is arc consistent $\iff$ all of its arcs (in both directions) are arc consistent
- by using AC, we can remove many incompatible values, but we do not necessiraly get a solution and we do not know if a solution exists
    - example: arc consistent, no solution $X,Y,Z \in {0,1}$ constraints $X \neq Y,Y \neq Z, Z \neq X$ 
    - reduces search space, sometimes solution

![](https://s3.hedgedoc.org/demo/uploads/5ec65b65-558c-4757-bc4e-8319151d9496.png)

### Arc revision

- *How to make the arc $(V_i, V_j)$ consistent?*
- idea: delete all the values $x$ from the domain $D_i$ that are inconsistent with all the values in $D_j$, i.e. there is no value $y$ in $D_j$ such that the valuation $V_i=x, V_j=y$ satisfies all binary constraints over $V_i, V_j$

```python
def revise((i,j)):
    deleted = False
    for x in Di:
        x_consistent = False
        for y in Dj:
           if consistent(Vi=x,Vj=y): 
               x_consistent = True
        if not x_consistent:
            Di.remove(x)
            deleted = True
    return deleted
```

- compelxity $O(k^2)$, $k$ maximum domain size

### Algorithm AC-1

- *How to make CSP arc consistent?*
- repeat arc revisions until some domain is changed

```python
def AC1(G):
    changed = True
    while changed:
        changed = False
        for (i,j) in G.edges:
            changed = revise((i,j))
```

- complexity $O(enk^3)$, $k$ maximum domain size, $e$ number of arcs, $n$ number of variables
    - go through all arcs $O(e)$
    - revision $O(k^2)$
    - one loop deletes (in worst case) just one value from the domain, in total of $nk$ values (each variable has up to k values in its domain) $\rightarrow O(nk)$
- inefficient
    - even if we change a single domain, all arcs must be revised, but these arcs may not be affected by the revision at all - we can do better $\rightarrow$ AC-3

### Algorithm AC-3

- *Which arcs to revise after a domain change?*
    - arcs which consistency may be broken by the domain change
        - arcs $(i, k)$ that lead to the variable $V_k$ with changed domain
    - queue of revisions to do
 
```python
def AC3(G):
    Q = G.edges
    while len(Q) > 0:
        (i,j) = Q.pop(0)
        if revise((i,j)):
            # ingoing edges to i, not yet in Q
            Q += [(k,i) in G.edges if k != j and (k,i) not in Q] 
```

- complexity $O(ek^3)$
    - revision $O(k^2)$
    - total number of arc/constraints $O(e)$
    - each constraint can be queued at most $2k$ times $\rightarrow O(k)$
        - once a constraint is added to the queue, the domain ($k$) of one of its two variables (2) has been reduced by at least one value
- often used, but not optimal

### Value support

- definition: support for $a \in D_i := \{<j,b>| b\in D_j, (a,b) \in c_{i,j}\}$
    - value $y$ in $D_j$ supports value $x$ in $D_i$ if $V_i=x, V_j=y$ satisfies all binary constraints over $V_i, V_j$
- use supports in a consistency algorithm
    - maintain for each value a list of values that it supports
        - it knows who to tell if it disappears
    - maintain for each value count of its supports 

```python
def initialize_supports(G):
    Q = {} # queue of deleted values
    S = {} # the set of pairs (i, a) which (j, b) supports
    counter = {}
    
    for (i,j) in G.edges:
        for a in Di:
            total = 0
            for b in Dj:
                if consistent(Vi=a,Vj=b):
                    total += 1
                    S[j][b].add((i,a))
            counter[(i,j),a] = total
            if counter[(i,j),a] == 0:
                Di.remove(a)
                Q.add((i,a))
                
    return Q
```

![](https://s3.hedgedoc.org/demo/uploads/51109b70-48f6-46aa-b062-2916e8032aba.png)

### Algorithm AC-4

```python
def AC4(G):
    Q = initialize_supports(G)
    while len(Q) > 0:
        (j,b) = Q.pop()
        for (i,a) in S[j][b]:
            counter[(i,j),a] -= 1
            if counter[(i,j),a] == 0 and a in Di:
                Di.remove(a)
                Q.add((i,a))
```

- complexity $O(ek^2)$ - (theoretically) optimal, but (practically) slow
    - $O(ek^2)$ initialization, $O(ek^2)$ main procedure (we have to remove all the supports one by one, and there are $O(ek^2)$ supports)
- AC-3 not optimal, because searching for support in REVISE always starts from zero 
    - better: AC-3.1 - remembers the last support in the constraint and the next time, it starts looking for a support at this value

### AC-2001 (optimal AC-3)

- AC-3 with a queue of variables instead of constraints
- used in practice, the use of variable queues is common


```python
def revise_2001((i,j)):
    deleted = False
    for x in Di:
        if last((i,x),j) in Dj: # change, check if the last support works
            return deleted
        x_consistent = False
        for y>last((i,x),j) in Dj:
           if consistent(Vi=x,Vj=y): 
               x_consistent = True
               last((i,x),j) = y
        if not x_consistent:
            Di.remove(x)
            deleted = True
    return deleted
```

```python
def AC_2001(G):
    Q = G.nodes
    while len(Q) > 0:
        j = Q.pop(0)
        for i in G.nodes:
            if (i,j) in G.edges:
            if revise_2001((i,j)):
                if len(Di) == 0: # it might happen during revise_2001
                    return fail
                Q.add(i)
```

## Path consistency

- path $(V_0,V_1,...,V_m)$ is consistent $\iff$ $\forall x \in D_0, y\in D_m$ satisfying the binary constraints on the arc $V_0,V_m$ there exists a valuation of the variables $V_0,V_1,...,V_{m-1}$ such that all binary constraints between the neighbors $V_j,V_{j+1}$ are satisfied
    - does not guarantee that all constraints over nodes in the path are satisfied, it only deals with constraints between neighbors!
- CSP is path consistent $\iff$ all paths are consistent
- not enough to guarantee solution:
    - example: $V,X,Y,Z \in \{1,2,3\}$ constraints $X\neq Y,Y\neq Z,Z\neq X, V\neq X,V\neq Y,V \neq Z$

### Theorem: CSP is PC $\iff$ every path of length 2 is PC

- "$\rightarrow$" 
    - if all paths are PC then all path of length 2 are PC
- "$\leftarrow$"
    - by induction by path length $n$
    - base: for $n=2$ all paths of length $n$ are PC
    - assume all paths of length $n$ are PC, are all paths of length $n+1$ PC?
        - take path consisting of nodes $V_0,V_1,...V_{n+1}$
        - take any two compatible values of $x_0 \in D_0$ and $x_{n+1} \in D_{n+1}$
            - all paths of length 2 are PC $\rightarrow$ path $V_0,V_n,V_{n+1}$ is PC $\rightarrow$ we find $x_{n} \in D_{n}$ such that ($x_0,x_n$) and ($x_n,x_{n+1}$) are consistent
        - by the induction step we find the remaining values on the path $V_0,V_1,...V_{n}$

### PC and AC relationship

- PC $\implies$ AC
    - arc is AC, if the path $(X,Y,X)$ is PC
- AC $\;\not\!\!\!\implies$ PC
    - example: $X,Y,Z \in {0,1}$ constraints $X \neq Y,Y \neq Z, Z \neq X$ 
    - is AC, but not PC
        - $X=1,Y=0$ cannot be extended along the path $(X,Y,Z)$

### Path revision algorithm

- for a pair of variables $V_i,V_j$, update elements of the relation $R_{ij}$ 

```python
def revise_p(i,m,j):
    deleted = False
    for (x,y) in Rij:
        xy_consistent = False
        for z in Dm:
            if (x,z) in Rim and (z,y) in Rmj:
                xy_consistent = True
        if not xy_consistent:
            Rij.remove((x,y))
            deleted = True
    return deleted
```

- complexity $O(k^3)$, $k$ maximum domain size

### Algorithm PC-1

- initialize $R_{ij}$ using existing binary (and unary) constraints
- path revision for each path of length 2
- repeat the path revisions until some pairs are deleted

```python
def PC1(G):
    changed = True
    while changed:
        changed = False
        for m in range(n): # n variables
            for i in range(n):
                for j in range(n):
                    changed = revise_p(i,m,j)
```

- complexity $O(n^5k^5)$
    - 3 cycles for triples of values $O(n^3)$
    - revise_p $O(k^3)$
    - the number of repetetion in the while cycle $O(n^2k^2)$
        - one cycle deletes in the worst case only one pair of values
        - $n^2$ pairs of variables, $k^2$ pairs of values for each pair of variables
- inefficiency
    - repeated revision of all paths even if no change has occurred for them
    - paths only need to be taken with one oriantation
    - $\rightarrow$ PC-2 solves the problems

### Algorithm PC-2

```python
def PC2(G):
    Q = {(i,m,j)|1<=i<=j<=n,1<=m<=n,m!=i,m!=j} # set of paths for revisions
    while len(Q) > 0:
        (i,m,j) = Q.pop(0)
        if revise_p(i,j,m):
            # add paths not yet queued
            if i!=j:
                # někde tady je chyba v pseudokódu
                for l in range(j):
                    if l!=i:
                        Q.add((l,i,j))
                for l in range(i):
                    if l!=j:
                        Q.add((l,j,i))
```

- complexity $O(n^3k^5)$
    - each triple can be queued at most $k^2$ times, because when it is added to the queue, $R_{ij}$ has been reduced by at least one value
- not optimal
    - PC-4 with counting supports and complexity $O(n^3k^3)$ optimal 
- memory - eliminates pairs of values from the constraints $\rightarrow$ it needs to use an extensional constraint representation
- effectivity
    - PC eliminates more (or as many) inconsistencies as AC
    - the performance to problem simplification ratio is much worse than that of AC
- changes to the constraint graph - adds arcs (constraints) even where they weren’t originally

# Non-binary constraints

- generalization of NC, AC, and PC principles towards $k$-consistency
    - interesting in terms of problem-solving complexity
    - works with arbitrary $k$-tuples of variables
    - not used in practice due to memory and time complexity
- **global constraints**
    - specific types of consistencies 
    - example: `allDifferent`

## k-consistency


CSP is $k$-consistent $\iff$ any consistent ($k$-1) valuation of different variables can be extended to any $k$-th variable.

CSP is strongly $k$-consistent $\iff$ it is $j$-consistent for every $j\leq k$.


- strong $k$-consitency $\implies$ $k$-consistency
- strong $k$-consitency $\implies$ $\forall j \leq k$: $j$-consistency
- $k$-consitency does not imply strong $k$-consistency
- NC = strong 1-consistency = 1-consistency
- AC = (strong) 2-consistency
- PC = (strong) 3-consistency

**Concepts and markup**
- **constraint scope** $scope(c)$ is set of variables on which constraint $c$ is defined
- **k-tuple $\vec{t}$** of values belonging to $c$: $\vec{t} \in c$
- for the variable $x\in scope(c)$ and k-tuple $\vec{t} \in c$, $\vec{t}[x]$ is the value of the variable $x \in \vec{t}$
    - example: $A, B, C \in \{0, 1, 2\}$
        - $scope(A < B) = \{A, B\}$
        - $(0,1)\in(A<B)$
        - $(0,1)[A]=0$

### Consistency and solutions

- CSP can be solved without bactracking with respect to the ordering of the variables $(x_1, ..., x_n)$ if for each $i\leq n$, each partial solution $(x_1, ..., x_i)$ can be consistently extended by a variable $x_{i+1}$
- solution without backtracking for any ordering of variables - strong $n$-consistency is required for a graph with $n$ vertices
    - $n$-consitency is not enough
    - strong $k$-consistency for $k<n$ is not enough
        - example: complete graph of $n$ nodes, edges $\neq$, domains $\{1,...,n-1\}$

### Generalized arc consistency (GAC)

- sometimes called domain consistency
- for each variable in the constraint and each of its values there is a valuation of the remaining variables in the constraint such that the constraint holds
    - example: $A + B = C, A \in \{1, 2, 3\}, B \in \{2, 3, 4\}, C \in \{3, ..., 7\}$
- the constraint $c$ is generalized arc consistent if every value $a$ of every variable $x \in scope(c)$ has a domain support in $c$
    - the value $a$ of variable $x \in scope(c)$ has a domain support $\vec{t}$ in $c$ if
        - $\vec{t}\in c$ 
        - $a\in \vec{t}[x]$
        - $\forall y \in scope(c): \vec{t}[y] \in D_y$
    - example:  $A = B + C, A \in \{0,1\}, B \in \{0,1\}, C \in \{1,2\}$
        - 0 at A - no support
        - 1 at A - support (1,0,1)
- CSP is generalized arc consistent $\iff$ all its constraints are generalized arc consistent

### Bound consitency (BC)

- the constraint $c$ is bound consistent if every value $a$ of every variable $x \in scope(c)$ has a interval support in $c$
    - the value $a$ of the variable $x \in scope(c)$ has interval support $\vec{t}$ in $c$, if 
        - $\vec{t}\in c$ 
        - $a\in \vec{t}[x]$
        - $\forall y \in scope(c): \vec{t}[y] \in [min(D_y),max(D_y)]$
- CSP is bound consistent $\iff$ all its constraints are bound consistent
- weaker than generalized arc consistency
- propagation only at change of minimum or maximum value in the domain of the variable
- examples of bound consistency
    - $A > B: min(A) \geq min(B)+1, max(B) \leq max(A)−1$
    - $A = B + C:$
        - $min(A) ≥ min(B)+min(C)$
        - $max(A) ≤ max(B)+max(C)$
        - $min(B) ≥ min(A)-max(C)$
        - $max(B) ≤ max(A)-min(C)$
        - $min(C) ≥ min(A)-max(B)$
        - $max(C) ≤ max(A)-min(B)$

### Algorithms

- extension of arc revision and path revision
    - gradually remove elements from the relation over $(k-1)$ variables
- update the relation over each $(k-1)$-tuple of variables
    - we have to remember the $(k-1)$-tuples of values
- general algorithm
    - extension of AC-1 and PC-1
    - repeating revisions over $(k-1)$-tuples as long as changes occur 
- not used in practice for higher k because of high memory and time complexity

#### General consistency algorithm for non-binary constraints

- extension of AC-2001 to non-binary constraints
- repeatedly revise constraints as long as domains change
- special revise procedures are defined depending on the type of constraints
    - examples:
        - general arc consistency
        - bound consistency
        - consistency algorithms for global constraints like allDifferent
        - ...
    - the user often has the option to define own revise procedure
    - it is necessary to determine the event that triggers the revision
    - event is a change to the domain of a variable (suspension)
        - invoking a revision only on a given variable change
            - on any domain change (for general arc consistency)
            - on bound change (for bound consistency)
            - on instantiation of the variable
        - different events can be used for each variable
        - basic consistency algorithm with events in slides  

```python 
def general_consistency(Q,G): # Q queue of variables
    while len(Q) > 0:
        Vj = Q.pop(0)
        for c in C:
            if Vj in scope(c):
                # revise procedures are defined depending on the type of constraints
                W = revise(Vj,c) # W is a set of variables whose domain is changed
            for Vi in W:
                if len(Di) == 0:
                    return fail
            Q.add(W) 
```

![](https://s3.hedgedoc.org/demo/uploads/3e3a9319-182a-4f7f-9a76-4aa8e572f59d.png)

# Global constraints

- constraint defined for any finite number of variables
- examples:
    - all variables different: allDifferent
    - disjunctive resources: noOveralp
    - cumulative resources: cumuFunction, pulse
    - alternative resources: alternative

## All variables different

- allDifferent vs. binary constraints
    - allDifferent has full propagation
    - binary constraints have weaker (incomplete) propagation
- to maintain consistency, it must hold $\forall \{X_1,...,X_k\} \subseteq V: cardinality(D_1\cup...\cup D_k)\geq k$
- inference rule:
    - $V$ is set of all variables
    - $U=\{X_1,...,X_k\}, dom(U)={D_1\cup...\cup D_k}$
    - $card(U) = card(dom(U)) \implies \forall v \in dom(U), \forall X \in (V-U): X \neq v$ (values in $dom(U)$ are unavailable for other variables)
- complexity of finding all subsets of the set $n$ of variables (naive) $O(2^n)$
- effective propagation for allDifferent can be based on matching in bipartite graphs 

### CSP as bipartite graph

- one set of vertices represents the variables, the other set of vertices represents the values of the variables
- edge from variable $X$ to value $a$ if the variable $X$ can take value $a$
- propagation for allDifferent
    - initialization
        - calculate maximum pairing
        - remove all edges that don’t belong to any maximum pairing
    - propagation of edge/value removals
        - remove corresponding edges
        - calculate new maximum paring
        - remove all edges that don’t belong to any maximum pairing 
    - algorithm uses domain consistency
        - each maximum pairing defines domain support
        - complexity $O(n^2 k^2)$, $n$ number of variables, $k$ maximum domain size

# Directional consistency

## Directional arc consitency (DAC)

- binary CSP is directionally arc consistent with respect to the ordering of variables $(x_1, ..., x_n)$, just when each arc $x_j,x_i$ is arc consistent for every $j\leq i$

```python
def DAC_binaryCSP(G,(x1,...xn)):
    for i in range(n,0,-1):
        for j in range(i):
            if (i,j) in G.edges:
                revise(j,i) 
```
- repeating revisions is not necessary, domains of revised variables unchanged in subsequent cycles
- complexity $O(ek^2)$ - each arc is revised once with complexity $O(k^2)$
- we can get solution without backtracking 
    - reminder: CSP can be solved without bactracking with respect to the ordering of the variables $(x_1, ..., x_n)$ if for each $i\leq n$, each partial solution $(x_1, ..., x_i)$ can be consistently extended by a variable $x_{i+1}$

![](https://s3.hedgedoc.org/demo/uploads/76ae8062-8144-4a1a-9041-7f9c72b9add5.png)

## Directional i-consistency

- CSP is directionally $i$-consistent with respect to the ordering of variables $(x_1, ..., x_n)$ if every $(i − 1)$ of the variables are $i$-consistent with respect to all variables that follow them in the ordering
- CSP is strongly directionally $i$-consistent (DIC-$i$) precisely when it is directionally $j$-consistent for every $j \leq i$
- DIC-2 equivalent to DAC
    - for both we consider unary and binary constraints
    - DAC is defined only on binary CSPs
    - DIC-2 is defined for general CSPs, but it is only capable to find a consistent value for a given variable in the domain of another variable, so it does not catch constraints with more than two variables

### Solution without backtracking with DIC-$i$

- if we want to assign value to a variable, it must be compatible with variables already assigned, i.e., with as many variables as there are "backward" arcs
- for $i$ backward arcs we need a directional $(i + 1)$-consistency
- if $m$ is the maximum of the number of backward arcs for all nodes, we just need a strong directional $(m + 1)$-consistency
- when the nodes are ordered differently,the number of backward arcs is different **!**
- $\rightarrow$ we search for the node ordering with the smallest number of bacward arcs 

### Graph width

- ordered graph - a graph with a linear ordering of nodes
- node width *in an ordered graph* - the number of arcs leading from this node to previous nodes
- width of ordered graph - maximum of all its node widths
- graph width - minimum of the widths of all its ordered graphs

![](https://s3.hedgedoc.org/demo/uploads/4431e16a-f41a-4de4-86ee-cdf10b3fe018.png)

```python
def min_width_ordering(G):
    Q = []
    for i in range(len(G.vertices)):
        n = node with the fewest arcs from G
        delete n from G with all its edges
        Q.apend(n)
    return Q.reversed
```

**A graph G is a tree $\iff$ G has a width 1.**

Proof:
- $\rightarrow$
    - lets have G without cycles - tree
    - we can create an oriented tree with a root such that all arcs point out from the root node
    - each node has at most one parent (root none, others one)
    - ordering where parents precedes its children has a width of 1
- $\leftarrow$
    - let us have a graph of width 1
    - can not contain cycle
    - tree

### Consistency for tree CSP

**Let $d = (x_1, ..., x_n)$ be the ordering of the tree constraint graph $T$ for a given CSP. If $T$ is directionally arc consistent (DAC) with respect to $d$, then the CSP has a solution without backtracking with respect to $d$.**

Proof:
- consider ordering $d = (x_1, ..., x_n)$ with width 1
    - constraint graph $T$ is a tree, so it exists
- assume $x_1, ..., x_i$ are consistently instantiated
- we want to instantiate $x_{i+1}$
    - $d$ is ordering of width 1, so there is only one parent $x_j$ $(j\leq i+1)$ of $x_{i+1}$
    - $(x_j,x_{i+1})$ is arc consistent (from DAC), so there is a value in the domain of $x_{i+1}$ consistent with the current assignment of $x_j$, we assign this value

```python
def tree_solving(tree T):
    (x1,...,xn) = min_width_ordering(T) # find an ordering with width 1 using tree T
    for i in range(n,0,-1):
        revise(xi.parent,xi)
        if len(D_xi.parent) == 0:
            return fail # solution does not exist
```

- algorithm finds DAC for tree CSP - possible to find a solution without backtracking
    - if we apply DAC with respect to the ordering of width 1, and then in the opposite direction, we achieve full arc consistency
- complexity $O(nk^2)$

### Graph width and degree of consistency

**Lets have a CSP whose ordered constraint graph with ordering $d$ has width $i − 1$. If the problem is strongly directionally $i$-consistent, then the problem is solvable without backtracking with respect to $d$.**

Proof:
- there is an ordered graph with ordering $d$ and width $i − 1$
- we assign the variables in the order of the graph ordering
- while assgning variable:
    - we have to find a value compatible with all the variables already assigned, that are linked to the variable by a constraint (arc)
    - let there be $m$ such variables, then $m \leq (i − 1)$ because of width $(i-1)$
    - graph is directionally $i$-consistent, so such a value must exist (by definition)

![](https://s3.hedgedoc.org/demo/uploads/a978d847-6000-4508-919b-e6c97632e311.png)

## Adaptive Consistency (ADC)

- variable must be compatible with already assigned variables, i.e., with as many variables as there are backward arcs
    - problem: strong directional $i$-consistency algorithms for $i \geq 3$ change constraint graph by adding arcs, the necessary value of $i$ may increase

#### Adaptive consistency algorithm to find a solution without backtracking

- ordering $d$ 
- recursively generate directional $i$-consistency
    - from last to first variable in $d$
    - change the value of $i$ from node to node so that we adapt to the current node width at the moment of processing
- why it works?
    - at the moment of node processing the final node width is determined
    - we know what level of directional $i$-consistency we need to achieve 

The class of CSPs whose graph width is bounded by the constant $b$ is solvable in polynomial time and space.

# Interval variables

- modeling a time interval (e.g. task, activity)
- value is an integer interval [start, end)
- might be `optional` - optional activities that may not be present in the solution
- we can define precedence constraints between interval variables 
    - `endBeforeStart(i, j);`, `endAtStart(i, j);`, ...
- unary constraints for the presence of interval
    - `presenceOf(x)` 
- expresions with interval variables
    - `startOf(x)`
	- `endOf(x)`
	- `sizeOf(x, V)`

## Sequence

- defined on the set of interval variables
- value is permutation of the present intervals
    - the permutation p does not yet imply any ordering in time!
- constraint `noOverlap(p)`
    - useful for unary/disjunctive resources
    - absent intervals are not included in the sequence

**Disjunctive scheduling: algorithm principle**

- answering questions like "What happens if activity A is not processed first?" and "What happens if activity A is not processed last?"

## Cumulative functions

- the value of of the cumulative function expression represents the evolution of the quantity over time, which can be incrementally changed (decreased or increased) by interval variables
- the interval variables x[i] contribute to the cumulative function for the duration of its execution

![](https://s3.hedgedoc.org/demo/uploads/60e67ad2-d97e-41aa-ba50-7c194d29d3f7.png)

## Alternative constraint

- if the interval x is present, then one of the intervals is y1,...,yn is present and is synchronized with x (it has the same start and end)
- if x is absent, then yi are also all absent

# Combining search and propagation

- search algorithm often applied after the initial constraint propagation
    - typically: initial propagation and then forward checking or initial consistency and then look-ahead
- search algorithms for CSP - overview
	- constructive tree search: extending partial consistent assignment
		- backtracking
		- look-ahead (constraint propagation)
		- look-back (intelligent backtracking)
		- incomplete tree search algorithms
	- iterative search: over complete inconsistent assignments
		- generate and test
		- local search
	- population-based search
		- genetic algorithms, ant colony optimization, ...

#### State space

- represented by tree
    - root - empty assignment -initial state in state (search) space
    - nodes - consistent partial assignments
    - edges - operators that extend the partial assignment
    - leafs - inconsistent or complete consistent assignments (goal states)
- CSP has a solution without backtracking with respect to a variable ordering d if all leaves in its state space represent solutions

## Backtracking (DFS)

- two phases
    - forward - variables are subsequently selected, expanding partial solution by assigning a consistent value (if any) for the next variable
    - backward - if there is no consistent value for the current variable, the algorithm backtracks to the previous assigned value
- variables
	- past: variables already selected (and have an assigned value)
	- current: a variable that is currently selected and is assigned a value
	- future: variables that will be selected in the future

```python
def select_value(Xi,D,C):
    while len(D[Xi]) > 0:
        a = D[Xi].pop()
        if consistent(Xi,a,C):
            return a
    return None

def backtracking(X,D_all,C): # D_all original domains, D domains in progress
    i = 1
    D[i] = D_all[i]
    A = [] # assigned values
    while 1 <= i <= n:
        A[i] = select_value(X[i],D,C) # value assignment
        if A[i] == None:
            i = i-1
        else:
            i = i+1
            D[X[i]] = D_all[X[i]]
    if i == 0:
        return fail # no consistent assignment
    return A
```

- returns only the first solution
- checks at each step the consistency of the constraints leading from past variables to the current variable
    - ensures consistency of constraints on
        - all past variables
        - between past variables and the current variable
- weaknes: repeated finding of the same inconsistencies and partial search successes 

## Look-ahead algorithms (LA)

- check the constraints ahead
    - filter the domains of unassigned variables
    - try to find out how variable and value selection will affect future search
- help us decide, which variable to assign
    - it is usually better to assign the variables that most constrain the rest of the state space 
- help us decide, which value to assign to the variable
    - trying to choose the value that allows the most choices in future assignments 

```python
def LA(X,D_all,C): # D_all original domains, D domains in progress
    i = 1
    D = D_all # copy ALL domains
    A = [] # assigned values
    while 1 <= i <= n:
        A[i] = select_value_XXX(X[i],D,C) # value assignment different for each LA algorithm type
        if A[i] == None:
            i = i-1
            for k in range(i,n): # differs from backtracking
                D[k] = value before the last instantiation of A[i]
        else:
            i = i+1
            D[X[i]] = D_all[X[i]]

    if i == 0:
        return fail # no consistent assignment
    return A
```

- `select_value_XXX` different for each LA algorithm type
- n copies of each D stored
    - one copy for each level in the state space

### Forward checking (FC)

- type of value selection for LA
- FC is an extension of backtracking
- ensures consistency between the current variable and future variables that are linked to it by constraints not yet satisfied

```python
def select_value(i,X,D,C):
    while len(D[X[i]]) > 0:
        a = D[X[i]].pop()
        for k in range(i,n):
            for b in D[X[k]]:
                if not consistent(Xi=a,Xk=b,C):
                    D[X[k]].remove(b)
            if len(D[X[k]]) == 0:
                a_dead_end = True
                break
        if a_dead_end:
            for k in range(i,n):
                D[X[k]] = value before selecting a 
        else:
             return a   
    return None
```

### Real full look-ahead (RFLA)

- sometimes called "maintaining arc-consistency (MAC)"
- after assigning value, run AC-1 (or other AC) algorithm

```python
def select_value(i,X,D,C):
    while len(D[X[i]]) > 0:
        a = D[X[i]].pop()
        deleted = True
        while deleted:
            deleted = False
            for j in range(i,n):
                for k in range(i,n):
                    for b in D[X[j]]:
                        cons = False
                        for c in D[X[k]]:
                            if consitent(Xi=a,Xj=b,Xc=c):
                                cons = True 
                        if not cons:
                            D[X[j]].remove(b)
                            deleted = True
        a_dead_end = False
        for k in range(i,n):
            if len(D[X[k]]) == 0:
                a_dead_end = True
        if a_dead_end:
            for k in range(i,n):
                D[X[k]] = value before selecting a 
        else:
            return a
    return None
```

### Comparison of search algorithms

- backtracking (BT) checks in step $a$ the constraints $c(V_1,V_a),...,c(V_{a-1},V_a)$
    - from past variables to current variable
- forward checking (FC) checks in step $a$ constraints $c(V_{a+1},V_a),...,c(V_{n},V_a)$
    - from future variables to the current variable
- look-ahead (LA) checks in step $a$ the constraints $\forall l (a\leq l \leq n), \forall k (a\leq k \leq n), k \neq l: c(V_k,V_l)$
    - from future variables to current variable and between future variables
    - extension of FC, maintains arc consistency
    - real full look-ahead RFLA
        - often referred to as the LA algorithm
        - checks the consistency of all arcs between future variables
    - full look-ahead FLA
        - only one repeat until pass in the algorithm Select-Value-Look-Ahead
    - partial look-ahead PLA
        - only one repeat until pass in the algorithm Select-Value-Look-Ahead
        - future variables compared only to those that follow them

![](https://s3.hedgedoc.org/demo/uploads/908a5ac8-17a9-46f4-bb9f-68df114274bb.png)

![](https://s3.hedgedoc.org/demo/uploads/e3996f29-dd22-479f-97eb-fd33c39907d5.png)

### Value selection

- trivial
    - domain traversed in ascending/descending order
- succeed first
    - choose the order so that we do not have to repeat the selection

#### Look-ahead with dynamic value selection

- value selection from the variable domain
    - we’ll try to assign each of the values in the domain to the variable and try the effect of using FC or another LA algorithm
- minimal conflict
    - selecting the value that deletes the smallest number of values from domains of future variables 
    - experimentally: this heuristic shows very good results
- maximum domain size
    - selecting the value that will create the largest minimum domain size among all future variables 

### Variable selection

- large effect on the state space size
- static ordering - selection of variables given in advance
    - trivial
        - i.e., the variable with the smallest index
    - maximum cardinality
        - the first variable (graph node) is chosen randomly
        - each node is associated with the maximum number of already ordered (= previously selected) neighbors
        - variables are selected in order of this number of nodes
    - minimum width
        - variables ordered to minimize graph width (minimizing the number of backward arcs)
        - we select the variables with the fewest backward arcs
- dynamic ordering
    - selection of variables calculated during the search
    - can be used (computed) by look-ahead algorithms 
    - first-fail
        - select the variable with smallest domain

## Look-back algorithms

- backjumping
    - try to backtrack as far as possible to the source of the failure
- no-good learning (constraint recording) 
    - no-good = incorrect partial assignment
    - adds new constraints that disallow the found no-goods

**Conflict set**

- fixed ordering of variables $(x_1,...,x_n)$
- for partial consistent assignment $\vec{a}=(a_{i_1},...,a_{i_k})$ of an arbitrary subset of variables, consider a previously unassigned variable $x$, if it is not possible to assign it consistently:
    - $\vec{a}$ is a conflict set of $x$
    - $\vec{a}$ is in conflict with $x$
    - if $\vec{a}$ does not contain a subtuple that is in conflict with $x$, it is a minimal conflict set

**No-good**

- fixed ordering of variables $(x_1,...,x_n)$
- for partial consistent assignment $\vec{a_{i-1}}=(a_1,...,a_{i-1})$, if $\vec{a_{i-1}}$ is in conflict with $x_i$, then we call it leaf dead-end and $x_i$ is called leaf dead-end variable
- no-good - partial assignment, which does not occur in any CSP solution
    - conflict sets are no-goods
- minimal no-good does not contain any no-good with less variables

**Culprit variable**

- $\vec{a_{i-1}}$ is leaf dead-end, then $x_j$ where $j<i-1$ is safe if $\vec{a_{j}}$ is a no-good
    - jump from $x_i$ to $x_j$ is safe, it does not omit any solution
    - culprit index relative to $\vec{a_{i-1}}$ is defined by $b=min\{k<i | \vec{a_k} \text{ is in conflict with } x_i\}$
    - $x_b$ culprit variable of $\vec{a_{i-1}}$
- jump to the culprit variable is safe
    - we are trying to assign $x_i$ - not possible (dead-end)
    - find the smallest part of assignment which is still causing conflict
        - change that - jump back here
        - changing anything later does not help, because assigning $x_i$ is not possible

![](https://s3.hedgedoc.org/demo/uploads/63925db8-1987-42d9-8b42-007a381b699c.png)

### Gaschnig’s backjumping (GBJ)

- not possible to assign $x_i$? - go back to culprit variable
- computation of culprit variable 
    - for each value we are trying to assign ($v_i \in D_i$), we find variable with with the smallest index that is in conflict with that assignment
        - the culprit variable is the one that has largest index

```python
def GBJ(X,D_all,C): # D_all original domains, D domains in progress
    i = 1
    D[i] = D_all[i]
    A = [] # assigned values
    latest_i = 0
    while 1 <= i <= n:
        A[i] = select_value_GBJ(i,X,D,C, latest_i) # value assignment
        if A[i] == None:
            i = latest_i
        else:
            i = i+1
            D[X[i]] = D_all[X[i]]
            latest_i = 0

    if i == 0:
        return fail # no consistent assignment
    return A

def select_value_GBJ(i,X,D,C,latest_i):
    while len(D[X[i]]) > 0:
        a = D[X[i]].pop()
        consistent = True
        k = 1
        while k<i and consistent:
            if k > latest_i:
                latest_i = k
            if not consistent(X[i]=a):
                consistent = False
            else:
                k += 1
        if consistent:
            return a   
    return None
```

### Conflict-directed backjumping (CBJ)

- constraint $R$ is earlier than the constraint $S$, if the latest variable in the scope of $R$ precedes the latest variable in the scope of $S$ 
    - for identical variables, the other variables decide
- when jumping back to the variable $x_j$ from leaf dead-end, we may not find any value in its domain to assign it
    - $\vec{a_{j-1}}$ internal dead-end
    - $x_j$ internal dead-end variable
- CBJ jumps back 
    - in the leaf dead-end
    - in the internal dead-end 
    - maintain jumpback set $J_i$ for each variable 
        - using unsatisfied constraints
        - represent no-goods calculated during the search
            - these no-goods may occur later in other paths in the search tree and are recalculated   
    - select the earliest constraint among the unsatisfied constraints
        - longest possible jump back
    - jump back to the latest variable in this constraint
        - safe to change the variable that was assigned last 

```python
def CBJ(X,D_all,C): # D_all original domains, D domains in progress
    i = 1
    D[i] = D_all[i]
    A = [] # assigned values
    J = [] # jumpback sets
    for i in range(n):
        J[i] = [] 
    while 1 <= i <= n:
        A[i] = select_value_CBJ(i,X,D,C, latest_i) # value assignment
        if A[i] == None:
            ip = i
            if len(J[i]) == 0:
                return fail # not possible to remove the conflict
            else:
                i = index of the last variable in J[i] # backjumping
                J[i] = J[i] join J[ip] - {X[i]} # join jumpback sets
        else:
            i = i+1
            D[X[i]] = D_all[X[i]]
            J[i] = []

    if i == 0:
        return fail # no consistent assignment
    return A

def select_value_CBJ(i,X,D,C,latest_i):
    while len(D[X[i]]) > 0:
        a = D[X[i]].pop()
        consistent = True
        k = 1
        while k<i and consistent:
            if k > latest_i:
                latest_i = k
            if not consistent(X[i]=a):
                R = the earliest unsatisfied constraint form C
                add variables in the scope of R, excluding X[i], to J[i]
                consistent = false
            else:
                k += 1
        if consistent:
            return a   
    return None
```

### No-good learning (constraint recording)

- no-good learning algorithm ≡ CBJ + adding new constraints
- add no-goods as new constraints when a dead-end is detected
    - after reaching the leaf dead-end $\vec{a_{i-1}}$, add constraint prohibiting $\vec{a_{i-1}}[J_i]$ projection on $J_i$
- state space prunning 
    - the smaller no-goods can be kept, the faster the search
- enlarging the constraint set
    - the cost of pruning the state space must not exceed the cost of enlarging of the constraint set


## Incomplete tree search

- do not search entire state space - too large
- no guarantee that the solution does not exist, even if the algorithm does not find it
    - loss of completeness
- in many cases, we find the solution faster
- algorithms often derived from the complete algorithm
    - cutoff
    - restart 

**Randomized backtracking**

- time-limited backtracking (cutoff)
    - terminates after specified time interval, can be increased for further runs
    - backtracking itself is complete
- random selection of values and variables (restart)
    - if we have a choice in value or variable selection (relative to given selection heuristic), we randomly choose one of the them
- randomized backtracking with learning
    - keep the no-goods of previous runs and use them

**Bounded-backtrack search (BBS)**

- bounded number of backtracks (cutoff)
    - backtracks to choice points where a new value can no longer be selected is not counted
- "limited number of lists visited"
- for completeness
    - on failure increase the number of backtracks by one (restart)

**Depth-bounded search (DBS)**

- limit the depth of the search tree (cutoff)
    - within a given depth of the tree, all alternatives are tried
    - in the rest of the tree, another incomplete method can be used
- for completeness
    - on failure increase the search depth by one (restart)

**Credit search (CS)**

- limited credit (number of backtracks) for search (cutoff)
    - credit is split evenly between alternative search branches
    - unit credit disables the (value) choice
        - continue without bactracking
- for completeness
    - on failure increase the credit by one (restart)

**Iterative broadening (IB)**

- limited maximum number of values for each variable selection (cutoff)
    - branching limited by a constant
    - if the maximum number of choices is exceeded, continue with the previous choice point
- for completeness
    - on failure increase the allowed number of choices by one (restart)

### Tree search and heuristics

- heuristics: advice on how to continue searching
    - recommend a value to assign
    - often lead directly to the solution
    - discrepancy = heuristic violation 
        - use a different value than recommended by the heuristic
- What to do if the heuristic fails?
	- DFS mainly takes care of the end of the search (bottom of the tree)
	    - spends most of the time there
	    - fixes the last heuristics rather than the first
	        - assumes that previously used heuristics have guided it well 
- observations
    - the number of heuristic violations on the successful path is small
        - few heuristic violations on the way to the solution
    - heuristics are less reliable at the beginning of the search than at its end 
        - heuristic violations mainly at the beginning of the path
        - we have more information and fewer options at the end

**Limited discrepancy search (LDS)**

- limited maximum number of discrepancies on the path (cutoff)
    - paths with discrepancies at the beginning are explored earlier
    - if unsuccessful, increase the number of allowed discrepancies by one (restart)

![](https://s3.hedgedoc.org/demo/uploads/9c5969b4-573b-405e-a9cd-2f5eaa002bea.png)


**Improved limited discrepancy search (ILDS)**

- LDS in each subsequent iteration also passes through the branches from the previous iteration
    - repeats the calculation already performed
    - must backtrack to already explored parts in each iteration
- ILDS
    - a given number of discrepancies on the path (cutoff)
        - paths with discrepancies at the end explored earlier

![](https://s3.hedgedoc.org/demo/uploads/fae3facf-df19-4554-8185-3fc0ab7752c1.png)

**Depth-bounded discrepancy search (DDS)**

- discrepancies allowed only up to a given depth (cutoff) and at this depth, there is always a discrepancy
    - it prevents traversal of branches from the previous iteration
    - paths with discrepancies at the beginning explored earlier

# Local search

- *works with complete inconsistent variable assignments, tries to reduce the number of conflicts*
- it is heuristic search method
    - incomplete
    - does not guarantee finding (or proving non-existence of) a solution even if it exists (does not exist)
    - small memory requirements

**Terminology**

- **state $\theta$** - assignment of all variables
- **evaluation $E$** - objective function value represented by the number of unsatisfied constraints = number of conflicts
- **local change** - change in value of (one) variable
- **neighborhood $o$** - set of states differing from the given state by one local change

![](https://s3.hedgedoc.org/demo/uploads/3838c4d6-4fa2-4d41-ac57-d75cc51e4a94.png)

## Local search algorithm

```python
def LS(max_trials, max_changes):
    for i in range(max_trials):
        A = initial assignment
        while GLOBAL_COND: # condition depends on alg. variant
            for j in range(max_changes):
                while LOCAL_COND: # condition depends on alg. variant
                    if E(A) == 0: # E depends on alg. variant
                        return A
                    A_new = select from A.neighborhood() # select and neighborhood depend on alg. variant
                    if acceptable(A_new): # acceptable depends on alg. variant
                       A = A_new                 
```

## Hill climbing

- starts in randomly selected state
- neighborhood = value of an arbitrary variable is changed
    - size $n\times (d-1)$
- always selects the best state in the neighborhood
- escape from strict local minimum by restarting

```python
def hill_climbing(max_changes):
    restart:
    A = random initial assignment
    for j in range(max_changes):
        if E(A) == 0: 
            return A
        if A strict local optimum:
            goto restart
        else:
            A = select best from A.neighborhood()
    if enough time:
        goto restart
    else:
        return fail # E(A) > 0          
```

**Conflict minimization (MC)**

- HC neighborhood large
    - change only the conflicting variables
        - conflicting variable = present in some unsatisfied constraint
- `neighborhood()` contains only conflicting variables
    - select the value that minimizes the number of conflicts

## Random walk (RW)

- possibility to escape from local optimum without restarting
- in local optimum
    - randomly select a state from the neighborhood and continue with it
    - combine this with HC or MC

**RW combined with heuristics using probability distribution**

- the probability of a random step is $p$, probability of using the directional heuristic is $1 − p$
    - random selection of a conflicting variable
- replaces restart

## Simulated annealing (SA)

- at the beginning, the probability that we accept deterioration of the solution, is higher
- accepting new state
    - on improvement - always
    - on deterioration - with a probability decreasig during the algorithm
- algorithm cycles
    - external: reducing temperature $T$ with time
    - internal: counting how many times we have not accepted the deterioration

**Metropolis criterion**

- $dE = E(A_{new})-E(A)$, we want to minimize E
- always accept solution if $dE <0$
- if $dE>0$, accept if $U<e^{-dE/T}$ for random number $U$ from the interval $(0,1)$, $T$ temperature

```python
def SA(init_T,min_T,max_iter):
A = initial assignment
best = A
T = init_T
while min_T < T:
    no_errors = 0
    while no_errors < max_iter:
        A_new = select from A.neighborhood()
        if E(A_new) < E(A):
            A = A_new
            if E(A) < E(best):
                best = A
        else:
            p = random(0,1)
            if p < e^((E(A)-E(A_new))/T):
                A = A_new
            else:
                no_errors += 1
    T = decrease(T) # e.g., each several iterations process 0.8 × T
```

### Local search for SAT problem

- SAT - satisfiability of the logical formula in CNF
    - conjunction of clauses, clause is disjunction of literals
    - example:  $(A ∨ B) ∧ (¬B ∨ C ) ∧ (¬C ∨ ¬A)$
    - NP-complete
- LS solves a rather large formula
    - flipping values
    - GSAT (greedy SAT)
        - flipping over variables sequentially
        - evaluation gives the (weighted) number of unsatisfied clauses
        - can be combined with other heuristics
            - random walks with conflict minimization
            - weighting of clauses
                - some clauses remain unsatisfied for a number of iterations, clauses thus have different importance
                - satisfying a "hard" clause can be preferred by adding a weight
                - weight can be derived by the algorithm
            - solution averaging
                - by default each trial starts from a random assignment, but the common parts of the previous assignments can be preserved

```python
def GSAT(CNF,max_trials,max_changes): # CNF is formula in CNF
    for i in range(max_trials):
        A = random initial assignment
        for j in range(max_changes):
            if CNF satisfiable by A:
                return A
            V = variable whose value change will reduce the number of unsatisfied clauses the most
            A = A.flip(V)
    return best A
```

# Hybrid search - local search + tree search

- local search before or after tree search
    - before: local search will give us heuristics on the value ordering
    - after: local search is trying to improve the computed solution locally (optimization)
- tree search can be complemented by a local search
    - in the leaves of the search tree and its nodes
        - e.g. local search to select a value or refine a computed solution
- local search can be complemented by tree search
    - to select a neighbor from the neighborhood or to prune the search space

## Iterative forward search

- hybrid search: a constructive non-systematic algorithm
- for optimization problems
- model with hard and soft constraints
    -  hard: must be satisfied
    -  soft: objective function
- works with consistent assignments
- **idea**
    - starts with an empty assignment
    - selects a new variable to assign
    - if it finds a conflict, unassigns all variables conflicting with the selected variable
    - value selection using conflict statistics
    - variable selection is not critical to the algorithm, because variables can be can be assigned repeatedly

```python
def ifs():
    iteration = 0
    current = None
    best = None
    while can_continue(current, iteration):
        iteration += 1
        var = select_variable(current)
        value = select_value(current,variable)
        # check for consistency of the constraints and
        # remove assignment of conflicting variables
        unassign(current, conflicting(current, variable, value))
        assign(current,variable,value)
        if E(current) > E(best):
            best = current
    return best
    
```

- conflicting statistics
    - remember how many times $[A=a \implies  B\neq b]$
    - when selecting a value
        - choose the values with the lowest number of conflicts weighted by their frequency
            - count a conflict if it leads to the removal of the assignment

# Comparison of search algorithms

![](https://s3.hedgedoc.org/demo/uploads/652bb8b2-12b4-402a-9a06-11860bcbd5b8.png)

*all nodes in $B$ are also in $A$ for $A\rightarrow B$*

- benchmarking
    - collection of real problems
    - randomly generated problems
- phase transition problem 
    - SAT
        - with a small number of clauses most problems satisfiable and easily solvable
        - with a large number of clauses most problems unsatisfiable, unsatisfiability can be easily detected
        - finding a solution is most difficult given that about 50 % of problems are satisfiable 

# Optimization and soft constraints


**Constraint optimization problem** is CSP with objective function from solutions to $\mathbb{R}$.



## COP in operational research

- **constraint optimization problem (COP): $(X , D, C_h, C_s)$**
    - $C_h=\{C_1,...,C_n\}$ is set of hard constraints
        - $C_i$ defined on $Y_i\subseteq X, Y_i = \{x_{i_1},...,x_{i_k}\}$
        - $C_i$ is a subset of $D_{i_1}\times ...\times D_{i_k}$
    - $C_s=\{F_1,...,F_n\}$ is set of soft constraints
        - $F_j$ defined on $Q_j\subseteq X, Q_j = \{x_{j_1},...,x_{j_l}\}$
        - $F_j$ is a function $D_{j_1}\times ...\times D_{j_l} \rightarrow \mathbb{R^+}$
    - objective function $$F(\vec{d})=\sum_{j=1}^l F_j(\vec{d}[Q_j])$$ where $\vec{d}[Q_j]$ is a projection of $\vec{d}$ to $Q_j$
    - solution: $\vec{d^o}$ satisfying all constraints of $C_h$ such that $F(\vec{d^o})=max_{\vec{d}}F(\vec{d})$ (min for minimization problem)
- problems
    - over-constrained problems: solution of CSP does not exist 
    - problems with uncertainties
    - ill-defined problems: it is not clear which constraints define the CSP

## Approaches for soft constraints

![](https://s3.hedgedoc.org/demo/uploads/606ae9b7-4a65-46b9-a2da-c16f659d5f84.png)


**Weighted CSPs**

- weight/cost associated with each constraint
- constraint: pair $(c,w(c))$
- problem: $(V,D,C_w)$
- satisfaction degree: function on the set of asignments, sum of weights of unsatisfied constraints $$\omega(\theta)=\sum_{\theta\models \neg c} w(c)$$
- solution: the assignment $\theta$ with mimimum $\omega(\theta)$
- degree of consistency: satisfaction degree of a solution

**MAX-CSP**

- weighted CSP with weights 1, maximizing the number of satisfied constraints

**Fuzzy CSP**

- fuzzy sets: element membership to the set is given by the number from interval (0, 1)
- fuzzy constraints: fuzzy relation $\mu_c(d_1,...,d_k)\in (0,1)$ indicates preference level
- tuple projection: $(d_1,...,d_l)\downarrow_X^Y$ where $X\subset Y$ and $d_i$ value of variable $i$
- combination: $c = c_X \oplus c_Y, dom(c) = Z = X \cup Y$ where $c,c_X,c_Y$ are constraints on sets of variables $Z,X,Y$
    - $\mu_c(d_1,...,d_k)=min(\mu_{c_X}(d_1,...,d_k)\downarrow_X^Z,\mu_{c_Y}(d_1,...,d_k)\downarrow_Y^Z)$
    - specifies satisfation degree for all assignments from $Z$ wrt. $c_X$ and $c_Y$
- projection: $c= c_Y \Downarrow_X, dom(c)=X,X\subseteq Y$ where $c,c_X,c_Y$ are constraints on $X,X,Y$   
    - $\mu_c(d_1,...,d_k)=max_{((d_{y_1},...,d_{y_l})\in D_{y_1}\times ...\times D_{y_l})\land ((d_{y_1},...,d_{y_l})\downarrow^Y_X=(d_{x_1},...,d_{x_k}))}(\mu_{c_Y}(d_{y_1},...,d_{y_k}))$
    - specifies degree of consistency for all assignments from $X$ wrt. $c_Y$
- satisfaction degree of an assignment $(d_1,...,d_n)$ is given as $\mu_{\oplus C}(d_1,...,d_n)$
- solution: assignment $(d_1,...,d_n)$ such that $$max_{(d_1,...,d_n)\in D_1\times ...\times D_n} \mu_{\oplus C}(d_1,...,d_n) = \mathbb{C}(P_{\mu})$$
    - $\mathbb{C}(P_{\mu})$ the degree of consistency
        - also given by the projection to the empty set of variables $\oplus C \Downarrow \emptyset$
        - inconsistency degree $1-\mathbb{C}(P_{\mu})$
- CSP ≡ fuzzy CSP on two-element $A$ (below)
    - fuzzy CSP can be polynomially transformed to weighted CSP

**Semiring-based CSP**

- $c$-semiring $<A,+,\times,0,1>$
    - $A$ semiring set (preference set)
    - $0\in A$ complete unsatisfaction, $1\in A$ complete satisfaction
    - $+$ commutative, idempotent ($a+a=a$), associative operations with unit element 0, with absorbing element 1
    - $\times$ commutative, associative operations, distributive over $+$, with unit element 1, with absorbing element 0
        - used to combine the preferences of several constraints
        - corresponds to min in fuzzy sets CSP
    - partial ordering $\leq_S$: $a\leq_S b$ when $a+b=b$
        - used to select "better" assignment
        - corresponds to max in fuzzy CSP
- semiring-based CSP: definitions
    - system $(S,D,V)$
        - $S$ $c$-semiring, $D$ finite domain, $V$ ordered set of variables
    - soft constraint $(def,con): con \subseteq V, def : D^{|con|}\rightarrow A$ 
    - soft problem $P$: $P=(C,con)$ over $(S,D,V)$ where $con\subseteq V$,$C$ set of constraints
    - tuple projection $\vec{t}\downarrow_X^Y$
    - combination $c=c_1 \otimes c_2$ where $c_1=(def_1,con_1),c_2=(def_2,con_2),c=(def,con), con = con_1 \cup con_2$ and $$def(\vec{t})=def_1(\vec{t}\downarrow_{con_1}^{con}) \times def_2(\vec{t}\downarrow_{con_2}^{con})$$
    - projection $c \Downarrow_l, c = (def, con),l \subseteq V, c'= (def',con'), con'= con \cap l$ $$def'(\vec{t})= \sum_{\vec{t}:\vec{t}\downarrow_{l\cap con}^{con}} def(\vec{t})$$
    - satisfaction degree of problem $P=(C,con)$ is given by constraint $sol(P)=(\otimes C)\Downarrow_{con}$
        - the combination of all constraints in $C$ and then the projection onto the variables in $con$
    - degree of consistency: $blevel(P)=sol(P)\Downarrow \emptyset$

## Optimization & soft constraints: algorithms

- soft propagation
    - propagation of preferences (costs) over $k$-value tuples of variables
    - trying to get more realistic preferences
    - calculating more realistic contributions to the objective function
- $C$ is soft $k$-consistent if for all subsets of $(k-1)$ variables $W$ and any other variable $y$ holds $$\otimes \{c_i|c_i \in C \land con_i\subseteq W\}=(\otimes\{c_i|c_i \in C \land con_i\subseteq W\})\Downarrow_W$$
    - $con_i$ denotes variables in constraint $c_i$
- basic algorithm for soft arc consistency (SAC) calculation
    - for each $x$ and $y$ change the definition of $c_x$ so that is corresponds to $(\otimes_{c_x,c_y,c_{xy}})\Downarrow_x$
    - iteration until stability
    - if $\times$ is idempotent operation (fuzzy CSP)
        - ensures equivalence, i.e., the original and the new (after reaching SAC) problem have the same solution
        - ensures termination of the algorithm
    - if $\times$ is not idempotent operation (weighted CSP)

## How to solve constraint optimization problem (COP)

- objective - finding the complete optimal solution
- search
    - local
        - direct inclusion of the optimization criterion in the evaluation
    - tree
        - branch and bount + its extensions
- *weighted CSP*  
    - minimizing the sum of constraint weights
    - weight: 0 full satisfaction, $(0,\infty)$ partial satisfaction, $\infty$ failure
    - value of objective function: assignment/solution cost
    - algorithms for fuzzy CSP on similar principles
    - trivial solution
        - COP a series of CSP problems
        - initial value of objective $W^0$
        - in each iteration, add constraint that objective should be smaller than $W^i$
        - if no new solution of CSP exists, then the last $W^i$ gives the optimum 

### Branch and bound

- generic algorithm extensible as an implementation of backtracking
    - possible extensions: look-ahead, lower bound calculation
- lower bound $LB(\vec{d})$ returns a lower bound cost estimate for each partial assignment $\vec{d}$
    - used when expanding by one variable $LB(\vec{a_{i-1}},a)$

![](https://s3.hedgedoc.org/demo/uploads/ceb91382-ec2b-489a-b50c-3902db64f237.png)

- lower bound quality very important for the efficiency of BB
- possible to find the optimal solution fast, but proof of optimality (explore of the rest of the search tree) takes time
- we can find just "good enough" solution
- speed up using both upper and lower bounds
    - new bound $TempB = (UB+LB)/2$
        - if solution not found, set $LB$ to $TempB+1$
- lower bound can be influenced by
    - good heuristics: good propagation through the objective constraint
	- good solution found early: using an initial bound may help
	- costs of past variables
	    - distance: sum of constraint weights on past variables
	- local costs of future variables relative to past variables:
	    - NC∗
	- local costs of future variables:
	    - AC∗
	- global costs of future variables:
	    - Russian doll search 

---

## Poznámky z Rozvrhování (část relevantní pro CSP)

### 11 Problém splňování podmínek

#### 11.1. V kontextu omezujících podmínek definujte množinu doménových proměnných a doménu.

- množina doménových proměnných - množina proměnných, kterým chceme přiřadit nějaké hodnoty z předem definovaných množin (domén)
- doména - jaké všechny možné hodnoty mohou nabývat proměnné, tedy $D = D_1 \cup D_2 \cup ... \cup D_n$ kde $D_i$ je doména proměnné $y_i$

#### 11.2. Definujte pojem omezení v kontextu programování s omezujícími podmínkami. Co musí platit, aby se omezení nazývalo splněné? Uveďte příklad omezení a ukažte, kdy je splněno a kdy není splněno.

- omezení je podmínka určující jakých hodnot mohou proměnné nabývat současně
    - formálně jde o podmnožinu kartézského součinu $D_1 \times D_2 \times ... \times D_n$
- př. všechny hodnoty musí být navzájem různé, splněno pro $x_1=1$, $x_2=2$, nesplněno pro $x_1=x_2=2$

#### 11.3. Popište problém splňování podmínek. Jak je definováno jeho řešení? Ukažte příklad problému splňování podmínek, který má právě dvě řešení.

Problém splňování podmínek je dán trojicí $(V,D,C)$ kde $V$ jsou porměnné, $D$ domény a $C$ podmínky. Řešením je přiřazení hodnot všem proměnným t.ž. není porušena žádná podmínka. 

Formálně: řešení $(d_1,...,d_n) \in D_1 \times ... \times D_n$, $\forall c_i \in C$ na $v_{i_1}...v_{i_k}$ platí $(d_{i_1}...d_{i_k}) \in c_i$

Příklad: $A\in\{0,1\}$, $B\in\{0,1\}$, $A \neq B$, řešení $A=1,B=0$ nebo $A=0,B=1$

#### 11.4. Uveďte konkrétní příklad problému splňování podmínek a na něm ukažte pojmy množina doménových proměnných, doména, omezení a řešení problému splňování podmínek. Příklad problému splňování podmínek navrhněte tak, aby měl alespoň pět proměnných a pět omezení.

- proměnné: A,B,C,D,E jsou vrcholy grafu, hrany AB, BC, CA, DE
- domény: pro každou proměnnou {0,1,2}, celkem doména {0,1,2}
- omezení: sousedé musí mít různé hodnoty, hodnota A musí být větší než hodnota B, hodnota D musí být větší než hodnota A, hodnota E musí být různá od A

#### 11.5. Co je to filtrace domén? K čemu slouží při řešení problému splňování podmínek? Uveďte příklad filtrace domény.

Během filtrace domén odebíráme z domén jednotlivých proměnných hodnoty, o kterých víme, že jich za daných podmínek nemohou nabývat.

#### 11.6. Co je to hranová konzistence (AC)? Kdy nazýváme problém splňování podmínek (CSP) hranově konzistentní? Ukažte příklad problému, který je hranově konzistentní a příklad problému, který není hranově konzistentní.

Podmínka je hranově konzistentní, pokud pro každou hodnotu každé její proměnné existuje kombinace hodnot pro další proměnné v podmínce t.ž. je podmínka splněna. Každá hodnota je tedy podporována. Během revizní procedury odstraňujeme z domén hodnoty, které nemají podporu. 

Příklad: mějme proměnné A,B,C, podmínka A=B+C
- konzistentní: A {2,3}, B,C {1,2}
- nekonzistentní: A {1,2,3}, B,C {1,2} protože hodnota 1 proměnné A nemá podporu

#### 11.7. Jak se v problému splňování podmínek zajišťuje hranová konzistence? Napište algoritmus AC-8 pro zajištění hranové konzistence. Čím se liší od algoritmu AC-3?

- postupně revidujeme podmínky a upravujeme domény (vyhazujeme proměnné, které nemají podporu)
- procedura AC-8 vytvoří frontu proměnných k revizi (na počátku všechny proměnné)
    - dokud fronta není prázdná, vybereme nějakou proměnnou
    - pro každou podmínku, který se proměnné týká, provedeme filtraci
    - pokud se po filtraci mohla změnit doména i jiné proměnné, tak přidáme do fronty
    - pokud se nějaká doména vyprázdní, tak fail

#### 11.8. Použijte algoritmus AC-8 pro zajištění hranové konzistence na problém splňování podmínek:
- $A, B, C \in \{1, 2, 3, 4\}$

||podmínky|
|---|---|
|c1 | $A \neq B$ |
|c2 | $B \neq C$ |
|c3 | $A + B = 4$|
|c4 | $B + C = 3$|

V algoritmu použijte uspořádání proměnných A, B, C při zařazování proměnných do fronty a uspořádání omezení c1, c2, c3, c4 při výběru k revizi. Ve svém řešení uveďte vždy stav fronty, všechny provedené revize omezení (i když nedojde ke změně domén) a domény proměnných, jejichž doména se změnila.

- Q: A,B,C
- vyberu A
- c1 nic
- c3 A odeberu 4, B odeberu 4, přidám A do fronty (B už tam je)
- $C \in \{1, 2, 3, 4\}, A, B \in \{1, 2, 3\}$
-
- Q: B,C,A
- vyberu B
- c1 nic
- c2 nic
- c3 nic
- c4 B odeberu 3, C odeberu 3,4, přidám B do fronty
- $B, C \in \{1, 2\}, A \in \{1, 2, 3\}$
- 
- Q: C,A,B
- vyberu C
- nic
- $B, C \in \{1, 2\}, A \in \{1, 2, 3\}$
- 
- Q: A,B
- vyberu A
- c1 nic
- c2 nic
- c3 A odeberu 1
- c4 nic
- $B, C \in \{1, 2\}, A \in \{2, 3\}$
- 
- Q: B
- vyberu B
- nic
- $B, C \in \{1, 2\}, A \in \{2, 3\}$
- 
- výsledek: $B, C \in \{1, 2\}, A \in \{2, 3\}$

#### 11.9. Uveďte kostru algoritmus přiřazování (labeling) používaný při řešení problému splňování podmínek. Jaký typ prohledávání se zde používá?

Používá se DFS. Zkusíme přiřadit hodnotu proměnné, uděláme problém konzistentní, v případě neúspěchu zkusíme přiřadit jinak.

#### 11.10. Proč není pro řešení problému splňování podmínek dostačující algoritmus pro hranovou konzistenci a musí být doplněn prohledávacím algoritmem?

Protože některé situace neumíme hranovou konzistencí rozhodnout. Například pro trojúhelník s vrcholy A {0,1}, B {0,1}, C {0,1} a podmínky, že každý vrchol má jiné ohodnocení, než jeho sousedi, je sice problém hranově konzistentní, ale nemá řešení, což najdeme pomocí labelingu.

#### 11.11. Popište techniku pohledu dopředu (MAC) pro prohledávání při řešení problému splňování podmínek.

- inicializace - konzistenční alg., abychom dosáhli úvodní konzistence
- dokud nejsou všechny proměnné ohodnoceny:
    - vyber neohodnocenou proměnnou
    - pro každou možnou hodnotu zkus ohodnotit
        - pokud konzistence ok: R = rekurze
        - pokud R not fail, vrať R
    - vrať fail

#### 11.12. Navrhněte a napište modifikaci techniky pohledu dopředu (MAC) tak, aby používala větvení stavového prostoru typu (Promenna=Hodnota ∨ Promenna =\= Hodnota).

Když ohodnocení selže, tak tuto hodnotu odeberu z domény dané proměnné.

#### 11.13. Použijte algoritmus MAC (technika pohledu dopředu) na problém splňování podmínek:
• A ∈ {0, 1}, B ∈ {0, 1, 2, 3}, C ∈ {0, 1, 2}
• A =\= B, A =\= C, A < B, A + B + C = 4
• při výběru proměnné preferujte tu, která má nejmenší doménu
• preferujte hodnoty, které jsou více podporovány v součtu přes všechny podmínky, při stejném součtu preferujte menší hodnoty v doméně

- konzistence - odebereme 0 z domény B, protože nemá podporu v poslední podmínce
- chceme přiřadit hodnotu pro A - pro 0 je 3 + 2 + 3 + 2 = 10 podpor, pro 1 je 2 + 2 + 2 + 3 = 9 podpor, přiřadíme 0
- konzistence - odebereme 0 z domény C a 1 z domény B
- chceme přiřadit hodnotu B, podpory mají stejně, přiřadíme 2
- konzistence - odstraníme 1 z domény C
- výsledek: A=0, B=2, C=2

#### 11.14. First-fail a succeed-first

- fail-first se využívá k výběru toho, kterou proměnnou ohodnotíme
    - definuje tvar prohledávacího stromu
- succeed-first se využívá k výběru hodnoty, kterou ji ohodnotíme
    - definuje pořadí procházení větví