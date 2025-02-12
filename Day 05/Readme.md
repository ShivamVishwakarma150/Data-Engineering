# **Big Data Fundamentals**  
#### **What is Big Data?**  
Big Data refers to extremely large datasets that are difficult to process using traditional data management tools. The volume and complexity of this data grow exponentially, requiring specialized frameworks like Hadoop.  

---

### **5 V’s of Big Data**  
1. **Volume** – Huge amounts of data are generated, ranging from terabytes to petabytes and exabytes.  
2. **Velocity** – Data is generated at high speed (e.g., real-time data from social media and IoT).  
3. **Variety** – Different formats of data: structured (databases), semi-structured (JSON, XML), and unstructured (images, videos, emails).  
4. **Veracity** – Ensuring the reliability and accuracy of data.  
5. **Value** – Extracting meaningful insights to make business decisions.  

---

### **Examples of Big Data**  
- **Social Media** – Data from Facebook, Twitter, Instagram.  
- **Healthcare** – Patient records, medical images.  
- **Finance** – Stock exchange data, transactions.  
- **E-commerce** – Purchase data from Amazon, eBay.  
- **IoT** – Data from smart devices.  
- **Transportation** – Traffic data, GPS logs from Google Maps, Uber.  

---

### **Types of Data**  
1. **Structured Data** – Stored in relational databases (e.g., SQL).  
2. **Unstructured Data** – Includes emails, images, videos, PDFs.  
3. **Semi-Structured Data** – A mix of both structured and unstructured data (e.g., JSON, XML, email with metadata).  

---

### **What is a Cluster?**  
A **cluster** is a collection of multiple computers (nodes) working together to act as a single system. This improves performance, scalability, and fault tolerance.  

---

### **Scaling in Clusters**  
1. **Vertical Scaling (Scaling Up)** – Adding more resources (CPU, RAM, storage) to a single machine.  
2. **Horizontal Scaling (Scaling Out)** – Adding more machines (nodes) to distribute the load.  

---

### **Hadoop Overview**  
Hadoop is an **open-source framework** designed for **storing and processing** large datasets across multiple computers.  

**Main Components of Hadoop:**  
1. **Hadoop Distributed File System (HDFS)** – Storage layer, distributing data across multiple nodes.  
2. **MapReduce** – Processing framework that breaks tasks into small chunks and processes them in parallel.  
3. **Yet Another Resource Negotiator (YARN)** – Manages cluster resources and schedules jobs efficiently.  

---

### **Properties of Hadoop**  
- **Scalability** – Can handle growing data volumes.  
- **Cost-effectiveness** – Uses inexpensive hardware.  
- **Flexibility** – Works with all types of data.  
- **Fault Tolerance** – Replicates data to prevent loss.  
- **Data Locality** – Moves computation to the node where data is stored, reducing network congestion.  
- **Simplicity** – Uses MapReduce for easy data processing.  
- **Open-source** – Free to use and modify.  

---

### **HDFS (Hadoop Distributed File System)**  
HDFS follows a **Master-Slave Architecture** with two key components:  
1. **NameNode (Master)** – Stores metadata, tracks file locations.  
2. **DataNode (Slave)** – Stores actual data blocks and responds to read/write requests.  

#### **Key HDFS Concepts**  
- **Block Size** – Default is **128MB or 256MB**, much larger than traditional file systems.  
- **Replication** – Default is **3 copies per block** for fault tolerance.  
- **Rack Awareness** – Distributes data across different racks for better fault tolerance.  

---

### **Secondary NameNode vs. Standby NameNode**  
1. **Secondary NameNode** – Helps in metadata checkpointing but **does not act as a failover** for NameNode failure.  
2. **Standby NameNode** – A **hot backup** that synchronizes with the active NameNode and **automatically takes over in case of failure**.  

---

### **HDFS Operations**  
1. **Write Operation** – Data is broken into blocks, replicated, and stored across multiple DataNodes.  
2. **Read Operation** – Data is fetched from the nearest DataNode to minimize latency.  

---


<br/>
<br/>

# **HDFS Blocks in Hadoop**  

A **block** in Hadoop is the smallest unit of storage in the **Hadoop Distributed File System (HDFS)**. Unlike traditional file systems (where block sizes are typically **4 KB to 64 KB**), Hadoop uses much larger blocks (default **128 MB or 256 MB**) to efficiently handle **big data processing**.  

---

## **1. Why Does Hadoop Use Large Blocks?**  
- **Reduces Metadata Overhead**: Each file stored in HDFS has metadata maintained by the NameNode. If the block size were small, the NameNode would need to manage too many blocks, leading to high memory usage.  
- **Optimized for Streaming Reads**: Hadoop is designed for processing large datasets, not small, random-access reads. Large blocks ensure **efficient sequential data processing**.  
- **Minimizes Network Traffic**: Since large blocks reduce the number of splits, **less data is transferred between nodes**, improving performance.  

---

## **2. HDFS Block Characteristics**  
- **Default Block Size**: **128 MB** (Hadoop 2.x and later) or **64 MB** (Hadoop 1.x). Can be configured based on workload.  
- **Logical Representation**: Unlike physical disk blocks, HDFS blocks are **logical**—a single block **can span multiple physical disks** across different DataNodes.  
- **File Splitting**: If a file is larger than the block size, it is split into **multiple blocks** (e.g., a **500 MB file** will be split into **four 128 MB blocks + one 4 MB block**).  
- **No Wastage of Space**: If a block **is not fully utilized**, the remaining space **is not shared** with other files (e.g., a 50 MB file in a 128 MB block still uses one full block).  

---

## **3. Example: How Data is Stored in HDFS Blocks**  
Let's assume:  
- Block size = **128 MB**  
- Replication factor = **3**  

| File Size  | Blocks Created |
|------------|--------------|
| 200 MB  | Block 1 (128 MB) + Block 2 (72 MB) |
| 300 MB  | Block 1 (128 MB) + Block 2 (128 MB) + Block 3 (44 MB) |

Each block is **replicated three times** across different DataNodes for **fault tolerance**.  

---

## **4. Block Replication (Fault Tolerance)**  
HDFS ensures **high availability** by replicating blocks across multiple DataNodes.  
- **Default Replication Factor**: **3 copies**  
- **Placement Strategy**:  
  - First copy → Stored on a **random node in the cluster**  
  - Second copy → Stored on a **different node in a different rack**  
  - Third copy → Stored on **another node in the same rack as the second copy**  

If a node fails, Hadoop automatically **re-replicates lost blocks** from existing copies to maintain the replication factor.  

---

## **5. Why is HDFS Block Size Larger than Traditional File Systems?**  
| Feature           | Traditional File Systems | HDFS |
|------------------|------------------------|------|
| Block Size       | 4 KB – 64 KB             | 128 MB – 256 MB |
| Storage Type     | Local disks              | Distributed across clusters |
| Read/Write Mode  | Random Access            | Sequential Processing |
| Performance      | Fast for small files     | Optimized for large files |

### **Key Takeaway**  
HDFS uses **large blocks** to handle **big data efficiently**, minimize **metadata overhead**, and support **fault tolerance through replication**.

---