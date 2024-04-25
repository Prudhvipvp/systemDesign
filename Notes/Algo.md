
&nbsp;

Stack =

```
Stack<Integer> stack = new Stack<Integer>();
stack.push();
stack.pop();
stack.peek(); -> just get , no pop
stack.empty() -> isEmpty
```

Heap / Priority Queue -

```
PriorityQueue<string>pq = new PriorityQueue<>();  
pq.add()  
pq.remove()  
pq.peek()

The default order is ascending - MinHeap

To make a maxHeap -  
PriorityQueue<Integer>maxHeap = new PriorityQueue<>(Collections.reverseOrder());

Creating Heap with some class Txn(int amount, String id)

PriorityQueue<Txn>txnPriorityQueue = new PriorityQueue<>(Comparator.comparingDouble(Txn::getAmount));

and similar maxHeap would be  
PriorityQueue<Txn>maxHeap = new PriorityQueue<>(Comparator.comparingDouble(Txn::getAmount).reversed());
```

BFS -

In problems, where u need to do multiple BFS from different roots  causing O(N2) complexity. Think of ways where we can add all those roots to the single bfs queue and do bfs on it -> completing with a single bfs and O(N)

```
Map<Integer,ArrayList<Integer>> map = buildMap(A);
Set<Integer> seen = new HashSet<Integer>();
int level = -1;
Queue<Integer> queue = new LinkedList<Integer>();
seen.add(B);
queue.add(B);
while(queue.size()!=0){
    int n = queue.size();
    for(int i = 0;i<n;i++){
        Integer head = queue.remove();
        for(Integer neighbour:map.get(head)){
            if(!seen.contains(neighbour)){
                seen.add(neighbour);
                queue.add(neighbour);
            }
        }
    }
    level+=1;
}
return level;
```

Java arraylist sort -  
`Collections.sort(A);`  
Java int\[\] sort - Arrays.sort(A);  
Read about min max heap

***Kadane's - Max Sum contiguous sub array and ending with each index -***

logic is maxEndTillNow keeps adding up with every index and thats the max value till that index i that can be obtained. Whenver at an index the val < 0. we reset it to 0 meaning that the subarray ending with that index cant give u any positive sum.

```
int maxSum = Integer.MIN_VALUE;
int maxEndTillNow = 0;
for(int i = 0;i<A.size();i++){
    maxEndTillNow+=A.get(i);
    maxSum = Math.max(maxSum,maxEndTillNow);
    if(maxEndTillNow < 0){
        maxEndTillNow = 0;
    }
}
return maxSum;
```

**Word Break Problem. - O(N2)**

```
public static boolean wordBreak(String s, List<String> dictionary) {
        // create a dp table to store results of subproblems
        // value of dp[i] will be true if string s can be segmented
        // into dictionary words from 0 to i.
        boolean[] dp = new boolean[s.length() + 1];
     
        // dp[0] is true because an empty string can always be segmented. 
        dp[0] = true;
     
        for(int i = 0; i <= s.length(); i++){
            for(int j = 0; j < i; j++){
                if(dp[j] && dictionary.contains(s.substring(j, i))){
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.length()];
    }
```

**Gas Station** - is Possible to make a circle by refilling using A\[i\] amount of gas and burning B\[i\] gas to reach from i to i+1. Find least possible station to start journey from?

```
    int gasIndex = 0;
    int n = A.size();
    int currGas = 0;
    int isPossible = 0;
    for(int i=0;i<n;i++){
        isPossible += A.get(i) - B.get(i);
        currGas+=A.get(i);
        if(currGas<B.get(i)){
            gasIndex = i+1;
            currGas=0;
        }
        else{
            currGas -= B.get(i);
        }
    }
    return isPossible >=0 ? gasIndex : -1;
```

Circle of Words -

```
The trick is instead of seeing each word as a vertex and edge as between two words. Just see the first letter and last letter of each word as 
vertices and edge as the word itself. Doing this way , is same as before also, but it simplies the solution(as there will be many edges to be considered in that case).

Now a circle will be formed by a graph iff 1. In degree = out degree for each vertex and graph should form a single connected component.

First check is easy, just traverse thru each word and do indegree[lastLetter]+=1 and out degree[firstLetter]+=1 . Now for any letter if indegree!=outdegree -> return false.

For single conencted component, start at any vertex, do dfs and at the end. If any unvisited vertex -> return false
```

&nbsp;

Move all negative to left and positives to right inplace in a array -

```
Two Pointer approach, left = 0, right = n-1
while(left<right){
if(left is -ve and right is +ve){
left+=1 and right-=1
}
else if(left is +ve nd right is -ve){
swap(left,right)
left+=1 and right-=1
}
else if(left is +ve and right is +ve){
right-=1
}
else{
left+=1
}
}
```

First Missing Positive - in a unsorted array containing negative , positive

```
public int firstMissingPositive(ArrayList<Integer> A) {
        int n = A.size();
        int maxVal = n+1;
        Boolean found = false;
        for(int i = 0;i<n;i++){
            if(A.get(i) == 1){
                found = true;
            }
           if(A.get(i) < 1 || A.get(i) > n){
                A.set(i,1);
            }
        }
        if(!found){

            return 1; // this is required specifically for 1, becoz we set all negatives as 1, so its presence will be marked falsely

        }
        for(int i = 0;i<n;i++){
                Integer shift = (A.get(i) % maxVal) -1 ;
                if(shift>=0)
                A.set(shift,A.get(shift)+maxVal);

        }
        for(int i = 0;i<n;i++){
            if(A.get(i) < maxVal){
                return i+1;
            }
        }
        return maxVal;
    }
```

Nearest Smallest Element before/Nearest Greater element after -

```
Logic is to maintain a stack in ascending order(for first case) and descending in second case

if u find currElement is lesser than currr stack top(for first case), pop till u find a greater top

```

Concentric Squares with outer most square value as A and goes on till 1

```
3 3 3 3 3
3 2 2 2 3
3 2 1 2 3
3 2 2 2 3
3 3 3 3 3

Given any index(x,y) - the value is
return Math.max(A-x, A-y, x-A+2, y-A+2);
```

**Distribute Candies** - u basically traverse from left to right and check for ascending order case - if arr\[i+1\]>arr\[i\]

and then u traverse from right to left and u see desceding order case if arr\[i\]>arr\[i+1\]. Note that it should be done in given below style only, if not, the end elements(arr\[0\] and arr\[n-1\] may get missed )

```
for(int i =0;i<n-1;i++){
    if(A.get(i+1)>A.get(i) && candies[i+1]<=candies[i]){
        candies[i+1] = candies[i]+1;
    }
}
for(int i = n-2;i>=0;i--){
    if(A.get(i)>A.get(i+1) && candies[i]<=candies[i+1]){
        candies[i] = candies[i+1]+1;
    }

}
```

## Smart Settle Splitwise Algo - using Greedy Algo

```
From the txns List, divide into two groups - Givers and Receivers  
Givers are the ones which has net positive cash(So they received money till now, to settle up , they have to give  
Similarly for Receivers

Now, create two max Heaps - giver heap and receiver heap(maintain absolute values in both case)`
finalTxnsList[]

while(heap.isNonempty()) - 
Loop till either heap is non empty(empty means - all the txns are settled. 
So in each iteration we are finding that one optimal txn and adding it to our list
{
maxGiver = giverHeap.remove();
maxReceiver = receiverHeap.remove();
int min = Min(maxGiver, maxReceiver);
maxGiver-=min;
maxReceiver+=min;
finalTxnsList.add(from=maxGiver's id, to = maxReceiver'sid, amount = min)
if(maxGiver != 0){ if its 0 , it means this is settled
giverHeap.add(maxGiver)
}
if(maxReceiver!=0){
receiverHeap.add(maxReceiver);
}
}
return finalTxnsList;
```

Find max points lying on a single straight Line -

```
Simply find slope between every pair of points(O(n2))
Note - skip finding slope between same point twice
     - In slope , if denominator is 0, make slope = MAX_INT 
Maintain a hashmap of slopes and frequencies, The max frequency + 1 is the count of max points on a line.

```

Sub arrays with atmost K distinct elements -

```
HashMap<Integer, Integer> map = new HashMap<>();
// Loop to calculate the count
while (right < n) {
    map.put(arr[right],
            map.getOrDefault(arr[right], 0) + 1);
    // Shrinking the window from left if the
    // count of distinct elements exceeds K
    while (map.size() > k) {
        map.put(arr[left], map.get(arr[left]) - 1);
        if (map.get(arr[left]) == 0)
            map.remove(arr[left]);
        left++;
    }
    // Adding the count of subarrays with at most
    // K distinct elements in the current window
    (i.e. num of subarrays ending with right)
    count += right - left + 1;
    right++;
}
return count;
```

This F can be used to get subarrays with exact K distinct elements - F(K) - F(K-1)

&nbsp;

Infinite coin Sum -

```
int [] dp = new int[B+1];
dp[0] = 1;
int mod = 1000007;
for(int i = 0;i<A.size();i++){
    for(int j = A.get(i);j<=B;j++){
        dp[j] = (dp[j] % mod + dp[j-A.get(i)] % mod)%mod;
    }
}
return dp[B];
```

3 SUM -

Sort the array, fix one element arr\[i\] , make s = i+1 and e =n-1 and start doing two ptr searching for targetsum

Random num in a range - Math.random() gives rand num 0<= x < 1

public int getRandomNumber(int min, int max) {  
return (int) ((Math.random() \* (max - min)) + min);  
}

Minstack - getMinStack() in constant time and space - 2x - minEle and 2\*minEle - x

```
maintain minEle
While pushing x
if(x>=minEle){
push x
}else{
push 2*x - minEle.  (Note that 2x - minEle < minEle)
minEle = x
}
While popping - 
if(top >= minEle){
pop
}else{
x = pop();
minEle = 2*minEle - x; (since its equal to 2*oldMin - 2*oldMin + newMin)
}

```

Max Pair

```
Max N Pair Combinations - 
Given two arrays of size N. Find N maximum pairs from both arrays.

Sort both arrays
Keep a maxHeap<Int,<Int,Int>> and a set<Int,Int>
initially , add maxHeap.add(a[-1]+b[-1], (n-1,n-1))
Now while(n--){
obj o = maxHeap.pop());
arr.add(o.first);
l = o.second.first;
r = o.second.second;
if((l-1,r) is not in set){
maxHeap.add(a[l-1]+b[r],(l-1,r)
}
if((l,r-1) is not in set){
maxHeap.add(a[l]+b[r-1],(l,r-1))
}
}
return arr;
```

Remove Duplicates 2 - Given a sorted array, remove the duplicates in place such that each element can appear atmost twice

```java
public int removeDuplicates(ArrayList<Integer> nums) {
        int i = 0;
        for (final int num : nums)
        if (i < 2 || num > nums.get(i - 2)) {
            nums.set(i,num);
            i+=1;
        }
        return i;
}
```

Best time to buy and sell stocks -2

Buy and sell any number of times. Note that the max transactions will anyways be N/2 (since a txn is a buy+sell)

The logic is whenever price increases sell it. So -

```
public int maxProfit(final List<Integer> A) {
        int profit = 0;
        for(int i = 1;i<A.size();i++){
            if(A.get(i) >A.get(i-1)){
                profit +=A.get(i) - A.get(i-1);
            }
        }
        return profit;
    }
}
```

Buy and sell stocks atmost B times =

```
public int solve(ArrayList<Integer> A, int B) {
        int n = A.size();
        if(B >= n/2){
        //unlimited times
            int profit = 0;
            for(int i =1;i<n;i++){
                if(A.get(i) >A.get(i-1)){
                    profit+=A.get(i)-A.get(i-1);
                }
            }
            return profit;
        }
        int dp[][] = new int[B+1][n]; //max profit with atmost B txn and till n index
        for(int i = 1;i<=B;i++){
            int localMax = dp[i-1][0]-A.get(0); // or simply -A.get(0) as always the first element we will buy
            //the localMax is basically - Max(dp[i-1][k]-A.get(k) where 0<=k<j))- so instead of an other loop we are keeping track of it
            for(int j=1;j<n;j++){
                dp[i][j] = Math.max(dp[i][j-1], A.get(j)+localMax);
                localMax = Math.max(localMax, dp[i-1][j] - A.get(j));
            }
        }
        return dp[B][n-1];
    }
```

Number of ways to (1,1) to (n,m) -

Total down moves = n-1 and total right moves = m-1 -> net = m+n-2

Now we need all combinations of these m+n-2 moves. Consider m+n-2 seats and now you need to select any m-1(/n-1) seats. the rest seats will automatically be the left over ones. So - total ways - m+n-2 C n-1

Time complexity of backtracking is generally loosely bound - and O(b^d) where b is branching factor. i.e. how many decisions we take at each point. In above case, we have 2 decisions right/down, so b = 2. D is the max depth we would go to while taking decisions. Here its m+n-2. So O(2^m+n-2). note that its a loose bound, the more tighter bound is hard to calculate.

Important -

```
Keshi is throwing a party and he wants everybody in the party to be happy.

He has 𝑛 friends. His 𝑖-th friend has 𝑖 dollars.

If you invite the 𝑖-th friend to the party, he will be happy only if at most 𝑎𝑖
 people in the party are strictly richer than him and at most 𝑏𝑖
 people are strictly poorer than him.

Keshi wants to invite as many people as possible. Find the maximum number of people he can invite to the party so that every invited person would be happy.
```

The solution is greedy binary search.

Consider a set { P1,P2,P3....PX} -> A set of X persons in order of their wealth. (The wealth is 'i' as told in problem - so this set need not just be 0,1,2,3,4,5.. coz say for 2th index the given condition didnt match - so it can be of - 1,5,7,9....)

Now lets say this set X all are happy. Take any person Pi. Let the original index of this person in set be v.

So the condition is b\[v\] >= i-1 ( i-1 are the people poorer than him) and  a\[v\] >= x-i

now merge both these conditions to get -> **x - a\[v\] <= i <= b\[v\] + 1**

**So, for each person in the set this condition should hold true. Let there be function allHappy(x) - which tells if x sized set can be made with all of them happy.**

**Note that this function is monotonic. So if we find the maximum x which is the answer. There wont be any x > ans (since ans is maximum) and for all x < ans , happy(x) will be true.**

**So, do binary search to get max X.**

Now problem is how to find allHappy(x) -> use greedy approach .

so as shown, we start searching for the first person i=1, and as soon as we find him, we increment i and start searching for P2

this will work becoz the given input persons are in order of their wealth(i is wealth). so each i+=1 is giving you a person richer than previous i . thats why your formed set is in order(can be something like 2,3,6,9,12,...)

```java
bool allHappy(int x){
    int i =1;
    for(int v=0;v<n;v++){
        if( x - a[v] <= i && i <= b[v] +1){
            //then we found the first entry in our set of {P1,P2,P3...PX)
            i+=1;
        }
    }
    return i-1>=x; (i-1 becoz we started index from i=1
}

int binSearch(int[] a, int[] b, int n){
    int hi = n; int lo = 1;
    int ans = 1;
    while(lo<=hi){
        mid = (lo+hi )/2
        if(allHappy(mid)){
            ans=mid;
            lo = mid+1;
        }
        else{
            hi=mid-1;
        }
    return ans;
}
```

Find cycle in a directed graph using DFS. Logic is cycle exists if theres a backedge - i.e.

```java
private boolean isCyclicUtil(int i, boolean[] visited,
                                 boolean[] recStack)
    {
 
        // Mark the current node as visited and
        // part of recursion stack
        if (recStack[i]) //means this node is present in current recursion stack of dfs
            return true;
 
        if (visited[i]) // means this node is already visited(may be by som other path), so no need to go again
            return false;
 
        visited[i] = true;
 
        recStack[i] = true;
        List<Integer> children = adj.get(i);
 
        for (Integer c : children)
            if (isCyclicUtil(c, visited, recStack))
                return true;
 
        recStack[i] = false;
 
        return false;
    }
```

&nbsp;Topological sort -

Its only possible for a DAG. And its a order such that in that order if u comes before v, then theres a edge from U to V.

So, the first vertex in topo sort will be the node with no incoming edge onto it.

```
static void findTopoSort(int node, int vis[], ArrayList<ArrayList<Integer>> adj, Stack<Integer> st) {
        vis[node] = 1; 
        for(Integer it: adj.get(node)) {
            if(vis[it] == 0) {
                findTopoSort(it, vis, adj, st); 
            } 
        }
        st.push(node); 
    }
    static int[] topoSort(int N, ArrayList<ArrayList<Integer>> adj) {
        Stack<Integer> st = new Stack<Integer>(); 
        int vis[] = new int[N]; 
        
        for(int i = 0;i<N;i++) {
            if(vis[i] == 0) {
                findTopoSort(i, vis, adj, st);
            }
        }
        
        int topo[] = new int[N];
        int ind = 0; 
        while(!st.isEmpty()) {
            topo[ind++] = st.pop();
        }
        // for(int i = 0;i<N;i++) System.out.println(topo[i] + " "); 
        return topo; 
    }
```

Example -  given a set of courses and each course might have a prereq course. Find order of courses completion. So each prereq becomes a directed edge. If theres no cycle , means courses can be completed using topo sort order.

You can also check cycle using topo sort. After topo sort is done and you have the stack as given above . Each popped element from the stack will give in order of topo i.e. parents first and then children(since edge is from par to child). Now if theres a cycle, there exists atleast one case in our stack, such that child comes before parent. To check that -

```
private boolean checkCycle(Stack<Intger> s, int n, List adj){
   Map<Integer, Integer> pos = new HashMap<>();
     
    int ind = 0;
     
    // Pop all elements from stack
    while (!s.isEmpty()) 
    {
        pos.put(s.peek(), ind); // putting the node and the count
        ind += 1;
        s.pop();
    }
 
    for(int i = 0; i < n; i++) 
    {
        for(Integer it : adj.get(i))
        {   
            // If parent vertex
            // does not appear first
            if (pos.get(i) > pos.get(it))
            {    
                // Cycle exists
                return true;
            }
        }
    }
    // Return false if cycle
    // does not exist
    return false;
```

max avg subtree =

```
private double ans;

    public double maximumAverageSubtree(TreeNode root) {
        dfs(root);
        return ans;
    }

    private int[] dfs(TreeNode root) {
        if (root == null) {
            return new int[2];
        }
        var l = dfs(root.left);
        var r = dfs(root.right);
        int s = root.val + l[0] + r[0];
        int n = 1 + l[1] + r[1];
        ans = Math.max(ans, s * 1.0 / n);
        re
```

Minimum iterations to pass info to all nodes - https://www.geeksforgeeks.org/minimum-iterations-pass-information-nodes-tree/

The logic is for root r -> it has k children. By recursion get minItr\[\] for each of the k children. minItr\[i\] gives min iterations starting from node i.

Now sort this minItr for these k children in descending order. (Descending coz we need minimum, so first we will iterate thru the child which is of max depth i.e. max minItr\[i\] value)

Iterate thru it  using i and count = 0 and minItr\[currNode\] = Math.max(minItr\[currNode\], minItr\[i\] + count+1); count+=1

```
public int minIterations(TreeNode root, int n){

int[] minItr = new int[n];
dfs(0, minItr);
return minItr[0];
}

private void dfs(int root, int[] minItr) { //
    int numChildren = graph[root].size(); 
    minItr[root] = numChildren //intialising it to min value
    int[] minItrChild = new int[numChildren];
    int k = 0;
    for(int i:graph[root]){
        dfs(i,minItr);
        minItrChild[k] = minItr[i]; //the above dfs call has populated this
    }
    Sort(minItrChild,-1);
    for(int i -> 0 to graph[root].size()){
        minItr[root] = Math.max(minItr[root],minItrChild[i] + i+1);
    }

}
```

Median for stream -

```
private PriorityQueue<Integer> small = new PriorityQueue<>(Collections.reverseOrder());
private PriorityQueue<Integer> large = new PriorityQueue<>();
private boolean even = true;

public double findMedian() {
    if (even)
        return (small.peek() + large.peek()) / 2.0;
    else
        return small.peek();
}

public void addNum(int num) {
    if (even) {
        large.offer(num);
        small.offer(large.poll());
    } else {
        small.offer(num);
        large.offer(small.poll());
    }
    even = !even;
}
```