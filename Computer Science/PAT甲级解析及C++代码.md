# PAT甲级解析及C++代码

## 第一套

### 1001 Format

给出两个整数，计算A+B，输出格式化答案（3位一逗号），无难度，注意结果等于0和前置逗号的情况

```
input : -1000000 9
output: -999,991
```



```c++
ll a,b,c;
string go(ll num){
    if(num == 0)
        return "0";

    string ret;
    ll cp = abs(num), cnt = 0;
    
    while(cp > 0){
        ret.append(1, cp%10 + '0');
        cp /= 10;
        cnt++;
        if(cnt == 3){
            ret.append(1, ',');
            cnt = 0;
        }
    }
    if(ret[ret.length()-1] == ',')
        ret.pop_back();
    if(num < 0)
        ret.append(1, '-');
    reverse(ret.begin(), ret.end());
    return ret;
}
int main(){
    cin>>a>>b;
    c = a+b;
    cout<<go(c)<<endl;
}
```

###  1002 A+B for Polynomials

题意：给出两个多项式，返回多项式结果

```
input:
2 1 2.4 0 3.2
2 2 1.5 1 0.5
output:
3 2 1.5 1 2.9 0 3.2
```

分析：so easy，注意一下幂的范围就好

```c++
double coeff[MAXN];
void go(){
    int K = 0, exp=0;
    double coe=0.0;
    cin>>K;
    for(int i=0;i<K;i++){
        cin>>exp>>coe;
        coeff[exp] +=  coe;
    }
}
int main(){
    memset(coeff, 0.0, sizeof(coeff));
    go();go();
    
    int cnt = 0;
    for(int i=0;i<=MAXN;i++)
        if(coeff[i] != 0.0)
            cnt++;
    printf("%d", cnt);
    for(int i=MAXN;i>=0;i--)
        if(coeff[i] != 0.0)
            printf(" %d %.1f", i, coeff[i]);
    printf("\n");
}
```





### 1003 Emergency

给定城市数量$N (N\le 500)$，道路数目$M$，每个城市的救援人数$P_i$，起点和终点$C_1,C_2$

求$C_1$到$C_2$间的最短路条数，和路径上人数总和的最大值，保证存在至少一条路。

思路：就是考最短路，直接上Dijkstra就行，可以堆优化，也可以不进行堆优化，但要注意最短路的最优化需要小顶堆，因此申明的时候去要`greater<pair<ll,ll>>`



花了不少时间，几个WA点：

* 最短的条数计算，加法原理还是乘法原理？
* 可以一次dijkstra把答案搞出来吗？需要额外dfs吗？
* 使用`pair<ll, ll>`的时候`.first`和`.second`造成代码冗长，很难debug
* `memset(a, INF, sizeof(a))`似乎不奏效，需要`for`循环初始化
* 带堆优化的dijkstra代码中，如何去重复点？

```
input:
5 6 0 2      // 5个城市，6条路，起点0，终点2
1 2 1 5 3    // 城市0-4分别有1，2，1，5，3人
0 1 1 	     // 城市0-1有一条长度为1的路
0 2 2
0 3 1
1 2 1
2 4 1
3 4 1

output:
2 4          // 有2条最短路，路径上人数最多为4
```



```C++
ll N,M,C1,C2,P[MAXN],dis[MAXN],Peo[MAXN]={0},cnt=1,path[MAXN]={0};
vector<pair<ll,ll> > city[MAXN];
vector<int> pre[MAXN];
bool vis[MAXN] = {false};

int main(){
    cin>>N>>M>>C1>>C2;
    for(int i=0;i<N;i++) cin>>P[i];
    for(int i=0;i<M;i++){
        int x, y, len=0;
        cin>>x>>y>>len;
        city[x].push_back({y, len});
        city[y].push_back({x, len});
    }
    
    for(int i=0;i<MAXN;i++) dis[i] = 0x3f3f3f3f;
    memset(Peo, 0, sizeof(Peo));
    
    Peo[C1] = P[C1];
    dis[C1] = 0;
    path[C1] = 1;
    
    priority_queue<pair<ll,ll>, vector<pair<ll,ll>>, greater<pair<ll,ll>>> pq;
    pq.push({dis[C1], C1});
    
    while(!pq.empty()){
        pair<ll, ll> head = pq.top();
        pq.pop();
        if(dis[head.second] != head.first || vis[head.second]) continue;
        vis[head.second] = true;

        for(auto next: city[head.second]){
            if(head.first + next.second < dis[next.first]){
                dis[next.first] = head.first + next.second;
                Peo[next.first] =  Peo[head.second] + P[next.first];
                path[next.first] = path[head.second];
            }
            else if(head.first + next.second == dis[next.first]){
                Peo[next.first] = max(Peo[next.first], Peo[head.second] + P[next.first]);
                path[next.first] += path[head.second];
            }
            if(vis[next.first] == false)
                pq.push({dis[next.first], next.first});
        }
    }
    cout<<path[C2]<<" "<<Peo[C2]<<endl;
}
```



### 1004 Counting Leaves

题意：给定一棵树，找出叶子节点个数，$N<100$。

思路：考点是`bfs`，因为需要层序输出答案，很简单。

```
input:
2 1        // 2个节点，1个非叶子节点
01 1 02    //接下来1行，非叶子节点是01，有1个儿子，编号02

output:
0 1        //树一共2层，第1层0个叶子节点，第2层1个叶子节点
```



```c++
int N, M, K, child, fa;
vector<int> son[MAXN], leaf;
void bfs(vector<int>& rt){
    if(rt.empty()) return ;

    vector<int> next;
    int cnt = 0;
    for(auto root: rt){
        if(son[root].size() == 0) cnt++;
        else
            for(auto s: son[root])
                next.push_back(s);
    }
    leaf.push_back(cnt);
    bfs(next);
}
int main(){
    cin>>N>>M;
    for(int i=0;i<M;i++){
        cin>>fa>>K;
        for(int j=0;j<K;j++){
            cin>>child;
            son[fa].push_back(child);
        }
    }

    vector<int> rt; rt.push_back(1);
    bfs(rt);
    for(int i=0;i<leaf.size();i++){
        if(i==0) cout<<leaf[i];
        else cout<<" "<<leaf[i];
    }
    cout<<endl;
}
```



## 第二套

### 1005 Spell It Right

给一个非负整数$N， N<10^{100}$，求出每一位数字和，并用英语输出。

思路：被秒题

```
input: 12345
output: one five
```



```c++
string s;
int sum;
string out[] = {"zero","one","two","three","four","five","six","seven","eight","nine"};
int main(){
    cin>>s;
    if(s == "0"){
        cout<<"zero"<<endl;
        return 0;
    }
    for(auto ch: s)
        sum += (ch-'0');
    vector<string> ans;
    while(sum>0){
        ans.push_back(out[sum%10]);
        sum /= 10;
    }
    for(int i=ans.size()-1;i>=0;i--){
        if(i==ans.size()-1) cout<<ans[i];
        else cout<<" "<<ans[i];
    }
    cout<<endl;
}
```

### 1006 Sign In and Sign Out

给定$M$条记录，每一条是`ID, 登陆时间， 登出时间`，找出最找开门的人，和最晚关门的人。

思路：被秒题

```
intput:
3
CS301111 15:30:28 17:00:10
SC3021234 08:00:00 11:25:25
CS301133 21:45:00 21:58:40

output:
SC3021234 CS301133
```

```c++
struct record{
    string ID, begin, end;
};
bool cmp1(record a, record b){return a.begin < b.begin;}
bool cmp2(record a, record b){return a.end > b.end;}
int M;
record a[MAXN];
int main(){
    cin>>M;
    for(int i=0;i<M;i++)
        cin>>a[i].ID>>a[i].begin>>a[i].end;
    sort(a,a+M,cmp1);
    cout<<a[0].ID<<" ";
    sort(a,a+M, cmp2);
    cout<<a[0].ID<<endl;
}
```



### 1007  Maximum Subsequence Sum

题意：给定$N$个数字，找出最大连续子序列和

思路：典型dp，$O(n)$搞定，有个地方卡了十分钟，要输出的是最小起始位置和最小结束位置的**数字**，然而我一直输出了位置。

```
input:
10
-10 1 2 3 4 -5 -23 3 7 -21

output:
10 1 4        //和，最小起始位置数字，最小终止位置数字
```

```C++
ll a[MAXN], n;
bool is_negetive = true;
int main(){
    cin>>n;
    for(int i=0;i<n;i++) {
        cin>>a[i];
        if(a[i] >= 0)
            is_negetive = false;
    }
    if(is_negetive){
        cout<<"0 "<<a[0]<<" "<<a[n-1]<<endl;
        return 0;
    }

    ll nowsum=0, maxsum=-1, indexi=0, indexj=0, tmp=0;
    for(int i=0;i<n;i++){
        nowsum += a[i];
        if(nowsum > maxsum){
            maxsum = nowsum;
            indexi = tmp;
            indexj = i;
        }
        else if(nowsum < 0){
            nowsum = 0;
            tmp=i+1;
        }
    }
    cout<<maxsum<<" "<<a[indexi]<<" "<<a[indexj]<<endl;
}
```

