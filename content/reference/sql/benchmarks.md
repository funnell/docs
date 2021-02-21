---
title: Benchmarks
---

# Benchmarks

This section provides benchmarks for Dolt. The current version of Dolt is 0.23.5, and we benchmark against MySQL 8.0.22.

## Data

Here we present the result of running `sysbench` MySQL tests against Dolt SQL for the most recent release of Dolt. We will update this with every release. The tests attempt to run as many queries as possible in a fixed 2 minute time window. The `Dolt` and `MySQL` columns show the median latency of each test during that 2 minute time window.

Dolt is slower than MySQL. The goal is to get Dolt to within 2-4 times the speed of MySQL common operations. If a query takes MySQL 1 second, we expect it to take Dolt 2-4 seconds. Or, if MySQL can run 8 queries in 10 seconds, then we want Dolt to run 2-4 queries in 10 seconds. The `multiple` column represents this relationship. Our custom lua tests can be found [here](https://github.com/dolthub/dolt/tree/master/benchmark/perf_tools/sysbench_scripts/lua).


| Read Tests | Dolt | MySQL | Multiple |
| :---  | :--- | :--- | :--- |
| covering\_index\_scan | 11.04 | 1.39 | 8.0 |
| index\_scan | 125.52 | 34.95 | 4.0 |
| oltp\_point\_select | 1.55 | 0.11 | 14.0 |
| oltp\_read\_only | 31.37 | 2.26 | 14.0 |
| select\_random\_points | 2.71 | 0.26 | 10.0 |
| select\_random\_ranges | 3.07 | 0.28 | 11.0 |
| table\_scan | 142.39 | 34.95 | 4.0 |
| mean | | | _9.29_ |

<br/>

| Write Tests | Dolt | MySQL | Multiple |
| :--- | :--- | :--- | :--- |
| bulk\_insert | 0.001 | 0.001 | 1.0 |
| oltp\_delete | 12.08 | 0.11 | 110.0 |
| oltp\_insert | 13.22 | 2.43 | 5.0 |
| oltp\_read\_write | 84.47 | 6.09 | 14.0 |
| oltp\_update\_index | 15.27 | 2.97 | 5.0 |
| oltp\_update\_non\_index | 9.06 | 2.86 | 3.0 |
| oltp\_write\_only | 55.82 | 3.75 | 15.0 |
| mean | | | _21.86_ |

<br/>

| Overall Mean Multiple | _15.57_ |
| ------ | ------ |

<br/>

In the spirit of ["dog fooding"](https://en.wikipedia.org/wiki/Eating_your_own_dog_food) we created a Dolt database on [DoltHub](https://www.dolthub.com/repositories/dolthub/dolt-benchmarks) with our performance metrics. You can find the full set of metrics produced by `sysbench` there, and explore them via our SQL console.

## Approach

We adopted an industry standard benchmarking tool, [`sysbench`](https://github.com/akopytov/sysbench). `sysbench` provides a series of benchmarks for examining various aspects of database performance, and was authored by developers who worked on MySQL.

You can read more about our benchmarking approach [here](https://github.com/dolthub/dolt/tree/master/benchmark/perf_tools). The basic idea is to provide our developers and contributors with simple tools for producing robust comparisons of Dolt SQL against MySQL. We do this by building and running on all benchmarks on the same hardware and associating the run with a unique identifier with each invocation of `run_benchmarks.sh`, our wrapper.

For example, suppose that a developer made a bunch of changes that are supposed to speed up bulk inserts:

```text
$ cd $DOLT_CHECKOUT/benchmark/perf_tools
$ ./run_benchmarks.sh \
  bulk_insert \
  10000 \
  <username> \
  current
```

This will do the following:

* build a copy of Dolt using the locally checked out Git repository located at `$DOLT_CHECKOUT` inside a Docker container
* build and launch a Docker container running Dolt SQL
* build and launch a Docker container running `sysbench` that executes `bulk_insert` using a table with 10000 records for the test
* repeat the process using MySQL for comparison

All of the data produced will be associated with a unique run ID.

## Code

The benchmarking tools are part of Dolt, which is free and open source. You can find a more detailed description of the tools on [GitHub](https://github.com/dolthub/dolt/tree/master/benchmark/perf_tools).
