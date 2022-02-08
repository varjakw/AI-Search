# Search in Prolog
We can describe computations as a Search. These two lines of code below are vital:
Given ```goal``` and ```arc```
```
search(Node) :- goal(Node).
search(Node) :- arc(Node,Next), search(Next)
```

The first line is you are trying to find a goal node. If you aren't in a place where you find this goal node, you move using arcs in line 2. Its then a recursive call to search. The arc predicate can be non-deterministic, meaning sometimes you must make a choice. Intelligence is about making choices. Frontier Search is a way that one can deal with these choices.

# Frontier Search
First, lets see a simple graph and how it can be described in Prolog.
![image](https://user-images.githubusercontent.com/78870995/153012659-2d21620f-acc2-4305-9ba3-8a0df2c741e0.png)

The 9 facts of arcs describe the connections between nodes.

The arc2 predicate says you can either go from X to Y or from Y to X.




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
