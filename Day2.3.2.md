# Excersice2.3.3--> Design MapReduce algorithm for matrix-vector multiplication where M is an r-by-c matrix for some number of rows r and columns c.

### We divide the matrix into vertical stripes. If we can not store in main memory, we just divide the matrix into square matrix and map reduce is applied 

Matrix-vector product is a vector x of length n xi = mij vj 
Matrix M and vector v stored in separate files in DFS
Compute nodes that are executing map task read vector v
Each map task operate on a chunk of matrix M
Map function: 
Apply to one element of M 
Produce key-value pair (i, mijvj)
All terms of the sum that make up component xi of the matrix-vector product will get the same key i.
Reduce function:
Sum up all the values associated with a given key i
Result is a pair (i, xi)

## Two stage map reduce:
	Stage 1:
        Map(key,line) =               // mapper for matrix M
            split line into 3 values: i, j, and v
            emit(j,new Elem(0,i,v))

        Map(key,line) =               // mapper for matrix N
            split line into 3 values: i, j, and v
            emit(i,new Elem(1,j,v))

        Reduce(index,values) =
        A = all v in values with v.tag==0
        B = all v in values with v.tag==1
        for a in A
        for b in B
        emit(new Pair(a.index,b.index),a.value*b.value)
    Stage 2:
        Map(key,value) = // do nothing
        emit(key,value)
        Reduce(pair,values) = // do the summation
        m = 0
        for v in values
        m = m+v
        emit(pair,pair.i+","+pair.j+","+m)

