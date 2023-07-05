# TPC-CH Benchmark on MySQL

## Description
- CH-benCHmark is a mixed workload containing both TPC-C and TPC-H tests. It is the most common workload to test HTAP systems. 
- Before running the CH-benCHmark test, you need to deploy TiFlash first, which is a TiDB's HTAP component. After you deploy TiFlash and create TiFlash replicas, TiKV will replicate the latest data of TPC-C online transactions to TiFlash in real time, and the TiDB optimizer will automatically push down OLAP queries from TPC-H workload to the MPP engine of TiFlash for efficient execution.
- The CH-benCHmark test in this document is implemented based on go-tpc. You can download the test program using the following TiUP command:

## Prerequisites and Installations

- Install Go-lang higher than 11.0
```bash
sudo apt install golang-go
```

- Install TiUP & TiDB
1. Install TiUP 
```bash
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```
2. Add PATH to the bash script
```bash
source ${your_shell_profile}
```
3. Start the cluster in the current session:
```bash
tiup playground
```
4. User the TiUP client to connect to TiDB
```bash
tiup client
```

- Install go-tpc
```bash
curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/pingcap/go-tpc/master/install.sh | sh
git clone https://github.com/pingcap/go-tpc.git
cd go-tpc
make build
```
- Install Bench
For detailed usage of the TiUP Bench component, see TiUP Bench.
```bash
tiup install bench
```
- Source the PATH to bash
```bash
source ~/.bashrc
```

## Run
### Start MySQL Server
```bash
./bin/mysqld --defaults-file=/home/lbh/scripts/my.cnf
```
### Load Data
- Load TPC-C Data
```bash
.tiup/bin/tiup bench tpcc -H 172.16.5.140 -P 4000 -D tpcc --warehouses 1000 prepare -T 32
```
- Load TPC-H Data
```bash
.tiup/bin/tiup bench ch -H 172.16.5.140 -P 4000 -D tpcc prepare
```

Sample Log Output
```bash
creating nation
creating region
creating supplier
generating nation table
generate nation table done
generating region table
generate region table done
generating suppliers table
generate suppliers table done
creating view revenue1
```

### Create TiFlash Replicas and Check
```bash
mysql>> ALTER DATABASE tpcc SET tiflash replica 2;
```

- To check whether replications are completed
```bash
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = 'tpcc';
```

- To collect statistics:
```bash
analyze table customer;
analyze table district;
analyze table history;
analyze table item;
analyze table new_order;
analyze table order_line;
analyze table orders;
analyze table stock;
analyze table warehouse;
analyze table nation;
analyze table region;
analyze table supplier;
```

### Run the TPCCH Test
```bash
tiup bench ch --host 172.16.5.140 -P4000 --warehouses 1000 run -D tpcc -T 50 -t 1 --time 1h
```
- Sample Output
```bash
Finished: 50 OLTP workers, 1 OLAP workers
[Summary] DELIVERY - Takes(s): 3599.7, Count: 501795, TPM: 8363.9, Sum(ms): 63905178.8, Avg(ms): 127.4, 50th(ms): 125.8, 90th(ms): 167.8, 95th(ms): 184.5, 99th(ms): 226.5, 99.9th(ms): 318.8, Max(ms): 604.0
[Summary] DELIVERY_ERR - Takes(s): 3599.7, Count: 14, TPM: 0.2, Sum(ms): 1027.7, Avg(ms): 73.4, 50th(ms): 71.3, 90th(ms): 109.1, 95th(ms): 109.1, 99th(ms): 113.2, 99.9th(ms): 113.2, Max(ms): 113.2
[Summary] NEW_ORDER - Takes(s): 3599.7, Count: 5629221, TPM: 93826.9, Sum(ms): 363758020.7, Avg(ms): 64.6, 50th(ms): 62.9, 90th(ms): 88.1, 95th(ms): 100.7, 99th(ms): 130.0, 99.9th(ms): 184.5, Max(ms): 570.4
[Summary] NEW_ORDER_ERR - Takes(s): 3599.7, Count: 20, TPM: 0.3, Sum(ms): 404.2, Avg(ms): 20.2, 50th(ms): 18.9, 90th(ms): 37.7, 95th(ms): 50.3, 99th(ms): 56.6, 99.9th(ms): 56.6, Max(ms): 56.6
[Summary] ORDER_STATUS - Takes(s): 3599.8, Count: 500318, TPM: 8339.0, Sum(ms): 7135956.6, Avg(ms): 14.3, 50th(ms): 13.1, 90th(ms): 24.1, 95th(ms): 27.3, 99th(ms): 37.7, 99.9th(ms): 50.3, Max(ms): 385.9
[Summary] PAYMENT - Takes(s): 3599.8, Count: 5380815, TPM: 89684.8, Sum(ms): 269863092.5, Avg(ms): 50.2, 50th(ms): 48.2, 90th(ms): 75.5, 95th(ms): 88.1, 99th(ms): 125.8, 99.9th(ms): 184.5, Max(ms): 1073.7
[Summary] PAYMENT_ERR - Takes(s): 3599.8, Count: 11, TPM: 0.2, Sum(ms): 313.0, Avg(ms): 28.5, 50th(ms): 10.0, 90th(ms): 67.1, 95th(ms): 67.1, 99th(ms): 88.1, 99.9th(ms): 88.1, Max(ms): 88.1
[Summary] STOCK_LEVEL - Takes(s): 3599.8, Count: 500467, TPM: 8341.5, Sum(ms): 13208726.4, Avg(ms): 26.4, 50th(ms): 25.2, 90th(ms): 37.7, 95th(ms): 44.0, 99th(ms): 62.9, 99.9th(ms): 96.5, Max(ms): 570.4
[Summary] STOCK_LEVEL_ERR - Takes(s): 3599.8, Count: 2, TPM: 0.0, Sum(ms): 7.6, Avg(ms): 3.7, 50th(ms): 3.1, 90th(ms): 4.7, 95th(ms): 4.7, 99th(ms): 4.7, 99.9th(ms): 4.7, Max(ms): 4.7
tpmC: 93826.9, efficiency: 729.6%
[Summary] Q1     - Count: 11, Sum(ms): 42738.2, Avg(ms): 3885.3
[Summary] Q10    - Count: 11, Sum(ms): 440370.3, Avg(ms): 40034.3
[Summary] Q11    - Count: 11, Sum(ms): 44208.6, Avg(ms): 4018.7
[Summary] Q12    - Count: 11, Sum(ms): 105320.3, Avg(ms): 9574.6
[Summary] Q13    - Count: 11, Sum(ms): 19199.5, Avg(ms): 1745.4
[Summary] Q14    - Count: 11, Sum(ms): 84582.1, Avg(ms): 7689.5
[Summary] Q15    - Count: 11, Sum(ms): 271944.8, Avg(ms): 24722.8
[Summary] Q16    - Count: 11, Sum(ms): 183894.9, Avg(ms): 16718.1
[Summary] Q17    - Count: 11, Sum(ms): 89018.9, Avg(ms): 8092.7
[Summary] Q18    - Count: 10, Sum(ms): 767814.5, Avg(ms): 76777.6
[Summary] Q19    - Count: 10, Sum(ms): 17099.1, Avg(ms): 1709.8
[Summary] Q2     - Count: 11, Sum(ms): 53513.6, Avg(ms): 4865.2
[Summary] Q20    - Count: 10, Sum(ms): 73717.7, Avg(ms): 7372.1
[Summary] Q21    - Count: 10, Sum(ms): 166001.4, Avg(ms): 16601.1
[Summary] Q22    - Count: 10, Sum(ms): 48268.4, Avg(ms): 4827.7
[Summary] Q3     - Count: 11, Sum(ms): 31110.1, Avg(ms): 2828.5
[Summary] Q4     - Count: 11, Sum(ms): 83814.2, Avg(ms): 7619.3
[Summary] Q5     - Count: 11, Sum(ms): 368301.0, Avg(ms): 33483.5
[Summary] Q6     - Count: 11, Sum(ms): 61702.5, Avg(ms): 5608.9
[Summary] Q7     - Count: 11, Sum(ms): 158928.2, Avg(ms): 14446.3
```


## References
- https://docs.pingcap.com/tidb/dev/quick-start-with-tidb
- https://github.com/pingcap/go-tpc
- https://docs.pingcap.com/tidb/dev/benchmark-tidb-using-ch#collect-statistics
