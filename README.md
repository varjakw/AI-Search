# Search in Prolog
Given ```goal``` and ```arc```
```
search(Node) :- goal(Node).
search(Node) :- arc(Node,Next), search(Next)
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
