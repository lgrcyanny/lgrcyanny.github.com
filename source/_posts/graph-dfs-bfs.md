title: Graph DFS and BFS
tags:
  - algorithm
id: 517
categories:
  - Algorithm
date: 2013-12-02 22:26:47
---

图论是一个重要的部分，以前数据结构课上过，没有作为考试要求，所以很不太熟悉，尤其是没有实现。考完试，无聊中重新看了看，想起以前在eBay，某帅哥突然开玩笑说DFS和BFS嘛，当然要会呀，Cyanny你会的吧，我说忘了，其实细细想起来，还真没有写过代码。In my view，没有实现过，就等于不会，今天实现了BFS和DFS，深深赞一个算导，说得明白和透彻，现在的算法书太多，这算是一本很不错的经典，厚了点，但要是能厚着脸皮看完确实受益匪浅。

### 广度优先搜索BFS

其算法思想简介：
1\. 利用队列
2\. 每一个Vertex引入三个变量
color：WHITE 表示没有访问过，GREY 表示发现，BLACK 表示完成访问
parent: 父亲
d: 表示从root到该顶点的长度
3\. 为了实现方便，我采用int来标识每一个顶点，采用邻接表来表示graph
深度优先搜索可以生成深度优先树，生成单源最短路径，即从root到每一个顶点的长度是最短路径。[more...]
BFS可以采用邻接矩阵表示，其复杂度较高为O(n^2)
如果采用邻接表表示，其复杂度为O(E+V),即顶点个数+边的个数
[java]
public void BFS(int root) {
	vertexes = new Vertex[n];
	// Initialize each vertex,
	// Each vertex has three segment: parent, d, color
	for (int i = 0; i &lt; n; i++) {
		vertexes[i] = new Vertex(i);
	}

	Vertex rootv = vertexes[root];
	rootv.d = 0;
	rootv.color = Color.GREY;
	queue.add(rootv);

	while (!queue.isEmpty()) {
		Vertex u = queue.remove(0);
		ArrayList&lt;Integer&gt; list = graph.get(u.vertex);
		for (int i = 0; i &lt; list.size(); i++) {
			Vertex temp = vertexes[list.get(i)];
			// Make sure each vertex enqueue only once
			if (temp.color == Color.WHITE) {
				temp.parent = u;
				temp.d = u.d + 1;
				temp.color = Color.GREY;
				queue.add(temp);
			}				
		}
	        u.color = Color.BLACK;
		res.add(u);
	}
}
[/java]
[BFS Source Code](https://github.com/lgrcyanny/Algorithm/blob/master/src/com/algorithm/graph/GraphBFS.java "BFS Source")

### 深度优先搜索DFS

广度优先搜索DFS，基本算法思想
1\. 采用递归，或stack来实现
2\. 每一个Vertex引入四个变量
color：WHITE 表示没有访问过，GREY 表示发现，BLACK 表示完成访问
parent: 父亲
timestamp1: 表示第一次访问该顶点的时间戳
timestamp2:表示访问结束时的时间戳 
3\. 为了实现方便，我采用int来标识每一个顶点，采用邻接表来表示graph

DFS采用邻接表表示Graph来实现，复杂度O(V+E), 矩阵复杂度依然不好，为O（n^2）
DFS是从多个源开始，最终会生成由多个源作为root的森林，即深度优先树（深度优先森林）。
同时引入时间戳，是为了获得DFS的进展情况，如果不需要时间戳，只是想简单地打印每一个顶点，可以不用。

以下是递归的实现方法：
[java]
    public void DFS() {
		int i = 0;
		for (i = 0; i &lt; n; i++) {
			vertexes[i] = new Vertex(i);
		}
		for (i = 0; i &lt; n; i++) {
			Vertex u = vertexes[i];
			if (u.color == Color.WHITE) {
				visitDFS(u);				
			}
		}
	}

	private void visitDFS(Vertex u) {
		res.add(u);
		u.color = Color.GREY;
		timer++;
		u.timestamp1 = timer;
		ArrayList&lt;Integer&gt; adjList = graph.get(u.vertex);
		for (int i = 0; i &lt; adjList.size(); i++) {
			Vertex v = vertexes[adjList.get(i)];
			if (v.color == Color.WHITE) {
				v.parent = u;
				visitDFS(v);
			}
		}
		u.color = Color.BLACK;
		timer++;
		u.timestamp2 = timer;
	}
[/java]

以下是采用栈的实现方法改写 “visitDFS”：
[java]
    /**
	 * DFS with iterative method 
	 * Make use of stack
	 * @param u
	 */
	private void visitDFSIterative(Vertex u) {
		res.add(u);
		u.color = Color.GREY;
		timer++;
		u.timestamp1 = timer;
		stack.add(u);
		while(!stack.isEmpty()) {
			Vertex v = stack.remove(stack.size() - 1);
			ArrayList&lt;Integer&gt; adjList = graph.get(v.vertex);
			for (int i = 0; i &lt; adjList.size(); i++) {
				Vertex temp = vertexes[adjList.get(i)];
				if (temp.color == Color.WHITE) {
					temp.parent = v;
					temp.color = Color.GREY;
					timer++;
					temp.timestamp1 = timer;
					res.add(temp);
					stack.add(temp);
				}
			}
			v.color = Color.BLACK;
			timer++;
			v.timestamp2 = timer;
		}
		u.color = Color.BLACK;
		timer++;
		u.timestamp2 = timer;
	}
[/java]

[DFS Source Code](https://github.com/lgrcyanny/Algorithm/blob/master/src/com/algorithm/graph/GraphDFS.java "DFS Source")

最终发现，起始实现起来代码不多，重在想法。