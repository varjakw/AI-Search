# Search in Prolog
We can describe computations as a Search. These two lines of code below are vital:
Given ```goal``` and ```arc```
```
search(Node) :- goal(Node).
search(Node) :- arc(Node,Next), search(Next)
```

The first line is you are trying to find a goal node. If you aren't in a place where you find this goal node, you move using arcs in line 2. Its then a recursive call to search. The arc predicate can be non-deterministic, meaning sometimes you must make a choice. Intelligence is about making choices. Frontier Search is a way that one can deal with these choices.

# Searching
First, lets see a simple graph and how it can be described in Prolog.
![image](https://user-images.githubusercontent.com/78870995/153012659-2d21620f-acc2-4305-9ba3-8a0df2c741e0.png)

The 9 facts of arcs describe the connections between nodes.

The arc2 predicate says you can either go from X to Y or from Y to X.

## Non-termination due to poor choices
![image](https://user-images.githubusercontent.com/78870995/153658196-3f731a44-78a6-40ab-a9db-9a947fd59094.png)

![image](https://user-images.githubusercontent.com/78870995/153658481-2e07b78e-0f20-4a0d-b538-8f1f30169fc4.png)

On the left here, it follows this infinite path.
 
 ```
 prove([],_).
 prove([H|T],KB) :- member([H|B], KB), append(B,T,Next), prove(Next, KB)
 
 ?- prove([i],[[i,p,q],[i,r],[p,i],[r]]).
 ```
 The clauses can be examined using the prove predicate above.
 
 This part ```[[i,p,q],[i,r],[p,i],[r]] ``` of the query is a data representation of the program in the previous image (i if p,q etc).
 
 We get non-termination due to the infinite path. The path we want is from ```[i]``` to the empty list, but upon execution we never get to explore the black path. How do we get around that? The infinite path was followed due to a poor choice so we can eliminate choice altogether to fix this.
 
 ## Determinization (we eliminate choice altogether)
 
 A FSM ```[Trans, Final, Q0]``` such that for all ```[Q, X, QN]``` and ```[Q,X,Qn']``` in ```Trans```, ```Qn = Qn'```.
 
 These triples represent: Q = The current state you are at, X = the symbol you are scanning, Qn being the next state and Qn' meaning there could be two possible post-states. So ```Qn = Qn'``` is stating that they have to be the same. Therefore its deterministic. There's no choice.
 
 See FiniteStateMachine repo.
 
 We say the FSM is deterministic if the trasitions are determined by the Q and X here, meaning there is a unique next state. There is no choice in the matter here.
 
 Fact: Every FSM has a Deterministic Finite Automaton accepting the same language.
 
 Proof of this: Subset (powerset) construction
 
 Apply to  ```Arc ```, ```goal ```,  ```contra trans ``` and  ```Final```. 
 ```
 arcD(NodeList, NextList) :- setOf(Next, arcLN(NodeList,Next), NextList).
 arcLN(NodeList,Next) :- member(Node, Nodelist), arc(Node, Next).
 goalD(NodeLidt) :- member(Node,NodeList), goal(Node).
 searchD(NL) :- goalD(NL); (arcD(NL,NL2), searchD(NL2))
 search(Node) :- searchD([Node]).
 ```
 
 We have an arc between nodes i.e. between q0 and q1 and here we have ```arcD ``` which is an arc between node lists. ```setOf``` refers to the set of possible states it can be at.
 
 
 We can turn any finite automata into a deterministic one by going from Nodes to Node Lists. ```arcD``` means 'determinised arc'. ```arcLN``` refers to 'List to a Node'.
 
With ```goalD```, as long as that NodeList has a Node in it that happens to be a goal node, then we are done.

Then the search for this determinised version, ```searchD``` has the same form as search we saw before, where the base case is 'are we at a node we are happy with?'. This is a brute force way of doing it.

# Frontier Search
![image](https://user-images.githubusercontent.com/78870995/153713118-f65a0531-6e50-4679-be94-e5f0c82df7e3.png)

Rather than eliminating choice, we might try to manage choices instead, through Frontier Search.

We have a start node, and we are trying to reach some goal node. We are going to keep track of a frontier of candidate goal nodes. The ```arc``` predicate is a hard constraint on what moves we can make. The frontier is a list of candidate goal nodes. Rather than eliminating the choices, we will be managing the choices using the predicate ```add2frontier```. This tells us how to make our choices.

```
search(Node) :- frontierSearch([Node]).
frontierSearch([Node| ]) :- goal(Node).
frontierSearch([Node|Rest]) :- 
          findall(Next, arc(Node,Next), Children),
          add2frontier(Children,Rest,NewFrontier),
          frontierSearch(NewFrontier).
```






Example:
accept(Trans, Final, QO, String)
Node as [Q,UnseenString]
```
goal(Q,[],Final) :- member(Q,Final).
arc([Q,[H|T]],[Qn,T],Trans):- member([Q,H,Qn],Trans).

search(Q,S,F,_) :- goal(Q,S,F).
search(Q,S,F,T) :- arc([Q,S],[Qn,Sn],T), search(Qn,Sn,F,T).

accept(T,F,Q,S) :- search(Q,S,F,T).
```

# Search Example
```
i :- p,q  %rule1
i :- r.   %rule2
p.        %fact1
r.        %fact2

?- i.     %query
```
How such a query works:
![image](https://user-images.githubusercontent.com/78870995/153006594-21eda286-eca6-481a-af7e-b8eb0c840865.png)

```
?-i       StartNode = [i]     %query
yes       goal([])

prove(Node) :- goal(Node).
prove(Node) :- arc(Node,Next), prove(Next).
```
# Knowledge Base (KB) and arc
![image](https://user-images.githubusercontent.com/78870995/153007453-f30040c6-daa3-426c-b047-a5d8bf185236.png)

```
arc(Node1,Node2,KB) :- ???

arc([H|T],N,KB) :- member([H|B],KB), append(B,T,N).
prove(Node,KB) :- goal(Node) ; arc(Node,Next,KB), prove(Next,KB).
```

#  Prolog Notes

goal
```arc``` is two connected nodes. It refers to an edge in a graph. Like AB for example.
prove
member
append

# Source
Taken from Tim Fernando's Artificial Intelligence lectures at Trinity College Dublin.
