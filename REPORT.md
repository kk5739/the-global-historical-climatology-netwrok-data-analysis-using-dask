# Lab 4: Dask

- Name: 
Kyeongmo Kang, Alexander Pegot-Ogier, Nikolas Prasinos

- NetID: 
kk5739, ap9283, np3106

## Description of Your Solution

We implemented a Dask-based pipeline to compute, for each weather station, the maximum daily temperature range (TMAX – TMIN) across three dataset sizes: **tiny**, **small**, and **all**. Our workflow evolved through three main approaches as we addressed performance bottlenecks and memory limitations:

1. **First approach – Dask writer with repartition tuning**  
   - We persisted the final result and experimented with different `npartitions`.  
   - On **tiny** and **small**, the pipeline completed successfully—using no manual repartition for tiny and `npartitions=1` for small yielded the fastest write times.  
   - On **all**, workers kept dying with `KilledWorker` errors during the parallel Parquet write.

2. **Second approach – pandas-based write**  
   - We bypassed Dask’s writer by calling `result.compute()` to bring the (small) final table into pandas and then writing via `df.to_parquet()`.  
   - Unfortunately, the initial compute itself (the global shuffle/groupby on the full dataset) still triggered `KilledWorker` failures.

3. **Third approach – two-stage reduction + pandas write**  
   - We replaced the one-step global aggregation with a **two-stage reduction**:  
     1. Per-partition max by station (no shuffle)  
     2. Global max on the much smaller intermediate  
   - Then we wrote the result via pandas.  
   - This eliminated the large shuffle, but under our current cluster resources the final compute still encountered memory-related worker restarts.

**Conclusion**  
Despite iterating through three strategies—parallel Dask writes, single-process pandas writes, and a two-stage reduction—the full dataset remains too large for our available workers’ memory. To complete the “all” run successfully, we would need to either scale up the cluster (more RAM or cores per worker) or further partition the problem (for example by station region or year).


## For Tiny ##
- **before optimization**
CPU times: user 2.13 s, sys: 374 ms, total: 2.5 s
Wall time: 11.2 s 

- **1st approach with no partition - Fastest** 
**CPU times: user 13.3 ms, sys: 2.53 ms, total: 15.8 ms
Wall time: 28.2 ms**

- **1st approach with 1 partition**
CPU times: user 14.5 ms, sys: 637 µs, total: 15.1 ms
Wall time: 29.2 ms

- **1st approach with 2 partition**
CPU times: user 62.7 ms, sys: 42.7 ms, total: 105 ms
Wall time: 420 ms 


## For Small ##
- **before optimization**
CPU times: user 6.53 s, sys: 1.35 s, total: 7.88 s
Wall time: 1min 25s 

- **1st approach with no partition** 
CPU times: user 16.8 ms, sys: 5.28 ms, total: 22.1 ms
Wall time: 71.7 ms

- **1st approach with 1 partition - Fastest**
**CPU times: user 11 ms, sys: 4.18 ms, total: 15.2 ms
Wall time: 33.5 ms**

- **1st approach with 2 partition**
CPU times: user 13.4 ms, sys: 2.11 ms, total: 15.5 ms
Wall time: 41.3 ms

- **# 1st approach with 5 partition**
CPU times: user 18.6 ms, sys: 0 ns, total: 18.6 ms
Wall time: 59.9 ms

## For All ##
**Error message: KilledWorker. Doesn't work with any approach**
