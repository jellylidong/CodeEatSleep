## Leetcode 网络延时问题
```
给定一个单向联通网络times[], times[i] = (u ,v, w), u为起点，v为目标点, w为u到v的距离
已知这个网络中有N个结点, 求当我们从节点K发出信号时，需要多久整个网络可以收到这个信号
```
- 例如
  ![002.png](https://raw.githubusercontent.com/jellylidong/CodeEatSleep/master/code/002_NetworkDelayTime/002.png)
    ```
    Input: times = [[2,1,1],[2,3,1],[3,4,1]], N = 4, K = 2
    Output: 2
    ```

- 这是一个典型的单向联通网络求最短路径的题目，我们可以用哦个Dijkstra算法来求得从点K到所有节点的最短路径，所有最短路径中的最大值即为信号传遍整个网络所需的时间, 具体算法可以参考这里: 

- 首先，我们来构建graph, 将所给的times[]转化成graph
  ```java
  // build graph
    HashMap<Integer, List<int[]>> graph = new HashMap<>();
    for(int[] time: times) {
        int src = time[0];
        int dst = time[1];
        int cost = time[2];
        if(!graph.containsKey(src)) {
            graph.put(src, new ArrayList<int[]>());
        }
        graph.get(src).add(new int[] {dst, cost});
    }
  ```

- 然后初始化 visited 集合和 toStart heap
  ```java
    HashMap<Integer, Integer> visited = new HashMap<>();
    PriorityQueue<int[]> toStart = new PriorityQueue<>((a1, a2) -> {
        return a1[1] - a2[1];
    }); 
    toStart.offer(new int[] {K, 0});
  ``` 

- 遍历及更新toStart, 同时更新visited
  ```java
    while(!toStart.isEmpty()) {
        int[] node = toStart.poll();
        int src = node[0];
        int dist = node[1];

        if(visited.containsKey(src))    continue;
        visited.put(src, dist);

        // src has no neighbors, skip directly
        if(!graph.containsKey(src)) continue;
        List<int[]> neighbors = graph.get(src);

        for(int[] neighbor: neighbors) {
            int dst = neighbor[0];
            int cost = neighbor[1];
            if(visited.containsKey(dst))    continue;
            toStart.offer(new int[] {dst, dist+cost});
        }
    }
  ```

- 遍历完toStart后, 从点K出发所有可以到达的点都被走了一遍. 个节点到K的距离存放在visited中
- 如果visited中含有的节点个数小于给定网络的节点个数N, 则说明K不能到达所有点, 返回-1
  ```
    if(visited.size() != N) return -1;
    int ans = 0;
    for(int n: visited.keySet()) {
        int dist = visited.get(n);
        ans = Math.max(ans, dist);
    }
    
    return ans;
  ```

- 完整代码
  ```java
    public int networkDelayTime(int[][] times, int N, int K) {
        
        HashMap<Integer, List<int[]>> graph = new HashMap<>();
        for(int[] time: times) {
            int src = time[0];
            int dst = time[1];
            int cost = time[2];
            if(!graph.containsKey(src)) {
                graph.put(src, new ArrayList<int[]>());
            }
            graph.get(src).add(new int[] {dst, cost});
        }
        HashMap<Integer, Integer> visited = new HashMap<>();
        PriorityQueue<int[]> toStart = new PriorityQueue<>((a1, a2) -> {
            return a1[1] - a2[1];
        });   
        toStart.offer(new int[] {K, 0});
        while(!toStart.isEmpty()) {
            int[] node = toStart.poll();
            int src = node[0];
            int dist = node[1];
            if(visited.containsKey(src))    continue;
            visited.put(src, dist);
            if(!graph.containsKey(src)) continue;
            List<int[]> neighbors = graph.get(src);
            for(int[] neighbor: neighbors) {
                int dst = neighbor[0];
                int cost = neighbor[1];
                if(visited.containsKey(dst))    continue;
                toStart.offer(new int[] {dst, dist+cost});
            }
        }
        if(visited.size() != N) return -1;
        int ans = 0;
        for(int n: visited.keySet()) {
            int dist = visited.get(n);
            ans = Math.max(ans, dist);
        }
        return ans;
    }
  ```

- 题目出处: [Leetcode 743. Network Delay Time](https://leetcode.com/submissions/detail/255549860/)
  