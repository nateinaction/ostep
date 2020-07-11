# 1 CPU Intro Homework (Simulation)
## Nate Gay - July 11, 2020
----
1. The CPU utilization will be 100% because neither process will be waiting on IO.
```
./process-run.py -l 5:100,5:100 -c -p
Time    PID: 0    PID: 1       CPU       IOs
  1    RUN:cpu     READY         1
  2    RUN:cpu     READY         1
  3    RUN:cpu     READY         1
  4    RUN:cpu     READY         1
  5    RUN:cpu     READY         1
  6       DONE   RUN:cpu         1
  7       DONE   RUN:cpu         1
  8       DONE   RUN:cpu         1
  9       DONE   RUN:cpu         1
 10       DONE   RUN:cpu         1

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
```
2. It takes 9 clock tickss to complete the processes. 4 clock tickss spent on CPU for the first process. 1 clock ticks on CPU and 4 clock tickss waiting on IO for the second process.
```
./process-run.py -l 4:100,1:0 -c -p
Time    PID: 0    PID: 1       CPU       IOs
  1    RUN:cpu     READY         1
  2    RUN:cpu     READY         1
  3    RUN:cpu     READY         1
  4    RUN:cpu     READY         1
  5       DONE    RUN:io         1
  6       DONE   WAITING                   1
  7       DONE   WAITING                   1
  8       DONE   WAITING                   1
  9       DONE   WAITING                   1
 10*      DONE      DONE

Stats: Total Time 10
Stats: CPU Busy 5 (50.00%)
Stats: IO Busy  4 (40.00%)
```
3. Yes, switching the process order in the previous example matters because the processor can work on the 4 CPU clock tickss of the first process while passing the 4 clock tickss waiting on the second process to complete IO. The total clock tickss taken when swapping order will be 5.
```
./process-run.py -l 1:0,4:100 -c -p
Time    PID: 0    PID: 1       CPU       IOs
  1     RUN:io     READY         1
  2    WAITING   RUN:cpu         1         1
  3    WAITING   RUN:cpu         1         1
  4    WAITING   RUN:cpu         1         1
  5    WAITING   RUN:cpu         1         1
  6*      DONE      DONE

Stats: Total Time 6
Stats: CPU Busy 5 (83.33%)
Stats: IO Busy  4 (66.67%)
```
4. Setting the `-S` flag to `SWITCH_ON_END` will recreate the lenghty 9 clock ticks runtime from example 2 only swapping the order of the processes.
```
./process-run.py -l 1:0,4:100 -S SWITCH_ON_END -c -p
Time    PID: 0    PID: 1       CPU       IOs
  1     RUN:io     READY         1
  2    WAITING     READY                   1
  3    WAITING     READY                   1
  4    WAITING     READY                   1
  5    WAITING     READY                   1
  6*      DONE   RUN:cpu         1
  7       DONE   RUN:cpu         1
  8       DONE   RUN:cpu         1
  9       DONE   RUN:cpu         1

Stats: Total Time 9
Stats: CPU Busy 5 (55.56%)
Stats: IO Busy  4 (44.44%)
```
5. Using the `SWITCH_ON_IO` flag will reproduce the same results as seen in example 3 with a 5 clock ticks runtime.
```
./process-run.py -l 1:0,4:100 -S SWITCH_ON_IO -c -p
Time    PID: 0    PID: 1       CPU       IOs
  1     RUN:io     READY         1
  2    WAITING   RUN:cpu         1         1
  3    WAITING   RUN:cpu         1         1
  4    WAITING   RUN:cpu         1         1
  5    WAITING   RUN:cpu         1         1
  6*      DONE      DONE

Stats: Total Time 6
Stats: CPU Busy 5 (83.33%)
Stats: IO Busy  4 (66.67%)
```
6. Running the example with the `IO_RUN_LATER` flag results in a long 26 clock ticks runtime. The process that contains 3 IO steps is scheduled initially but then remains in a `READY` state until the remaining CPU processes complete. This does not appear to be an effective utilization of resources.
```
./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p
Time    PID: 0    PID: 1    PID: 2    PID: 3       CPU       IOs
  1     RUN:io     READY     READY     READY         1
  2    WAITING   RUN:cpu     READY     READY         1         1
  3    WAITING   RUN:cpu     READY     READY         1         1
  4    WAITING   RUN:cpu     READY     READY         1         1
  5    WAITING   RUN:cpu     READY     READY         1         1
  6*     READY   RUN:cpu     READY     READY         1
  7      READY      DONE   RUN:cpu     READY         1
  8      READY      DONE   RUN:cpu     READY         1
  9      READY      DONE   RUN:cpu     READY         1
 10      READY      DONE   RUN:cpu     READY         1
 11      READY      DONE   RUN:cpu     READY         1
 12      READY      DONE      DONE   RUN:cpu         1
 13      READY      DONE      DONE   RUN:cpu         1
 14      READY      DONE      DONE   RUN:cpu         1
 15      READY      DONE      DONE   RUN:cpu         1
 16      READY      DONE      DONE   RUN:cpu         1
 17     RUN:io      DONE      DONE      DONE         1
 18    WAITING      DONE      DONE      DONE                   1
 19    WAITING      DONE      DONE      DONE                   1
 20    WAITING      DONE      DONE      DONE                   1
 21    WAITING      DONE      DONE      DONE                   1
 22*    RUN:io      DONE      DONE      DONE         1
 23    WAITING      DONE      DONE      DONE                   1
 24    WAITING      DONE      DONE      DONE                   1
 25    WAITING      DONE      DONE      DONE                   1
 26    WAITING      DONE      DONE      DONE                   1
 27*      DONE      DONE      DONE      DONE

Stats: Total Time 27
Stats: CPU Busy 18 (66.67%)
Stats: IO Busy  12 (44.44%)
``` 
7. Running the example over again with the `IO_RUN_IMMEDIATE` flag results in a much shorter runtime. It might be a good idea to switch back to a process that has just completed IO to keep that process from hanging for an extended period of time. I'm not sure if this question or the example is suggesting that it is typical for processes that conduct IO to continue to conduct IO but it would be good to schedule this process again sooner if it were to continue making IO requests. Rescheduling this process immediately after IO completion makes for a much faster runtime.
```
./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE -c -p
Time    PID: 0    PID: 1    PID: 2    PID: 3       CPU       IOs
  1     RUN:io     READY     READY     READY         1
  2    WAITING   RUN:cpu     READY     READY         1         1
  3    WAITING   RUN:cpu     READY     READY         1         1
  4    WAITING   RUN:cpu     READY     READY         1         1
  5    WAITING   RUN:cpu     READY     READY         1         1
  6*    RUN:io     READY     READY     READY         1
  7    WAITING   RUN:cpu     READY     READY         1         1
  8    WAITING      DONE   RUN:cpu     READY         1         1
  9    WAITING      DONE   RUN:cpu     READY         1         1
 10    WAITING      DONE   RUN:cpu     READY         1         1
 11*    RUN:io      DONE     READY     READY         1
 12    WAITING      DONE   RUN:cpu     READY         1         1
 13    WAITING      DONE   RUN:cpu     READY         1         1
 14    WAITING      DONE      DONE   RUN:cpu         1         1
 15    WAITING      DONE      DONE   RUN:cpu         1         1
 16*      DONE      DONE      DONE   RUN:cpu         1
 17       DONE      DONE      DONE   RUN:cpu         1
 18       DONE      DONE      DONE   RUN:cpu         1

Stats: Total Time 18
Stats: CPU Busy 18 (100.00%)
Stats: IO Busy  12 (66.67%)
```
8. With some randomly generated processes and seeing the previous examples, I predict that we will see more efficient resource use with the `IO_RUN_IMMEDIATE` and `SWITCH_ON_IO` flags.

Using either the `SWITCH_ON_IO` and `IO_RUN_IMMEDIATE` or the `SWITCH_ON_IO` and `IO_RUN_LATER` flag pairings resulted in the most efficient resource use.
```
        CPU     IO      CPU     IO      CPU     IO      CPU     IO
        SWITCH_ON_IO & IO_RUN_IMMEDIATE SWITCH_ON_IO & IO_RUN_IMMEDIATE SWITCH_ON_END & IO_RUN_IMMEDIATE        SWITCH_ON_END & IO_RUN_IMMEDIATE        SWITCH_ON_END & IO_RUN_LATER    SWITCH_ON_END & IO_RUN_LATER    SWITCH_ON_IO & IO_RUN_LATER     SWITCH_ON_IO & IO_RUN_LATER
Seed 1  50%     66.7%   42.86%  57.14%  42.86%  57.14%  50%     66.7%
Seed 2  46.15%  84.62%  26.09%  69.75%  26.09%  69.75%  46.15%  84.62%
Seed 3  46.15%  69.23%  33.33%  66.67%  33.33%  66.67%  46.15%  69.23%
Average 47.43%  73.52%  34.09%  64.52%  34.09%  64.52%  47.43%  73.52%
```

