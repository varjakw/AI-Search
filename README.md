# Search in Artificial Intelligence
Computations can be considered search and intelligence refers to choices in these searches


# Knowledge Base (KB) and arc
![image](https://user-images.githubusercontent.com/78870995/153007453-f30040c6-daa3-426c-b047-a5d8bf185236.png)

```
arc(Node1,Node2,KB) :- ???

arc([H|T],N,KB) :- member([H|B],KB), append(B,T,N).
prove(Node,KB) :- goal(Node) ; arc(Node,Next,KB), prove(Next,KB).
```



# Search in Prolog
We can describe computations as a Search. These two lines of code below are vital:
Given ```goal``` and ```arc```
```
search(Node) :- goal(Node).
search(Node) :- arc(Node,Next), search(Next)
```

The first line is you are trying to find a goal node. If you aren't in a place where you find this goal node, you move using arcs in line 2. Its then a recursive call to search. The arc predicate can be non-deterministic, meaning sometimes you must make a choice. Intelligence is about making choices. Frontier Search is a way that one can deal with these choices.


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

We have a start node, and we are trying to reach some goal node. We are going to keep track of a frontier of candidate goal nodes. The ```arc``` predicate is a hard constraint on what moves we can make. The frontier is a list of candidate goal nodes. Rather than eliminating the choices, we will be managing the choices using the predicate ```add2frontier```. This tells us how to make our choices. We are still working with a Node List but we are going to explore the nodes one at a time. Just like the ```searchD``` predicate previously, the frontier will always be a on-empty list and we are always examining the head. The base case of ```frontierSearch``` is 'is the head of the list the goal node? If yes, we are done.'. Otherwise,  our inductive case where we have the head and the rest of the frontier. We collect all the ndoes we can get to from the node here, and put them into ```children```. Then we add those children to the rest of the frontier. Then we have the tail recursion to run frontierSearch with the new frontier.

Its the same as the first Search method we looked at. First you have a search that checks if the current node is the node we want, and then you have the recursive one with an arc and search call with the next node. Except in the case of frontier search, the arc predicate is essentially given by ```findall``` and ```add2frontier```. 

```
search(Node) :- frontierSearch([Node]).
frontierSearch([Node| ]) :- goal(Node).
frontierSearch([Node|Rest]) :- 
          findall(Next, arc(Node,Next), Children),
          add2frontier(Children,Rest,NewFrontier),
          frontierSearch(NewFrontier).
```

## Breadth-first i.e. a FIFO queue

![image](https://user-images.githubusercontent.com/78870995/153713891-01f51219-9cff-46a8-a78b-166e4e5e201e.png)

The children of the head are only explored if the rest of the frontier is empty. If the frontier is not empty, we want to explore the rest of the nodes of the frontier before the children. Thats what this code is doing.

## Depth-first i.e. LIFO stack
This method leads to the non-termination we saw previously. (the infinite red path)

![image](https://user-images.githubusercontent.com/78870995/153714047-d7f74351-90cf-4675-92b6-a91523e0711d.png)

For depth-first, the rest of the frontier is only investigated if there arent any chidlren of the head. If the head has children, you examine them first.

## Explaining cut (destroys backtracking)

![image](https://user-images.githubusercontent.com/78870995/153714603-37ed17bb-6c48-4b50-bacd-c4186a8eee8f.png)

We are commited to q, meaning we will get a no at the end for the query seen. If we didn't have a cut, we would be able to get to r and would get a yes for the query:

![image](https://user-images.githubusercontent.com/78870995/153714634-a3f3c8dd-84ae-471a-86dc-9af052a34af8.png)

Cut is useful for if-then-else.  How do we understand cut in terms of frontier search?

## Depth-first as frontier search
```
prove([], ). % goal([]).
prove(Node,KB) :- arc(Node,Next,KB), prove(Next,KB).
```

```
fs([[]| ], ).
fs([Node|More],KB) :- findall(X,arc(Node,X),L),
append(L,More,NewFrontier),
fs(NewFrontier,KB).
```
For ```fs``` (frontier search), we move from a single node to a node list. And if that goal ndoe happens to be the head of our list, then we are done. 

How do we bring cut into this? Lets track the frontier to see.

![image](https://user-images.githubusercontent.com/78870995/153714740-2a354aa4-b11c-4c51-ac2d-0c18e34ff680.png)
Our frontier consists of a list of nodes. There are two brackets around i because the node happens to be a list. A list of ndoes becomes a list of lists.

![image](https://user-images.githubusercontent.com/78870995/153714770-b99f20e7-8130-48ac-9bba-4926388dc61f.png)

![image](https://user-images.githubusercontent.com/78870995/153714779-d86c55bd-e801-411e-909a-80ca80e5c253.png)

We have lost r from our frontier. 

![image](https://user-images.githubusercontent.com/78870995/153714794-63bb0ed0-592d-4ea5-9e9c-6442076721ca.png)

This empty list we are left with is bad news because it means there aren't any candidate nodes left so our frontier is empty. Usually you might htink having an empty list is good news because it means we at a goal node but the problem is that here, its representing an empty frontier with no nodes in it. So how do we encode the cut in a frontier search?

## Cut via frontier depth-first search
```
fs([[]| ], ).
fs([[cut|T]|_],KB)) :- fs([T],KB).  % _ is rest of frontier
fs([Node|More],KB) :- Node = [H| ], H\== cut,
                      findall(X,arc(Node,X),L),
                      append(L,More,NewFrontier),
                      fs(NewFrontier,KB).

%reminder of if-then-else and negation-as-failure
if(p,q,r) :- (p,!,q); r.    % contra (p,q);r
negation-as-failure(p) :- (p,!,fail); true.
```

At line 2 cut is getting rid of the rest of the frontier (which is given by the underscore). As such we have to modify line 3 to ```Node = [H| ], H\== cut``` in case the head of our node isnt a cut. If its a cut, we succeed but get rid of the rest of the frontier. But here we are keeping track of the rest; the ```More``` in the append is the response to the underscore.

See GoalNodes repo for exercise.

# Costs & Heuristics

![image](https://user-images.githubusercontent.com/78870995/154045281-ceeb6c2e-2907-4b2b-ade8-5ace29b6deff.png)

```add2frontier(Children, Rest, [Head|Tail])``` where ```[Head|Tail]``` is the NewFrontier we are constructing from Children and Rest. 

# A*
![image](https://user-images.githubusercontent.com/78870995/154046086-2e46f943-96d1-48b0-9f45-bf3950366239.png)

See AStar.pl for a Prolog implementation of A* search algorithm.

A solution is a path which is a list of nodes connected by links (arcs) to a goal node. 

The crucial property of A* is that it ensures ```Frontier = [Head|Tail]``` where ```Head``` has minimal ```f```. The new frontier we are constructing has a head with minimal f value among all other nodes in the frontier. 

A* is admissibile under cost and heuristics if it returns a solution of minimum cost whenever a solution exists.

# Assignment 1

![image](https://user-images.githubusercontent.com/78870995/154078838-c38aae3b-ce23-4353-ad6a-d89f4daa8566.png)



# Source
Taken from Tim Fernando's Artificial Intelligence lectures at Trinity College Dublin.
