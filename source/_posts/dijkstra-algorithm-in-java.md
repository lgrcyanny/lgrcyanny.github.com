title: Dijkstra Algorithm
tags:
  - algorithm
id: 676
categories:
  - Algorithm
date: 2014-04-12 11:55:04
---

计算单源最短路径的算法有Bellman-Ford算法，主要是计算带负权有向图的单源最短路径。第二种算法：对于DAG图（Directed Acyclic Graph, 有向无环图）可以通过拓扑排序后再计算单源最短路径。第三，可以使用BFS(广度优先搜索)计算单源最短路径。第四，就是著名的Dijkstra算法，要求有向图的所有边的权值非负，当然图可以不是DAG图。

<!--more-->
Dijkstra的实现思想主要是采用最小优先级队列，队列排序的标准是source节点到当前节点的距离，每次从堆顶取出一个节点，遍历该节点的邻接表，进行松弛(Relax)操作，松弛操作的伪代码如下：
[java]
relax(u, v) {
  if (v.distance &gt; u.distance + weight[u, v]) {
    v.distance = u.distance + weight[u, v];
    v.parent = u;
  }
}
[/java]

Dijkstra算法的伪代码如下：
[java]
dijkstraShortestPath(graph, s) {
  for each v in graph {
    v.parent = null;
    v.distance = MaxValue;
  }
  MinPriorityQueue queue;
  s.distance = 0;
  queue.add(s);
  while (queue != empty) {
    u = queue.extractMin();
    for each v in adjacentlist of u {
      if (v.distance &gt; u.distance + weight[u, v]) {
        v.distance = u.distance + weight[u, v];
        v.parent = u;
        queue.add(v);
      }
    }  
  }
}
[/java]

Dijkstra算法实现时的一个难点是如何实现最小优先级队列以及图如何表示，我在实现中采用Java提供的PriorityQueue，同时Graph的表示采用邻接表。而邻接表最初的想法是一个ArrayList<ArrayList<Edge>>, 每个Edge对象 = Vertex + weight; 但这个实现的限制是，在ArrayList中每个list的index就十分重要，当从queue中取得一个节点，需要按index找到他的邻接list。在[GitHub上](https://github.com/mburst/dijkstras-algorithm/blob/master/Dijkstras.java)看到一个实现，是用LinkedHashMap表示邻接表，但凭什么给每个节点按string的key进行标识呢？不是很认同。
最后采取的办法是，将每个节点的邻接list放入每个节点对象中，主要是受[这个博客](http://www.algolist.com/code/java/Dijkstra%27s_algorithm)的启发。
最后实现代码如下，也不是很长。

### 1\. 定义Vertex

[java]
public class Vertex implements Comparable&lt;Vertex&gt;{
	public String id = null;
	public Vertex parent = null;
	public Edge[] adjacencyList = null;
	public int distance = Integer.MAX_VALUE; // The min distance from source to the current vertex

	public Vertex() {

	}

	public Vertex(String id) {
		this.id = id;
	}

	/**
	 * Override this method for min priority queue
	 */
	@Override
	public int compareTo(Vertex o) {
		return this.distance &lt; o.distance ? -1 : (this.distance == o.distance ? 0 : 1);
	}

	@Override
	public String toString() {
		return &quot;Vertex [id=&quot; + id + &quot;, distance=&quot; + distance + &quot;]&quot;;
	}
}
[/java]

### 2\. 定义Edge对象

[java]
public class Edge {
	public Vertex vertex = null;
	public int weight = 0;

	public Edge() {

	}

	public Edge(Vertex vertex, int weight) {
		this.vertex = vertex;
		this.weight = weight;
	}

}
[/java]

### 3\. Dijkstra算法实现

测试的图如下：

[![Image and video hosting by TinyPic](http://i59.tinypic.com/9ggih1.jpg)](http://tinypic.com?ref=9ggih1.jpg)
[java]
public class DijkstraGraph {
	public Vertex[] graph = null;

	public DijkstraGraph(Vertex[] graph) {
		this.graph = graph;
	}

	public void computShortestPath(Vertex source) {
		PriorityQueue&lt;Vertex&gt; queue = new PriorityQueue&lt;Vertex&gt;();
		source.distance = 0;
		queue.add(source);
		while (!queue.isEmpty()) {
			Vertex u = queue.remove();
			for (int i = 0; i &lt; u.adjacencyList.length; i++) {
				Edge v = u.adjacencyList[i]; // The neighbor of vertex u
				if (v.vertex.distance &gt; u.distance + v.weight) {
					v.vertex.distance = u.distance + v.weight;
					v.vertex.parent = u;
					queue.add(v.vertex);
				}
			}
		}
	}

	public void printShortestPath() {
		for (int i = 0; i &lt; graph.length; i++) {
			Vertex target = graph[i];
			Stack&lt;Vertex&gt; path = new Stack&lt;Vertex&gt;();
			while (target != null) {
				path.push(target);
				target = target.parent;
			}
			printPath(path);
		}
	}

	private void printPath(Stack&lt;Vertex&gt; path) {
		System.out.println(&quot;=============The path is=========&quot;);
		while (!path.isEmpty()) {
			System.out.println(path.pop());
		}
	}

	public static void main(String[] args) {
		Vertex v0 = new Vertex(&quot;s&quot;);
		Vertex v1 = new Vertex(&quot;t&quot;);
		Vertex v2 = new Vertex(&quot;x&quot;);
		Vertex v3 = new Vertex(&quot;z&quot;);
		Vertex v4 = new Vertex(&quot;y&quot;);

		v0.adjacencyList = new Edge[] { new Edge(v1, 10), new Edge(v4, 5) };
		v1.adjacencyList = new Edge[] { new Edge(v2, 1), new Edge(v4, 2) };
		v2.adjacencyList = new Edge[] { new Edge(v3, 4) };
		v3.adjacencyList = new Edge[] { new Edge(v0, 7), new Edge(v2, 6) };
		v4.adjacencyList = new Edge[] { new Edge(v1, 3), new Edge(v2, 9),
				new Edge(v3, 2) };

		Vertex[] graph = { v0, v1, v2, v3 };

		DijkstraGraph dijkstraGraph = new DijkstraGraph(graph);
		dijkstraGraph.computShortestPath(v0);
		dijkstraGraph.printShortestPath(); // Each path from v0

	}

}
[/java]

源码在[Cyanny GitHub](https://github.com/lgrcyanny/Algorithm/tree/master/src/com/algorithm/graph/dijkstra)