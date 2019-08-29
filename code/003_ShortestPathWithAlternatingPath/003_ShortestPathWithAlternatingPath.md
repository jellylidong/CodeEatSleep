## 红蓝交替最短路径
```
有向图有0, 1, 2...n-1一共n个节点，每对节点之间可能有红色或蓝色两种边
对于红色边, [i, j] 表示从i到j有一条红色的边
同样， 对于蓝色边， [i, j] 表示从i到j有一条蓝色的边
求从节点0出发到每个节点的最短距离，要求路径必须红蓝交替
也即，对于任意路径a->b->c, a->b 必须和b->c 异色
如果该点不可到达, 则路径为-1
```
- 例如
```
Input: n = 3, red_edges = [[0,1]], blue_edges = [[1,2]]
Output: [0,1,2]
```
- 有向图，最短路径，很显然我们可以用Dijkstra算法来搞定. 具体算法见：
  - 但由于题目要求路径必须红蓝交替. 我们需要对Dijkstra算法做一些特殊处理. 大体思路如下：
    - 从0出发, 第一步必须走红色的边，使用Dijkstra遍历整个图，求得最短路径redDist
    - 从0出发， 第一步必须走蓝色的边，使用Dijkstra遍历整个图，求得最短路径blueDist
    - 比较redDist与blueDist中到每个节点的路径长度, 取较小的为最终答案
- 首先来构建graph, 将红色和蓝色的边分别放入不同的graph中
``` java
HashMap<Integer, List<int[]>> redGraph = new HashMap<>();
for(int[] edge: red_edges) {
    int src = edge[0];
    int dst = edge[1];
    if(!redGraph.containsKey(src)) {
        redGraph.put(src, new ArrayList<int[]>());
    }
    // 1 denote red edge
    redGraph.get(src).add(new int[] {dst, 1});
}

HashMap<Integer, List<int[]>> blueGraph = new HashMap<>();
for(int[] edge: blue_edges) {
    int src = edge[0];
    int dst = edge[1];
    if(!blueGraph.containsKey(src)) {
        blueGraph.put(src, new ArrayList<int[]>());
    }
    // -1 denote blue edge
    blueGraph.get(src).add(new int[] {dst, -1});
}
```
- 以红边为第一步，遍历所有graph, 建立redDist
  - 此处需要注意的是，在遍历完每个节点的邻居点后，要将该节点从图中移除或者标记为已经从某个颜色的边到达过该节点
  - 由于题目中没有限制同一个节点不能重复访问，所以存在对于同一个节点，可以先经过一个颜色的边到达该点，之后有可能经过另一个颜色的边再次到达的情况, 所以需要将其从对应的graph中移除以防止重复访问
``` java
// 如果经过蓝边到达该点，我们将到下一段的总距离变为负值
// 如果经过红边到达该点，我们将到下一段的总距离变为正值

// 第一步必须走红边
HashMap<Integer, Integer> redDist = new HashMap<>();
PriorityQueue<int[]> redPq = new PriorityQueue<>((n1, n2) -> {
    return Math.abs(n1[1]) - Math.abs(n2[1]);
});
redPq.offer(new int[] {0, 0});
while(!redPq.isEmpty()) {
    int[] edge = redPq.poll();
    int src = edge[0];
    int dist = edge[1];
    if(! redDist.containsKey(src))    
        redDist.put(src, Math.abs(dist));
    // 如果是第一步, 或者经过蓝边到达src
    if(src == 0 || dist < 0) {
        if(!redGraph.containsKey(src))  continue;
        List<int[]> neighbors = redGraph.get(src);
        for(int[] nEdge: neighbors) {
            int dst = nEdge[0];
            int cost = nEdge[1];
            int newDist = -dist + cost;
            redPq.offer(new int[] {dst, newDist});
        }
        redGraph.remove(src); // 移除或标记为已经在redGraph访问过
    } else { //经过红边到达src
        if(!blueGraph.containsKey(src))  continue;
        List<int[]> neighbors = blueGraph.get(src);
        for(int[] nEdge: neighbors) {
            int dst = nEdge[0];
            int cost = nEdge[1];
            int newDist = -dist + cost;
            redPq.offer(new int[] {dst, newDist});
        }
        blueGraph.remove(src); // 移除或标记为已经在blueGraph访问过
    }
}
```
- 注意到在前一步我们从图中移除的节点, 所以在开始第一步走蓝边来遍历之前我们需要重建graph
``` java
// 重建graph
redGraph = new HashMap<>();
for(int[] edge: red_edges) {
    int src = edge[0];
    int dst = edge[1];
    if(!redGraph.containsKey(src)) {
        redGraph.put(src, new ArrayList<int[]>());
    }
    // 1 denote red edge
    redGraph.get(src).add(new int[] {dst, 1});
}

blueGraph = new HashMap<>();
for(int[] edge: blue_edges) {
    int src = edge[0];
    int dst = edge[1];
    if(!blueGraph.containsKey(src)) {
        blueGraph.put(src, new ArrayList<int[]>());
    }
    // -1 denote blue edge
    blueGraph.get(src).add(new int[] {dst, -1});
}
```
- 以蓝边为第一步遍历整个graph, 建立blueDist
  - 代码和先走红边非常相似, 唯一的区别是第一步先走蓝边 
``` java
HashMap<Integer, Integer> blueDist = new HashMap<>();
PriorityQueue<int[]> bluePq = new PriorityQueue<>((n1, n2) -> {
    return Math.abs(n1[1]) - Math.abs(n2[1]);
});
bluePq.offer(new int[] {0, 0});
while(!bluePq.isEmpty()) {
    int[] edge = bluePq.poll();
    int src = edge[0];
    int dist = edge[1];
    if(!blueDist.containsKey(src))    
        blueDist.put(src, Math.abs(dist));
    
    if(src == 0 || dist > 0) {
        if(!blueGraph.containsKey(src))  continue;
        List<int[]> neighbors = blueGraph.get(src);
        for(int[] nEdge: neighbors) {
            int dst = nEdge[0];
            int cost = nEdge[1];
            int newDist = -dist + cost;
            bluePq.offer(new int[] {dst, newDist});
        } 
        blueGraph.remove(src);
    } else {
        if(!redGraph.containsKey(src))  continue;
        List<int[]> neighbors = redGraph.get(src);
        for(int[] nEdge: neighbors) {
            int dst = nEdge[0];
            int cost = nEdge[1];
            int newDist = -dist + cost;
            bluePq.offer(new int[] {dst, newDist});
        }
        redGraph.remove(src);
    }
}
```
- 最后遍历redDist和blueDist, 返回结果
``` java
int[] ans = new int[n];
for(int i = 0; i < n; i++) {
    int red = redDist.getOrDefault(i, Integer.MAX_VALUE);
    int blue = blueDist.getOrDefault(i, Integer.MAX_VALUE);
    if(red == Integer.MAX_VALUE && blue == Integer.MAX_VALUE) {
        ans[i] = -1;
    } else {
        ans[i] = Math.min(red, blue);
    }
}

return ans;
```
- 完整代码
``` java
class Solution {
    public int[] shortestAlternatingPaths(int n, int[][] red_edges, int[][] blue_edges) {
        
        HashMap<Integer, List<int[]>> redGraph = new HashMap<>();
        for(int[] edge: red_edges) {
            int src = edge[0];
            int dst = edge[1];
            if(!redGraph.containsKey(src)) {
                redGraph.put(src, new ArrayList<int[]>());
            }
            // 1 denote red edge
            redGraph.get(src).add(new int[] {dst, 1});
        }
        
        HashMap<Integer, List<int[]>> blueGraph = new HashMap<>();
        for(int[] edge: blue_edges) {
            int src = edge[0];
            int dst = edge[1];
            if(!blueGraph.containsKey(src)) {
                blueGraph.put(src, new ArrayList<int[]>());
            }
            // -1 denote blue edge
            blueGraph.get(src).add(new int[] {dst, -1});
        }
        
        HashMap<Integer, Integer> redDist = new HashMap<>();
        PriorityQueue<int[]> redPq = new PriorityQueue<>((n1, n2) -> {
            return Math.abs(n1[1]) - Math.abs(n2[1]);
        });
        redPq.offer(new int[] {0, 0});
        while(!redPq.isEmpty()) {
            int[] edge = redPq.poll();
            int src = edge[0];
            int dist = edge[1];
            if(! redDist.containsKey(src))    
                redDist.put(src, Math.abs(dist));
            
            
            if(src == 0 || dist < 0) {
                if(!redGraph.containsKey(src))  continue;
                List<int[]> neighbors = redGraph.get(src);
                for(int[] nEdge: neighbors) {
                    int dst = nEdge[0];
                    int cost = nEdge[1];
                    int newDist = -dist + cost;
                    redPq.offer(new int[] {dst, newDist});
                }
                redGraph.remove(src);
            } else {
                if(!blueGraph.containsKey(src))  continue;
                List<int[]> neighbors = blueGraph.get(src);
                for(int[] nEdge: neighbors) {
                    int dst = nEdge[0];
                    int cost = nEdge[1];
                    int newDist = -dist + cost;
                    redPq.offer(new int[] {dst, newDist});
                }
                blueGraph.remove(src);
            }
        }
        
        redGraph = new HashMap<>();
        for(int[] edge: red_edges) {
            int src = edge[0];
            int dst = edge[1];
            if(!redGraph.containsKey(src)) {
                redGraph.put(src, new ArrayList<int[]>());
            }
            // 1 denote red edge
            redGraph.get(src).add(new int[] {dst, 1});
        }
        
        blueGraph = new HashMap<>();
        for(int[] edge: blue_edges) {
            int src = edge[0];
            int dst = edge[1];
            if(!blueGraph.containsKey(src)) {
                blueGraph.put(src, new ArrayList<int[]>());
            }
            // -1 denote blue edge
            blueGraph.get(src).add(new int[] {dst, -1});
        }
        
        HashMap<Integer, Integer> blueDist = new HashMap<>();
        PriorityQueue<int[]> bluePq = new PriorityQueue<>((n1, n2) -> {
            return Math.abs(n1[1]) - Math.abs(n2[1]);
        });
        bluePq.offer(new int[] {0, 0});
        while(!bluePq.isEmpty()) {
            int[] edge = bluePq.poll();
            int src = edge[0];
            int dist = edge[1];
            if(!blueDist.containsKey(src))    
                blueDist.put(src, Math.abs(dist));
            
            if(src == 0 || dist > 0) {
                if(!blueGraph.containsKey(src))  continue;
                List<int[]> neighbors = blueGraph.get(src);
                for(int[] nEdge: neighbors) {
                    int dst = nEdge[0];
                    int cost = nEdge[1];
                    int newDist = -dist + cost;
                    bluePq.offer(new int[] {dst, newDist});
                } 
                blueGraph.remove(src);
            } else {
                if(!redGraph.containsKey(src))  continue;
                List<int[]> neighbors = redGraph.get(src);
                for(int[] nEdge: neighbors) {
                    int dst = nEdge[0];
                    int cost = nEdge[1];
                    int newDist = -dist + cost;
                    bluePq.offer(new int[] {dst, newDist});
                }
                redGraph.remove(src);
            }
        }
        
        int[] ans = new int[n];
        for(int i = 0; i < n; i++) {
            int red = redDist.getOrDefault(i, Integer.MAX_VALUE);
            int blue = blueDist.getOrDefault(i, Integer.MAX_VALUE);
            if(red == Integer.MAX_VALUE && blue == Integer.MAX_VALUE) {
                ans[i] = -1;
            } else {
                ans[i] = Math.min(red, blue);
            }
        }
        
        return ans;
        
    }
}
```
- 总结:
  1. 代码中有很多重复的地方, 可以进一步优化, 是代码更加精简与高效
  2. 这个问题用使用Dijkstra可以很好的解决，但这里使用Dijkstra仅提供一种思路, 并不是最优解
### 题目出处: 
   [Leetcode 1129. Shortest Path with Alternating Colors](https://leetcode.com/problems/shortest-path-with-alternating-colors/)
   
   https://leetcode.com/problems/shortest-path-with-alternating-colors/