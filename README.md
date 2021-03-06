# 1) Read the tabular data


```python
import pandas as pd
import numpy as np

df = pd.read_csv("exo3.csv")
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Age</th>
      <th>Income</th>
      <th>Student</th>
      <th>Credit_rating</th>
      <th>Buys_computer</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>&lt;=30</td>
      <td>Hight</td>
      <td>No</td>
      <td>Fair</td>
      <td>No</td>
    </tr>
    <tr>
      <td>1</td>
      <td>&lt;=30</td>
      <td>Hight</td>
      <td>No</td>
      <td>Excellent</td>
      <td>No</td>
    </tr>
    <tr>
      <td>2</td>
      <td>31...40</td>
      <td>Hight</td>
      <td>No</td>
      <td>Fair</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>3</td>
      <td>&gt;40</td>
      <td>Medium</td>
      <td>No</td>
      <td>Fair</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>4</td>
      <td>&gt;40</td>
      <td>Low</td>
      <td>Yes</td>
      <td>Fair</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>5</td>
      <td>&gt;40</td>
      <td>Low</td>
      <td>Yes</td>
      <td>Excellent</td>
      <td>No</td>
    </tr>
    <tr>
      <td>6</td>
      <td>31...40</td>
      <td>Low</td>
      <td>Yes</td>
      <td>Excellent</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>7</td>
      <td>&lt;=30</td>
      <td>Medium</td>
      <td>No</td>
      <td>Fair</td>
      <td>No</td>
    </tr>
    <tr>
      <td>8</td>
      <td>&lt;=30</td>
      <td>Low</td>
      <td>Yes</td>
      <td>Fair</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>9</td>
      <td>&gt;30</td>
      <td>Medium</td>
      <td>Yes</td>
      <td>Fair</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>10</td>
      <td>&lt;=30</td>
      <td>Medium</td>
      <td>Yes</td>
      <td>Excellent</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>11</td>
      <td>31...40</td>
      <td>Medium</td>
      <td>No</td>
      <td>Excellent</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>12</td>
      <td>31...40</td>
      <td>High</td>
      <td>Yes</td>
      <td>Fair</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>13</td>
      <td>&gt;40</td>
      <td>Medium</td>
      <td>No</td>
      <td>Excellent</td>
      <td>No</td>
    </tr>
  </tbody>
</table>
</div>



# 2) Generate the optimal split Decision Tree based on entropy


```python
def entropy(ni, n):
    if ni == 0 or ni == n:
        return 0
    tmp =  ni / n
    return - tmp * np.log2(tmp)


def computeNodeEntropy(node, label):
    e, n = 0, len(node)
    for c in node[label].unique(): # loop trough each class
        node_i = node[node[label] == c]
        n_i = len(node_i)
        e_i = entropy(n_i, n)
        e += e_i
        #print(f"{c} : {n_i}/{n} -> ei={e_i:.2f}")
    #print(f'entropy = {e:.2f}\n')
    return e
        

def DecisionTree(t , label):
    # get repartion of the label
    classes = t[label].unique()
    if len(classes) == 1: return classes[0]
    
    # compute the base node entropy
    H = computeNodeEntropy(t, label)
    #print(f'H : {H}')
    n = len(t)
    
    #if H == 0:
    #    print(len(classes))
    #    print(classes)
    #    raise Exception('H == 0')
    #    return classes
        
    # retrieve all the features that can be splited on
    features = t.columns.to_numpy().tolist()
    try:
        features.remove(label)
    except ValueError:
        raise Exception('label not present !')
    #print(features)
    
    # find the best split
    split_opti = None
    gain_split_max = 0
    for split in features:
        split_classes = t[split].unique()
        gain_split = H
        
        for split_class in split_classes:
            #print(f"{split} : '{split_class}'")
            ti = t[t[split] == split_class]
            ni = len(ti)
            ei = computeNodeEntropy(ti, label)
            #print(ni, ei)
            gain_split -= (ni/n) * ei

        #print('gain_split', gain_split)
        if gain_split > gain_split_max:
            gain_split_max = gain_split
            split_opti = split
        #print()
    
    #print(f"features : {features}")
    #print(f'best split feature : {split_opti}')
    out = {split_opti : {}}
    for split_class in t[split_opti].unique():
        ti = t[t[split_opti] == split_class]
        ti = ti.drop(split_opti, inplace=False, axis=1)
        
        out[split_opti][split_class] = DecisionTree(ti, label)
        
    return out
        

tree = DecisionTree(df, 'Buys_computer')
print(tree)
```

    {'Age': {'<=30': {'Student': {'No': 'No', 'Yes': 'Yes'}}, '31...40': 'Yes', '>40': {'Credit_rating': {'Fair': 'Yes', 'Excellent': 'No'}}, '>30': 'Yes'}}
    

## 3) Plot the Tree `conda install graphviz`


```python
import pydot
import uuid
from IPython.display import Image

def generate_unique_node():
    """ Generate a unique node label."""
    return str(uuid.uuid1())

def create_node(graph, label, shape='oval'):
    node = pydot.Node(generate_unique_node(), label=label, shape=shape)
    graph.add_node(node)
    return node

def create_edge(graph, node_parent, node_child, label):
    link = pydot.Edge(node_parent, node_child, label=label)
    graph.add_edge(link)
    return link

def walk_tree(graph, dictionary, prev_node=None):
    """ Recursive construction of a decision tree stored as a dictionary """
    for parent, child in dictionary.items():
        # root
        if not prev_node: 
            root = create_node(graph, parent)
            walk_tree(graph, child, root)
            continue
            
        # node
        if isinstance(child, dict):
            for p, c in child.items():
                n = create_node(graph, p)
                create_edge(graph, prev_node, n, str(parent))
                walk_tree(graph, c, n)
    
        # leaf
        else: 
            leaf = create_node(graph, str(child), shape='box')
            create_edge(graph, prev_node, leaf, str(parent))

def plot_tree(dictionary, filename="DecisionTree.png"):
    graph = pydot.Dot(graph_type='graph')
    walk_tree(graph, tree)
    graph.write_png(filename)
        

plot_tree(tree, filename="exo3.png")
Image("exo3.png")

```




![png](exo3.png)


