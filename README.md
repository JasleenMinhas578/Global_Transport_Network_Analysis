# Global Air Transportation Network: Structure, Communities, and Resilience

> **Course:** COMP 6917 – Complex Networks  
> **Author:** Jasleen Minhas  
> **Term:** Fall 2025  

This project analyzes the **Global Air Transportation Network (ATN)** as a complex network using the OpenFlights dataset.  
Airports are modeled as nodes, flight routes as directed edges.

The main goals are to:

- Understand the **global structure** of the ATN  
- Identify **influential hubs** and **regional communities**  
- Quantify **small-world** and **scale-free** properties  
- Evaluate **network resilience** under different failure scenarios (random, targeted, and community-level)

---

## 1. Research Questions

1. **Scale-free structure**  
   Does the ATN exhibit a heavy-tailed / near scale-free degree distribution?

2. **Small-world effect**  
   Does the ATN show the small-world property: **short paths + high clustering** compared to a random graph?

3. **Global hubs**  
   Which airports act as structurally influential hubs according to **PageRank, betweenness, and closeness centrality**?

4. **Communities**  
   How is the network organized into **regional communities**, and how are these communities interconnected?

5. **Resilience**  
   How robust is the ATN to:
   - random airport failures,  
   - targeted removal of high-centrality hubs, and  
   - failure of entire **communities/regions**?

---

## 2. Dataset

- **Source:** [OpenFlights](https://openflights.org/data.html) – Airport & Route data  
- **Files used (raw):**
  - `airports.dat`
  - `routes.dat`

Each **airport** record includes:

- Name, city, country  
- IATA code  
- Latitude, longitude, altitude  

Each **route** record includes:

- Airline, source airport, destination airport

### 2.1 Raw vs Cleaned Data

- **Raw size:**
  - ~7,698 airport entries  
  - ~67,663 flight routes  

- **After preprocessing and cleaning:**
  - **6,072 airports** (nodes)  
  - **37,042 directed routes** (edges)

### 2.2 Why This Dataset?

- Real-world, high-impact infrastructure (economy, tourism, logistics, pandemics)  
- Natural complex network: **airports → nodes**, **routes → directed edges**  
- Large and heterogeneous enough for rich structure, but small enough to run:
  - betweenness centrality (Brandes’ algorithm),
  - PageRank (power iteration),
  - Louvain community detection,
  - repeated resilience simulations
- Includes geographic attributes for:
  - **world-map visualizations**
  - **geographic interpretation of communities**
- Public and reproducible; widely used in network science examples

---

## 3. Graph Model

- **Graph type:** Directed graph \( G = (V, E) \)
- **Vertices:** Airports (identified by IATA code)  
- **Edges:** Directed routes (A → B) representing at least one scheduled flight from airport A to airport B

### 3.1 Preprocessing & Cleaning

Steps:

1. **Filter airports**
   - Keep only rows where:
     - `type == "airport"`
     - IATA code is valid and non-missing
   - Drop non-airport entries and rows with IATA `\N` or `NaN`.

2. **Filter routes**
   - Keep only routes whose source and destination IATA codes appear in the cleaned airport list.
   - Remove **self-loops** (routes from an airport to itself).
   - For structural analysis, treat multiple parallel routes between two airports as a single edge.

3. **Exports**
   - `data/processed/airports_clean.csv`
   - `data/processed/routes_clean.csv`

This yields a **consistent directed network** where all nodes and edges correspond to real airports and valid flight routes.

---

## 4. Methods & Algorithms

The analysis is implemented in:

- **Notebook:** `notebooks/Final_Code.ipynb`  
- **Language:** Python  
- **Main libraries:**
  - `pandas`, `numpy`
  - `networkx`
  - `matplotlib`, `plotly`
  - `python-louvain` or `networkit` (for Louvain / PLM)

### 4.1 Basic Network Metrics

Computed on:

- The full directed graph \(G\)
- The **Largest Strongly Connected Component (LSCC)**
- The **Largest Weakly Connected Component (LWCC)**

Metrics include:

- Number of nodes \(N\)
- Number of edges \(|E|\)
- Average in-degree / out-degree / total degree
- **Density**
- **Average shortest path length** \(L\) (on LSCC)
- **Diameter** (on LSCC / largest undirected component)
- **Global clustering coefficient** (transitivity)
- **Average local clustering coefficient**
- **Reciprocity** (fraction of bidirectional edges)
- **Degree assortativity**

**Algorithms:**  
Standard NetworkX implementations using:

- BFS / Dijkstra for shortest paths  
- Built-in clustering and component decomposition (linear in \(|V| + |E|\))

---

### 4.2 Core Components: LSCC and LWCC

Using NetworkX’s strongly and weakly connected component algorithms, we extract:

- **Largest Strongly Connected Component (LSCC)**
  - ~3,190 airports (~52.5% of all nodes)
  - Contains ~99.75% of all routes
- **Largest Weakly Connected Component (LWCC)**
  - ~3,231 airports (~53.2% of all nodes)
  - Contains ~99.88% of all routes

**Interpretation:**  
Just over half of the airports form a dense **directional backbone** that carries almost all routes; the rest are peripheral.

---

### 4.3 Degree Distribution & Scale-Free Behavior

To investigate **Research Question 1 (scale-free structure)**:

- Compute the degree sequence (in-, out-, total degree).
- Plot:
  - Degree distribution (PDF) on log–log scale.
  - Complementary cumulative distribution function (CCDF) on log–log scale.

**Observation:**

- Many airports have low degree.
- A few airports have **very high degree** (hundreds of connections).
- The tail appears approximately linear on log–log plots → **heavy-tailed / near scale-free** behavior.

This supports the picture of a **hub-dominated network** with a small number of mega-hubs and many small airports.

---

### 4.4 Centrality Measures & Global Hubs

To address **Research Question 3 (global hubs)**, we compute the following on the **LSCC**:

- **Degree centrality**
- **Eigenvector centrality**
- **Katz centrality**
- **PageRank**
- **Betweenness centrality**
- **Closeness centrality**

**Algorithmic details:**

- **PageRank:**  
  - Power iteration with damping factor (e.g., 0.85) until convergence.  
- **Betweenness centrality:**  
  - Brandes’ algorithm (exact) with complexity \(O(nm)\).  
- **Eigenvector & Katz:**  
  - Linear algebra-based solvers on the adjacency matrix.  
- **Closeness:**  
  - Based on all-pairs shortest paths from each node.

**Role of each centrality:**

- Degree/PageRank/Katz → high-traffic **hubs** and “important” airports.  
- Betweenness → **bridges** that lie on many shortest paths between regions.  
- Closeness → airports that are “globally central” in distance.

**Typical recurrent hubs:**

- FRA (Frankfurt), CDG (Paris), AMS (Amsterdam),  
- IST (Istanbul), ATL (Atlanta), ORD (Chicago),  
- PEK (Beijing), LHR (London Heathrow), DXB (Dubai),  
- LAX (Los Angeles), ANC (Anchorage), and others.

---

### 4.5 Community Detection (Louvain / PLM)

To address **Research Question 4 (communities)**, we use **Louvain community detection** on the undirected version of the LSCC.

**Algorithm:**

- Greedy **modularity maximization**:
  - Start with each node in its own community.
  - Iteratively move nodes between communities if modularity \(Q\) improves.
  - Aggregate communities into super-nodes and repeat.

**Result:**

- ~20 communities found.
- Modularity \(Q \approx 0.655\) → strong community structure.

**Interpretation:**

- Communities align closely with **geographic regions**, e.g.:
  - North America (ATL hub)
  - Western/Central Europe (CDG/FRA/AMS hubs)
  - China / East Asia (PEK hub)
  - Latin America (BOG hub)
  - Russia / Eastern Europe (DME hub)
  - Arctic / Northern Canada (ANC / YZF)
  - Pacific islands (POM, PPT), etc.

This gives a **mesoscale view** of the ATN as a set of regional blocks tied together by a few key inter-community bridges.

---

### 4.6 Resilience Analysis

To address **Research Question 5 (resilience)**, we simulate different failure scenarios on the **LSCC**.

#### 4.6.1 Node-Level Attacks

We consider three strategies:

1. **Random failures**
   - Iteratively remove random sets of airports.
   - After each removal, compute the size of the **largest SCC**.

2. **Targeted attacks by degree**
   - Sort airports by degree (descending).
   - Remove highest-degree airports first.
   - Track the size of the largest SCC after each step.

3. **Targeted attacks by betweenness**
   - Sort airports by betweenness centrality.
   - Remove highest-betweenness airports first.
   - Track the size of the largest SCC.

**Key metric:**  
Fraction of airports that need to be removed before the LSCC shrinks to **< 50%** of its original size.

**Results (approx.):**

- **Random failures:**  
  - ~**38%** of airports must be removed.

- **Targeted by degree:**  
  - ~**9.8%** of airports (highest degree) must be removed.

- **Targeted by betweenness:**  
  - ~**6.1%** of airports (highest betweenness) must be removed.

**Conclusion:**  
The ATN is:

- **Robust to random failures**, but  
- **Highly vulnerable** to targeted removal of hubs and bridges.

---

#### 4.6.2 Community-Level Failures (Novelty)

To introduce a **novel angle**, we also simulate **community-level failures** using the Louvain partition:

- Sort communities by size.
- Remove all airports belonging to one community at a time, starting from the largest.
- After each community removal, recompute the size of the largest connected component.

**Finding:**

- Removing the **two largest communities** (≈40.8% of airports) reduces the largest component to ≈49.2% of its original size.

**Interpretation:**  
The ATN’s resilience is not only about individual hubs; it also relies heavily on **entire regional communities** functioning as structural building blocks.

---

## 5. Results Summary

- **Scale-free-like structure:**  
  - Degree distribution is heavy-tailed: many small degree nodes, a few mega-hubs.

- **Small-world effect:**  
  - Average shortest path length ≈ 4 (on LSCC).  
  - Clustering coefficient ≈ 0.26, much higher than a comparable random graph.  

- **Global hubs & bridges:**  
  - Centrality analysis identifies:
    - Large hubs (FRA, CDG, AMS, IST, ATL, PEK, etc.).
    - Bridge airports (DXB, LAX, ANC, YYZ) that connect distant regions.

- **Communities & geography:**  
  - Louvain finds ~20 communities with high modularity (~0.655).  
  - Communities align with real-world regions and regional flight systems.

- **Resilience:**
  - Robust to random failures (≈38% random node removal to halve the backbone).
  - Fragile to targeted hub attacks (≈6–10% node removal suffices).
  - Vulnerable to **community-level failures** (loss of two major regional communities halves the backbone).

Overall, the ATN is **efficient and well-connected**, but this efficiency comes with **structural fragility** concentrated in a small set of airports and regions.

---


