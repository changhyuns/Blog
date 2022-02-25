Spanning Tree  스패닝 트리(신장 트리)
============================================
***

 ## Spanning Tree 신장 트리
* 최소한의 간선 개수로 그래프 내의 모든 정점을 포함하는 트리
  - N 개의 정점이 있는 그래프의 경우 (N-1) 개의 간선으로 트리 구성 가능
  - 하나의 그래프에 여러 가지 spanning tree 존재 가능
* 사이클이 없는 트리
  - 정확히 (N-1)개의 간선으로 N개의 정점을 모두 이어주고 있기 때문에 사이클이 존재하지 않아야 한다.
***
 ## MST (Minimum Spanning Tree) 최소 신장 트리
 * 여러 Spanning Tree 중에서, 사용된 간선들의 가중치 합이 가장 적은 Spanning Tree
  - 예) 도시를 모두 연결하면서 도시를 연결하는 도로들의 길이의 합도 최소로 하는 방법 


### MST 구현 
  - 정점을 잇는 간선들의 경우를 모두 구해서 (조합) 최소를 찾는 방법은 
      간선의 수가 많아지면 경우의 수가 너무 많아져서 비효율적이다
  - 직관적인 Greedy식으로 접근하는 것이 좋다 (Kruskal , Prim)
***
  ### Kruskal 알고리즘 (간선 기반)
  - 간선을 하나씩 선택해서 MST를 찾는 알고리즘<br>
  - 이전 단계에 만들어지는 신장 트리를 이용하는게 아니라, 항상 최소의 간선만 선택하는 방식
       ① 모든 간선을 가중치에 따라 오름차순 정렬한다. ( 최소비용의 간선부터 사용하기 위해 )<br>
       ② 가중치가 가장 낮은 간선부터 선택하면서 트리를 증가시킨다.<br>
         - 이 때, 사이클이 존재하게 된다면, 다음으로 가중치가 낮은 간선을 선택한다.<br>
       ③ n-1 개의 간선이 선택될 때 까지 ② 과정을 반복한다.<br>
```java
public class Kruskal {

	static int V, E;
	static Edge[] edgeList;
	static int[] parents; // 부모원소관리

	private static void make() {
		// 모든 원소의 대표자를 자기 자신으로 만듬
		// 연결된 정점 없이 각각 하나의 원소로만 구성된 집합을 만들어주는 부분 (시작단계)
		for (int i = 0; i < V; i++) {
			parents[i] = i;
		}
	}

	// a가 속한 집합의 대표자 찾기
	private static int find(int a) {
		if (a == parents[a])
			return a; 						  // 자신이 대표자일 경우
		return parents[a] = find(parents[a]); // 자신이 대표자가 아닐경우 path compression을 이용하여 재귀
	}

	private static boolean union(int a, int b) {
		int aRoot = find(a);
		int bRoot = find(b);
		if (aRoot == bRoot)
			return false; // a가 속한 집합의 대표자와 b가 속한 집합의 대표자가 같으면 return

		parents[bRoot] = aRoot; // b의 부모(대표자)를 a가 속한 집합의 대표자의 밑으로 붙인다
		return true;

	}

	static class Edge implements Comparable<Edge> {
		int start, end, weight;

		public Edge(int start, int end, int weight) {
			super();
			this.start = start;
			this.end = end;
			this.weight = weight;
		}

		@Override
		public int compareTo(Edge o) {
			return this.weight - o.weight;
		}

	}

	public static void main(String[] args) throws IOException {
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		StringTokenizer st = new StringTokenizer(br.readLine());
		V = Integer.parseInt(st.nextToken());
		E = Integer.parseInt(st.nextToken());

		// 간선 리스트 작성
		edgeList = new Edge[E];
		parents = new int[V];

		for (int i = 0; i < E; i++) {
			st = new StringTokenizer(br.readLine());
			int start = Integer.parseInt(st.nextToken());
			int end = Integer.parseInt(st.nextToken());
			int weight = Integer.parseInt(st.nextToken());
			edgeList[i] = new Edge(start, end, weight);
		}

		Arrays.sort(edgeList);
		parents = new int[V];
		make();
		
		// 간선 하나씩 시도하며 트리 만들어감
		int cnt = 0, result = 0;
		for(Edge edge : edgeList) {
			if(union(edge.start, edge.end)) {
				result += edge.weight;
				if(++cnt == V-1) break;   // 신장트리 완성
			}
		}
		
		System.out.println(result);
	}

}
```

  ### Prim 알고리즘 (정점 기반)
  - 하나의 정점에서 연결된 간선들 중 하나씩 선택하면서 MST를 만들어가는 알고리즘<br>
  - 이전 단계에 만들어진 신장 트리를 계속해서 확장해 나가는 방식
       ① 임의 정점을 하나 선택해서 시작한다.<br>
       ② 선택한 정점과 인접하는 정점들 중의 최소 비용의 간선이 존재하는 정점을 선택한다.<br>
         - 이 때, 사이클이 존재하게 된다면, 다음으로 가중치가 낮은 간선을 선택한다.<br>
       ③ 모든 정점이 선택될 때 까지 ①,② 과정을 반복한다.<br>
```java
public class Prim {
	public static void main(String[] args) throws NumberFormatException, IOException {
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
		StringTokenizer st;
		StringBuilder sb = new StringBuilder();
		
		int N = Integer.parseInt(br.readLine());
		int[][] adjMatrix = new int[N][N];
		boolean[] visited = new boolean[N];
		int[] minEdge = new int[N];
		
		for(int i=0; i<N; i++) {
			st = new StringTokenizer(br.readLine());
			for(int j=0; j<N; j++) {
				adjMatrix[i][j] = Integer.parseInt(st.nextToken());
			}
			minEdge[i] = Integer.MAX_VALUE;
		}
		
		int result=0; // 최소신장트리 비용
		minEdge[0]=0; // 임의의 시작점 0의 간선 비용을 0으로 셋팅
		
		for(int i=0; i<N; i++) {
			// 1. 신장트리에 포함되지 않은 정점 중 최소간선비용의 정점 찾기
			int min = Integer.MAX_VALUE;
			int minVertex = -1;
			for(int j=0; j<N; j++) {
				if(!visited[j] && (min > minEdge[j])) {  // 이미 신장트리에 있지 않은 애 중에서 (후보) 더 최소비용 간선을 가진 정점이 잇으면
					min = minEdge[j]; // 그때 간선의 최소비용값 기억
					minVertex = j;    // 그때의 정점 기억
				}
			}
			visited[minVertex] = true;   // 정점을 신장트리에 포함시킴
			result += min;               // 간선비용 누적
			
			// 2. 선택된 정점 기준으로 신장트리에 연결되지 않은 타 정점과의 간선 비용 최소로 업데이트
			for(int j=0; j<N; j++) {						//  이미 신장트리에 있는 애들은   최소의 비용을 가지고 신장트리에 포함되었기 때문에 고려대상 아님
				if(!visited[j] && adjMatrix[minVertex][j]!=0 &&(minEdge[j] > adjMatrix[minVertex][j])) {    // (다른정점에서 j정점으로 오는 ) 간선비용이 더 작은게 있다면
					minEdge[j] = adjMatrix[minVertex][j];
				}
			}	
		}
		System.out.println(result);
	}

}
```
