## 算法概览
- 一个在weighted graph上使用的算法
- 初始条件为给定起点start和终点end
- 找到从起点到终点的least cost path

## 步骤
  1. 定义两个集合：
    visited存放存放[node, dist] pair，node为已经找到最短路径的结点，dist为对应的最短距离
    toStart为一个heap, 存放[node, dist] pair, node为结点，dist为从起点到该node的距离，heap按距离排序
  2. 将 [startNode, 0] 放入toStart 
  3. 遍历toStart中的node：
    - 如果该node已经在visited中，则从heap中移移除
      否则将该[node, dist]放入visited，并移除
    - 遍历该node所有邻居结点n
       - 如果当前邻居结点n已经在visited中则跳过
         否则，此时 从startNode到该n的距离dist 与 node到n的距离cost 之和
         将[n, dist+n.cost]放入toStart中
  4. 重复步骤3直到toStart变为控
  5. 此时visited中存放的就是node以及startNode到该node的最短距离

  - PS: 由于node放入toStart heap后会自动排序， 所以每次从heap中移出node，总能保证最短路径的node被放入visited中：
    也即，如果有node可以从多个路径到达，总是该node与最短路径的pair首先被从toStart heap中取出，并被放入visited中

## 伪代码
``` js
// visited: a set already get shortest cost
// toStart: a heap contains [node, cost] pairs with ordered distance,
// cost is distance from start node to current node, initialy only has [startNode, 0]

// initially, start from the starNode, and its cost is 0
toStart.add([startNode, 0]);

for node_cost_pair in toStart:
    if visited contains(node) {
        continue;
    }
    visited.add(node); // 
    for neighborNode in node.neighbors {
        if not visited neighborNode {
            toStart.add([neighborNode, cost+neighborNode.dist])
        }
    }
}

// after the loop, vivisted contains all nodes that are reachable from the given start node 
//and corresponding shorted distance (from start node)

```

## 演示
![alt text](https://raw.githubusercontent.com/jellylidong/CodeEatSleep/master/code/001_Dijkstra/Dijkstra_Animation.gif)






