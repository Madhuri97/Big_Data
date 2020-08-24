# Exercise 2.4--> Suppose a job consists of n tasks, each of which takes time t seconds. Thus, if there are no failures, the sum over all compute nodes of the time taken to execute tasks at that node is n*t. Suppose also that the probability of a task failing is p per job per second, and when a task fails, the overhead of management of the restart is such that it adds 10t seconds to the total execution time of the job. What is the total expected execution time of the job?

Minimum time to complete the execution = n*t
Probability of a task failing per second = p
Let the number of tasks failed = tn
The probability of x tasks failing  = tn*p*t
When a task fails, it adds 10t seconds to the execution time
Total Execution time of failed tasks = tn*p*t + 10*p*t = (tn+10)p*t
### Total execution time of the job = n*t + (tn+10)p*t

# Suppose a Pregel job has a probability p of a failure during any superstep. Suppose also that the execution time (summed over all compute nodes) of taking a checkpoint is c times the time it takes to execute a superstep. To minimize the expected execution time of the job, how many super steps should elapse between checkpoints?

Let the number of supersteps = n
Let the execution time of superstep = t
The execution time of n supersteps = n*t
Execution time of n checkpoints = n*c*t
Then the execution time of taking a checkpoint = c*t
Probability of a failure during superstep = p
Probability of n supersteps failing = n*p*t
Total expected execution time = n*t + n*c*t + n*p*t
Let the minimum total expected execution time = execTime
Number of supersteps elapsed between checkpoints => 
n*t + n*c*t + n*p*t = execTime
n*t(1+c+p) = execTime
### n = execTime/t(1+c+p)
