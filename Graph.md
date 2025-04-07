# 📘 Graph

A **graph** is a set of **vertices** (nodes) connected via **edges**, forming a network.

---

## 🔹 Vertex

- Structures used to store data in a graph.
- Represented as nodes (e.g., 1, 3, 7…).
- Also known as **vertices**.

 <p align="center">
  <img width="385" alt="Vertices" src="https://github.com/user-attachments/assets/ce47de48-8fed-4edc-8e46-eddb077ec74f" />
</p>

---

## 🔹 Edge

- A pair (x, y) is called an **edge**, indicating that vertex `x` is connected to vertex `y`.
- May contain **weight/cost** (e.g., distance or time).
- Usually represented using **straight lines**.

  <p align="center"><img width="565" alt="A Graph having 3 Edges" src="https://github.com/user-attachments/assets/b98d3a54-db3a-4d4c-9b38-d795763797d8" /></p>


---

# 📊 Types of Graphs

## Undirected Graph

- Edges are **bi-directional**.
- Example: pair (0,1) indicates you can go from vertex 0 to 1 or 1 to 0.

  <p align="center"><img width="411" alt="An Undirected Graph having 3 vertices and 2 edges" src="https://github.com/user-attachments/assets/b9d92c61-f330-47e4-934a-1d52bff70dbc" /></p>


## Directed Graph

- Edges are **unidirectional**.
- Example: pair (0,1) means you can only go from vertex 0 to vertex 1.

  <p align="center""><img width="425" alt="A Directed Graph having 3 vertices and 2 edges" src="https://github.com/user-attachments/assets/87db695d-7b41-4ef3-a8f5-f34d623d25a5" /></p>


---

# 🧠 Graph Terminologies

### Degree of Vertex

- Total number of edges connected to a vertex.

  <p align="center"><img width="768" alt="A Graph having 6 nodes and 7 edges" src="https://github.com/user-attachments/assets/1d0401f6-1592-4efe-adf9-3b4d5efb33c4" /></p>


#### Types of Degree:

- **In-Degree**: Number of **incoming** edges to a vertex.
- **Out-Degree**: Number of **outgoing** edges from a vertex.
  
<p align="center"><img width="701" alt="Pasted Graphic 1" src="https://github.com/user-attachments/assets/3468463d-554c-48f7-a018-1ede8536456f" /></p>

  

---

### Parallel Edges

- Two or more edges between the **same two vertices**.

  <p align="center"><img width="590" alt="Pasted Graphic 2" src="https://github.com/user-attachments/assets/eaead978-3272-41ab-b1d8-30f020b27c71" /></p>


---

### Self Loop

- An edge that connects a vertex to **itself**.
- Example: pair (x, x).

  <p align="center"><img width="257" alt="A graph showing self-loop" src="https://github.com/user-attachments/assets/0d76bdf0-fc9a-49b4-9195-698f9bc73931" /></p>


---

# ⚖️ Graph Representation: Adjacency List vs Adjacency Matrix

## 🔍 General Overview

| Feature / Operation       | Adjacency List                                | Adjacency Matrix                                   |
|---------------------------|-----------------------------------------------|----------------------------------------------------|
| **Description**           | List where each vertex stores adjacent nodes | 2D array where cell (i, j) = 1 if edge i→j exists |
| **Space Complexity**      | O(V + E)                                      | O(V²)                                              |
| **Add Edge**              | O(1) (amortized)                              | O(1)                                               |
| **Remove Edge**           | O(E/V)                                        | O(1)                                               |
| **Check Edge Existence**  | O(E/V)                                        | O(1)                                               |
| **Get All Neighbors**     | O(E/V)                                        | O(V)                                               |
| **Iterate Over All Edges**| O(V + E)                                      | O(V²)                                              |

---

## ✅ Advantages & ❌ Disadvantages

| Aspect                    | Adjacency List                                  | Adjacency Matrix                                 |
|---------------------------|--------------------------------------------------|--------------------------------------------------|
| **Best For**              | Sparse graphs (E << V²)                         | Dense graphs (E ≈ V²)                            |
| **Worst For**             | Dense graphs                                    | Sparse graphs (space waste)                      |
| **Pros**                  | ✔ Space-efficient for sparse graphs <br> ✔ Fast neighbor iteration | ✔ Constant-time edge check <br> ✔ Simple to implement |
| **Cons**                  | ❌ Slower edge existence check <br> ❌ Complex weighted edge lookup | ❌ Space-wasting in sparse graphs <br> ❌ Slower neighbor iteration |

---

## 📌 When to Use Which

| Use Case                       | Recommended Representation  |
|--------------------------------|-----------------------------|
| Sparse Graph                   | ✅ Adjacency List           |
| Dense Graph                    | ✅ Adjacency Matrix         |
| Fast Edge Lookup Needed        | ✅ Adjacency Matrix         |
| Fast Neighbor Iteration Needed| ✅ Adjacency List           |
| Memory Efficient Applications | ✅ Adjacency List           |
| Floyd-Warshall Algorithm       | ✅ Adjacency Matrix         |
| DFS, BFS, Dijkstra w/ Heap     | ✅ Adjacency List           |

---

## ⚙️ Algorithm Suitability

| Algorithm / Pattern                | Adjacency List | Adjacency Matrix            |
|-----------------------------------|----------------|-----------------------------|
| DFS / BFS                         | ✅              | ✅ (slower neighbor access) |
| Dijkstra (Min Heap)               | ✅ Efficient    | ❌ Slower row iteration     |
| Prim's (Min Heap)                 | ✅              | ✅ Simple with arrays       |
| Floyd-Warshall                    | ❌              | ✅ Very efficient           |
| Warshall’s Transitive Closure     | ❌              | ✅                          |
| Matrix Exponentiation / DP on Graphs | ❌           | ✅                          |
| Topological Sort / Cycle Detection| ✅              | ❌ Inefficient              |

---

# 🌐 Sparse vs Dense Graphs

## ✅ Sparse Graph

- Has **few edges**.
- Most vertex pairs are **not connected**.
- Edges (E) << V².
- Example: Social networks with few friends.

### Pros:

- ✔ Memory-efficient (Adjacency List)
- ✔ Faster DFS/BFS

### Visual:

```
A — B    C    D — E
```

---

## 🟡 Dense Graph

- Has **many edges**, close to maximum.
- Most vertices are highly connected.
- Edges (E) ≈ V².
- Example: Complete graph with 1000 nodes.

### Pros:

- ✔ Edge existence check is fast (Adjacency Matrix)

### Visual:

```
A — B
| \/ |
C — D
```

---

## 🔁 Formula for Maximum Edges

- **Undirected Graph**: `V(V - 1) / 2`
- **Directed Graph**: `V(V - 1)`

---

# 💡 TL;DR

| Feature        | Sparse Graph              | Dense Graph              |
|----------------|---------------------------|--------------------------|
| **Edges**      | Few (E << V²)             | Many (E ≈ V²)           |
| **Storage**    | Adjacency List preferred  | Adjacency Matrix preferred |
| **Use Case**   | Road/social networks      | Flight/mesh networks    |

---
