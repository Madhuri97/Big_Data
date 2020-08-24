# Excersice 2.3.3--> In the form of relational algebra implemented in SQL, relations are not sets, but bags; that is, tuples are allowed to appear more than once. There are extended definitions of union, intersection, and difference for bags, which we shall define below. Write MapReduce algorithms for computing the following operations on bags R and S:

## Bag Union, defined to be the bag of tuples in which tuple t appears the sum of the numbers of times it appears in R and S.
    Map : for each tuple t in R and S emit key-value pair(t,t)
    Reduce : input → (t, t) output → (t, t) (identity)

## Bag Intersection, defined to be the bag of tuples in which tuple t appears the minimum of the numbers of times it appears in R and S.
    Map : for each tuple t in R emit key-value pair((t, R), 1) & for each tuple t in S emit key-value pair((t, S), 1)
    Reduce : input → ((t, R), [1,..,1] ) or ((t, S), [1,..,1]) output → ((t, R), sum[1,..,1]=m) or ((t, S), sum[1,..,1] = n)
    Map : input → ((t, R), m) or ((t, S), n) output → (t, m) or (t, n)
    Reduce : input → (t, [m, n]) output → (t, min(m , n) = d)
    Map : input → (t, d) output → (t, d)
    Reduce : input → (t, d) output → produce d tuple(t, t)

## Bag Difference, defined to be the bag of tuples in which the number of times a tuple t appears is equal to the number of times it appears in R minus the number of times it appears in S. A tuple that appears more times in S than in R does not appear in the difference.
    Map : for each tuple t in R emit key-value pair((t, R), 1) & for each tuple t in S emit key-value pair((t, S), 1)
    Reduce : input → ((t, R), [1,..,1] ) or ((t, S), [1,..,1]) output → ((t, R), sum[1,..,1]=m) or ((t, S), sum[1,..,1] = n)
    Map : input → ((t, R), m) or ((t, S), n) output → (t, m) or (t, n)
    Reduce : input → (t, [m, n]) output → (t, m - n = d)
    Map : input → (t, d) output → (t, d)
    Reduce : input → (t, d) output → produce d tuple(t, t) (if d =< 0, produce nothing)
