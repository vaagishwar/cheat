
### 1. A* Campus Navigation
```python
import heapq

g = {
    'Gate': [('C',2,1),('B',4,2)],
    'B':[('D',3,1)], 'C':[('D',2,1),('E',4,2)],
    'D':[('E',2,1)], 'E':[]
}
h = {'Gate':6,'B':4,'C':4,'D':2,'E':0}

def astar(start, goal):
    q=[(h[start],0,start,[start])]
    while q:
        f,c,u,path=heapq.heappop(q)
        if u==goal: return path,c
        for v,d,crowd in g[u]:
            cost=d+crowd
            heapq.heappush(q,(c+cost+h[v],c+cost,v,path+[v]))

path,cost=astar('Gate','E')
print("Route:",' -> '.join(path))
print("Total Cost:",cost)
print("Optimal because it has minimum distance + crowd cost.")
```

### 2. K-Means Customer Segmentation + Elbow
```python
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans

X=pd.DataFrame({
'income':[25,30,80,90,45,50,85,35],
'frequency':[2,3,10,12,5,6,11,3],
'spending':[20,25,90,95,50,55,85,30],
'order':[500,600,2500,3000,1000,1200,2800,700]})

X=StandardScaler().fit_transform(X)

w=[]
for k in range(1,6):
    m=KMeans(n_clusters=k,n_init=10,random_state=1).fit(X)
    w.append(m.inertia_)
plt.plot(range(1,6),w,'o-'); plt.xlabel("Clusters"); plt.ylabel("WCSS"); plt.show()

m=KMeans(n_clusters=3,n_init=10,random_state=1).fit(X)
print("Labels:",m.labels_)
print("Centroids:\n",m.cluster_centers_)
print("Segments: Low-value, Regular, High-value customers")
```

### 3. Machine Fault Expert System — Forward Chaining
```python
rules={
'overheating':'Cooling Failure',
'abnormal vibration':'Bearing Damage',
'unusual noise':'Bearing Damage',
'low output':'Motor Fault',
'power failure':'Electrical Fault'
}

symptoms=input("Enter symptoms: ").lower().split(',')
facts=[x.strip() for x in symptoms]
used=[]

for s in facts:
    if s in rules:
        used.append((s,rules[s]))

fault=max(set(f for s,f in used),key=lambda x:sum(f==x for s,f in used)) \
       if used else "Unknown Fault"

actions={
'Cooling Failure':'Check cooling system',
'Bearing Damage':'Inspect/replace bearing',
'Motor Fault':'Inspect motor',
'Electrical Fault':'Check power supply'}

print("Diagnosis:",fault)
print("Maintenance:",actions.get(fault,"Inspect machine"))
print("Rules Used:",used)
```

### 4. ANN Student Academic Risk Prediction
```python
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.neural_network import MLPClassifier
from sklearn.metrics import accuracy_score,precision_score,recall_score,f1_score

X=np.array([[90,85,80,5,85],[60,55,50,2,60],[80,75,70,4,78],
            [50,45,40,1,50],[95,90,92,6,90],[65,60,55,2,65],
            [85,80,82,5,88],[45,40,35,1,45]])
y=[0,1,0,1,0,1,0,1]   # 0=Not Risk, 1=At Risk

a,b,c,d=train_test_split(X,y,test_size=.25,random_state=1)
m=MLPClassifier(hidden_layer_sizes=(8,),max_iter=1000,random_state=1).fit(a,c)
p=m.predict(b)

print("Accuracy:",accuracy_score(d,p))
print("Precision:",precision_score(d,p,zero_division=0))
print("Recall:",recall_score(d,p,zero_division=0))
print("F1:",f1_score(d,p,zero_division=0))

new=[[55,50,45,2,55]]
print("Prediction:", "At Risk" if m.predict(new)[0] else "Not At Risk")
```

### 5. Bayesian Network + HMM Weather Prediction
```python
import numpy as np

# Bayesian inference: P(Rain | Humidity, Cloud)
rain=0.7
p_rain=.8*.9*.7 + .2*.3*.7
print("P(Rain):",round(p_rain,2))

# HMM: states = Dry, Damp, Wet
states=['Dry','Damp','Wet']
A=np.array([[.7,.2,.1],[.2,.6,.2],[.1,.3,.6]])
B=np.array([[.8,.2,.0],[.2,.6,.2],[.0,.2,.8]])
obs=[0,1,2,2]

dp=np.zeros((len(obs),3)); back=np.zeros((len(obs),3),int)
dp[0]=B[:,obs[0]]/3

for t in range(1,len(obs)):
    for j in range(3):
        x=dp[t-1]*A[:,j]
        back[t,j]=np.argmax(x)
        dp[t,j]=max(x)*B[j,obs[t]]

path=[np.argmax(dp[-1])]
for t in range(len(obs)-1,0,-1):
    path.append(back[t,path[-1]])
path=path[::-1]

print("State Sequence:",[states[i] for i in path])
print("Final Prediction:", "Rainy" if p_rain>.5 else "Dry")
```
