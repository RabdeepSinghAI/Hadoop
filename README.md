# Hadoop MapReduce — Word Length Statistics
**New York Institute of Technology**  
**Student:** Rabdeep Singh (NetID: 1319229)  
**Assigned File:** `austen-emma.txt` (Jane Austen's *Emma*)

---

## Overview

This project implements three **Hadoop MapReduce jobs** in Java to compute statistical properties of word lengths across a large text corpus. All jobs were run on a local Hadoop cluster using HDFS and verified via `hadoop fs -cat`.

| Statistic | Result |
|-----------|--------|
| Total Words | 158,167 |
| Total Characters | 722,615 |
| **Mean Word Length** | **4.57** |
| **Median Word Length** | **4** |
| **Standard Deviation** | **2.6396** |

---

## How It Works

### Job 1 — Mean Word Length

**Map phase:** For each word in the text, emit two key-value pairs:
- `(count, 1)` — to track total word count
- `(length, word.length())` — to accumulate total character length

**Reduce phase:** Sum all values per key, then compute:
```
mean = total_length / total_count = 722615 / 158167 = 4.57
```

**Output (`output_mean/part-r-00000`):**
```
count   158167
length  722615
```

---

### Job 2 — Median Word Length

**Map phase:** For each word, emit `(word.length(), 1)`.

**Reduce phase:** Sum counts per word length, producing a full frequency distribution.

**Output (`output_median/part-r-00000`):**
```
1    5694
2    27162
3    35279
4    28806
5    17661
6    11669
7    9788
...
35   1
```

**Median calculation:** Cumulative sum to find the middle value (word #79,084) → **Median = 4**

---

### Job 3 — Standard Deviation

**Map phase:** For each word, emit three aggregates:
- `(count, 1)`
- `(length, word.length())`
- `(square, word.length()²)`

**Reduce phase:** Compute standard deviation in a single pass:
```
σ = sqrt( (Σx²/n) - (Σx/n)² )
  = sqrt( 4403385/158167 - (722615/158167)² )
  = 2.6396
```

**Output (`output_std/part-r-00000`):**
```
count   158167
length  722615
square  4403385
```

---

## How to Run

### Prerequisites
- Java 8+
- Hadoop 3.x installed and configured
- `austen-emma.txt` available (from NLTK corpus or Project Gutenberg)

### Steps

```bash
# 1. Compile the Java source files
javac -classpath $(hadoop classpath) -d . *.java

# 2. Create JAR
jar cf wordlength.jar *.class

# 3. Upload input to HDFS
hadoop fs -mkdir -p /user/$USER/input
hadoop fs -put austen-emma.txt /user/$USER/input/

# 4. Run Mean job
hadoop jar wordlength.jar MeanDriver /user/$USER/input /user/$USER/output_mean

# 5. Run Median job
hadoop jar wordlength.jar MedianDriver /user/$USER/input /user/$USER/output_median

# 6. Run Std Dev job
hadoop jar wordlength.jar StdDriver /user/$USER/input /user/$USER/output_std

# 7. View results
hadoop fs -cat /user/$USER/output_mean/part-r-00000
hadoop fs -cat /user/$USER/output_median/part-r-00000
hadoop fs -cat /user/$USER/output_std/part-r-00000
```

---

## Repository Structure

```
hadoop-word-length/
├── src/
│   ├── MeanMapper.java
│   ├── MeanReducer.java
│   ├── MeanDriver.java
│   ├── MedianMapper.java
│   ├── MedianReducer.java
│   ├── MedianDriver.java
│   ├── StdMapper.java
│   ├── StdReducer.java
│   └── StdDriver.java
├── output_mean/
│   └── part-r-00000
├── output_median/
│   └── part-r-00000
├── output_std/
│   └── part-r-00000
├── hadoop_results.txt
└── README.md
```

---

## Key Concepts Demonstrated

- **MapReduce paradigm** — distributing computation across map and reduce phases
- **HDFS** — storing and retrieving large text files in a distributed filesystem
- **Single-pass statistics** — computing mean, median, and σ without loading the full dataset into memory
- **Frequency distributions** — building a histogram of word lengths to derive the median

---

## Tech Stack

![Java](https://img.shields.io/badge/Java-ED8B00?style=flat&logo=openjdk&logoColor=white)
![Apache Hadoop](https://img.shields.io/badge/Apache%20Hadoop-66CCFF?style=flat&logo=apachehadoop&logoColor=black)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)

---

*Completed as part of a distributed systems / big data assignment at New York Institute of Technology.*
