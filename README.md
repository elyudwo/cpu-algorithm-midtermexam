# cpu-algorithm-midtermexam  
  
  
## 1. 프로젝트 개요

##### 그래프에서 두 정점 사이에 얼마나 많은 유량(flow)을 보낼 수 있는지 계산하는 알고리즘을 네트워크 유량(Network Flow)혹은 최대유량 (Maximum Flow) 알고리즘이라고 합니다. 방향 그래프를 사용해 각 간선의 용량이 정해져 있을 때, 정해진 출발점 (위에서 말한 두 정점중 첫번째) 에서 도착점 (두 정점중 두번째) 까지 보낼 수 있는 최대 유량을 계산할 수 있어야합니다. 

##### 이러한 네트워크 흐름은 도로망의 교통 흐름을 분석하거나 전자 회로의 전류, 파이프를 흐르는 유체등 네트워크를 통해 묘사될 수 있는 다양한 대상들의 특성을 연구하는데 사용되므로 삶에 밀접해있는 알고리즘 입니다.
#
#
#
  
## 2. 분석

### 2-1. 사전 지식

##### 용량(Capacity) : c(u,v)는 정점 u에서 v로 가는 간선의 용량(가중치) 입니다.

##### 유량(Flow) : f(u,v)는 정점 u에서 v로의 간선에 실제로 흐르는 유량을 의미합니다.

##### 잔여 용량(Residual Capacity) : 간선의 용량과 유량의 차이를 의미합니다. r(u, v) = c(u, v)-f(u,v) 즉, 해당 간선에 추가적으로 더 흘려 보낼 수 있는 유량입니다.

##### 소스(source) 유량이 시작되는 정점으로 모든 유량은 소스에서 부터 싱크로 흐르게 됩니다.

##### 싱크(sink) 모든 유량이 도착하는 정점이 싱크입니다. 네트워크 유량 알고리즘은 소스에서 싱크로 흐를 수 있는 최대 유량을 계산합니다.

##### 용량 제한 속성 flow(u,v) <= capacity(u,v) : 간선의 유량은 간선의 용량을 초과할 수 없습니다.

##### 유량의 대칭성  flow(u,v) = -flow(v,u) : u가 v에게 보낸 유량은 v가 u에게 음의 유량을 보낸 것과같습니다.

##### 유량의 보존 ∑flow(u,v) = 0 : 시작점과 도착점을 제외한 정점에서 보낸 유량과 나간 유량은 같습니다.             

#
#


### 2-2. 시작 전 어려웠던 부분

##### 알고리즘을 이해하며 가장 어려웠던 부분은 유량의 대칭이었습니다.

![다운로드 (1)](https://user-images.githubusercontent.com/97587573/164615378-4cedcad1-b6c9-413e-9913-2c87b42b5896.png)

##### 유량의 대칭성이란 위의 그림에서 2->3 으로 1이 흐를때 3->2 로는 -1이 흐른다고 생각하는 것입니다

![다운로드 (2)](https://user-images.githubusercontent.com/97587573/164615416-51cd435d-33f2-47c0-92cb-552d9876c087.png)
##### 위에서 설명한 잔여 용량 공식을 보면 r(u,v) = c(u,v) - f(u,v)입니다. 이때 r(3,2) = c(3,2)-f(3,2) 입니다. c(3,2)는 0 이고 f(3,2)는 -1이므로 r(3,2) = 0 - (-1) = 1이 됩니다

#### 왜 3->2는 연결이 안되어있는지 현실세계의 파이프를 생각해서 이해가 안됐습니다. 
#### 사고의 범위를 넓혀서 우리가 단순하게 유량을 더해주는 과정에서 사실은 **보이지 않게 반대로 가는 유량이 있다고 가정** 해야합니다. 

#
#

### 2-3. 알고리즘 분석
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
##### ford fulkerson 알고리즘을 표현하기 위해 2차원 배열과 c++의 vector container를 사용했습니다 2차원 배열의 경우 간선의 용량을 표현하기 위해 선언해줬고 vector container는 각 간선을 연결해주기 위해 선언했습니다.
##### 간선의 개수를 정해주는 n, 유량이 시작되는 정점과 도착되는 정점인 source와 sink를 선언하고 for문을 이용해 간선연결, 간선정보를 입력해주고 maxFlow 함수에 source와 sink를 입력해서 최대유량을 계산하는 과정입니다.
##### 다음으로는 maxFlow 함수에 대해 살펴보겠습니다.

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
##### while 반복문을 시작하면 fill 함수가 나오는데 이때 위에서 선언해준 int형 배열 d를 -1로 초기화 해줍니다.
##### 여기서 d 배열을 선언해준 이유가 나오는데 d 배열은 간선으로 유량이 이동하지 않았을때 -1 인 상태를 유지해주기 위함입니다. true와 false로 구성된 bool 함수라 생각하면 이해하기가 편합니다.

##### 여기서 bool 함수와의 차이점이 있는데 true 와 false 로만 구성된 bool 함수와는 다르게 d 배열에 연결된 서로 다른 간선 정보를 저장해주는점이 다릅니다. 이때 연결된 간선 정보를 이용해 아래와같은 거꾸로 최소유량을 탐색해주는 반복문을 돌릴수 있습니다
```
int flow = INF;
for(int i = end; i != start; i = d[i]) {
	flow = min(flow, c[d[i]][i] - f[d[i]][i]);
}
```
#### MaxFlow 함수의 진행과정을 살펴보겠습니다.
##### while 반복문 실행을 하면 BFS를 통해 source -> sink로 가는 최단경로를 찾게됩니다. 
![20220422224548](https://user-images.githubusercontent.com/97587573/164726988-af20c079-38ba-40de-83bb-769056db99a2.png)

##### (그림판으로 그려서 그림이 고르지못한점 죄송합니다)
##### 간선연결을 작은수부터 차례로 해준다 가정했을때 위의 그림과 같은 상황에서 BFS 코드를 실행하면 1 -> 2 -> 3 -> 4 로의 첫번째 source -> sink 식이 완성됩니다.

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
##### 그 후 위의 코드를 따라가보면 1 -> 2 -> 3 -> 4 으로 가는 간선중 가장작은 용량을 가진 간선을 찾고 해당 간선들의 유량을 가장작은 용량으로 더해줍니다.
```
	for(int i = end; i != start; i = d[i]) {
		f[d[i]][i] += flow;
		f[i][d[i]] -= flow;
	}
```
##### 2-2 에서 말했던 보이지 않는 유량을 선언하기 위해 f[i][d[i]] -= flow 를 해주었습니다.
##### 1 -> 2 -> 3 -> 4 를 탐색했으니 BFS를 이용해 다음 source -> sink로의 간선을 탐색해보면 1 -> 2 로 가려다
```
if(c[x][y] - f[x][y] > 0 && d[y] == -1) {
			q.push(y);
			d[y] = x;
			if(y == end) break;
			}
```

##### 위의 식에서 걸리게 되며 1 -> 3 -> 2 -> 4 로의 경로를 찾게되며 이후 반복문을 계속 실행하면 
```
if(c[x][y] - f[x][y] > 0 && d[y] == -1)
```
##### 위의 코드에서 계속 걸리므로 result 값이 2가되며 최대유량을 찾아냅니다.

#
#
### 2-4. 오류 상황

**가정. 위의 코드에서 f[i][d[i]] -= flow 부분이 없다.**


##### 먼저 1 -> 2 -> 3 -> 4 가 실행되면 result = 1 이되며 f[1][2] = 1 ,f[2][3]= 1
##### f[3][4] = 1 이 저장됩니다. 그 후 반복문을 실행하면 1 -> 2가 if문에 걸리게되고 1->3->4 또한 if문에 걸리게 되며 최종 result 값이 1이 되어서 틀린 코드가 됩니다.
#
#

### 2-5. 오류 해결

##### 만약 가정을 없앤 상태로 코드를 실행하면 어떻게 될까?
##### 1 -> 2 -> 3 -> 4 가 실행되면 f[1][2] = 1, f[2][1] = -1, f[2][3] = 1, f[3][2] = -1 이 저장됩니다 
##### 그 후 반복문을 실행하면 1 -> 2 로는 f[1][2]에 의해서 갈수없고 1 -> 3 으로 가게된다. 이 때 3 -> 4 또한 f[3][4]에 의해 갈수없다.

#### 이 때 f[i][d[i]] -= flow 에 의해서 f[2][1] = -1 이므로 
```
if(c[x][y] - f[x][y] > 0 && d[y] == -1) 
```
##### 위의 if문에 걸리지 않고 실행되며 1->3->2->4 로의 경로가 완성된다.
##### 따라서 result 값은 2가 되므로 알고리즘이 정상작동 하게된다.

#
#
#
## 3. 시간 복잡도

##### ford-fulkerson 알고리즘의 시간복잡도에 대해서 알아보겠습니다. 
##### 먼저 ford-fulkerson 알고리즘의 시간 복잡도는 반복문 관점에서 계산하지 않습니다.
##### 최대 유량이 f 라면 각 증가 경로 당 유량이 최소 1 이기 때문에 증가 경로의 최대 개수는 f 가 됩니다.
##### 하나의 증가 경로를 찾는데 방문하는 정점과 간선은 최대로 계산할 시, 각각 V, E 개 입니다.
##### 따라서 최대 유량을 얻어내는데 필요한 시간 복잡도는 O((|E|+|V|)f) 입니다.
##### 정점의 개수가 무시할 만큼 작다면 O(|E|f) 가 되겠습니다.

##### 위 코드에서는 BFS를 이용했기 때문에 f를 탐색하는데 |E||V| 가 소요되어 시간 복잡도는 O(|V||E|^2)가 됩니다. 

![화면 캡처 2022-04-25 140925](https://user-images.githubusercontent.com/97587573/165024639-53ca5284-4df1-4507-8908-16a4c1be67b6.png)

y = v 그래프입니다.

![화면 캡처 2022-04-25 141020](https://user-images.githubusercontent.com/97587573/165024666-3c9c19e2-bee0-4ce0-9d24-f6b6055c3998.png)

y = e^2 그래프입니다

##### 따라서 시간복잡도는 위의 그래프의 v와 e의 값에따른 y값을 곱해준것입니다.

#
#
#
## 4. 실행결과 

##### 위의 그림판 그림대로 출력했을때의 화면은 다음과 같습니다.
![실행화면](https://user-images.githubusercontent.com/97587573/164970828-42082673-7ef5-4525-99ef-895058aa442c.png)

##### 차례대로 간선의 개수, source와 sink 간선정보와 용량 마지막줄에 결과가 출력 됩니다.

#### 아래 그림과 같이 간선의 개수를 늘려서 실행해보겠습니다

![asdasdsa](https://user-images.githubusercontent.com/97587573/164975675-f486fac9-962a-4e0f-9ffe-ec2f2830fa47.png)


##### 실행화면은 다음과 같습니다

![실행화면 3](https://user-images.githubusercontent.com/97587573/164971775-083d70a3-daa8-4edc-8f2c-c025623a1381.png)

##### 1 -> 2 -> 3 -> 6 으로 6 만큼
##### 1 -> 2 -> 6 으로 6 만큼
##### 1 -> 4 -> 5 -> 3 -> 6 으로 2 만큼
##### 1 -> 4 -> 5 -> 6 으로 4 만큼 흐르게 되어서 18의 유량이 흐르게 됩니다
##### 이 때 유량의 대칭성을 이용하면 유량을 더 보낼수 있게됩니다.
##### 2 -> 3 으로 6만큼의 유량이 흐르고 있으므로 3 -> 2 로는 -6만큼의 유량이 흐릅니다. 
#### 이를 이용하면 1 -> 4 -> 5 -> 3 -> 2 -> 6 으로 1 만큼의 유량을 흘려보낼 수 있습니다. 
##### 그렇기 때문에 최대 유량은 19 입니다.


#
#
#
## 5. 추가 지식

#### 증가 경로 탐색을 할 때 DFS를 사용하면 시간복잡도에서 효율성 문제가 발생할 수 있습니다.
##### 왜 그런지 아래 그림을 통해 설명드리겠습니다.

![다운로드 (3)](https://user-images.githubusercontent.com/97587573/165021706-e35e482b-2a35-446d-abc6-6932c3237656.png)

##### BFS를 통해 그림과 같은 상황일때 최대유량을 구해보겠습니다
#### 1. A -> B -> D 로 1000의 유량을 보냄
#### 2. A -> C -> D 로 1000의 유량을 보냄
##### 총 2000의 유량을 두번의 탐색으로 보내게됩니다.

##### 이번엔 DFS를 통해 그림과 같은 상황일때 최대유량을 구해보겠습니다
#### 1. A -> B -> C -> D 로 1의 유량을 보냅니다.
#### 2. 유량의 대칭성을 이용해 A -> C -> B -> D 로 1의 유량을 보냅니다.
#### 이 과정을 2000번 반복해주면 2000의 유량을 보내게 됩니다.
##### DFS를 이용하면 2000의 유량을 2000번의 탐색으로 보내게됩니다.

#### 3에서 구한 DFS 시간복잡도 공식을 살펴보면  O((|E|+|V|)f) 입니다.
##### 이때 위의 그림과 같은 경우 간선의 개수는 5개 정점은 4개 유량은 2000 이므로 계산결과는 18000 입니다.
##### 3에서 구한 BFS 시간복잡도 공식은 O(|V||E|^2) 입니다.
##### 공식을 통해 계산해보면 4 * 5 * 5 = 100 입니다.

### 그러므로 위와같은 최악의 경우 시간복잡도를 고려해 보았을때 DFS에 비해 BFS가 더 효율적입니다.

#### 반대로 생각해보면 유량(f)이 적고 정점(V)과 간선(E)의 개수가 많다면 BFS에 비해 DFS가 시간복잡도 면에서 더욱 효율적입니다.
#
#
#

## 6. 정리

#### 언뜻 보았을때 간단한 알고리즘이라 생각했지만 상황에 따라 BFS와 DFS를 다르게 선택해 시간복잡도를 줄일수 있다는 점이 흥미로웠습니다. 또한 유량의 대칭을 이용해 음의 유량을 생성해준다는 아이디어는 눈에 보이는것만 생각하는 꽉막힌 사고에 대해서 변화의 필요성을 느끼게 해주었습니다. 
#### 이번 보고서를 통해 BFS와 DFS의 활용에 대해 다시한번 일깨우는 계기가 되었고 앞으로 다른 알고리즘 문제를 풀때도 좀 더 신선한 아이디어를 사고할수있는 밑거름이 되었습니다.
