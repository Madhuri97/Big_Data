# Advanced MapReduce Techniques

## Question 1: There are two datasets: A is the large one, B is small enough to fit in the memory of the cluster node. What type of join do you choose to make their intersection A&B?
## Records in A: keyA, valueA
## Records in B: keyB, valueB
## Records in the result:
## key (=keyA=keyB), valueA, valueB
    Map

## Question 2: There are two datasets: A is the large one, B is small enough to fit in the memory of the cluster node. What type of join do you choose to make the difference A\B (records from A not found in B)?
## A: keyA, valueA
## B: keyB, valueB
## Result: keyA, valueA, null
    Map

## Question 3: There are two datasets: A is the large one, B is small enough to fit in the memory of the cluster node. What type of join do you choose to make the difference B\A (records from B not found in A)?
## A: keyA, valueA
## B: keyB, valueB
## Result: keyB, null, valueB
    Reduce

## Question4: There are two datasets: A is the large one, B is small enough to fit in the memory of the cluster node. What type of join do you choose to make the union A U B (records from A or from B or from the both datasets)?
## A: keyA, valueA
## B: keyB, valueB
## Result has three types of records:
## keyA, valueA, null
## keyB, null, valueB
## key (=keyA=keyB), valueA, valueB
    Reduce

## Question 5: How do you distinguish records of two datasets on the Reduce phase? 
    By a some tag added to the records on the Map phase; tags are selected by the filename from the environment
    
    By format of the values

## Question 6: What parameters do you specify in the Hadoop Streaming command to perform Secondary sort (i.e. sort by two fields of the key, partition by the first field)?
    -D stream.num.map.output.key.fields=2

    -D mapred.output.key.comparator.class=org.apache.hadoop.mapred.lib.KeyFieldBasedComparator

    -D mapred.text.key.comparator.options=-k1,2

    -D mapred.text.key.partitioner.options=-k1,1

    -partitioner org.apache.hadoop.mapred.lib.KeyFieldBasedPartitioner

## Question 7: When is Secondary Sort really useful?
    When you join two datasets with a Reduce-side join and one of them has many records with repeating keys

    When you want to avoid containers in memory on the reducers and therefore decrease the memory required by your tasks.

## Question 8: In what type of join could this code be used?
## for line in sys.stdin:
##    key, value = line.strip().split('\t', 1)
##    if key in dataset:
##        print "%s\t%s" % (key, value)
    Map

## Question 9: In what type of join could this script be used?
## for line in sys.stdin:
##    key, tag, value = line.strip().split('\t', 2)
##    if key != current_key:
##        if current_value is not None:
##            print “%s\t%s” % (current_key, current_value)
##        current_key = key
##        current_value = None
##        if tag == 0:
##            continue	
##    current_value = min(current_value, value) if current_value is not None else value   
## if current_value is not None:
##    print “%s\t%s” % (current_key, current_value)
    Reduce

## Question 10: What file is in the output directory of the succeeded MapReduce job (input the exact filename)?
    _SUCCESS
