## Suppose a job consists of n tasks, each of which takes time t seconds. Thus, if there are no failures, the sum over all compute nodes of the time taken to execute tasks at that node is n*t. Suppose also that the probability of a task failing is p per job per second, and when a task fails, the overhead of management of the restart is such that it adds 10t seconds to the total execution time of the job. What is the total expected execution time of the job?

Minimum time to complete the execution = n*t
Probability of a task failing per second = p
Let the number of tasks failed = tn
The probability of x tasks failing  = tn*p*t
When a task fails, it adds 10t seconds to the execution time
Total Execution time of failed tasks = tn*p*t + 10*p*t = (tn+10)p*t
Total execution time of the job = n*t + (tn+10)p*t
