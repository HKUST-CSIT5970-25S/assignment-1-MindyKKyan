[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/IAASVEAZ)
# CSIT5970 Assignment-1: EC2 Measurement (2 questions, 4 marks)

### Deadline: 11:59PM, Feb, 28, Friday

---

### Name: Mingzhen JIANG
### Student Id: 21108128
### Email: mjiangar@connect.ust.hk

---

## Question 1: Measure the EC2 CPU and Memory performance

1. (1 mark) Report the name of measurement tool used in your measurements (you are free to choose *any* open source measurement software as long as it can measure CPU and memory performance). Please describe your configuration of the measurement tool, and explain why you set such a value for each parameter. Explain what the values obtained from measurement results represent (e.g., the value of your measurement result can be the execution time for a scientific computing task, a score given by the measurement tools or something else).

    > I used the Phoronix Test Suite to measure CPU and memory performance. The Phoronix Test Suite is a comprehensive testing and benchmarking platform that supports a wide range of tests for various system components. 

    > **Configuration:**
    > - **CPU Test:** Used the `pts/compress-7zip` test profile, which includes a variety of CPU-intensive benchmarks such as compression. I set the number of test iterations to 5 to ensure consistent results.
    > - **Memory Test:** Used the `pts/ramspeed` test profile, which includes benchmarks for memory read and write speeds. I configured the test to run with a block size of 1MB to simulate typical application workloads.

    > **Reason for Configuration:**
    > - The number of iterations was set to 5 to balance between obtaining reliable results and minimizing the total test duration.
    > - The block size of 1MB for memory tests was chosen to reflect common usage patterns in real-world applications.

    > **Measurement Results:**
    > - **CPU Performance:** The results are reported as a score, which is a composite of the various benchmarks included in the `pts/compress-7zip` profile. Higher scores indicate better CPU performance.
    > - **Memory Performance:** The results include memory bandwidth (measured in MB/s) and latency (measured in nanoseconds). Higher bandwidth and lower latency values indicate better memory performance.

    > **Commands Used:**
    > - **Install Phoronix Test Suite:**
    >   ```sh
    >   sudo add-apt-repository ppa:phoronix-test-suite/stable
    >   sudo apt update
    >   sudo apt install -y phoronix-test-suite php-cli php-common php-xml unzip
    >   ```
    > - **Run CPU Test:**
    >   ```sh
    >   sudo apt-get install php-zip
    >   phoronix-test-suite run pts/compress-7zip
    >   ```
    > - **Run Memory Test:**
    >   ```sh
    >   phoronix-test-suite run pts/ramspeed
    >   ```

2. (1 mark) Run your measurement tool on general purpose `t2.micro`, `t2.medium`, and `c5d.large` Linux instances, respectively, and find the performance differences among these instances. Launch all the instances in the **US East (N. Virginia)** region. Does the performance of EC2 instances increase commensurate with the increase of the number of vCPUs and memory resource?

    In order to answer this question, you need to complete the following table by filling out blanks with the measurement results corresponding to each instance type.

    | Size        | CPU performance | Memory performance |
    | ----------- | --------------- | ------------------ |
    | `t2.micro` |      3069 MIPS    |   10533.45 MB/s   |
    | `t2.medium`  |    5866 MIPS    |   19622.88 MB/s    |
    | `c5d.large` |     4914 MIPS    |   14181.12 MB/S    |
    
    > Region: US East (N. Virginia). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI.

    #### **Key Observations:**
    1. **CPU Performance:**  
    - `t2.micro` (1 vCPU): Lowest performance (3069 MIPS).  
    - `t2.medium` (2 vCPUs): Nearly double the performance of `t2.micro` (5866 MIPS).  
    - `c5d.large` (2 vCPUs): Lower than `t2.medium` (4914 MIPS) despite being a compute-optimized instance.  
    - **Reason:** The `c5d.large` uses Intel Xeon Scalable processors optimized for sustained workloads, while the `pts/compress-7zip` test may favor burstable `t2` instances due to initial CPU credits.  

    2. **Memory Performance:**  
    - `t2.micro`: 10,533.45 MB/s.  
    - `t2.medium`: ~1.86x higher than `t2.micro` (19,622.88 MB/s).  
    - `c5d.large`: Lower than `t2.medium` (14,181.12 MB/s).  
    - **Reason:** The `pts/ramspeed` test focuses on memory bandwidth, which may not fully leverage the `c5d.large`’s NVMe storage or DDR4 memory architecture.  

    ---

    ### Does Performance Increase with Resource Scaling?**
    **Answer:** **Yes, but only within the same instance family.**  

    #### **Analysis:**
    - **Within `t2` Family (`t2.micro` → `t2.medium`):**  
    - **vCPUs:** 1 → 2 (+100%).  
    - **Memory:** 1 GiB → 4 GiB (+300%).  
    - **CPU Performance:** +91% (3069 → 5866 MIPS).  
    - **Memory Performance:** +86% (10,533 → 19,623 MB/s).  
    - **Conclusion:** Scaling resources in the `t2` family yields near-linear performance gains.  

    - **Cross-Family (`t2.medium` vs. `c5d.large`):**  
    - **Same vCPUs/Memory (2 vCPUs, 4 GiB):**  
        - **CPU Performance:** `c5d.large` is 16% slower than `t2.medium`.  
        - **Memory Performance:** `c5d.large` is 28% slower than `t2.medium`.  
    - **Reason:**  
        - `t2` instances use burstable CPU credits, which boost short-term performance.  
        - `c5d.large` is optimized for sustained compute workloads, not short benchmarks.  

    ---

   
## Question 2: Measure the EC2 Network performance

1. (1 mark) The metrics of network performance include **TCP bandwidth** and **round-trip time (RTT)**. Within the same region, what network performance is experienced between instances of the same type and different types? In order to answer this question, you need to complete the following table.

     | Type                      | TCP b/w (Mbps) | RTT (ms) |
    | ------------------------- | -------------- | -------- |
    | `t3.medium` - `t3.medium` |     3819.51    |   0.298  |
    | `m5.large` - `m5.large`   |     5079.03    |   0.169  |
    | `c5n.large` - `c5n.large` |     5079.03    |   0.140  |
    | `t3.medium` - `c5n.large` |     2518.93    |   0.418  |
    | `m5.large` - `c5n.large`  |     3077.01    |   0.298  |
    | `m5.large` - `t3.medium`  |     1916.08    |   1.003  |

    > Region: US East (N. Virginia). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI. Note: Use private IP address when using iPerf within the same region. You'll need iPerf for measuring TCP bandwidth and Ping for measuring Round-Trip time.
    #### **Key Observations:**
    - **Same Instance Type:**  
    - Higher-tier instances (e.g., `c5n.large`) achieve maximum bandwidth (~5 Gbps) and sub-0.2 ms RTT.  
    - **Cross-Type Pairs:**  
    - Bandwidth is limited by the slower instance (e.g., `t3.medium` caps at ~3.8 Gbps).  
    - RTT increases slightly due to heterogeneous hardware configurations.  

    ---


2. (1 mark) What about the network performance for instances deployed in different regions? In order to answer this question, you need to complete the following table.

    | Connection                | TCP b/w (Mbps) | RTT (ms) |
    | ------------------------- | -------------- | -------- |
    | N. Virginia - Oregon      |      33.6      |   64.596 |
    | N. Virginia - N. Virginia |     5058.36    |   0.218  |
    | Oregon - Oregon           |     5078.84    |   0.156  |
 
    > Region: US East (N. Virginia), US West (Oregon). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI. All instances are `c5.large`. Note: Use public IP address when using iPerf within the same region.


    #### **Key Observations:**
    - **Intra-Region:** Near-maximum bandwidth (~5 Gbps) and sub-0.3 ms RTT.  
    - **Cross-Region:** Bandwidth drops to ~33 Mbps, and RTT increases to ~65 ms due to AWS inter-region traffic limits.  
