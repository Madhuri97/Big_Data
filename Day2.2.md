# Excersice 2.2 

## Suppose we execute the word-count MapReduce program described in this section on a large repository such as a copy of the Web. We shall use 100 Map tasks and some number of Reduce tasks.

### 1)	Suppose we do not use a combiner at the Map tasks. Do you expect there to be significant skew in the times taken by the various reducers to process their value list? Why or why not?

There will be significant skew since some keys will have a large number of occurrences while some have less occurrences, so different reducers take different amounts of time. We can take an example from real world dictionaries where the word distribution follows power law. So, if we reduce based on keys, the skew in time is bound to happen.

### 2)	If we combine the reducers into a small number of Reduce tasks, say 10 tasks, at random, do you expect the skew to be significant? What if we instead combine the reducers into 10,000 reduced tasks?

The skew will be present but not as in the first case. After combination of reducers to some reduce tasks cause an averaging over execution times of several reducer tasks. If we combine 10,000 reduce tasks, then skew may be even lesser as way, long reduce tasks might occupy a compute node fully, while several shorter Reduce tasks might run sequentially at a single compute node.

### 3)	Suppose we do use a combiner at the 100 Map tasks. Do you expect skew to be significant? Why or why not?

As per my knowledge the skew will be less. Since words will be combined in the mapping phase only. So during the reduce phase not very large value lists will be present to be reduced.
