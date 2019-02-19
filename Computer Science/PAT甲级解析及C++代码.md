# PAT甲级解析及C++代码

[TOC]

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





### 1003 Emergency ***

给定城市数量$N (N\le 500)$，道路数目$M$，每个城市的救援人数$P_i$，起点和终点$C_1,C_2$

求$C_1$到$C_2$间的最短路条数，和路径上人数总和的最大值，保证存在至少一条路。

思路：就是考最短路，直接上Dijkstra就行，可以堆优化，也可以不进行堆优化，但要注意最短路的最优化需要小顶堆，因此申明的时候去要`greater<pair<ll,ll>>`



花了不少时间，几个WA点：

* 最短的条数计算，加法原理还是乘法原理？
* 可以一次dijkstra把答案搞出来吗？需要额外dfs吗？
* 使用`pair<ll, ll>`的时候`.first`和`.second`造成代码冗长，很难debug
* `memset(a, INF, sizeof(a))`似乎不奏效，需要`for`循环初始化
* 带堆优化的dijkstra代码中，如何去重复点？
*  TODO：自己写的结构体如何用优先队列排序？

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



### 1007  Maximum Subsequence Sum **

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



## 第三套

### 1008 Elevator

电梯上一层花6秒，下一层花4，每一次停驻花5秒，给定一个$N$个整数的请求，在电梯初始0层最少需要花多少秒完成所有请求。

```
input: 3 2 3 1
output: 41
```

```c++
ll req, n, ans=0, cur=0;
int main(){
    cin>>n;
    for(int i=0;i<n;i++) {
        cin>>req;
        if(req >= cur)
            ans += (req-cur)*6 + 5;
        else
            ans += (cur-req)*4 + 5;
        cur = req;
    }
    cout<<ans<<endl;
}
```



### 1008 Product of Ploynomials

题意，给定两个多项式，计算它们的乘积。

```
input:
2 1 2.4 0 3.2
2 2 1.5 1 0.5

output:
3 3 3.6 2 6.0 1 1.6
```

```c++
ll k=0, cnt=0, expo;
double poly[3][MAXN] = {0.0}, coeff;
int main(){
    cin>>k;
    for(int i=0;i<k;i++) {
        cin>>expo>>coeff;
        poly[0][expo] = coeff;
    }
    cin>>k;
    for(int i=0;i<k;i++) {
        cin>>expo>>coeff;
        poly[1][expo] = coeff;
    }

    for(int i=0;i<MAXN;i++){
        for(int j=0;j<MAXN;j++){
            if(poly[0][i] != 0.0 && poly[1][j] != 0.0){
                poly[2][i+j] += poly[0][i]*poly[1][j];
            }
        }
    }

    for(int i=0;i<MAXN;i++) 
        if(poly[2][i] != 0.0)
            cnt++;
    cout<<cnt<<" ";

    bool f=true;
    for(int i=MAXN-1;i>=0;i--){
        if(poly[2][i] != 0.0)
            if(f==false)
                printf(" %d %.1f",i, poly[2][i]);
            else{
                printf("%d %.1f",i, poly[2][i]); 
                f = false; 
            }   
    }
    cout<<endl;
}
```





### 1010 Radix *

给定两个数字$N_1, N_2$，告诉其中一个进制$r$，判断是否有另一个进制$x$使得$N_1 = N_2$

思路：转换已知数字到10进制，再搜索另一个数字的进制，此题WA点很多

```
input: 6 110 1 10
output: 2
```

* 没有告诉进制上限
* $N$最长9位，最大就是10亿，若另一个数组表示为$10$，则它是999999999进制，需要特判。

```c++
string N1, N2;
ll tag = 0, radix = 0, n1=0, n2=0;
ll go(string num, int radix){
    ll sum = 0, cnt=1;
    for(int i=num.length()-1;i>=0;i--){
        if(num[i]>='0' && num[i] <='9'){
            if(num[i]-'0' >= radix) return -1;
            sum += (num[i]-'0')*cnt;
        }
        else{
            if((num[i]-'a'+10) >= radix) return -1;
            sum += (num[i]-'a'+10)*cnt;
        }
        cnt *= radix;
    }
    return sum;
}
int main(){
    cin>>N1>>N2>>tag>>radix;
    if(tag == 1){
        n1 = go(N1, radix);
        if(N2=="10") {cout<<n1<<endl;return 0;}
        for(int i=1; i<=1000;i++){
            if(go(N2, i ) == n1 && n1!=-1){
                cout<<i<<endl;
                return 0;
            }
        }
    }
    else{
        n2 = go(N2, radix);
        if(N1=="10") {cout<<n2<<endl;return 0;}
        for(int i=1; i<=1000;i++){
            if(go(N1, i ) == n2 && n2!=-1){
                cout<<i<<endl;
                return 0;
            }
        }
    }
    cout<<"Impossible"<<endl;
    return 0;
}
```





## 第四套

### 1011 World Cup Betting

题意：给定三场比赛的（赢，平局，输）三种胜率，计算最大收益率

思路：被秒题

```c++
double a[3][3], ans=0.0,p=1.0;
int main(){
    for(int i=0;i<3;i++){
        double maxp = 0.0;
        for(int j=0;j<3;j++){
            cin>>a[i][j];
            maxp = max(maxp, a[i][j]);
        }
        for(int j=0;j<3;j++){
            if(maxp == a[i][j]){
                switch(j){
                    case 0: cout<<"W ";break;
                    case 1: cout<<"T ";break;
                    case 2: cout<<"L ";break;
                }
            }
        }
        p *= maxp;
    }
    printf("%.2f\n",(p*0.65-1.0)*2.0);
}
```



### 1012 The Best Rank *

题意：给$N$个学生，每个学生3门成绩$C, M,E$和平均分$A$，找出需要查找学生的最好排名并输出，若排名相同按优先级$A>C>M>E$输出。

思路：

这题很有意思，输出上的数据结构处理可以决定代码写50行还是150行，当然还包括排序

* 一个WA点坑了我将近2个小时，排名不能是`1 1 2 3 4`，而应该是`1 1 3 4 5`

```c++
struct rec{
    string ID;
    int score[4], rank[4];
};
int n,m,C,M,E, idx=-1;
char ch[] = {'A','C','M','E'};
string name;
rec stu[MAXN];
map<string, int> mp;
bool cmp(rec a, rec b){return a.score[idx] > b.score[idx];}
void go(int r){
    int rk = 1;
    stu[0].rank[r] = rk;
    for(int i=1;i<n;i++){
        rk++;
        if(stu[i].score[r] == stu[i-1].score[r]) 
            stu[i].rank[r] = stu[i-1].rank[r];
        else 
            stu[i].rank[r] = rk;
    }
}
int main(){
    cin>>n>>m;    
    for(int i=0;i<n;i++){
        cin>>name>>C>>M>>E;
        stu[i].ID=name; stu[i].score[1]=C; stu[i].score[2]=M; stu[i].score[3]=E;
        stu[i].score[0]= C+M+E;
    }

    for(idx=0;idx<4;idx++){
        sort(stu, stu+n, cmp);
        go(idx);
    }

    for(int i=0;i<n;i++) mp[stu[i].ID] = i;
    for(int i=0;i<m;i++){
        cin>>name;
        if(mp.count(name) == 0){
            cout<<"N/A"<<endl;
            continue;
        }
        int rk = INF, pos=INF;
        for(int j=0;j<4;j++){
            if(stu[mp[name]].rank[j] < rk){
                rk = stu[mp[name]].rank[j];
                pos = j;
            }
        }
        cout<<rk<<" "<<ch[pos]<<endl; 
    }
}
```



### 1013 Battle Over Cities *

题意：给定一幅图包含$N(<1000)$个点，$M$条边，以及$K$个城市，若这$K$中的每一个城市的邻边断后，还需要修几条路使剩下的$N-1$个城市连通。

思路：本质是，在图中去掉一个点后，计算连通分量，用并查集就能很好完成。刚开始不敢写并查集感觉会TLE，其实仔细分析一些，外层是$K$个点，内层并查集计算是$M$条边，和两个常数$N$，复杂度就是$O(K*(M+2N))$，稍微有点卡时间，用`cin`过不了，用`scanf`就行。

```c++
int n,m,k, ID[MAXN],u,v,ans=0;
vector<pii> edge;
int find(int x){return ID[x]==x?x:ID[x]=find(ID[x]);}
void UNION(int x, int y) {ID[find(x)] = find(y);}
int main(){
    scanf("%d%d%d",&n,&m,&k);
    for(int i=0;i<m;i++){
        scanf("%d%d",&u,&v);
        edge.push_back({u,v});
    }
    for(int i=0;i<k;i++){
        scanf("%d",&u);
        for(int j=1;j<=n;j++) ID[j] = j;
        for(auto e: edge){
            if(e.first==u || e.second==u) continue;
            UNION(e.first, e.second);
        }
        set<int> st;
        for(int i=1;i<=n;i++) st.insert(find(i));
        cout<<st.size()-2<<endl;
    }
}
```



### 1014 Waiting in Line ***

题意：银行有N个窗口，每个窗口最多排M个客户，客户总是选择最短的队排，如果一样短就选序号小的窗口。银行早上8点开始服务。每个客户办业务要花费的时间是$T_i$，输出$Q$个请求，对每个请求$r_i$，对于客户的交易完成时间？但注意银行17:00前关门，在此之前无法被服务的客户输出`Sorry`

思路：模拟题，但要非常非常注意题面（**尤其注意代码第24行**）

几个wa点

* 一定是在17:00前无法被服务的用户输出Sorry，也就是说最晚16:59就要服务到
* 答案输出是hh:mm, hh是8-17，mm是0-59，所以有一部分用户就算被服务了，也可能超时
* 一旦黄线区域内人数少于N*M，多余的人就要去排队，选择人最少的。
* 经过几次惨痛的debug教训：下标一定要从0开始

```c++
int N, M, K, Q, T[MAXN], ans[MAXN]={INF}, cur=0, t[22]={0}, q, out=-1;
queue<int> win[22];
int check(){
    int ret = INF, pos = -1;
    for(int i=0;i<N;i++)
        if(!win[i].empty() && ret > t[i]+T[win[i].front()]){
            ret = t[i]+T[win[i].front()];
            pos = i;
        }
    return pos;
}
int main() {
    cin>>N>>M>>K>>Q;
    for(int i=0;i<K;i++) cin>>T[i];
    while(cur < K && cur < N*M) {
        int pos = cur%N;
        win[pos].push(cur);
        cur++;
    }
    while((out = check()) != -1){
        int customer = win[out].front(); win[out].pop();
        t[out] += T[customer];
        ans[customer] = t[out];
        if(ans[customer] - T[customer] >= 540 || ans[customer] >= 600) ans[customer] = INF;
        if(cur < K )
            win[out].push(cur++);
    }
    
    while(Q--){
        cin>>q;q--;
        if(ans[q] == INF)
            cout<<"Sorry"<<endl;
        else
            printf("%02d:%02d\n", ans[q]/60+8, ans[q]%60);
    }
}
```



## 第五套

### 1015 Reversible Primes *

题意：给定一个数字，如果它本身和翻转后的数字都是质数，它就是翻转指数，给定$N,D$，判断数字$N$在$D$进制下是不是翻转质数。

思路：秒题，wa点在于**0和1不是质数**

```c++
int n, rn, d;
int go(int n, int r){
    int ret = 0, cnt=1;
    string s="";
    while(n>0){
        s.append(1, n%r+'0');
        n /= r;
    }
    for(int i=s.length()-1;i>=0;i--){
        ret += (s[i]-'0')*cnt;
        cnt *= r;
    }
    return ret;
}
bool is_prime(int n){
    if(n==0 || n==1) return false;
    for(int i=2;i*i<=n;i++){
        if(n%i == 0)
            return false;
    }
    return true;
}
int main() {
    while(cin>>n){
        if(n < 0) return 0 ;
        cin>>d;
        rn = go(n, d);
        if(is_prime(n) && is_prime(rn)) cout<<"Yes"<<endl;
        else cout<<"No"<<endl;
    }
}
```





### 1016 Phone Bills **

题意：给定一些电话记录`姓名，时间，tag`，给定每个小时区间价格，计算这个月的话费。

思路：模拟题，非常恶心，犯了一个很傻的错误，导致一直段错误。

尤其注意第57和67行，否则**某些时候会往答案里push空值**，导致输出的时候发生段错误。

```c++
struct node{
    string name, time, tag;
};
struct result{
    string name;
    vector<string> start, end;
    vector<int> bill, mins;
    int tot;
};
node a[MAXN];
vector<node> user[MAXN];
vector<result> ans;
int n, rate[34],id=0, minutes=0;
map<string, int> mp;
bool cmptime(node a, node b){return a.time < b.time;}
bool cmpname(result a, result b){return a.name < b.name;}
int fee(string begin, string end){
    int ret = 0; minutes=0;
    while(begin < end){
        int hh = begin[7]-'0' + (begin[6]-'0')*10;
        ret += rate[hh]; minutes++;
        begin[10] += 1;
        if(begin[10] > '9'){
            begin[10] = '0';
            begin[9] += 1;
            if(begin[9] >= '6')
            {begin[9]='0'; begin[7]+=1;}
        }
        if(begin[6]=='2' && begin[7]=='4'){
            begin[6] = begin[7] = '0';
            begin[4] += 1;
        }
        else if(begin[7]>'9'){
            begin[7] = '0';
            if(begin[6] == '0') begin[6]='1';
            else if(begin[6]=='1') begin[6] = '2';
        }
        if(begin[4] > '9'){
            begin[4] = '0';
            begin[3] += 1;
        }
    }
    return ret;
}
int main(){
    for(int i=0;i<24;i++) cin>>rate[i];
    cin>>n;
    for(int i=0;i<n;i++) {
        cin>>a[i].name>>a[i].time>>a[i].tag;
        if(mp.count(a[i].name) == 0)
            mp[a[i].name] = id++;
        user[mp[a[i].name]].push_back(a[i]);
    }   
    for(int i=0;i<id;i++){
        sort(user[i].begin(), user[i].end(), cmptime);
        result res;res.tot=0;
        bool f = false;
        for(int j=1;j<user[i].size();j++){
            if( !(user[i][j].tag=="off-line" && user[i][j-1].tag=="on-line")) continue;
            res.name = user[i][j].name;
            res.start.push_back(user[i][j-1].time);
            res.end.push_back(user[i][j].time);
            int charge = fee(user[i][j-1].time, user[i][j].time);
            res.bill.push_back(charge);
            res.tot += charge;
            res.mins.push_back(minutes);
            f = true;
        }
        if(f)
            ans.push_back(res);
    }
    sort(ans.begin(), ans.end(), cmpname);
    for(auto usr: ans){
        cout<<usr.name<<" "<<usr.start[0].substr(0,2)<<endl;
        for(int i=0;i<usr.bill.size();i++){
            cout<<usr.start[i].substr(3,8)<<" "<<usr.end[i].substr(3,8)<<" "<<usr.mins[i];
            printf(" $%.2f\n",(double)usr.bill[i]/100.0);
        }
        printf("Total amount: $%.2f\n",(double)usr.tot/100.0);
    }
}
```



### 1017 Queueing at Bank **

题意：银行有K个窗口，营业时间是8:00到17:00，给N个客户的到达时间和办业务时间，计算平均等待时间。（每个窗口黄线内只能排1人），参考1014。

思路：模拟题，但是要注意

* 17:00到达之前的所有人，都要被服务（因此输入阶段就可以过滤17:00后到达的人）
* 特判一下无有效人数的情况
* 以时间模拟的情况下，每一秒钟，**应该先处理出队，再处理进队**

```c++
int n, k, hh, mm, ss, p, out=0, cur=0,tic=8*3600;
ll sum = 0;
struct node{
    int in, last, out;
    bool operator< (const node& a) const {return in<a.in;}
} a[MAXN];
int main(){
    scanf("%d%d",&n, &k);
    for(int i=0;i<n;i++){
        scanf("%d:%d:%d", &hh, &mm, &ss);
        scanf("%d", &p);
        int tt = hh*3600 + mm*60 + ss;
        if(tt > 17*3600){
            i--; n--;
            continue;
        }
        a[i].in = tt;
        a[i].last = p*60;
    }
    sort(a, a+n);
    queue<int> w[k];
    while(cur < n) {
        for(int i=0;i<k;i++){
            if(!w[i].empty() && tic == a[w[i].front()].out){
                w[i].pop();
                out++;
            }
        }    
        bool f = false;
        if(cur < n && tic >=  a[cur].in){
            for(int i=0;i<k;i++){
                if(w[i].empty()){
                    a[cur].out = tic + a[cur].last;                 
                    sum += (tic - a[cur].in);
                    w[i].push(cur++);
                    f = true;
                    break;
                }
            }
        } 
        tic += f?0:1;
    }
    if(n == 0) printf("0.0\n");
    printf("%.1f\n", (double)sum/60.0/n);
}
```





### 1018 Public Bike Management ***

题意：n个自行车站，每个站点数量达到$C_{max}/2$为完美状态，若有一站数量为$0$或$C_{max}$，中心将选择最短路，将沿路车站都变成完美状态，若存在多条最短路，选择需要从中心运出车量最少的一个。

思路：这道题有点恶心，主要考察最短路。所以先求最短路，算出pre数组，然后通过dfs找到所有最短路，然后在这些最短路里找符合答案的一条，注意以下几点

* 多条最短路时，选择需要从出发点携带最少数量自行车的最短路，若这也是多条，选择回起点时需要携带的最少数量自行车的最短路。
* 多条路径最短路如何记录路径？ **(14，33，42，69行很重要，是出错点)**



```c++
int cmax, n, sp, pbmc=0, m, num[MAXN], u, v, t, d[MAXN] = {INF};
vector<pii> G[MAXN];
vector<int> pre[MAXN], p;
vector<vector<int>> path;
bool vis[MAXN] = {false};
void find_path(int u, vector<int>& p){
    p.push_back(u);
    if(u == 0){
        path.push_back(p);
        return ;
    }
    for(auto son: pre[u]){
        find_path(son, p);
        p.pop_back();
    }
}
int main(){
    cin>>cmax>>n>>sp>>m;
    for(int i=1;i<=n;i++) cin>>num[i];
    for(int i=0;i<m;i++){
        cin>>u>>v>>t;
        G[u].push_back({v,t});
        G[v].push_back({u,t});
    }
    for(int i=1;i<=n;i++) d[i] = INF;
   
    priority_queue<pii, vector<pii>, greater<pii>> Q;
    Q.push({0,0}); d[0] = 0;
    while(!Q.empty()){
        int city = Q.top().second, dist = Q.top().first;
        Q.pop();
        vis[city] = true;
        if(d[city] != dist) continue;
        for(auto son: G[city]){
            if(d[city] + son.second < d[son.first]){
                d[son.first] = d[city] + son.second;
                pre[son.first].clear();
                pre[son.first].push_back(city);
            }
            else if (d[city] + son.second == d[son.first])
                pre[son.first].push_back(city);
            if(vis[son.first] == false)
                Q.push({d[son.first], son.first});  
        }
    }

    find_path(sp, p);
    for(auto& x: path) reverse(x.begin(), x.end());
    int pos = 0, minhave =INF, minneed = INF, ansneed, anshave;
    for(int i=0;i<path.size();i++){
        int need = 0, have = 0;
        for(int j=0; j<path[i].size();j++){
            if(path[i][j] == 0 || num[path[i][j]] == cmax/2) continue;
            if(num[path[i][j]] < cmax/2){
                int diff = cmax/2 - num[path[i][j]];
                if(have >= diff ) have -= diff;
                else {
                    diff -= have; have = 0;
                    need += diff;
                }
            }
            else if(num[path[i][j]] > cmax/2)
                have += (num[path[i][j]] -  cmax/2);
        }
        if(need < minneed){
            pos = i;
            minneed = need; minhave = have;
        }
        else if(need == minneed && have < minhave){
            pos = i;
            minneed = need; minhave = have;
        }
    }
    cout<<minneed<<" ";
    for(auto x: path[pos]) printf(x==0?"0":"->%d",x);
    cout<<" "<<minhave<<endl;
}
```



## 第六套

### 1019 General Palindromic Number

题意：给定b进制数字n，判断它是不是回文数

思路：被秒题

```c++
int n, b, f=0;
vector<int> num;
int main(){
    cin>>n>>b;
    while(n>0){
        num.push_back(n%b);
        n /= b;
    }
    reverse(num.begin(), num.end());
    for(int i=0,j=num.size()-1; i<j;i++,j--) {
        if(num[i] != num[j]){
            cout<<"No"<<endl;
            f = 1;
            break;
        }
    }
    if(f == 0)
        cout<<"Yes"<<endl;
    for(int i=0;i<num.size();i++)
        printf(i==0?"%d":" %d", num[i]);
}
```



### 1020 Tree Traversals *

题意：给定二叉树的中序和后序遍历，输出层序遍历

思路：考察二叉树遍历，递归写法和BFS。注意**判断递归终止条件**。

```c++
struct node{
    int val;
    node* left, *right;
    node(){val=-1;left=right=NULL;}
};
int n, t;
vector<int> post, in, level;
node* build(vector<int> post, vector<int> in){
    if(post.size() == 0 && in.size() == 0) return NULL;
    node* rt = new node();
    rt->val = post[post.size()-1];
    vector<int> p1,p2,i1,i2;
    bool f = false;
    for(int i=0;i<in.size();i++){
        if(!f && in[i] != rt->val){
            p1.push_back(post[i]);
            i1.push_back(in[i]);
        }
        else if(in[i] == rt->val)
            f = true;
        else{
            p2.push_back(post[i-1]);
            i2.push_back(in[i]);
        }
    }
    rt->left = build(p1, i1);
    rt->right = build(p2, i2);
    return rt;
}
int main(){
    cin>>n;
    for(int i=0;i<n;i++){
        cin>>t;
        post.push_back(t);
    }
    for(int i=0;i<n;i++){
        cin>>t;
        in.push_back(t);
    }
    node* rt = build(post, in);
    queue<node*>  q;
    q.push(rt); 
    while(!q.empty()){
        node* rt = q.front(); q.pop();
        if(rt->left != NULL) q.push(rt->left);
        if(rt->right != NULL) q.push(rt->right);
        level.push_back(rt->val);
    }
    for(int i=0;i<level.size();i++)
        printf(i==0?"%d":" %d",level[i]);
}
```



### 1021 Deepest Root **

题意：给定一棵树($N<10^4$)，求最深深度，输出此时根节点编号，若不唯一，升序输出。若不是树，输出连通分量个数。

思路：由于时间给了2s，可以直接$O(n^2)$暴力，并查集+暴力dfs。dfs内联可以使时间从859ms降到647ms。

```c++
vector<pii> e;
vector<int> G[MAXN], a;
int n,u,v, ID[MAXN], ans[MAXN], maxdeep=-1, deep=0;
int find(int x){return x==ID[x]?x:ID[x]=find(ID[x]);}
void UNION(int u, int v){ID[find(u)] = find(v);}
inline void dfs(int u, int fa, int p){
    deep = max(deep, p);
    for(auto son: G[u])
        if(son != fa)
            dfs(son, u, p+1);
}
int main(){
    cin>>n;
    for(int i=0;i<=n;i++) ID[i] = i;
    for(int i=0;i<n-1;i++){
        cin>>u>>v;
        G[u].push_back(v); G[v].push_back(u);
        e.push_back({u,v});
        UNION(u,v);
    }
    set<int> s;
    for(int i=1;i<=n;i++) s.insert(find(i));
    if(s.size() > 1){
        cout<<"Error: "<<s.size()<<" components"<<endl;
        return 0;
    }
    for(int i=1;i<=n;i++){
        deep = 0;
        dfs(i, -1, 0);
        ans[i] = deep;
        maxdeep = max(maxdeep, deep);
    }
    for(int i=1;i<=n;i++)
        if(ans[i] == maxdeep)
            cout<<i<<endl;
}
```



### 1022 Digital Library **

题意：给n本书，包括编号，标题，作者，出版社，关键字（多个），年份。再进行m次查找，给出对于查找形式，输出符合内容的书的编号，升序排列。

思路：纯粹是考输入。

* 不要从键盘输入，文件输入debug
* 使用getliney要非常小心，尤其注意**11, 31**行，原因不明。似乎每一次循环前要多读一次清掉缓存。

```c++
struct book{
    string id, title, author, publisher, year;
    vector<string> keywords;
    bool operator<(const book& a) const {return id < a.id;}
} a[MAXN];
int n, m, q;
string tmp, t;
vector<book>  ans;
int main(){
    cin>>n;
    getline(cin, tmp);
    for(int i=0;i<n;i++){
        tmp=""; t="";
        getline(cin, a[i].id);
        getline(cin, a[i].title);
        getline(cin, a[i].author);
        getline(cin, tmp);
        for(auto x: tmp){
            if(x == ' ') {
                a[i].keywords.push_back(t);
                t = "";
            }
            else
                t.append(1, x);
        }
        a[i].keywords.push_back(t);
        getline(cin, a[i].publisher);
        getline(cin, a[i].year);
    }
    cin>>m;
    getline(cin, tmp);
    while(m--){
        tmp = "";
        getline(cin, tmp); 
        cout<<tmp<<endl;
        int num = tmp[0]-'0';
        t = tmp.substr(3, tmp.length()-3);
        ans.clear();
        switch(num){
            case 1: {
                for(int i=0;i<n;i++)
                    if(a[i].title == t)
                        ans.push_back(a[i]);
                break;
            }
            case 2: {
                for(int i=0;i<n;i++)
                    if(a[i].author == t)
                        ans.push_back(a[i]);
                break;
            }
            case 3: {
                for(int i=0;i<n;i++)
                    for(auto word: a[i].keywords) {
                        if(word == t){
                            ans.push_back(a[i]);
                            break;
                        }
                    }
                break;
            }
            case 4: {
                for(int i=0;i<n;i++)
                    if(a[i].publisher == t)
                        ans.push_back(a[i]);
                break;
            }
            case 5: {
                for(int i=0;i<n;i++)
                    if(a[i].year == t)
                        ans.push_back(a[i]);
                break;
            }
        }
        sort(ans.begin(), ans.end());
        for(auto x: ans) cout<<x.id<<endl;
        if(ans.empty()) cout<<"Not Found"<<endl;
    }
}
```



## 第六套

### 1023 Have Fun with Numbers

题意：给一个长度小于20位的数字，乘2以后的数字是否是前面数字的一个排列？

思路：简单模拟题，大整数乘法。

```c++
string n1, n2;
int a1[10]={0}, a2[10]={0};
string go(string s){
    string ret = "";
    int carry = 0;
    for(int i=s.length()-1; i>=0;i--){
        int num = (s[i]-'0')*2 + carry;
        ret.append(1, num%10+'0');
        carry = num/10;
    }
    if(carry)
        ret.append(1, carry+'0');
    reverse(ret.begin(), ret.end());
    return ret;
}
int main(){
   cin>>n1;
   n2 = go(n1);
   for(int i=0;i<n1.length();i++) a1[n1[i]-'0']++;
   for(int i=0;i<n2.length();i++) a2[n2[i]-'0']++;
   bool f = true;
   for(int i=0;i<10;i++)
        if(a1[i] != a2[i])
            f = false;
    printf(f?"Yes\n":"No\n");
    cout<<n2<<endl;
}
```



### 1024 Palindromic Number

题意：给一个数字，若它不是回文数字，则加上它的翻转数，直到它是回文数字，或者操作次数大于了k次。

思路：字符串加法，很简单。

```c++
string n,p;
int cnt = 0, k;
string add(string s,string t){
    string ret = "";
    reverse(s.begin(), s.end()); reverse(t.begin(), t.end());
    int carry = 0, idx=0;
    for(idx=0;idx<s.length();idx++){
        int num = s[idx]+t[idx]-'0'-'0'+carry;
        ret.append(1, num%10+'0');
        carry = num/10;
    }
    if(carry)
        ret.append(1, carry+'0');
    reverse(ret.begin(), ret.end());
    return ret;
}
bool is_palindrom(string s){
    for(int i=0,j=s.length()-1;i<j;i++,j--)
        if(s[i] != s[j])
            return false;
    return true;
}
int main(){
   cin>>n>>k;
   while(cnt < k){
       if(is_palindrom(n)){
           cout<<n<<endl;
           cout<<cnt<<endl;
           return 0;
       }
       string r = n;
       reverse(n.begin(), n.end());
       n = add(r, n);
       cnt++;
   }
   cout<<n<<endl;
   cout<<k<<endl;
}
```



### 1025 PAT Ranking 

题意：给n个考场，每个考场k人，每人有id和分数。求每人的考场排名和总排名，若排名一样按id升序。

题意：考排序，很简单。

```c++
struct node{
    string id; 
    int score, localrank, globalrank, loc;
    bool operator<(const node& a) const {
        return score != a.score ? score > a.score : id < a.id;
    }
};
vector<node> a[MAXN], global;
int n,k, rk=1;
string id;
void gorank(vector<node>& a, int flag){
    int rk = 1;
    if(flag == 0)
        a[0].localrank = rk;
    else
        a[0].globalrank = rk;
    
    for(int i=1;i<a.size();i++){
        if( a[i].score != a[i-1].score) {
            if(flag == 0) a[i].localrank = ++rk;
            else a[i].globalrank = ++rk;
        }
        else{
            if(flag == 0) a[i].localrank = a[i-1].localrank;
            else a[i].globalrank = a[i-1].globalrank;
            rk++;
        }
    }
}
int main(){
    cin>>n;
    for(int i=1;i<=n;i++){
        cin>>k;
        for(int j=0;j<k;j++){
            node t;
            cin>>t.id>>t.score;
            t.loc = i;
            a[i].push_back(t);
        }
        sort(a[i].begin(), a[i].end());
        gorank(a[i], 0);
        for(auto x: a[i]) global.push_back(x);
    }
    sort(global.begin(), global.end());
    gorank(global, 1);
    cout<<global.size()<<endl;
    for(auto x: global)
        cout<<x.id<<" "<<x.globalrank<<" "<<x.loc<<" "<<x.localrank<<endl;
}
```



### 1026 Table Tennis ***

题意：n队人来玩乒乓球，有到达时间，玩耍时间，是否是vip用户。球厅有k张桌子，其中m个是vip。接待时间是8:00-21:00。若有多余桌子，那么队列头的用户选择序号最小的。特殊的地方是，如果有vip桌且队列中有vip客户，那么队列中第一个vip客户可以选择序号最小的vip桌。否则按正常用户对待。求出每队人的等待时间，以及每张桌子的接客数量。

思路：巨坑题模拟，以下是注意的地方。

* 题目中说“It is assumed that every pair of players can play for at most 2 hours.”，要理解成：**输入的玩耍时间大于2小时按2小时处理**，而不是**假设所有输入的玩耍时间最大2小时**

* vip可以插队的时候，选择vip座位中序号最小
* **无vip座位或队伍中无vip用户的时候**，用户总是选择序号最小的座位，不管是不是vip座位
* 等待时间折算成分钟，四舍五入，**（题目中用的“round up”，容易理解成向上取整）**
* 按秒数处理程序，输入的玩耍时间需要乘个60
* 输出按“开始服务时间”升序排列

tips：

* “选择序号最小”的这种性质，用队列维护（开始用的vector，浪费不少时间）
* 模拟时间主循环里，串行写好先后步骤，再一步一步实现（while里的注释部分）
* 出错的时候，二分查找错误点。
* 答案错误的时候，思考良久没有思路的话，再仔细读题

```c++
struct node{
    int tt, hh, mm, ss, len, vip, ser, wait, out;
    bool operator< (const node& a) const {return tt<a.tt;}
    node(){tt=hh=mm=ss=len=vip=ser=wait=out=0;}
} a[MAXN];
vector<node> win[111], ans;
map<int, int> viptable;
int n,hh,mm,ss,len,tag,k,m,table, t=8*3600, idx=0, cnt[111] = {0};
bool cmpser(node a, node b){return a.ser<b.ser;}
int main(){
    cin>>n;
    for(int i=0;i<n;i++){
        scanf("%d:%d:%d", &a[i].hh, &a[i].mm, &a[i].ss);
        scanf("%d%d", &a[i].len, &a[i].vip);
        a[i].tt = a[i].ss + a[i].mm*60 + a[i].hh*3600;
        a[i].len *= 60;
        if(a[i].len > 2*3600) a[i].len = 2*3600;
    }
    cin>>k>>m;
    for(int i=1;i<=m;i++){
        cin>>table;
        viptable[table] = 1;
    }
    sort(a, a+n);
    queue<node> q, vipq;
    while(t <= 23*3600){
        //clear table first
        for(int i=1;i<=k;i++){
            if(!win[i].empty() && win[i][0].out == t && win[i][0].ser < 21*3600){
                ans.push_back(win[i][0]);
                cnt[i]++;
                win[i].clear();
            }
        }
        //enqueue
        if(idx < n && t >= a[idx].tt){
            if(a[idx].vip == 1) vipq.push(a[idx]);
            else q.push(a[idx]);
            idx++;
        }
        //gather empty table info
        queue<int> avialable_table, aviablable_vip;
        for(int i=1;i<=k;i++){
            if(win[i].empty()){
                if(viptable[i] == 1) 
                    aviablable_vip.push(i);
                else 
                    avialable_table.push(i);
            }
        }
        //assign table for vip 
        while(!vipq.empty() && aviablable_vip.size() > 0){
            node cus = vipq.front(); vipq.pop();
            cus.out = t + cus.len;
            cus.wait = t - cus.tt;
            cus.ser = t;
            win[aviablable_vip.front()].push_back(cus);
            aviablable_vip.pop();
        }
        //assign table for customer, if no vip customer nor vip tables
        while(aviablable_vip.size()+avialable_table.size()>0 && q.size()+vipq.size()>0){
            node cus;
            if(!vipq.empty()) {
                cus = vipq.front();
                if(!q.empty() && q.front().tt < cus.tt){
                    cus = q.front();
                    q.pop();
                }
                else
                    vipq.pop();
            }
            else {
                cus = q.front(); 
                q.pop();
            }   
            cus.out = t + cus.len;
            cus.wait = t - cus.tt;
            cus.ser = t;
            int p1 = INF, p2 = INF, pos=INF;
            if(!avialable_table.empty()) p1 = avialable_table.front();
            if(!aviablable_vip.empty())  p2 = aviablable_vip.front();
            pos = min(p1, p2);
            win[pos].push_back(cus);
            if(pos == p1) avialable_table.pop();
            else aviablable_vip.pop();
        }
        t++;
    }
    sort(ans.begin(), ans.end(), cmpser);
    for(auto x: ans){
        int p = x.wait/60 + (x.wait%60>=30?1:0);
        printf("%02d:%02d:%02d %02d:%02d:%02d %d\n",x.hh,x.mm,x.ss,x.ser/3600,x.ser%3600/60,x.ser%60,p);
    }
    for(int i=1;i<=k;i++)
        printf(i==1?"%d":" %d", cnt[i]);
}
```



## 第七套

### 1027 Colors in Mars 

题意：给3个整数代表R,G,B，转换成RGB数值（13进制）

```c++
int a,b,c;
string go(int num){
    string ret = "";
    if(num/13 < 10) ret.append(1, num/13+'0');
    else ret.append(1, num/13-10+'A');
 
    if(num%13 < 10) ret.append(1, num%13+'0');
    else ret.append(1, num%13-10+'A');
    return ret;
}
int main(){
    cin>>a>>b>>c;
    cout<<"#"<<go(a)<<go(b)<<go(c)<<endl;
}
```



### 1028 List Sorting 

题意：给n条记录`id,name,grade`，按需求排序，id,name升序，grade降序，若存在一样的按id升序

思路：排序，注意规模是$10^5$，用`cin,cout`会超时，特殊处理一下

```c++
struct node{
    string id, name, grade;
} a[MAXN];
int n, c;
bool cmp(node a, node b){
    if(c == 1) return a.id<b.id;
    if(c == 2) return a.name==b.name?a.id<b.id:a.name<b.name;
    else return a.grade==b.grade?a.id<b.id:a.grade<b.grade;
}
int main(){
    ios::sync_with_stdio(false);cin.tie(0);cout.tie(0);
    cin>>n>>c;
    for(int i=0;i<n;i++)
        cin>>a[i].id>>a[i].name>>a[i].grade;
    sort(a, a+n, cmp);
    for(int i=0;i<n;i++)
        printf("%s %s %s\n",a[i].id.c_str(), a[i].name.c_str(), a[i].grade.c_str());
}
```



### 1029 Median \****

题意：给2个有序序列，每个的规模$N<2*10^5$，求合并后的有序序列的中位数。**内存1.5Mb，时间200ms**

思路：第一个数组存起来，读第二个数组里的数字，到（sa+sb+1/2停下，就是答案，但注意处理边界情况，一个是a数组读到末尾，一个是b数组读到末尾。**注意12和15行**

```c++
int sa, sb, a[MAXN], ans=0, cnt=0, num=0, idxa=0, in=0;
int main() {
    scanf("%d",&sa);
    for(int i=0;i<sa;i++) scanf("%d", a+i);
    scanf("%d",&sb);
    bool f = true;
    while(cnt < (sa+sb+1)/2){
        if(f && in < sb){
            scanf("%d", &num);
            in++;
        }
        else if(f)
            num = INF;

        if(idxa<sa && a[idxa] < num) {
            ans = a[idxa++];
            f = false;
        }
        else{
            ans = num;
            f = true;
        }
        cnt++;
    }
    printf("%d\n",ans);
}
```



### 1030 Travel Plan

给定一个图，每条边上有距离和花费，求最短路，如有多条求最小花费最短路

```java
import java.util.*;
import java.lang.*;
import java.io.*;
public class Main {
    public static void main(String[] args) {
        FastScanner in = new FastScanner(System.in);
        int n, m, s, d, u, v;
        n = in.nextInt();
        m = in.nextInt();
        s = in.nextInt();
        d = in.nextInt();
        int[][] D = new int[n+1][n+1];
        int[][] C = new int[n+1][n+1];
        for(int i=0;i<n+1;i++) for(int j=0;j<n+1;j++){
            D[i][j] = -1;
            C[i][j] = -1;
        }
        for(int i=0;i<m;i++){
            u = in.nextInt();
            v = in.nextInt();
            int dist = in.nextInt();
            int cost = in.nextInt();
            D[u][v] = dist;
            D[v][u] = dist;
            C[u][v] = cost;
            C[v][u] = cost;
        }

        int totdis=0, totcost=0;
        int[] cost = new int[505];
        int[] dist = new int[505];
        int[] pre = new int[505];
        for(int i=0;i<505;i++){
            cost[i] = 0x7fffffff;
            dist[i] = 0x7fffffff;
        }
        boolean vis[] = new boolean[505];

        cost[s] = 0;
        dist[s] = 0;
        myPair e = new myPair(0, s);
        PriorityQueue<myPair> q = new PriorityQueue<>();
        q.add(e);

        while(!q.isEmpty()){
            myPair top = q.remove();
            if(dist[top.city] != top.dist || vis[top.dist]) continue;

            int city = top.city;
            int nowdist = top.dist;

            for(int i=0; i<n+1;i++){
                if(D[city][i] == -1) continue;
                int dd = D[city][i];

                if(dd + nowdist < dist[i]){
                    dist[i] = dd + nowdist;
                    pre[i] = city;
                    cost[i] = cost[city] + C[city][i];
                }
                else if(dd + nowdist == dist[i] && cost[city] + C[city][i] < cost[i]) {
                    pre[i] = city;
                    cost[i] = cost[city] + C[city][i];
                }
                myPair a = new myPair(dist[i], i);
                if(!vis[i])
                    q.add(a);
            }
            vis[city] = true;
        }

        ArrayList<Integer> path = new ArrayList<>();
        int now = d;
        while(now != s){
            path.add(now);
            now = pre[now];
        }
        path.add(s);
        Collections.reverse(path);
        for(int i=0;i<path.size();i++)
            System.out.print(path.get(i) + " ");
        System.out.print(dist[d]+" ");
        System.out.println(cost[d]);
    }
}

class myPair implements Comparable<myPair>{
    public int dist, city;
    myPair(int a, int b){
        this.dist = a;
        this.city = b;
    }
    public int compareTo(myPair other){
        if(other.dist == this.dist)
            return this.city - other.city;
        else
            return this.dist - other.dist;
    }
}
class FastScanner {
    BufferedReader br;
    StringTokenizer st;
    FastScanner(InputStream ins){
        try{
            br = new BufferedReader(new InputStreamReader(ins));
        } catch (Exception e){
            e.printStackTrace();;
        }
    }
    public String next() {
        while(st == null || !st.hasMoreTokens()){
            try{
                st = new StringTokenizer(br.readLine());
            } catch (IOException e){
                e.printStackTrace();
            }
        }
        return st.nextToken();
    }
    public int nextInt() {
        return Integer.parseInt(next());
    }
}
```



## 第八套

### 1031 Hello World for U

题意：“u”型输出字符串，先从上到下，再从左到右，再从下到上，越正方形越好。

```java
public class Main {
    public static void main(String[] args) {
        FastScanner in = new FastScanner(System.in);
        String s = in.next();
        int len = s.length(), idx=0;
        int n1=(len+2)/3, n3 = (len+2)/3;
        int n2 = len-n1-n3;

        char ans[][] = new char[n1][n2+2];
        for(int i=0;i<n1;i++) for(int j=0;j<n2+2;j++) ans[i][j] = '.';

        for(int i=0;i<n1;i++) ans[i][0] = s.charAt(idx++);
        for(int j=1;j<=n2;j++) ans[n1-1][j] = s.charAt(idx++);
        for(int i=n1-1;i>=0;i--) ans[i][n2+1] = s.charAt(idx++);

        for(int i=0;i<n1;i++){
            for(int j=0;j<n2+2;j++){
                if(ans[i][j] != '.')
                    System.out.print(ans[i][j]);
                else
                    System.out.print(" ");
            }
            System.out.println();
        }
    }
}
```



### 1032 Sharing

题意：给两个链表，确认它们第一个重合的节点地址

思路：模拟链表，数据的规模时$10^5$，Java过不了最后一个测试点，c++关掉同步也需要125ms(map)，96ms(unordered_map)

```c++
string a,b,add,nxt,data;
int n;
map<string, string> mp;
int main() {
    ios::sync_with_stdio(false);cin.tie(0);
    cin>>a>>b>>n;
    for(int i=0;i<n;i++){
        cin>>add>>data>>nxt;
        mp[add] = nxt;
    }
    vector<string> p1, p2;
    while(a != "-1"){
        p1.push_back(a);
        a = mp[a];
    }
    p1.push_back("-1");
    while(b != "-1"){
        p2.push_back(b);
        b = mp[b];
    }
    p2.push_back("-1");
    int pos1 = p1.size()-1, pos2 = p2.size()-1;
    while(pos1>=0 && pos2>=0 && p1[pos1] == p2[pos2]){
        pos1--;
        pos2--;
    }
    cout<<p1[pos1+1]<<endl;
}
```



### 1033 1033 To Fill or Not to Fill

### 1034 Head of a Gang

题意：给n个记录`name1 name2 time`，表示两人电话联系时间，若一个cluster大于2人，且总的电话时间大于k说明这是一个gang，求gang的个数和每个gang的头目，头目是电话时间最长的人

思路：并查集

```c++
int n, k, t, cnt = 0, w[MAXN], ID[MAXN];
string a, b;
map<string,int> mp;
map<int, string> rmp;
int find(int x){return x==ID[x]?x:ID[x]=find(ID[x]);}
void UNION(int x, int y){ID[find(x)] = find(y);}
int main() {
    ios::sync_with_stdio(false);cin.tie(0);
    cin>>n>>k;
    for(int i=0;i<MAXN;i++) ID[i] = i;
    for(int i=0;i<n;i++){
        cin>>a>>b>>t;
        if(mp.count(a) == 0){
            mp[a] = cnt;
            rmp[cnt] = a;
            cnt++;
        }
        if(mp.count(b) == 0){
            mp[b] = cnt;
            rmp[cnt] = b;
            cnt++;
        }
        w[mp[a]] += t;
        w[mp[b]] += t;
        UNION(mp[a],mp[b]);
    }
    vector<string> gang[MAXN];
    for(int i=0;i<cnt;i++)
        gang[find(i)].push_back(rmp[i]);
    
    vector<pair<string, int>> ans;
    for(int i=0;i<cnt;i++){
        if(gang[i].size() > 2){
            string head = gang[i][0];
            int weight = w[mp[head]], tot=0;
            for(auto x: gang[i]){
                tot += w[mp[x]];
                if(w[mp[x]] > weight){
                    weight = w[mp[x]];
                    head = x;
                }
            }
            if(tot/2 > k)
                ans.push_back({head, gang[i].size()});
        }
    }
    sort(ans.begin(), ans.end());
    cout<<ans.size()<<endl;
    for(auto x: ans)
        cout<<x.first<<" "<<x.second<<endl;
}
```





## 第九套

### 1035 Password 

题意：常规字符串处理

思路：注意`StringBuffer`的使用和`replace ` 方法的使用

```java
public class Main {
    public static void main(String[] args) {
        FastScanner in = new FastScanner(System.in);
        int n = in.nextInt();
        ArrayList<myPair> a = new ArrayList<myPair>();
        for(int i=0;i<n;i++){
            StringBuffer name = new StringBuffer(in.next());
            StringBuffer passwd = new StringBuffer(in.next());
            boolean modi = false;
            for(int j=0;j<passwd.length();j++){
                switch (passwd.charAt(j)){
                    case '1':{
                        passwd.replace(j,j+1,"@");
                        modi = true;
                        break;
                    }
                    case '0':{
                        passwd.replace(j,j+1,"%");
                        modi = true;
                        break;
                    }
                    case 'l':{
                        passwd.replace(j,j+1,"L");
                        modi = true;
                        break;
                    }
                    case 'O':{
                        passwd.replace(j,j+1,"o");
                        modi = true;
                        break;
                    }
                }
            }
            if(modi)
                a.add(new myPair(name.toString(), passwd.toString()));
        }

        if(a.size() > 0){
            System.out.println(a.size());
            for(int i=0;i<a.size();i++)
                System.out.println(a.get(i).name+" "+a.get(i).passwd);
        }
        else if(n == 1)
            System.out.println("There is 1 account and no account is modified");
        else if(n>0)
            System.out.println("There are " + n + " accounts and no account is modified");
    }
}
```



### 1036 Boys vs Girls

题意：找出分数最高的女孩和分数最低的男孩

思路：一道无聊的排序题

```java
public class Main {
    public static void main(String[] args) {
        FastScanner in = new FastScanner(System.in);
        String name, gender, id;
        int grade, n ;
        n = in.nextInt();
        ArrayList<Student> male = new ArrayList<>();
        ArrayList<Student> female = new ArrayList<>();
        for(int i=0;i<n;i++){
            name=in.next(); gender=in.next(); id = in.next(); grade=in.nextInt();
            Student a = new Student(name, gender, id, grade);
            if(gender.equals("M"))
                male.add(a);
            else
                female.add(a);
        }
        Collections.sort(male);
        Collections.sort(female);

        int a=-1, b=-1;
        if(!female.isEmpty()) {
            System.out.println(female.get(female.size() - 1).name + " " + female.get(female.size() - 1).ID);
            a = female.get(female.size()-1).grade;
        }
        else
            System.out.println("Absent");
        if(!male.isEmpty()) {
            System.out.println(male.get(0).name + " " + male.get(0).ID);
            b = male.get(0).grade;
        }
        else
            System.out.println("Absent");

        if(a==-1 || b==-1)
            System.out.println("NA");
        else
            System.out.println(a-b);

    }
}
class Student implements Comparable<Student>{
    public String name, gender, ID;
    public int grade;
    Student(String a, String b, String c, int d){
        this.name = a;
        this.gender= b;
        this.ID = c;
        this.grade = d;
    }
    @Override
    public int compareTo(Student o) {
        return this.grade-o.grade;
    }
}
```



### 1037 Magic Coupon

题意：给两个数组，可以每次从两个数组里分别拿两个数字出来，每个数字只能拿一次，求它们的乘积的最大和

思路：双指针

```c++
int nc,nb, a[MAXN], b[MAXN];
ll sum = 0;
int main() {
    scanf("%d",&nc);
    for(int i=0;i<nc;i++) scanf("%d",a+i);
    scanf("%d", &nb);
    for(int i=0;i<nb;i++) scanf("%d", b+i);
    sort(a,a+nc);
    sort(b,b+nb);
    int al=0,bl=0,ar=nc-1,br=nb-1;
    while(al<=ar && bl<=br){
        ll L= a[al]*b[bl],  R = a[ar]*b[br];
        if(R >= L && R > 0){
            sum += R;
            ar--;br--;
        }
        else if(L > R && L > 0){
            sum += L;
            al++;bl++;
        }
        else
            break;
    }
    cout<<sum<<endl;
}
```

