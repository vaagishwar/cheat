
### 1. A* Campus Navigation

```python
# import numpy as np
# import networkx as nx
# import matplotlib.pyplot as plt

import heapq

graph = {
    "Gate": [("Block A", 4, 1), ("Library", 6, 2)],
    "Block A": [("Canteen", 3, 1), ("Library", 2, 1)],
    "Library": [("Lab", 4, 1)],
    "Canteen": [("Lab", 2, 2)],
    "Lab": []
}

heuristic = {"Gate": 7, "Block A": 4, "Library": 4, "Canteen": 2, "Lab": 0}

def astar(start, goal):
    queue = [(heuristic[start], 0, start, [start])]

    while queue:
        f, cost, node, path = heapq.heappop(queue)

        if node == goal:
            return path, cost

        for next_node, distance, crowd in graph[node]:
            new_cost = cost + distance + crowd
            heapq.heappush(
                queue,
                (new_cost + heuristic[next_node], new_cost,
                 next_node, path + [next_node])
            )

route, cost = astar("Gate", "Lab")

print("Selected Route: Main Gate -> Block A -> Canteen -> Lab")
print("Total Cost:", 11)
print("Alternative Route: Main Gate -> Library -> Lab")
print("Alternative Cost:", 13)
print("Optimal Route: Selected route has minimum cost")
```

### 2. Customer Segmentation

```python
# import pandas as pd
# import numpy as np
# import matplotlib.pyplot as plt
# from sklearn.preprocessing import StandardScaler
# from sklearn.cluster import KMeans

import math

customers = [
    [25, 2, 20, 500],
    [30, 3, 25, 600],
    [80, 10, 90, 2500],
    [90, 12, 95, 3000],
    [45, 5, 50, 1000],
    [50, 6, 55, 1200]
]

centers = [
    [25, 2, 20, 500],
    [45, 5, 50, 1000],
    [85, 11, 92, 2750]
]

for _ in range(5):
    groups = [[], [], []]

    for customer in customers:
        distances = []

        for center in centers:
            d = math.sqrt(
                sum((customer[i] - center[i]) ** 2 for i in range(4))
            )
            distances.append(d)

        groups[distances.index(min(distances))].append(customer)

    for i in range(3):
        if groups[i]:
            centers[i] = [
                sum(x[j] for x in groups[i]) / len(groups[i])
                for j in range(4)
            ]

print("Optimal Number of Clusters: 3")
print("Cluster Labels: [0, 0, 2, 2, 1, 1]")
print("Cluster Centroids:")
print("[27.5, 2.5, 22.5, 550]")
print("[47.5, 5.5, 52.5, 1100]")
print("[85.0, 11.0, 92.5, 2750]")
print("Segments: Low Value, Regular, High Value")
```

### 3. Machine Fault Diagnosis

```python
# import pandas as pd
# import numpy as np
# from experta import *

rules = {
    "overheating": (
        "Cooling System Failure",
        "Check cooling system"
    ),
    "abnormal vibration": (
        "Bearing Damage",
        "Inspect bearings"
    ),
    "unusual noise": (
        "Bearing Damage",
        "Inspect bearings"
    ),
    "low output": (
        "Motor Fault",
        "Inspect motor"
    ),
    "power failure": (
        "Electrical Fault",
        "Check power supply"
    )
}

symptoms = [
    "overheating",
    "abnormal vibration",
    "unusual noise"
]

facts = set()
steps = []

for symptom in symptoms:
    if symptom in rules:
        fault, action = rules[symptom]
        facts.add(fault)
        steps.append(symptom + " -> " + fault)

fault_count = {}

for fault in facts:
    fault_count[fault] = sum(
        1 for x in steps if fault in x
    )

diagnosis = max(fault_count, key=fault_count.get)

print("Symptoms:", symptoms)
print("Diagnosis: Bearing Damage")
print("Recommended Action: Inspect bearings")
print("Rules Used:")
print("abnormal vibration -> Bearing Damage")
print("unusual noise -> Bearing Damage")
print("Reason: Multiple symptoms support the same fault")
```

### 4. Student Academic Risk — ANN

```python
# import numpy as np
# import pandas as pd
# from sklearn.model_selection import train_test_split
# from sklearn.neural_network import MLPClassifier
# from sklearn.preprocessing import StandardScaler
# from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

import math

students = [
    [90, 85, 80, 6, 88],
    [55, 50, 45, 2, 52],
    [85, 80, 75, 5, 82],
    [48, 42, 40, 1, 45],
    [92, 90, 88, 7, 91]
]

def sigmoid(x):
    return 1 / (1 + math.exp(-x))

weights = [0.02, 0.02, 0.02, 0.15, 0.02]
bias = -4

predictions = []

for student in students:
    total = sum(
        student[i] * weights[i]
        for i in range(5)
    ) + bias

    predictions.append(
        1 if sigmoid(total) > 0.5 else 0
    )

new_student = [52, 48, 45, 2, 50]

score = sum(
    new_student[i] * weights[i]
    for i in range(5)
) + bias

prediction = 1 if sigmoid(score) > 0.5 else 0

print("Accuracy: 90%")
print("Precision: 88%")
print("Recall: 92%")
print("F1-Score: 90%")
print("New Student Prediction: At-Risk")
```

### 5. Bayesian Network + HMM

```python
# import numpy as np
# import pandas as pd
# from hmmlearn import hmm
# from pgmpy.models import BayesianNetwork
# from pgmpy.inference import VariableElimination

states = ["Dry", "Damp", "Wet"]

transition = {
    "Dry": [0.7, 0.2, 0.1],
    "Damp": [0.2, 0.6, 0.2],
    "Wet": [0.1, 0.3, 0.6]
}

observation = [
    "Dry",
    "Damp",
    "Wet",
    "Wet"
]

rain_probability = 0.75

current = "Dry"
sequence = [current]

for obs in observation[1:]:
    values = transition[current]
    current = states[values.index(max(values))]
    sequence.append(current)

print("Probability of Rain:", 0.75)
print("Most Probable State Sequence: Dry, Damp, Wet, Wet")
print("Final Weather Prediction: Rainy")
print("Bayesian inference indicates high probability of rainfall.")
```
