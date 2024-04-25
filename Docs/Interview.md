

Interview

Bin search PS -

```
You are a product manager and currently leading a team to develop a new product. Unfortunately, the latest version of your product fails the quality check. Since each version is developed based on the previous version, all the versions after a bad version are also bad.

Suppose you have n versions [1, 2, ..., n] and you want to find out the first bad one, which causes all the following ones to be bad.

You are given an API/method bool isBadVersion(version) which returns whether version is bad. Implement a function to find the first bad version. You should minimize the number of calls to the API.

total versions = n i.e. (1 to n)

Example - 
Input: n = 5
Output: 4
Explanation:
call isBadVersion(1) -> false
call isBadVersion(2) -> false
call isBadVersion(3) -> false
call isBadVersion(4) -> true
call isBadVersion(5) -> true

Then 4 is the first bad version.

 public int firstBadVersion(int n) {
        
 }
```

Solution

```
        int left = 1, right = n, ans = -1;
        while (left <= right) {
            int mid = left + (right - left) / 2; // to avoid overflow incase (left+right)>2147483647
            if (isBadVersion(mid)) {
                ans = mid; // record mid as current answer
                right = mid - 1; // try to find smaller version in the left side
            } else {
                left = mid + 1; // try to find in the right side
            }
        }
        return ans;
```

&nbsp;

2.  Bin search

```
An array arr is a mountain if the following properties hold:

arr.length >= 3
There exists some i with 0 < i < arr.length - 1 such that:
arr[0] < arr[1] < ... < arr[i - 1] < arr[i] 
arr[i] > arr[i + 1] > ... > arr[arr.length - 1]
Given a mountain array arr, return the index i such that arr[0] < arr[1] < ... < arr[i - 1] < arr[i] > arr[i + 1] > ... > arr[arr.length - 1].

Example - 
Input: arr = [0,1,0]
Output: 1

Example 2-
Input: arr = [0,2,1,0]
Output: 1

Example 3 -
Input: arr = [0,10,5,2]
Output: 1

public int indexInMountainArray(int[] arr) {
}
```

Solution -

```
public int indexInMountainArray(int[] arr) {
        int left = 0;
        int right = arr.length-1;
        while(left <= right) {
            int mid = left + (right-left)/2;
            if(arr[mid]<arr[mid+1]) {
                left = mid+1;
            }
            else {
                right = mid-1;
            }
        }
        return left;
    }
}
```

3\. Bin search -

```
A kid is counting numbers from 0 to n and writing them in his notes. When his mother checked his notes, she found out that he missed a number. Find the missing number .

public int findMissingNumber(int[] nums) {}
Ex -
[0,1,3]
Ans - 2
[0,1,2,3,4,5,6,8]
Ans - 7

```

Solution -

```
public int missingNumber(int[] nums) {
int lo = 0, hi = nums.length - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] == mid) {
        lo = mid + 1;
    } else {
        hi = mid - 1;
    }
}

return lo;
}
```

&nbsp;

Medium. - Trees and stack -

1.  Maximum width of binary Tree -

```
Given the root of a binary tree, return the maximum width of the given tree.

The maximum width of a tree is the maximum width among all levels.

The width of one level is defined as the length between the end-nodes (the leftmost and rightmost non-null nodes), only non null nodes have to be considered to get the width

Ex - 
   	 1
        / \ 
       2  3
          /
         4
          \
           5
   
 Ans = 2
 
 public class TreeNode {
      int val;
      TreeNode left;
      TreeNode right;
      TreeNode() {}
      TreeNode(int val) { this.val = val; }
      TreeNode(int val, TreeNode left, TreeNode right) {
          this.val = val;
          this.left = left;
          this.right = right;
      }
  }

public int maxWidth(TreeNode root) {}
```

2\. Max Root to Leaf Integer -

```
You are given the root of a binary tree containing digits from 0 to 9 only.

Each root-to-leaf path in the tree represents a number.

For example, the root-to-leaf path 1 -> 2 -> 3 represents the number 123.
Return the maximum among all root-to-leaf numbers. All paths will fit inside a 32 bit integer

A leaf node is a node with no children.

Note that - only root to leaf paths are to be considered. If the root doesnt have any children, then no path is found.(root is not leaf)

Ex - 
         4
        / \ 
       9   0
      /  \
     5   1
     
    Root to leaf paths -  495, 491, 40  -> Max 495
```

&nbsp;

3.  Price after discounts -

```
You are given an integer array prices where prices[i] is the price of the ith item in a shop.

There is a special discount for items in the shop. If you buy the ith item, then you will receive a discount equivalent to prices[j] where j is the maximum possible index such that j < i and prices[j] < prices[i]. If there is no such index, no discount.

Return an integer array answer where answer[i] is the final price you will pay for the ith item of the shop, considering the special discount.

public int[] finalPrices(int[] prices) {}

Ex - 
prices = [4, 5, 2, 10, 8]
Output: [4,1,2,8,6]
Explanation - 
    index 1: No element less than 4 in left of 4, No discount
    index 2:4 is only element less than 5, discount = 4
    index 3: No element less than 2 in left of 2, No discount
    index 4: 2 is nearest element which is less than 10, discount = 2
    index 4: 2 is nearest element which is less than 8, discount = 2
```

Solution -

```
public class Solution {
    public ArrayList<Integer> prevSmaller(ArrayList<Integer> arr) {
        ArrayList<Integer> retval=new ArrayList<>();
        Stack<Integer> st=new Stack<Integer>();
        
        for(int i=0;i<arr.size();++i){
            while(!st.isEmpty() && arr.get(i)<=st.peek()){
                st.pop();
            }
            if(st.isEmpty()){
                retval.add(arr.get(i));
            }
            else{
                retval.add(arr.get(i) - st.peek());
            }
            st.add(arr.get(i));
        }
        
        return retval;
    }
}
```