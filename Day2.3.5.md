# The relational-algebra operation R(A, B) Join S(C, D) with B < C produces all tuples (a, b, c, d) such that tuple (a, b) is in relation R, tuple (c, d) is in S, and b < c. Give a MapReduce implementation of this operation, assuming R and S are sets.

    Map: 
    for each tuple (a, b) of R, produce the key-value pair (1,((R,b) , (R, a))). 
    for each tuple (c,d) of S, produce the key-value pair (1,((S,c),(S,d)))
    Reduce: 
    input →  (1,[((R,b),(R, a)),....,((S,c),(S,d)),...] 
    output → Construct all pairs consisting of one with first component R and the other with first component S, The output for key (b,c) is ((b,c),((R,a),(S,d))
    Map:
    input → ((b,c),((R,a),(S,d)) 
    outpt → (if b<c ) ((b,c),((R,a),(S,d))
    Reduce: 
    Input →  ((b,c),((R,a),(S,d)) 
    Output →  (a,b,c,d)
