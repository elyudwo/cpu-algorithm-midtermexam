## cpu-algorithm-midtermexam

### 1. 프로젝트 개요

###### 그래프에서 두 정점 사이에 얼마나 많은 유량(flow)을 보낼 수 있는지 계산하는 알고리즘을 네트워크 유량(Network Flow)혹은 최대유량 (Maximum Flow) 알고리즘이라고 합니다. 방향 그래프를 사용해 각 간선의 용량이 정해져 있을 때, 정해진 출발점 (위에서 말한 두 정점중 첫번째) 에서 도착점 (두 정점중 두번째) 까지 보낼 수 있는 최대 유량을 계산할 수 있어야합니다. 

###### 이러한 네트워크 흐름은 도로망의 교통 흐름을 분석하거나 전자 회로의 전류, 파이프를 흐르는 유체등 네트워크를 통해 묘사될 수 있는 다양한 대상들의 특성을 연구하는데 사용되므로 삶에 밀접해있는 알고리즘 입니다.


### 2. 분석

#### 2-1. 사전 지식

###### 용량(Capacity) : c(u,v)는 정점 u에서 v로 가는 간선의 용량(가중치) 입니다.

###### 유량(Flow) : f(u,v)는 정점 u에서 v로의 간선에 실제로 흐르는 유량을 의미합니다.

###### 잔여 용량(Residual Capacity) : 간선의 용량과 유량의 차이를 의미합니다. r(u, v) = c(u, v)-f(u,v) 즉, 해당 간선에 추가적으로 더 흘려 보낼 수 있는 유량입니다.

###### 소스(source) 유량이 시작되는 정점으로 모든 유량은 소스에서 부터 싱크로 흐르게 됩니다.

###### 싱크(sink) 모든 유량이 도착하는 정점이 싱크입니다. 네트워크 유량 알고리즘은 소스에서 싱크로 흐를 수 있는 최대 유량을 계산합니다.

###### 용량 제한 속성 flow(u,v) <= capacity(u,v) : 간선의 유량은 간선의 용량을 초과할 수 없습니다.

###### 유량의 대칭성  flow(u,v) = -flow(v,u) : u가 v에게 보낸 유량은 v가 u에게 음의 유량을 보낸 것과같습니다.

###### 유량의 보존 ∑flow(u,v) = 0 : 시작점과 도착점을 제외한 정점에서 보낸 유량과 나간 유량은 같습니다.         


#### 2-2. 시작 전 어려웠던 부분

###### 알고리즘을 이해하며 가장 어려웠던 부분은 유량의 대칭이었습니다.

![다운로드 (1)](https://user-images.githubusercontent.com/97587573/164615378-4cedcad1-b6c9-413e-9913-2c87b42b5896.png)

###### 유량의 대칭성이란 위의 그림에서 2->3 으로 1이 흐를때 3->2 로는 -1이 흐른다고 생각하는 것입니다

![다운로드 (2)](https://user-images.githubusercontent.com/97587573/164615416-51cd435d-33f2-47c0-92cb-552d9876c087.png)
###### 위에서 설명한 잔여 용량 공식을 보면 r(u,v) = c(u,v) - f(u,v)입니다. 이때 r(3,2) = c(3,2)-f(3,2) 입니다. c(3,2)는 0 이고 f(3,2)는 -1이므로 r(3,2) = 0 - (-1) = 1이 됩니다

###### 왜 3->2는 연결이 안되어있는지 현실세계의 파이프를 생각해서 이해가 안됐습니다. 사고의 범위를 넓혀서 우리가 단순하게 유량을 더해주는 과정에서 사실은 **보이지 않게 반대로 가는 유량이 있다고 가정** 해야합니다. 


#### 2-3. 알고리즘 분석
```
#define MAX 100
#define INF 100000000

int n = 6, result;
int c[MAX][MAX], f[MAX][MAX], d[MAX];
vector<int> a[MAX];

int main(void) {
	
	int n;
	cin >> n; 
	
	for(int i=0; i<n; i++) {
		int m1,m2,m3;
		cin >> m1 >> m2 >> m3;
		a[m1].push_back(m2);
		a[m2].push_back(m1);
		c[m1][m2] = m3;
	} 
	maxFlow(1,6);
	cout << result;
}
```
###### ford fulkerson 알고리즘을 표현하기 위해 2차원 배열과 c++의 vector container를 사용했습니다 2차원 배열의 경우 간선의 용량을 표현하기 위해 선언해줬고 vector container는 각 간선을 연결해주기 위해 선언했습니다.
###### 간선의 개수를 정해주는 n, 유량이 시작되는 정점과 도착되는 정점인 source와 sink를 선언하고 for문을 이용해 간선연결, 간선정보를 입력해주고 maxFlow 함수에 source와 sink를 입력해서 최대유량을 계산하는 과정입니다.
###### 다음으로는 maxFlow 함수에 대해 살펴보겠습니다.

```
void maxFlow(int start, int end) {
	while(1) {
		fill(d, d + MAX, -1);
		queue<int> q;
		q.push(start);
		while(!q.empty()) {
			int x = q.front();
			q.pop();
			for(int i=0; i<a[x].size(); i++) {
				int y = a[x][i];
				
				if(c[x][y] - f[x][y] > 0 && d[y] == -1) {
					q.push(y);
					d[y] = x;
					if(y == end) break;
				}
			}
		}
		if(d[end] == -1) break;
		int flow = INF;
		for(int i = end; i != start; i = d[i]) {
			flow = min(flow, c[d[i]][i] - f[d[i]][i]);
		}
		
		for(int i = end; i != start; i = d[i]) {
			f[d[i]][i] += flow;
			f[i][d[i]] -= flow;
		}
		result += flow;
	}	
}
```
###### while 반복문을 시작하면 fill 함수가 나오는데 이때 위에서 선언해준 int형 배열 d를 -1로 초기화 해줍니다.
###### 여기서 d 배열을 선언해준 이유가 나오는데 d 배열은 간선으로 유량이 이동하지 않았을때 -1 인 상태를 유지해주기 위함입니다. true와 false로 구성된 bool 함수라 생각하면 이해하기가 편하다.

###### 여기서 bool 함수와의 차이점이 있는데 true 와 false 로만 구성된 bool 함수와는 다르게 d 배열에 연결된 서로다른 간선 정보를 저장해주는점이 다릅니다. 이때 연결된 간선 정보를 이용해 아래와같은 거꾸로 최소유량을 탐색해주는 반복문을 돌릴수있습니다
```
int flow = INF;
for(int i = end; i != start; i = d[i]) {
	flow = min(flow, c[d[i]][i] - f[d[i]][i]);
}
```
##### MaxFlow 함수의 진행과정을 살펴보겠습니다.
###### while 반복문 실행을 하면 BFS를 통해 source -> sink로 가는 최단경로를 찾게됩니다. 
![제목 없음](https://user-images.githubusercontent.com/97587573/164718601-e00c0120-3fad-476b-a9e2-213e8c3dd4f6.png)
###### (그림판으로 그려서 그림이 고르지못한점 죄송합니다)
###### 간선연결을 작은수부터 차례로 해준다 가정했을때 위의 그림과 같은 상황에서 BFS 코드를 실행하면 1 -> 2 -> 4 로의 첫번째 source -> sink 식이 완성된다.

```
	int flow = INF;
	for(int i = end; i != start; i = d[i]) {
		flow = min(flow, c[d[i]][i] - f[d[i]][i]);
	}
	
	for(int i = end; i != start; i = d[i]) {
		f[d[i]][i] += flow;
		f[i][d[i]] -= flow;
	}
	result += flow;
```
###### 그 후 위의 코드를 따라가보면 1 -> 2 -> 3 으로 가는 간선중 가장작은 용량을 가진 간선을 찾고 해당 간선들의 유량을 가장작은 용량으로 더해준다
```
	for(int i = end; i != start; i = d[i]) {
		f[d[i]][i] += flow;
		f[i][d[i]] -= flow;
	}
```
###### 2-2 에서 말했던 보이지 않는 유량을 선언하기 위해 f[i][d[i]] -= flow 를 해주었다.
