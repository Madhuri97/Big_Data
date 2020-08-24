# Exercise 2.3.1: Design MapReduce algorithms to take a very large file of integers and produce as output:

## a) The largest integer.
### The map function is used to get the various combinations of the largest number in different files, each file representing as single chunk which produces output as key-value pair form
	Map(file_name,  value):
		    emit(‘large’, max(value))
	Then the grouping by is done for the emitted output by map function
### The reduce function takes the key-valuelist pair and again finds the largest which provides output as a single key-value pair with the largest of all chunks.
	Reduce(key, values):
		emit(‘largest’, max(values))
 
## b) The average of all the integers.
Map(file_name, value):
	Sum = 0 
	Count = 0
	for num in value:
		Sum += num
		Count += 1
	emit(‘sum_of_numbers’, (Count,Sum))
Reduce(key, values):
	Total = 0
	Totalcount = 0
	for (Count,Sum) in values:
		Total += Sum
		Totalcount += Count
	emit(‘average’, Sum/Count)
 
The same set of integers, but with each integer appearing only once.
the map function:for each integer in s such t,produce key-value pair (t,t)
reduce function:For each key t produced by any of the Map tasks, there will be one or more key-value pairs (t,t). The Reduce function turns (t,[t,t,...,t]) into (t',t'), so it produces exactly one pair (t,t) for this key t.
Map(file_name, value):
	for num in value:
		emit(num, 1)

After map function the shuffle step will group all same values together like integer: (int, [1, 1, 1, ...])
Reduce(key, values):
	emit(uniq_number, 1)
Eliminate duplicates for each integer key and emit integer

## c)The count of the number of distinct integers in the input.
### Continuation to previous solution map2:for each (t,t) output (s,1) reduce 2:after grouping we have (s,[1...1]),output (s,sum[1,.....,1])
	Map1(file_name, value):
		for num in value:
			emit(num, 1)
	Reduce1(uniq_number, values):
		emit(uniq_number, 1)
	Map2(num, value):
		for num in value:
			emit(num, 1)
	Reduce2(uniq_number, values):
		emit(uniq_number, sum(values))
