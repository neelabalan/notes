---
marp: true
theme: default
class: invert
paginate: true
---

# AWS EC2: Processor Selection & Right-Sizing

### Understanding Processor Options, Pricing & Performance Optimization

*Neelabalan Nandakumar*  

---

# Agenda

- **EC2 Processors Overview**  
- **EC2 Naming Conventions**  
- **Pricing Models & Spot Instances**  
- **Processor Architectures Deep Dive**
  - Intel Xeon Detailed Lineup
  - AMD EPYC Detailed Lineup
  - Core Differences
- **Compute & Memory Optimized Instances**
  - Examples & Memory Configuration Insights
- **Right-Sizing & Performance Adjustments**
- **Workload-Specific Instance Recommendations**

---

# Overview of AWS EC2 Processors

- **Diverse Instance Families:** Tailored for compute, memory, storage, or general purposes.
- **Processor Options:**  
  - **Intel Xeon:** Industry-standard, high clock speeds, and advanced features.
  - **AMD EPYC:** High core counts, competitive pricing, and superior memory bandwidth.
  - **AWS Graviton:** ARM-based, cost-efficient, and optimized for scale.
- **Goal:** Align workload demands with the optimal blend of CPU performance and cost structure.

---

# AWS EC2 Naming Conventions

- **Instance Family Prefixes:**  
  - **T** – Burstable Performance Instances  
  - **M** – General Purpose Instances  
  - **C** – Compute Optimized Instances  
  - **R** – Memory Optimized Instances  
  - **P, G, F, Inf** – Specialized (Machine Learning, Graphics, FPGA, Inference)

---

- **Generation Numbers:**  
  - e.g., T2 vs. T3 indicate improvements in architecture and performance.
- **Suffix Details:**  
  - a – AMD processors
  - n – Enhanced networking
  - d – NVMe storage
  - g – AWS Graviton processors
  - z – High frequency capabilities

---

## An example

**c5n.large**

- c: Compute-optimized family
- 5: 5th generation of this family
- n: Enhanced networking (100 Gbps)
- large: Instance size within the family

---

## Memory allocation logic

- Standard Instances (t3.large, m5.large): Follow predictable memory multiples (mostly 4:1)
- Compute Optimized (c5.large): 4GB memory per 2 vCPUs (ratio of 2:1)
- Network Enhanced (c5n.large): 5.25GB memory per 2 vCPUs (ratio of 2.63:1 or 2.67:1)
- Memory Optimized (r5.large): 16GB memory per 2 vCPUs (ratio of 8:1)

---

# Pricing Models & Spot Instances

- **On-Demand:**  
  - Pay-as-you-go flexibility.
- **Reserved Instances:**  
  - Cost savings for 1- or 3-year commitments.
- **Spot Instances:**  
  - Leverage unused capacity at significant discounts.
  - **Ideal For:** Fault-tolerant or flexible workloads that can handle interruptions.

---

# Processor Architectures: In-Depth Look

## Intel Xeon
  - Cascade Lake vs. Ice Lake: Newer generations offer up to 36% better price-performance ratio
  - High clock speeds with Turbo Boost and Hyper-Threading.
  - Advanced instruction sets (AVX-512, VNNI) to accelerate specific workloads.
  - Typical use cases are workloads requiring strong single-thread performance and low latency.

## AMD EPYC
  - Chiplet design enabling very high core counts and more PCIe lanes.
  - Superior memory bandwidth with support for more memory channels.
  - Typical uses cases are multi-threaded, data-intensive applications and virtualized environments.

---

# Core Differences: Intel Xeon vs AMD EPYC

| Feature              | **Intel Xeon**                                                                 | **AMD EPYC**                                                              |
|---------------------|---------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| **Performance Focus** | Excels in high clock speeds and single-thread performance.                      | Designed for throughput and parallel processing with many cores.         |
| **Memory & I/O**     | Traditional memory configurations tuned for latency-sensitive tasks.             | Enhanced I/O capabilities and higher memory bandwidth for data-heavy applications. |
| **Market Positioning** | Best for workloads where per-core performance is critical.                    | Favored for cost-effective scalability and multi-core performance.      |

---

# Compute & Memory Optimized Instances: Examples

## Compute Optimized Instances
  - **C5 & C5n:** Utilize Intel Xeon or AMD EPYC; optimized for compute-intensive tasks.
  - **C6g:** Powered by AWS Graviton2, offering cost-effective performance.
  - C5n specifically: Enhanced for network-intensive workloads with up to 100 Gbps bandwidth
  - Use cases: High-performance computing, video encoding, gaming servers, network appliances
  - **Why?** They are tuned to prioritize CPU performance for compute-bound tasks while keeping memory overhead minimal, ensuring an optimal balance between processing power and memory for workloads that do not require high memory capacity.

---

## Memory Optimized Instances
- **Examples:**
  - **R5 & R6:** Ideal for memory-intensive applications.
  - R5b: Includes up to 3x higher EBS performance (60k IOPS) and pptimized for In-memory databases (Redis, Memcached), real-time big data analytics
  - **X1 & X2:** Provide very high memory-to-CPU ratios for large in-memory databases.
  - Designed to support data caching, in-memory analytics, and large dataset processing with ample RAM.

---

## When to Choose C5n Over Standard C5:

- High-Throughput Applications: Network traffic-intensive services
- Low-Latency Requirements: Trading platforms, real-time data processing
- Tightly-Coupled Workloads: Distributed computing requiring frequent node communication

## Memory Engineering Logic:

- Network buffers require additional memory allocation
- Higher bandwidth instances need more memory for packet processing
- Real-world application profiling drives these non-standard allocations

---

# Right-Sizing: Concept & Best Practices

- Aligning the instance size and type precisely with actual workload demands.
- **Steps for Right-Sizing:**  
  - **Monitor:** Utilize CloudWatch or Datadog and Observability tools to track resource utilization.
  - **Analyze:** Evaluate CPU, memory, I/O, and network metrics.
  - **Adjust:** Scale vertically (switch to a different instance type) or horizontally (add/remove instances).

---

## Right-Sizing: Advanced Techniques

- Dynamic Rightsizing
    - Use Auto Scaling with custom metrics beyond CPU (memory, network I/O)
    - Implement predictive scaling based on historical patterns

- Granular Monitoring
    - CloudWatch Agent for detailed memory and disk metrics
    - Enhanced monitoring for per-second granularity

- Workload Profiling
    - Analyze application thread behavior to match with vCPU architecture
    - Measure memory access patterns to select optimal instance family

---


# Workload-Specific Instance Recommendations

## Running a Simple Web App (React & Flask Combo)
- **Recommended Instance:** General Purpose (e.g., **T3** or **T3a**)
    - Provides burstable performance for variable traffic.
    - Balanced CPU and memory suitable for lightweight web frameworks.
    - Cost-effective for development and small-scale production deployments.

## Running an NGINX Server
- **Recommended Instance:** Compute Optimized (e.g., **C5** or **C6g**)
    - NGINX is lightweight but benefits from high CPU performance when managing numerous concurrent connections.
    - Compute optimized instances deliver the necessary processing power with a lean memory footprint.
    - Excellent for handling network I/O and load balancing tasks.

## Running a Database (MongoDB / Postgres)
- **Recommended Instance:** Memory Optimized (e.g., **R5** or **R6**)
    - Databases require substantial memory for efficient caching and fast query performance.
    - Memory optimized instances offer high RAM-to-CPU ratios, ensuring quick access to data.
    - Adequate I/O throughput supports high transaction volumes and complex query loads.

---

## Beyond the Basics: Specialized Instance Selection

- Storage-Optimized Workloads
    - I3 & I3en: NVMe SSD storage with up to 60TB capacity
        - Use cases: NoSQL databases, time-series data, log processing
        - Key feature: I3en instances combine high storage with enhanced networking

- Bare Metal Options
    - x1e.metal, c5.metal: Direct access to hardware
        - Use cases: specialized virtualization
        - Consideration: No hypervisor overhead

---

# Additional Resources

- **AWS Documentation:**  
  - [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
  - [AWS Pricing Models](https://aws.amazon.com/ec2/pricing/)
- **Tools & Best Practices:**  
  - [Instances Vantage](https://instances.vantage.sh)
  - AWS Cost Explorer, CloudWatch, Compute Optimizer.
- **Further Reading:**  
  - Whitepapers on right-sizing and performance optimization.
