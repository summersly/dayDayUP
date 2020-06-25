### 剑指offer

#### 输入输出

```java
import java.util.Scanner;
public void Main{
  public static void main(String[] args){
    Scanner in = new Scanner(System.in);
    while(in.hasNextInt()){
      int a = in.nextInt();
      int b = in.nextInt();
    
    String str = sc.nextLine();
    String[] strings = str.split(" ");
    //转为整数数组
    int[] ints = new int[strings.length];
    for (int i = 0; i < strings.length; i++) {
       ints[i] = Integer.parseInt(strings[i]);
    }
    int k = sc.nextInt();
  }
}
```

#### T1 单例-2

```java
// 饿汉
private static Singleton instance = new Singleton();
public static Singleton getInstance(){
  return this.instance;
}
// 双重锁 
private volatile static Singleton instance = null;
public static Singleton getInstance(){
  if(instance == null){
    synchronized(Singleton.class){
      if(instance == null){
        instance = new Singleton();
      }
    }
  }
  return instance;
}
```

#### T3 数组中的重复数字-6

在回答问题前，要问清，是找出全部的重复数字还是找到任意一个。时间空间要求是怎么样？

查找重复数字的思路整理：

+ 哈希：使用了额外的空间
+ 求和：只能在确定重复个数和次数的特殊情况使用
+ 抽屉原理：边扫描边排序，把所有元素放在他的下标处，遇到位置被占用了就是答案，改变了原数组
+ 位运算：元素重复偶数次的时候可以使用，找到奇数次的数字
+ 二分：把区间分成两半，总有一半区间中数字个数超标了（因为重复）
+ 快慢指针：可怕的技巧



1. 先排序（nlogn），再从头到尾扫描（n）；可以找到其中一个重复

   ```java
   Arrays.sort(nums);
   for (int i = 1; i < nums.length; i++) {
       if (nums[i] == nums[i-1]) {
           return nums[i];
       }
   }
   return -1;
   ```
   
2. 哈希表存储（n），空间（n）；如果遍历完就能找到其他重复

   ```java
   Set<Integer> seen = new HashSet<Integer>();
   for (int num : nums) {
       if (seen.contains(num)) {
           return num;
       }
       seen.add(num);
   }
   return -1;
   ```
   
3. 边扫描边排序，0-n-1的数字，不重复则会出现在对应的下标位置；时间（n），空间（1）

   ```java
   int i = 0;
   // 只要第i为数字不是i，就一直交换，直到出现重复数字
   while(i < nums.length) {
   	if(nums[i] == i) {
     i++;
     continue;
   	}
   	if(nums[nums[i]] == nums[i]) return nums[i];
   	int tmp = nums[i];
   	nums[i] = nums[tmp];
   	nums[tmp] = tmp;
   }
   return -1; // 边界情况：数组长度0，1或者不包含重复数字的情况
   ```

4. 不修改原数组的情况：二分法查找重复数字，调用countRange得到符合二分区间的数值个数，调用（logn），每次（n），总时间（nlogn），空间（1）

   ```java
   int start = 1, end = nums.length-1;
   while(end>start){
     int mid = (end+start)/2;
     //
     int count = 0;
     for(int i = 0; i < nums.length; i++){
         if(nums[i]>=start && nums[i]<=mid) count++;
     }
     if(count>(mid-start+1)) end = mid;
     else start = mid+1;
   }
   return start;
   ```

5. 快慢指针：先通过快慢指针找到开始循环（即相同数字的地方），然后从头用一个慢指针，找到♻️的进入口。

   ```java
   // 快慢指针
   int kuai = nums[0], man = nums[0];
   do{
       man = nums[man];
       kuai = nums[nums[kuai]];
   } while(kuai!=man);
   int i = nums[0];
   while(i!=man){
       i = nums[i];
       man = nums[man];
   }
   return i;
   ```

   

#### T4 二维数组中的查找

1. 数组有序，从左到右，从上到下递增。从右上角开始检查，逐渐缩小范围：

   + 若右上角大于目标，可以剔除 `列`

   + 若等于目标，返回

   + 若右上角小于目标，可以剔除 `行` 

     ```java
     int i = 0, j = matrix[0].length-1;
     while (i < matrix.length && j >=0){
     	if(matrix[i][j]==target) return true;
     	else if(matrix[i][j]>target) j--;
     	else i++;
     }
     return false;
     ```



#### T5 替换字符串中的内容

从后往前遍历，这样可以减少挪动的次数。类似的还有合并两个数组。



#### T6从尾到头打印链表 & 18/22/24/25/52/62/36/35

在不能改变链表结构的情况下：**栈或者递归**，能用栈就少用递归，因为函数调用层级太多可能会导致溢出。

```java
LinkedList<ListNode> stack = new LinkedList<>();
int count = 0;
while(head != null){
	stack.push(head);
	head = head.next;
	count++;
}
int[] res = new int[count];
int k = 0;
while(!stack.isEmpty()){
	res[k++] = stack.pop().val;
}
return res;
```



#### T7 重建二叉树 & 36/68

```java
// 前序遍历第一个节点就是root，第二个节点是左子树的root，第三个是左子树的左子树的root。。。。。
// 遍历map（n），每个节点new （n），平均情况下递归深度（log2n）
// 空间map的（n）
List<Integer> inorder;
public TreeNode buildTree(int[] preorder, int[] inorder){
  this.preorder = preorder;
  this.inorder = Arrays.asList(inorder);//不好用的 error
  return help(0,inorder.length);
}
// java中的array没有indexOf！！！所以还是用map吧
private TreeNode help (int inStart, int inEnd){
  if(inStart == inEnd) return null;
  int rootIndex = inorder.indexOf(preorder[preIndex]);
  TreeNode root = new TreeNode(preorder[preIndex++]);
  root.left = help(inStart, rootIndex);
  root.right = help(rootIndex+1, inEnd);
  return root;
}
```



#### T8 二叉树的下一个节点

给定一个二叉树和其中的一个节点，求这个节点中序遍历下的下一个节点（左右孩子和父节点已知）

```java
// 已知parent的情况下 左 中 右
// 先找右子树的左下节点
// 没有右子树
// 再往上找，直到自己是父节点的左子树
public TreeNode getNext(TreeNode root){
  if(root == null) return null;
  if(root.right != null){
    root = root.right;
    while(root.left!=null) root = root.left;
    return root;
  }
  while(root.parent != null){
    if(root == root.parent.left) return root.parent;
    else root = root.parent;
  }
  return null;
}
```



#### T9 用两个栈实现队列

两个栈实现队列，一个栈负责新插入的元素，另一个栈用来转换顺序

插入数据，放入A的顶

删除数据，看B还有没有的剩，有则直接返回B顶；否则把A的元素全倒进B，进行倒序。

```java
private Stack<Integer> A;
private Stack<Integer> B;
public CQueue() {
    A = new Stack<>();
    B = new Stack<>();
}
public void appendTail(int value){
  A.push(value);
}
public int deleteHead(){
  if(!B.empty()) return B.pop();
  while(!A.empty()) B.push(A.pop());
  if(B.empty()) return -1;
  return B.pop();
}
```



#### T10 斐波那契数列

**二维数组搜索路径，适合用回溯（递归），不能用回溯的时候，用栈模拟递归**

**某个问题的最优解，将一个问题划分为多个子问题，用动态规划：先自上而下的写出规划方程，再自下而上的循环实现 -------- 贪婪？？**

```java
// （n）
if(n == 0) return 0;
if(n == 1) return 1;
int one = 0, two = 1;
for(int i = 2; i <=n ; i++){
  int sum = one + two; // 取模：两个数分别取模之后相加再取模 等价于 先相加再取模
  one = two;
  two = sum;
}
return two;
//（logn）
// 太难了要数学的
```

#### 查找和排序！& 39/40

顺序查找，二分查找，哈希表查找，二叉搜索树查找

插入排序、冒泡排序、归并排序、快速排序的比较！从额外空间消耗、平均时间复杂度、最差时间复杂度三方面来比较。

交换成本高的情况下，可以用选择排序：选择排序的交换次数最少；

插入排序是稳定的，在数组接近有序的情况下使用，表现最优，短数组也可以选择插入排序；



|                  | 平均时间复杂度 | 最差时间复杂度 | 空间复杂度 |
| ---------------- | -------------- | -------------- | ---------- |
| 冒泡排序（交换） | $$n^2$$        | $$n^2$$        | 1          |
| 快速排序（交换） | $$nlogn$$      | $$n^2$$        | $$nlogn$$  |
| 插入排序（插入） | $$n^2$$        | $$n^2$$        | 1          |
| 归并排序（二路） | $$nlogn$$      | $$nlogn$$      | n          |

```java
// 冒泡模版 从小到大
private void bubbleSort(int[] nums){
  for(int i = nums.length-1; i >= 1; i--){
    for(int j = 0 ; j < i; j++){
      if(nums[j]>nums[j+1]) swap(nums, j, j+1);
    }
  }
}
// 带标记的冒泡优化
private static void bubbleSort2(int[] nums){
  boolean change = false;
  for(int i = nums.length-1; i >= 1; i--){
      for(int j = 0 ; j < i; j++){
          if(nums[j]>nums[j+1]) {swap(nums, j, j+1); change = true;}
      }
      if(!change) break;
  }
}
// 标记冒泡尾巴的优化
private static void bubbleSort3(int[] nums){
  int i ,j;
  int flag = nums.length-1;
  while(flag>0){
      i = flag;
      flag = 0;
      for(j = 0; j < i; j++){
          if(nums[j]>nums[j+1]) {
              swap(nums, j, j+1); 
              flag = j+1;
          }
      }
  }        
}
```

```java
// 快速排序
private void quickSort(int[] nums, int start, int end){
  if(start = end) return;
  int pivotIndex = partition(nums, start, end);
  if(pivotIndex > start) quickSort(nums, start, pivotIndex-1);
  if(pivotIndex < end) quickSort(nums, pivotIndex+1, end);
}
private int partition(int[] nums, int start, int end){
  int pivot = nums[start];
  int lt = start +1;
  int gt = end;
  while(true){
    while(lt<=end&&nums[lt]<pivot) lt++;
    while(gt>start&&nums[gt]>pivot) gt--;
    if(lt>gt) break;
    swap(nums, lt, gt);
    lt++;
    gt--;
  }
  swap(nums, start, gt);
  return gt;
}
```

```java
// 插入排序
private void insertSort(int[] nums){
  for(int i = 1; j < nums.length; j++){
    int tmp = nums[i];
    int j = i;
    while(j > 0 && nums[j-1] > tmp){
      nums[j] = nums[j-1];
      j--;
    }
    nums[j] = tmp;
  }
}
```

```java
// 归并排序
mergeSort(nums, 0, len - 1);
private void mergeSort(int[] nums, int start, int end){
  int mid = (start+end)>>>1;
  mergeSort(nums, start, mid);
  mergeSort(nums, mid+1, end);
  if(nums[mid] <= nums[mid+1]) return;
  mergeTwo(nums, start, mid, end);
}

private void mergeTwo(int[] nums, int start, int mid, int end){
  int[] tmp = new int[nums.length];
  System.arraycopy(nums, start, tmp, start, end-start+1);
  int i = start, j = mid+1;
  for(int k = start; k <= end; k+){
    if(i == mid+1) nums[k] = tmp[j++]; // 数组1 到底了
    else if(j == end+1) nums[k] = tmp[i++];
    else if(tmp[i]<=tmp[j]) nums[k] = tmp[i++];
    else nums[k] = nums[j++];
  }
}
```

```java
// 选择排序
// selectSort
private static void selectSort(int[] nums){
    for(int i = 0; i < nums.length-1; i++){
        int max = i, j;
        for(j = i+1; j < nums.length; j++){
            if(nums[j] > nums[max]) max = j;
        }
        int tmp = nums[max];
        nums[max] = nums[i];
        nums[i] = tmp;
    }
}
```



#### T11 旋转数组的最小数字

**在有序数组中查找或者统计数字出现的次数，都可以用二分查找**

```java
// 二分查找
// 二分查找场景：寻找一个数，寻找左侧边界，寻找右侧边界
// 初始条件【0,len-1】--》while(left<=right)
// 初始条件【0,len】--> while(left<right)
// 模版 
int binarySearch(int[] nums, int target) {
    int left = 0; 
    int right = nums.length - 1; // 注意
    while(left <= right) {
        int mid = (right + left) / 2;
        if(nums[mid] == target)
            return mid; 
        else if (nums[mid] < target)
            left = mid + 1; 
        else if (nums[mid] > target)
            right = mid - 1;
    }
    return -1;
}
// 左边界/右边界 初始化[0, len) 左闭右开
// while(left<right) left = mid+1; right = mid 这三个都不变，只改变相等情况下的收敛方向
// 左边界：需要定位到左边，所以相等的时候改变 右界
int binarySearchLeft(int[] nums, int target){
  	int left = 0;
  	int right = nums.length;
  	while(left<right){
      int mid = (left + end)>>>1;
      if(nums[mid] < target) left = mid + 1;
      else right = mid;
    }
    return left; //出循环的时候，left和right相等，但是左闭右开所以搜索区间内没有数据
  // 这个数不存在的时候，左边界返回的是第一个大于target的位置
}
// 右边界：需要定位到右边，所以相等的时候改变 左界
int binarySearchRight(int[] nums, int target){
  	int left = 0;
  	int right = nums.length;
  	while(left<right){
      int mid = (left + end)>>>1;
      if(nums[mid] <= target) left = mid + 1;
      else right = mid;
    }
    return left; // 此时返回的是第一个大于target的位置（存在或者不存在都一样）
  // 如果需要返回确切的右边界，则 return left-1
 
 // 综上，如果要同时查找左右边界，可以查找target的右边界（值为target+1）和target-1的右边界（值为target），两个直接相减就是target的数量
}
```

```java
int i = 0, j = numbers.length - 1;
while (i < j) { 
  //这里不用i<=j ，是因为最小元素必存在，最后这一个数值，不需要再比较了，可以直接退出循环
  int m = (i + j) / 2;
  if (numbers[m] > numbers[j]) i = m + 1;
  else if (numbers[m] < numbers[j]) j = m;
  else j--; 
  // j--可能会丢失旋转点，111231，但是去掉这个旋转点后剩下的数列中仍有和它相等的值存在，返回的答案还是一样的
}
return numbers[i];
```



#### T12 矩阵中的路径

**回溯法：暴力的优化，一个问题可以分步骤求解的时候，适用该方法；例如，在二维矩阵上寻找路径**

```java
public boolean exist(char[][] board, String word) {
  if(board.length<=0||board[0].length<=0) return false;
  if(word.length()<=0) return true;
  char[] words = word.toCharArray();
  for(int i=0; i<board.length; i++){
    for(int j=0; j<board[0].length; j++){
      if(dfs()) return true;
    }
  }
  return false;
}
private boolean dfs(char[][] board, char[] word, int i, int j, int k){
  if(i<0 || i>=board.length || j<0 || j>=board[0].length || board[i][j] != word[k]) 
    return false;
  if(k == word.length-1) return true;
  // 已经访问过的地方标记为‘#’ 表示不能访问第二次
  char tmp = board[i][j];
  board[i][j] = '#';
  boolean res = dfs(board, word, i+1, j, k+1) || dfs(board, word, i-1, j, k+1) || dfs(board, word, i, j+1, k+1)|| dfs(board, word, i, j-1, k+1);
  board[i][j] = tmp;
  return res;
}
```

#### T13 机器人的运动范围-2

```java
// DFS
private boolean[][] visited;
public int movingCount(int m, int n, int k) {
  this.visited = new boolean[m][n];
  return dfs(0,0,m,n,k);
}
private int dfs(int i, int j, int m, int n, int k){
  // 只向右向下搜索，i、j不会小于0 ,下方和右方有重叠区域，所以需要visited来记录全局的访问情况
  if(i>=m || j>=n || sum(i)+sum(j) > k || visited[i][j]) return 0;
  visited[i][j] = true;
  return 1 + dfs(i+1,j,m,n,k) + dfs(i,j+1,m,n,k); 
}
// BFS
public int movingCount(int m, int n, int k) {
  LinkedList<int[]> queue = new LinkedList<>();
  int res = 0;
  boolean visited = new boolean[m][n];
  queue.offerLast(new int[]{0,0});
  while(!queue.isEmpty()){
    int[] cur = queue.pollFirst();
    // 
    int x = current[0];
    int y = current[1];
    if((sum(x)+sum(y)) <= k) res++;
    else continue;
    if(x+1<m && visited[x+1][y] != 1) {
        queue.offerLast(new int[]{x+1,y});
        visited[x+1][y] = 1;
    }
    if(y+1<n && visited[x][y+1] != 1) {
        queue.offerLast(new int[]{x,y+1});
        visited[x][y+1] = 1;
    } 
  }
}
// sum 这里可以找数学规律来减少运算量
private int sum(n){
  int sum = 0;
  while(n>0){
    sum += n%10;
    n /= 10;
  }
  return sum;
}
```



####T14 剪绳子-2

**动态规划与贪婪算法：有点类似，但是贪婪算法是需要通过数学推导得到最优的方式，然后一直用这种最优的状态一步步推进；动态规划需要知道基础值。**

```java
// 贪婪/数学
// 以相同的长度分为多段，此时乘积最大，当每段为3（x=e，2.7）时取到最大
// 因此n % 3

//动态规划

```

#### T15 二进制中1的个数 & 56/65

```java
private int numberOf(int n){
  int count = 0;
  while(n){
    ++count;
    n = n&(n-1); //n&(n-1) 相当于把最后一个1变成0
  }
  return count;
}

private int numberOf(int n){
  int count = 0;
  while(n!=0){
    count += n&1;
    n = n >>>1;
  }
  return count;
}
```

可以用相同方法的题目：

判断一个数是不是2的整数次方？如果是，这个数就只有1bit是1，所以n&（n-1）是0；否则不是。

两个整数m,n，计算需要改变m的二进制表示中的多少位才能得到n？m^n会得到mn之间不相同的bit。统计为1的bit数量即可。

**1. 基本功能；2. 边界值； 3. 不合规范的非法输入；！！！！返回值、全局变量和异常**

#### T16 数值的整数次方

```java
// 指数 正 负 0
// 底数 0 非零
// 位与运算代替求余数判断奇偶；右移运算代替除法
public double myPow(double x, int n) {
  //if(x == 0) return 0; // double类型不能直接使用==
  if(Double.compare(x,0.0) == 0) return 0;//包括了0的0次方这种非法情况
  if(n == 0) return 1;
  long N = (long) n; //避免Integer.MIN_VALUE取正之后溢出 -8 7
  if(N<0){
    x = 1/x;
    N = -N;
  }
  double res = 1.0;
  while(N>0){  //快速幂
    if((N&1)==1) res *= x;
    x *= x;
    N >>= 1;
  }
  return res;
}
```



####T17 打印从1最大的n位数

**大数问题！！！：没有规定n的范围，所以可能long都不够用。一般会用字符串或者数组来表示大数！**

```java
// 用字符串表示，每一个char代表一个数位
public void printNumbers(int n) {
  StringBuilder sb = new StringBuilder();
  for(int i = 0; i<n; i++){
    sb.append("0");
  }
  while(!increaseNum(sb)){
    printStringNumber(str);
  }
}
private boolean increaseNum(StringBuilder sb){
  boolean overFlow = false;
  for(int i = sb.length()-1 ; i >=0 ; i--){
    char s = (char) (sb.charAt(i) + 1);
    if(s > '9'){
      sb.replace(i, i+1, '0');
  		if(i == 0) overFlow = true;    
    } else {
      sb.replace(i, i+1, s);
      break;
    }
  }
  return overFlow;
}
private void printStringNumber(StringBuilder sb){
  int i = 0;
  while(i < nums.length && sb.charAt(i) == '0') i++;
  System.out.println(sb.substring(i).toString());
}
```



#### T18 删除链表节点-1/2个节点

**leetcode中的待删除节点是用int来表示的，所以只能从头遍历来定位它；而原题是用ListNode来表示的，可以分成三种情况来处理：待删除节点不是头节点也不是尾节点，将next的val复制到当前节点，去掉next；待删除节点是头节点；待删除节点是尾节点，需要遍历。整体时间复杂度O(1）。**



变体：删除两个相加和为0的节点

```java
public ListNode removeZeroSumSubLists(ListNode head){
  ListNode dummy = new ListNode(-1);
  dummy.next = head;
  ListNode h = dummy;
  
  while(h.next!=null){
    ListNode p = h.next;
    ListNOde q = p;
    int sum = p.val;
    whle(q.next!=null || sum== 0){
      if(sum == 0){
        h.next = q.next;
        break;
      }
      q = q.next;
      sum += q.val;
    }
    if(sum != 0) h = h.next;
  }
  return dummy.head;
}
```



#### T19 正则表达式匹配

动态规划：

一开始从顶向下看：如果pattern的最后一个字符是*，有两种情况

+ 忽略pattern后两位，意味着匹配0个；
+ 忽略string的后一位，意味着匹配上了，这是最后一个字符；

如果最后一个字符不是*，检查匹配或者跳过（。的情况）

写代码的时候从下往上看，从0，0开始推：

+ 空串，空正则：true；dp\[0][0] = true
+ 非空串，空正则：false；dp\[i][0] = false;
+ 空串，非空正则：计算；
+ 非空串，非空正则：计算

```java
public boolean isMatch(String A, String B){
  int n = A.length();
  int m = B.length();
  boolean[][] dp = new boolean[n+1][m+1];
  
  for(int i = 0; i <= n; i++){
    for(int j = 0; j <= m; j++){
      if(j == 0){
        dp[i][j] = i==0;
      } else {
        // 非空正则串 ,第一个字符，i，j为1
        if (B.charAt(j-1) != '*'){
          // 正常字符或者.
          if (i >= 1 && (A.charAt(i-1) == B.charAt(j-1) || B.charAt(j-1) == '.')){
            dp[i][j] = dp[i-1][j-1];
          }
        } else {
          // *
          if (j >=2){
            dp[i][j] = dp[i][j] | dp[i][j-2];
          }
          if (i>=1 && j>=2 && (A.charAt(i-1) == B.charAt(j-2) || B.charAt(j-2) == '.')){
            dp[i][j] = dp[i][j] | dp[i-1][j];
          }
        }
      }
    }
  }
  return dp[n][m];
}
```



#### T20 表示数值的字符串

【A】【。【B】

```java
public boolean isNumber(String s){
  if(s == null || s.length() == 0){
    return false;
  }
  char[] str = s.trim().toCharArray();
  boolean numSeen = false;
  boolean dotSeen = false;
  boolean eSeen = false;
  for(int i = 0; i< str.length; i++){
    if(str[i] >= '0' && str[i] <= '9'){
      numSeen = true;
    } else if(str[i] == '.'){
      if(dotSeen || eSeen) return false; // 。只能出现一次，并且在e之前
      dotSeen = true;
    } else if(str[i] == 'e' || str[i] == 'E'){
      if(eSeen || !numSeen) return false;
      eSeen = true;
      numSeen = false; // 确保e之后出现数字
    } else if(str[i] == '-' || str[i] == '+'){
      if(i != 0 && str[i-1] != 'e' && str[i-1] != 'E') return false;
    } else {
      return false;
    }
  }
  return numSeen;
}
```

#### T21 调整数组顺序使奇数位于偶数的前面

```java
public int[] exchange(int[] nums){
  if(nums.length <= 0) return nums;
  int i = 0;
  int j = nums.length-1;
  while(i<j){
    while(i<j && (nums[i]&1) == 1) i++;
    while(i<j && (nums[i]&1) == 0) j--;
		int temp = nums[j];
    nums[j--] = nums[i];
    nums[i++] = temp;
  }
  return nums;
}
```

#### T22 链表中倒数第k个节点

```java
public ListNode getKthFromEnd(ListNode head, int k){
  // 鲁棒性：head == null 或者 k超过最大长度 如果k是0
  if(head == null || k == 0) return null;
  ListNode first = head;
  ListNode second = head;
  while(k>0 && first != null){
    firsr = first.next;
    k--;
  }
  if(k>0) return null; // 此时k超过链表长度
  while(first != null){
    first = first.next;
    second = second.next;
  }
  return second;
}
```

#### T23 链表中环的入口节点

```java
// 链表有环？？
// 入口节点在哪？？
// 环的长度多少？？
    public boolean hasCycle(ListNode head) {
        if (head == null ) return false;
        ListNode first = head;
        ListNode second = head;
        while(true){
            if(first == null || first.next == null) return false;
            first = first.next.next;
            second = second.next;
            if(first == second) return true;
        }
      // 环的长度
        int count = 0;
        do{
            first = first.next.next;
            second = second.next;
            count ++;
        }while(first != second);
        System.out.println(count);
          
      // 进一步寻找入口， 上面的return true 改成break
        ListNode p = head;
        while(p != second){
            p = p.next;
            second = second.next;
        }
        return p;
    }
```

#### T24 反转链表

```java
public ListNode reverseList(ListNode head){
  // null 一个 多个
  if( head == null) return null;
  ListNode preNode = head;
  ListNode nextNode = head.next;
  ListNode tmp = null;
  head.next = null;
  while(nextNode != null){
    // 一个节点的时候不进入循环
    tmp = nextNode.next;
    nextNode.next = preNode;
    preNode = nextNode;
    nextNode = tmp;
  }
  return preNode;
}
```

#### T25 合并两个排序的链表

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2){
  // null null / null 非空 / 非空非空
  ListNode dummy = new ListNode(-1);
  ListNode cur = dummy;
  while(l1 != null && l2 != null){
    if(l1.val < l2.val){
      cur.next = l1;
      l1 = l1.next;
    } else {
      cur.next = l2;
      l2 = l2.next;
    }
    cur = cur.next;
  }
  while(l1 != null){
    
  }
  while(l2 != null){
    
  }
  return dummy.next;
}
```

#### T26 树的子结构

如果 TreeNode中的value是double类型：Double.compare(x,0.0) == 0

**整体思路：**

先序遍历，先**检查**root，再左右子树；用|| 短路或连接，当前面的条件满足true，则不必再检查后面的情况了。

**检查：**

B访问到叶节点，检查完成；true；

A到叶节点，B却没有到，false；

A，B的value不等，false

**特例**：如果B是null，空树不是任何树的子结构，false；

如果A是null，也不是子结构，false

```java
public boolean isSubStructure(TreeNode A, TreeNode B){
  if(B == null || A == null) return false;
  return help(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
}
private boolean help(TreeNode A, TreeNode B){
  if(B == null) return true;
  if(A == null || A.val != B.val) return false;
  return help(A.left,B.left) && help(A.right, B.right);
}
```

#### T27 二叉树的镜像

```java
// 递归方法
public TreeNode mirrorTree(TreeNode root){
  if(root == null) return root;
  if(root.left == null &&root.right ==null) return root;
  TreeNode tmp = root.right;
  root.right = root.left;
  root.left = tmp;
	mirrorTree(root.left);
  mirrorTree(root.right);
  return root;
}
// 迭代方法 层次遍历
public TreeNode mirrorTree(TreeNode root){
  if(root ==null ) return root;
  LinkedList<TreeNode> queue = new LinkedList<>();
  queue.offerLast(root);
  while(!ueue.isEmpty()){
    TreeNode cur = queue.pollFirst();
    TreeNode tmp = cur.left;
    cur.left = cur.right;
    cur.right = tmp;
    if(cur.left != null) queue.offerLast(cur.left);
    if(cur.right != null) queue.offerLast(cur.right);
  }
  return root;
}
```

#### T28 对称的二叉树

```java
public boolean isSymmetric(TreeNode root){
  if(root == null) return true;
  return help(root.left, root.right);
}
private boolean help(TreeNode p, TreeNode q){
  if(p == null && q == null) return true;
  if(p == null || q ==null || p.val != q.val) return false;
  return help(p.left, q.right) && help(p.right , q.left);
}
```

#### T29 顺时针打印矩阵

​     Zuo  you      

1	2	  3	  4     

5	6	  7	  8   

9	10	11	12 xia  shang

13  14	15	16   

```java
public int[] spiralOrder(int[][] matrix){
	if(matrix.length <= 0 || matrix[0].length <=0) return new int[]{};
  int row = metrix.length;
  int col = metrix[0].length;
  int[] res = new int[row*col];
  int k = 0, zuo = 0, you = col-1, shang = 0, xia = row-1;
  while(true){
    for(int i = zuo; i <= you; i++) res[k++] = matrix[shang][i];
    if(shang++ == xia) break;
    for(int i = shang; i <= xia; i++) res[k++] = matrix[i][you];
    if(you-- == zuo) break;
    for(int i = you; i >= zuo; i--) res[k++] = matrix[xia][i];
    if(xia-- == shang) break;
    for(int i = xia; i >= shang; i--) res[k++] = matrix[i][zuo];
    if(zuo++ == you) break;
  }
  return res;
}
```

#### T30 包含min函数的栈

```java
// 辅助栈 用于保存最小的值
class MinStack{
  
  private LinkedList<Integer> stackMain;
  private LinkedList<Integer> stackMin;
  
  public MinStack(){
    stackMain = new LinkedList<>();
    stackMin = new LinkedList<>();
  }
  
  public void push(int x){
    if(stackMin.isEmpty() || x <= stackMin.peek()){
      stackMin.push(x);
    }
    stackMain.push(x);
  }
  public void pop(){
    int remove = stackMain.pop();
    if(stackMin.peek() == remove){
      stackMin.pop();
    }
  }
  public void top(){
    stackMain.peek();
  }
  public void min(){
    stackMin.peek();
  }
}
// 辅助int， 两次压栈，两次弹出
class MinStack{
  
  private LinkedList<Integer> stackMain;
  private int min;
  
  public MinStack(){
    stackMain = new LinkedList<>();
    min = Integer.MAX_VALUE;
  }
  
  public void push(int x){
    if(!stackMain.isEmpty() && x <= min){
      stackMain.push(min);
      min = x;
    }
    stackMain.push(x);
  }
  public void pop(){
    int remove = stackMain.pop();
    if(min == remove){
      min = stackMain.pop();
    }
  }
  public int top(){
    return stackMain.peek();
  }
  public int min(){
    return min;
  }
}


```

#### T31 栈的压入弹出

```java
// 模拟stack的压入弹出
public boolean validateStackSequence(int[] pushed, int[] popped){
  if(pushed.length <= 0) return true;
  LinkedList<Integer> stack = new LinkedList<>();
  int j = 0;
  for(int push : pushed){
    stack.push(push);
    while(!stack.isEmpty() && stack.peek() == popped[j]){
      stack.pop();
      j++;
    }
  }
  return stack.isEmpty();
}
// 不用stack，直接用pushed当作栈，pushed[size-1]可以作为栈顶
public boolean validateStackSequences(int[] pushed, int[] popped){
  if(pushed.length <= 0) return true;
  int size = 0;
  for (int i = 0, j = 0; i < pushed.length; i++){
    pushed[size++] = pushed[i]; //入栈
    while(size != 0 && pushed[size-1] == popped[j]){
      size--;j++;
    }
  }
  return size == 0;
}
```

#### T32 从上到下打印二叉树

以层次遍历的方法为框架，在其中变化

反向打印，则LinkedList.offerFirst，不改变queue，只改变list的顺序。

```java
// 层次遍历
public int[] levelOrder(TreeNode root){
  if(root == null) return new int[]{};
  LinkedList<TreeNode> queue = new LinkedList<>();
  List<Integer> ans = new ArrayList<>();
  queue.offer(root);
  while(!queue.isEmpty()){
    TreeNode cur = queue.poll();
    ans.add(cur.val);
    if(cur.left != null) queue.offer(cur.left);
    if(cur.right != null) queue.offer(cur.right);
  }
  int[] res = new int[ans.size()];
  int i = 0;
  for(int num: ans) res[i++] = num;
  return res;
}

// 结果分层打印
public List<List<Integer>> levelOrder(TreeNode root){
    LinkedList<TreeNode> queue = new LinkedList<>();
    List<List<Integer>> res = new ArrayList<>();
  	if(root == null) return res;
    queue.offer(root);
    while(!queue.isEmpty()){
      List<Integer> ans = new ArrayList<>();
      int count = queue.size();
      while(count>0){
        TreeNode cur = queue.poll();
      	ans.add(cur.val);
        count--;
      	if(cur.left != null) queue.offer(cur.left);
      	if(cur.right != null) queue.offer(cur.right);
      }
      res.add(ans);
    }
    return res;
}

// Z型打印
//     1
//   2   3
//  4 5 6  7
public List<List<Integer>> levelOrder(TreeNode root){   
    LinkedList<TreeNode> queue = new LinkedList<>();
    List<List<Integer>> res = new ArrayList<>();
  	if(root == null) return res;
    queue.offer(root);
  	int level = 0;
    while(!queue.isEmpty()){
      LinkedList<Integer> ans = new LinkedList<>();
      int count = queue.size();
      while(count>0){
        TreeNode cur = queue.poll();
        count--;
        if ((level&1)==0){
          // 从左到右
          ans.offerLast(cur.val);
        } else {
          // 从右到左
          ans.offerFirst(cur.val);
        }
      	if(cur.left != null) queue.offer(cur.left);
      	if(cur.right != null) queue.offer(cur.right);
      }
      level++;
      res.add(ans);
    }
    return res;
}
```

#### T32 二叉搜索树的后续遍历

```java
//如果数组长度小于等于2，一定可以构成二叉搜索树
public boolean verifyPostorder(int[] postorder) {
    if(postorder.length <= 2) return true;
    return help(postorder, 0, postorder.length-1);
}
private boolean help(int[] postorder, int start, int end){
    if(start >= end) return true; 
    int root = postorder[end];
    int i = end-1;
    while(i >= start && postorder[i]>root) i--;
    int m = i;
    while(i >= start && postorder[i]<root) i--;
    return i == start-1 && help(postorder, start, m) && help(postorder, m+1, end-1);
}
```

#### T34 二叉树中和为某一值的路径

回溯

```java
private List<List<Integer>> res = new ArrayList<>();
public List<List<Integer>> pathSum(TreeNode root, int sum){
    help(root, new LinkedList<Integer>(), sum);
    return res;
}
private void help(TreeNode root, LinkedList<Integer> path, int sum){
    if(root == null) return;
    path.offerLast(root.val);
    if(root.left == null && root.right == null && sum == root.val){
        res.add(new LinkedList<Integer>(path));
        path.pollLast();
        return;
    }
    help(root.left, path, sum-root.val);
    help(root.right, path, sum-root.val);
    path.pollLast();
}
```

#### T35 复杂链表的复制

```java
// 哈希map
public Node copyRandomList(Node head){
  Node dummy = new Node(-1);
  HashMap<Node,Node> map = new HashMap<>();
  dummy.next = head;
  // 通过hashmap将新旧node对应起来，第二次遍历的时候加上node之间的联系
  while(head!=null){
    Node tmp = new Node(head.val);
    map.put(head, tmp);
    head = head.next;
  }
  for(Node old : map.keySet()){
    Node now = map.get(old);
    now.next = map.get(old.next);
    now.random = map.get(old.random);
  }
  return map.get(dummy.next);
}
// 把新node放在旧node的后面，链接，分成两个链表
public Node copyRandomList(Node head){
  if(head == null) return null;
  Node dummy = new Node(-1);
  dummy.next = head;
  while(head!=null){
    Node tmp = new Node(head.val);
    tmp.next = head.next;
    head.next = tmp;
    head = head.next.next;
  }
  head = dummy.next;
  while(head!=null){
    if(head.random!=null){
       head.next.random = = head.random.next;
    }
    head = head.next.next;
  }
  head = dummy.next;
  Node newHead = head.next;
  Node cur = newHead;
  head.next = cur.next;
  head = head.next;
  while(head!=null){
      cur.next = head.next;
      cur = cur.next;
      head.next = head.next.next;
      head = head.next;
  }
  return newHead;  
}
```





#### T36 二叉搜索树与双向链表-递归

```java
// 递归
private Node pre = null, head = null;
public Node treeToDoublyList(Node root){
    if(root == null) return null;
    help(root);
    pre.right = head;
    head.left = pre;
    return head;
}
private void help(Node root){
    if(root == null) return;
    help(root.left);
    if(pre == null) head = root;
    else pre.right = root;
    root.left = pre;
    pre = root;
    help(root.right);
}
// 迭代
算了记不住
```

#### **T37 序列化/反序列化二叉树









#### T38 字符串的排列-N皇后问题

```java
// 递归/回溯
private List<String> res = new ArrayList<>();
public String[] permutation(String s){
  char[] arr = s.toCharArray();
  Arrays.sort(arr);
  int[] used = new int[arr.length];
  help(arr, new StringBuilder(), used);
  return res.toArray(new String[res.size()]);
}
private void help(char[] arr, StringBuilder sb, int[] used){
  if(sb.length() == arr.length){
    res.add(new String(sb));
    return;
  }
  for(int i = 0; i < arr.lengh; i++){
    if(used[i] ==1 ||(i>0 && arr[i-1]==arr[i] && used[i-1]==0 )) continue;
    sb.append(arr[i]);
    used[i] = 1;
    help(arr, sb, used);
    used[i] = 0;
    sb.deleteCharAt(sb.length()-1);             
  }
}
// swap回溯，不用used查表
private List<String> res = new LinkedList<>();
private char[] c;
public String[] permutation(String s){
  c = s.toCharArray();
  dfs(0);
  return res.toArray(new String[res.size()]);
}
private void dfs(int x){
  if(x == c.length-1){
    res.add(String.valueOf(c));
    return;
  }
  HashSet<Character> set = new HashSet<>();
  for(int i = x; i < c.length; i++){
    if(set.contains(c[i])) continue;
    set.add(c[i]);
    swap(i,x);
    dfs(x+1);
    swap(i,x);
  }
}
private void swap(int a, int b){
  char tmp = c[a];
  c[a] = c[b];
  c[b] = tmp;
}

```

**N皇后问题**

```java
class Solution {
    private List<List<String>> res = new ArrayList<>();
    public List<List<String>> solveNQueens(int n) {
        char[][] track = new char[n][n];
        int[] colIndex = new int[n]; // colIndex[i] 第i行Q所在的列
        for(char[] row : track) Arrays.fill(row, '.');
        Arrays.fill(colIndex, -1);
        dfs(track, 0, colIndex);
        return res;
    }
    private void dfs(char[][] track, int row, int[] colIndex){
        if(row == colIndex.length){
            List<String> tmp = new ArrayList<>();
            for(char[] r : track){
                tmp.add(new String(r));
            }
            res.add(tmp);
            return;
        }
        for(int i = 0; i < colIndex.length; i++){
            if(!validate(row, i, colIndex)) continue;
            track[row][i] = 'Q';
            colIndex[row] = i;
            dfs(track, row+1, colIndex);
            track[row][i] = '.';
            colIndex[row] = -1;
        }
    }
    private boolean validate(int row, int col, int[] colIndex){
        // 检查列
        for(int in : colIndex){
            if (in == col) return false;
        }
        // 检查对角线
        for(int i = 0; i < colIndex.length; i++){
            if(colIndex[i]!=-1 && (colIndex[i] - col)==(i-row)) return false;
            if(colIndex[i]!=-1 && (colIndex[i] - col)==(row-i)) return false;
        }
        return true;
    }
}
```



#### 最大公约数/最小公倍数

辗转相除：

+ 用较大的a除以较小的b，余数r
+ a = b, b = r
+ 重复直到r为0 

```java
public int getHighestConventionNumber(int m,int n){
    //辗转相除法：求取最大公约数
  	// 1001 ， 0101
  	// 1100 ， 0101
  	// 1100 ， 1001
  	// 0101 ， 1001 
    if(m < n){
        m= m ^ n;
        n= m ^ n;
        m= m ^ n;
    }
    m % n == 0 ? return n : return getHighestConventionNumber(n, m % n);
}

public int getLowestCommonMultiple(int m, int n){
    // 求取最小公倍数：m*n/最大公约数
    return m * n / getHighestConventionNumber(m,n);
}
```

最小公倍数 = a*b/最大公约数

#### T39 数组中出现超过一半的数字-排序/投票

```java
// 快排中的partition方法，直到pivot是中位数

// 投票：初始化：第一个数投一票，下一个数如果一样，票数+1；下一个数不一样，票数-1；票数为0，换一个数保存
public int majorityElement(int[] nums){
  // 空数组，返回0；长度为1的数组，返回第一个值
  int x = 0, vote = 0;
  for(int n : nums){
    if(vote == 0) x = n;
    vote += (x==n)?1:-1;
  }
  return x;
}
```

#### T40 最小的K个数-排序/堆

TopK问题：

+ 快排，改变pivot对应的index就好
+ 堆：使用PriorityQueue
+ 二叉搜索树
+ 数据范围有限使用计数排序

```java
// 找出数组nums中最小的k个数
// 快排的partition方法，直到pivot是k-1
public int[] getLeastNumber(int[] arr, int k){
  if(k == 0 || arr.length == 0) return new int[]{};
  return quickSort(arr, 0, arr.length-1, k-1);
}
private int[] quickSort(int[] nums, int start, int end, int k){
  int index = partition(nums, start, end);
  if(index == k) return Arrays.copyOf(nums, index+1); // index+1 是 newlength = k
  return index > k ? quickSort(nums, start, index-1, k) : quickSort(nums, index+1, end, k);
}
private int partition(int[] nums, int start, int end){
  int pivot = nums[start];
  int lt = start +1;
  int gt = end;
  while(true){
    while(lt<=end&&nums[lt]<pivot) lt++;
    while(gt>start&&nums[gt]>pivot) gt--;
    if(lt>gt) break;
    swap(nums, lt, gt);
    lt++;
    gt--;
  }
  swap(nums, start, gt);
  return gt;
}
```

```java
// 前K小，就用大根堆，从而保证堆中的k个数是最小的，大的都被poll
public int[] getLeastNumber(int[] arr, int k){
  if(k == 0 || arr.length == 0) return new int[]{};
  // 默认是小根堆
  PriorityQueue<Integer> heap = new PriorityQueue<Integer>((i1, i2)-> i2-i1);
  for(int num: arr){
    if(queue.size()<k){
      queue.offer(num);
    } else if(num<queue.peek()) {
      queue.poll();
      queue.offer(num);
    }
  }
  int[] res = new int[queue.size()];
  int k = 0;
  for(int num: queue){
    res[k++] = num;
  }
  return res;
}

```



#### T41 数据流中的中位数-堆

用最大堆和最小堆来实现

最小堆： 大大大大

​					小

中位数：中位数 =  

最大堆：    大

​				小小小小

当总数为偶数的时候，新数据插入最大堆（但如果新数据很大，比最小堆的最小数据大，即本来应该放进最小堆的。那就把新数据插入最小堆，拿最小堆的最小值放进最大堆）

总数为奇数的时候，新数据插入最小堆（同理）

```java
class MedianFinder {

  	private long SUM = 0;
  	private PriorityQueue<Integer> lowerMedian;
  	private PriorityQueue<Integer> higherMedian;
    /** initialize your data structure here. */
    public MedianFinder() {
			lowerMedian = new PriorityQueue<Integer>((i1,i2)->i2-i1);
      higherMedian = new PriorityQueue<Integer>((i1,i2)->i1-i2);
    }
    
    public void addNum(int num) {
			if((SUM&1) == 1){
        if(!lowerMedian.isEmpty() && lowerMedian.peek() > num){
          lowerMedian.add(num);
          higherMedian.add(lowerMedian.poll());
        } else {
          higherMedian.add(num);
        }
        SUM++;
      } else {
        if(!higherMedian.isEmpty() && higherMedian.peek() < num){
          higherMedian.add(num);
          lowerMedian.add(higherMedian.poll());
        } else {
          lowerMedian.add(num);
        }
        SUM++;
      }
    }
    
    public double findMedian() {
		if(lowerMedian.size() == higherMedian.size()){
          return (((double)lowerMedian.peek() + (double)higherMedian.peek())/2);
        } else if(lowerMedian.size() > higherMedian.size()){
          return (double) lowerMedian.peek();
        } else {
          return (double) higherMedian.peek();
        }
    }
}
```



#### T42 连续子数组的最大和-动态规划

计算包括当前数字nums[i]的字数组的最大值 = 当前值 或者 i-1时的最大值+i

```java
public int maxSubArray(int[] nums){
	if(nums.length <= 0) return 0;
  int i = 1, sum = nums[0], max = nums[0];
  while(i < nums.length){
    if(sum <= 0) sum = nums[i++];
    else sum += nums[i++];
    max = Math.max(max, sum);
  }
  return max;
}
```



#### T43 1-n正数中1出现的个数

暴力遍历，对每个数检索1，时间复杂度（nlogn）

找规律：                                                                                                                                                                                                                                                                                                                                                                       

```java
// 从个位开始计算。
// X位出现1的次数
// X位是0的时候，只和比它高的位数值相关，high*digit
// X位是1的时候，还和低位相关，high*digit+low+1
// X位是其他的时候，X本身1？有10^X = digit个数， （high+1）*digit
public int countDigitOne(int n) {
    int digit = 1, res = 0;
    int high = n / 10, cur = n % 10, low = 0;
    while(high != 0 || cur != 0) {
        if(cur == 0) res += high * digit;
        else if(cur == 1) res += high * digit + low + 1;
        else res += (high + 1) * digit;
        low += cur * digit;
        cur = high % 10;
        high /= 10;
        digit *= 10;
    }
    return res;
}

```

#### T44 数字序列中某一位的数字

1-9			1			9

10-99		2 		180 		

100-999 	3		3*900=2700

先通过数学关系，确定这是几位数？

确定几位数之后，计算是具体哪一个数？

确定是哪一个数之后，计算是具体哪一位？

```java
public int findNthDigit(int n) {
    int digit = 1;
    long start = 1;
    long count = 9;
    while (n > count) { // 1.
        n -= count;
        digit += 1;
        start *= 10;
        count = digit * start * 9;
    }
    long num = start + (n - 1) / digit; // 2.
    return Long.toString(num).charAt((n - 1) % digit) - '0'; // 3.

}
```

#### T45 把数组排成最小的数-字符排序

"30" + "3" < "3" + "30"  ======== 303 < 330 

所以“30” 排在“3”的前面

```java
public String minNumber(int[] nums){
  String[] str = new String[nums.length];
  for(int i = 0; i < nums.length; i++){
    str[i] = String.valueOf(nums[i]);
  }
  Arrays.sort(str, (s1,s2)->(s1+s2).compareTo(s2+s1));
  StringBuilder sb = new StringBuilder();
  for(String s: str){
    sb.append(s);
  }
  return sb.toString();
}
```



#### T46 把数字翻译成字符串-动态规划

动态规划的关键：找到递推方程

```java
public int translateNum(int num){
  int dp1 = 1, dp2 = 1;
  int gewei = num%10, shiwei; 
  while(num > 0){
    num /= 10;
    shiwei = num %10;
    int tmp = shiwei*10+gewei;
    int dp = (tmp >=10 && tmp <=25) ? dp1 + dp2 : dp1;
    dp2 = dp1;
    dp1 = dp;
    gewei = shiwei;
  }
  return dp1;
}
```



#### T47 礼物的最大价值-动态规划

```java
public int maxValue(int[][]grid){
  int m = grid.length;
  int n = grid[0].length;
  // 初始化第一行
  for(int j = 1 ; j < n ; j++){
    grid[0][j] += grid[0][j-1];
  }
  // 初始化第一列
  for(int i = 1 ; i < m; i++){
    grid[i][0] += grid[i-1][0];
  }
  for(int i = 1 ; i < m ; i++){
    for(int j = 1; j < n ; j++){
      grid[i][j] += Math.max(grid[i][j-1], grid[i-1][j]);
    }
  }
  return grid[m-1][n-1];
}
```

#### T48 最长不含重复字符的子字符串

Hashmap存储

```java
public int lengthOfLongestSubstring(String s){
  int max = 0, start = 0, end = 0;
  HashMap<Character, Integer> map = new HashMap<>();
  while(end<s.length()){
    char c = s.charAt(end++);
    if(map.containsKey(c)){
      start = Math.max(map.get(c), start);
    }
    map.put(c, end); // char -> char对应的index+1
    max = Math.max(max, end-start);
  }
  return max;
}
```



#### T49 丑数

```java

```



#### T50 第一个只出现一次的字符

```java
// map保存: 字符->出现的次数

```



#### T51 逆序对-归并排序

归并排序每交换一次，就要计算这时有多少逆序对（！！！并不是直接+1）

**count += (mid-i+1);**

应用情况：排队，从低到高排序，每次只能交换相邻的两位，最少需要交换多少次？



```java
public int reversePairs(int[] nums) {
  if(nums.length < 2) return 0;
  int len = nums.length;
  return mergeSort(nums, 0, len - 1，new int[len]);
}


private int mergeSort(int[] nums, int start, int end, int[] tmp){
  if(start == end) return 0;
  int mid = (start+end)>>>1;
 	int left = mergeSort(nums, start, mid, tmp);
  int right = mergeSort(nums, mid+1, end, tmp);
  if(nums[mid] <= nums[mid+1]) return left + right;
  int cross = mergeTwo(nums, start, mid, end, tmp);
  return left + right + cross;
}

private int mergeTwo(int[] nums, int start, int mid, int end, int[] tmp){
  
  System.arraycopy(nums, start, tmp, start, end-start+1);
  int i = start, j = mid+1, count = 0;
  for(int k = start; k <= end; k++){
    if(i == mid+1) nums[k] = tmp[j++]; // 数组1 到底了
    else if(j == end+1) nums[k] = tmp[i++];
    else if(tmp[i]<=tmp[j]) nums[k] = tmp[i++];
    else {
      count += (mid-i+1);
      nums[k] = tmp[j++];
    }
  }
  return count;
}
```

#### T52 两个链表的第一个公共节点

```java
// 辅助栈，将链表倒序
// 遍历两个链表求长度，长的那一条先走k步，使他们同时到达终点。第一次相等的地方就是公共节点
// a遍历到尾的时候，返回b的开头，b到尾的时候返回a的开头，ab会相遇在公共节点
```



#### T53 在排序数组中查找数字

> 数字在排序数组中出现的次数
>

因为是排序数组，所以应该第一时间想到二分查找。需要找到这个数字的左边界和右边界。

```java
public int search(int[] nums, int targer){
  return binarySearch(target) - binarySearch(target-1);
}
// 查找右边界，即不小于target的
private int binarySearch(int[] nums, int target){
  int start = 0, end = nums.length;
  while(start < end){
    int mid = (start + end)>>>1;
    if(nums[mid] <= target) start = mid+1;
    else end = mid;
  }
  return start; // 返回的是第一个大于target的位置，start-1才是最后一个target（如果存在）
}
```



> 0~n-1之间缺失的数字，数组是排序的

如果数组是不排序的，可以用n的数字的总和减去数组的总和，差值就是缺少的数字；或者用位运算，0到n-1的数字按位异或，再对数组中的数字逐个进行异或，这样相当于只有缺失的数字参加了1次异或，其他数字是两次异或，最后的答案就是缺少的数。时间复杂度都是（n）

但是由于数组是排序的，首选还是用二分进行查找。

```java
public int missingNumber(int[] nums){
  int start = 0, end = nums.length;
  while(start < end){
    int mid = (start+end)>>>1;
    if(nums[mid] == mid) start = mid + 1;
    else end = mid;
  }
  return start;
}
```



> 数组中数值和下标相等的元素

```java

```



#### T54 二叉搜索树的第K大节点

二叉搜索树的中序遍历，就是递增的。

```java
public int kthLargest(TreeNode root, int k){
  this.k = k;
  dfs(root);
  return res;
}
private int res = 0;
private int k;
private void dfs(TreeNode root){
  if(root == null) return;
  dfs(root.right);
  if(k == 0) return;
  k--;
  if(k == 0) {
    res = root.val;
    return;
  }
  dfs(root.left);
}
```



#### T55 二叉树的深度

> 一颗普通的二叉树，求其深度

二叉树的题目首选用递归的方法，树的深度 =  1+ Max（左子树， 右子树）

```java
public int maxDepth(TreeNode root){
  if(root == null) return 0;
  return 1 + Math.max(maxDepth(root.right), maxDepth(root.left));
}
```



> 检查一棵二叉树，是不是平衡二叉树？

后序遍历可以保证每个节点只遍历一次

```java
public boolean isBalanced(TreeNode root) {
  return depth(root) != -1;
}
private int depth(TreeNode root){
  // 返回值是高度（平衡），-1（不平衡）
  if(root == null) return 0;
  int left = depth(root.left);
  int right = depth(root.right);
  if(left == -1 || right == -1) return -1;
  return Math.abs(left-right)<2?Math.max(left, right):-1;
}
```



#### T56 数组中数字出现的次数

一个数组中除了两个数字之外，其他数都出现了两次。

通过一个bit作为标志位，将数组分为两组，每组各有一个单独数。这样就把问题转化为只有一个数字是单独的。

```java
public int[] singleNumbers(int[] nums) {
  int key = 0;
  for(int n : nums) key ^= n;
  int bit = 1;
  while((key&bit)==0) bit <<= 1;
  
  int group1 = 0, group2 = 0;
  for(int n : nums){
    if((n&key) == 0) group1 ^= n;
    else group2 ^= n;
  }
  return new int[]{group1, group2};
}
```



> 数组中唯一出现一次的数字

一个数组中只有一个数字是单独的，其他数字都是出现了3次

按照bit为粒度，bit位上出现1的次数能够被3整除，则是出现了3次的数字。而不能被整除的就是出现了1次的那个数字所在的bit。

```java
public int singleNumber(int[] nums){
  int[] map = new int[32];
  for(int n : nums){
    int j = 0;
    while(n!=0){
      map[j++] += n&1;
      n >>>= 1;
    }
  }
  int res = 0, i = 0;
  for(int count: map){
    if(count%3!=0) res += 1<<i;
    i++;
  }
  return res;
}
```



#### T56 和为S的数字

> 从一个递增排序数组中找到两个数字，他们和为S。找到任意一组即可

如果不是排序数组，那就类似TwoSum；

但是因为是排序的，所以可以不需要辅助空间。双指针头尾扫描。

```java
public int[] twoSum(int[] nums, int target) {
  int i = 0, j = nums.length-1, sum;
  while(i<j){
    sum = nums[i]+nums[j];
    if(sum == target) return new int[]{nums[i], nums[j]};
    else if(sum > target) i++;
    else j--;
  }
  return new int[]{};
}
```



> 输入一个正整数S，输出所有和S的正整数序列（2个数及以上）

双指针指向序列的头和尾，类似滑动窗口，sum不到S，窗口增大；sum大于S，窗口缩小；

终止条件，左边指针超过了mid

```java
public int[][] findContinuousSequence(int target) {
  LinkedList<int[]> res = new LinkedList<>();
  int i = 1, j = 2;
  int sum = i;
  while(i<=target/2){
    if(sum < target){
      sum += j;
      j++;
    } else if(sum == target){
      int tmp = new int[j-i];
      for(int k = 0; k < j-i; k++) tmp[k] = i + k;
      res.add(tmp);
      sum = sum - i + j;
      i++;
      j++;
    } else {
      sum -= i;
      i++;
    }
  }
  return res.toArray(new int[res.size()][]);
}
```



#### T58 翻转字符串

> 翻转句子中单词的次序，但是不翻转单词本身

```java
public String reverseWords(String s) {
  StringBuilder sb = new StringBuilder();
  s = s.trim();
  char[] arr = s.toCharArray();
  int i = arr.length-1, j = arr.length-1;
  while(i>=0){
    while(i>=0 && arr[i] != ' ') i--;
    res.append(s.substring(i+1,j+1) + ' ');
    while(i>=0 && arr[i] == ' ') i--;
    j = i;
  }
  return sb.toString().trim(); //转化成string才能trim
}
```



> 左旋转字符串

```java
public String reverseLeftWords(String s, int n) {
  return s.substring(n, s.length()) + s.substring(0,n);
}
// 如果不能用substring
public String reverseLeftWords(String s, int n) {
  StringBuilder sb = new StringBuilder();
  for(int i = n; i < s.length(); i++){
    sb.append(s.charAt(i));
  }
  for(int i = 0; i < n; i++){
    sb.append(s.charAt(i));
  }
  return sb.toString();
}
```



#### T59 队列中的最大值

> 滑动窗口中的最大值

维护一个最大值队列：如果左边出窗口的值在队列中，出队；如果右边进窗口的值更大，队尾的值出来，新值进去

```java
public int[] maxSlidingWindow(int[] nums, int k) {
  LinkedList<Integer> queue = new LinkedList<>();
  if(nums.length <= 0 || k <= 0) return new int[]{};
  int[] res = new int[nums.length-k+1];
  int i = 0;
  for(; i < k; i++){
    while(!queue.isEmpty() && queue.peekLast() < nums[i]) queue.pollLast();
    queue.offerLast(nums[i]);
  }
  int j = 0;
  while(i < nums.length){
    res[j] = queue.peek();
    if(queue.peek() == nums[j++]) queue.pollFirst();
    while(!queue.isEmpty() && queue.peekLast() < nums[i]) queue.pollLast();
    queue.offerLast(nums[i++]);
  }
  res[j] = queue.peek();
  return res;
}
```



> 队列的最大值

```java
// 关键：将新入队的数和队尾做比较，直到队列为空或者队尾的数不小于nums[i]
// 主队列，辅助队列
```



#### T60 n个骰子的点数 = 新21点

```java
public double[] twoSum(int n) {
  double[] res = new int[6*n+1];
  double p = 6;
  for(int i = 1; i <= 6 ; i++){
    res[i] = p;
  }
  for(int k = 2 ; k <= n ; k++){
      for(int i = 6*k; i >= k ; i--){
          double tmp = 0.0;
          for(int j = 1; (i-j)>=(k-1)&&(j<=6);j++){
              tmp += res[i-j];
          }
          res[i] = tmp*p;
      }
  }
  return Arrays.copyOfRange(res,n,6*n+1);
}

public double[] twoSum(int n) {
    double pre[]={1/6d,1/6d,1/6d,1/6d,1/6d,1/6d};
    for(int i=2;i<=n;i++){
        double tmp[]=new double[5*i+1];
        for(int j=0;j<pre.length;j++)
            for(int x=0;x<6;x++)
                tmp[j+x]+=pre[j]/6;
        pre=tmp;
    }
    return pre;
}

```

```java
// 新21点 概率1/w 总分在K和N之间的概率
public double new21Game(int N, int K, int W) {
    // 动态规划
    // dp[i] 总分为i的时候的概率
    // 答案是 K<= i <= N 的dp[i] 总和
    if (K == 0) return 1;
    double[] dp = new double[N+1];
    double sum = 0.0, res = 0.0;
    dp[0] = 1;
    for(int i = 1 ; i <= N ;i++){
        // dp[i] = 前w个dp的总和 / w
        if(i <= K) sum += dp[i-1];
        if(i > W && i <= K+W) sum -= dp[i-W-1];
        dp[i] = sum / W;

        if(i >= K) res += dp[i];
        // System.out.println(dp[i] + "****" +res);
    }
    return res;

}
```



#### T61 扑克牌中的顺子

```java
public boolean isStraight(int[] nums) {
  HashSet<Integer> set = new HashSet<>();
  int max = 0, min = 14;
  for(int n : nums){
    if(n == 0) continue;
    if(set.contains(n)) return false;
    set.add(n);
    max = Math.max(max, n);
    min = Math.min(min, n);
  }
  return max-min < 5;
}
```



#### T62 圆圈中最后剩下的数字

一般方法：模拟环形链表的删除过程，直到最后一个数字

```java
public int lastRemaining(int n, int m) {
     if(n==0||m==0)
		return -1;
     List<Integer> list=new ArrayList<>();
     for(int i=0;i<n;i++)
     	list.add(i);
     int c=(m-1)%n;
     while(list.size()!=1) {
     	list.remove(c);
     	c=(c+m-1)%list.size();  
     }
     return list.get(0);
 }
```

通过数学规律，推倒最后一个数: last = (last + m) % i;

```java
public int lastRemaining(int n, int m){
  if(m == 0 || n == 0) return -1;
  int last = 0;
  for(int i = 2; i <= n; i++){
    // 倒退第一轮，2个数
    // 直到总数为n
    last = (last + m) % i;
  }
  return last;
}
```



#### T63 股票的最大利润

```java
// 记住前i-1天里的最小值，可以计算第i天获得的最大利润
public int maxProfit(int[] prices){
  if(prices.length <= 0) return 0;
  int min = Integer.MAX_VALUE, max = 0;
  for(int i = 0; i < prices.length; i++){
    min = Math.min(prices[i], min);
    max = Math.max(prices[i]-min, max);
  }
  return max;
}
```



#### T64 求1+2+3+...+n

```java
public int sum(int n){
  boolean x = (n>0) && (n+=sum(n-1))>0;
  return n;
}
```



#### T65 不用加减法做加法

```java
public int add(int a, int b){
  while(b!=0){
    int c = (a&b)<<1;
    a ^= b;
    b = c;
  }
  return a;
} 

```



#### T66 构建乘积数组

```java
public int[] constructArr(int[] a) {
    int[] output = new int[nums.length];
    int left = 1;
    for(int i = 0; i < nums.length; i++){
        // 计算左侧乘积
        output[i] = left; 
        left *= nums[i];
    }
    int right = 1;
    for(int i = nums.length-1; i >= 0 ; i--){
        output[i] *= right;
        right *= nums[i];
    }
    return output;
}
```



#### T67 把字符串转化成整数

```java
public int strToInt(String str) {
	// 首先检查空格，找到非空字符
  // 第一个非空字符可能是 + -
  // 有效整数之后的其他字符忽略不计
  // 无效情况下，返回0
  // 超过int范围，返回MAX_VALUE或者MIN_VALUE
  if(str.length() == 0) return 0;
  char[] arr = str.toCharArray();
  int len = arr.length;
  int i = 0, res = 0, sign = 1;
  while(i<len && arr[i] == ' ') i++;
  if(i == len) return 0; // 全是0，如果能用trim这里可以省略
  if(arr[i] == '+'){
    i++;
  } else if(arr[i] == '-'){
    sign = -1;
    i++;
  }
  while(i < len){
    if(arr[i] < '0' || arr[i] > '9') break;
    if(res > Integer.MAX_VALUE/10 || res == Integer.MAX_VALUE/10 && arr[i] > '7') 
      return sign == 1? Integer.MAX_VALUE:Integer.MIN_VALUE;
    res = res*10 + (arr[i]-'0');
    i++;
  }
  return sign*res;
}
```



#### T68 









```

```









### 面试题

#### 字节跳动

1. 先自我介绍
2. 线程池的线程数怎么确定？
3. 如果是IO操作为主怎么确定？
4. 如果计算型操作又怎么确定？
5. Redis熟悉么，了解哪些数据结构?
6. 跳表的查询过程是怎么样的，查询和插入的时间复杂度?
7. 红黑树了解么，时间复杂度?
8. 既然两个数据结构时间复杂度都是O(logN)，zset为什么不用红黑树
9. 说下Dubbo的原理?
10. CAS了解么？
11. 那我们做一道题吧，数组A，2*n个元素，n个奇数、n个偶数，设计一个算法，使得数组奇数下标位置放置的都是奇数，偶数下标位置放置的都是偶数
12. 先说下你的思路
13. 下一个奇数？怎么找？
14. 有思路么？
15. 你这样时间复杂度有点高，如果要求O(N)要怎么做
16. 时间差不多了，先到这吧。你有什么想问我的？
17. 你对服务治理怎么理解的？
18. 项目中的限流怎么实现的？具体怎么实现的？
19. 如果突然很多线程同时请求令牌，有什么问题？怎么解决呢？如果不用消息队列怎么解决？
20. 分布式追踪的上下文是怎么存储和传递的？
21. Dubbo的RpcContext是怎么传递的？
22. 你说的内存泄漏具体是怎么产生的？
23. 线程池的线程是不是必须手动remove才可以回收value？
24. 那你说的内存泄漏是指主线程还是线程池？
25. 可是主线程不是都退出了，引用的对象不应该会主动回收么？
26. 那你说下SpringMVC不同用户登录的信息怎么保证线程安全的？
27. 这个直接用ThreadLocal不就可以么，你见过SpringMVC有锁实现的代码么？
28. 我们聊聊mysql吧，说下索引结构
29. 为什么使用B+树？
30. 什么是索引覆盖？
31. Java为什么要设计双亲委派模型？
32. 什么时候需要自定义类加载器？
33. 手写一个对象池



#### 阿里巴巴

第一道题：蚂蚁森林n个小动物，1~n,小动物编号越小能力越强，现在筛选国王，每个小动物都会崇拜别的小动物或者自己，但只会崇拜比自己能力强的小动物。问每个人最多可以获得多少票。

图的路径搜索，但是不会就没做，后来上网查了floyd和Dijkstra(迪杰斯特拉)算法都可以破解，但floyd可能会超时

1. 单机部署项目，问题很多，版本迭代就要kill进程，有什么问题？你有想过怎么解决好点吗？

   后面问了问朋友，生产直接kill是不行的，强制杀死容易出问题，Kill这个操作不是生产操作，最简单就是布完包重启tomcat。就是取代kill操作啊，但还是以前的部署操作。（唉，我确实菜，项目经验确实不够）

2. 你的项目很慢，你应该做什么排查？例如查询一个数据很慢，你会怎么做？

   我那时候一开始就说对项目的优化嘛，我直接就说建立索引什么的，我以为他会要我说说索引原理什么嘛，结果她问我怎么确定就是数据库的问题，不是其他的问题，要怎么排查

   我确实不知道，后面补充一下：

   可以从应用，数据库，和运行环境分析；可以考虑用户网络环境，然后是应用中的调用链路是否存在问题（循环调用？外部依赖过多？），然后是数据库。有应用监控系统最好不过了，可以比较具体的排查。

   数据库的原因也很多，sql问题，也有可能是数据库本身延时高。

   Linux查看一下进程也需要，查看进程排查应用所在环境因素（其他进程对该应用程序的影响）

   而且索引也不是绝对的优化方法，可能本身的sql是否可以优化，如果连表过多，是否可以拆解到应用层做，或是压根表设计不合理

3. 类的加载？——》

   双亲委派的过程？——》

   还问为什么要双亲委派？

   补充：
   确实是安全，如果没有这种机制，编写了一个java.lang.Object的同名类并放在ClassPath中，程序一跑，多个Object继续加载，就不能保证object的唯一性。

   因此，解决就说通过类加载机制。底层是代理模式，对于 Java 核心库的类的加载工作由引导类加载器来统一完成，保证了 Java 应用所使用的都是同一个版本的 Java 核心库的类，是互相兼容的。

4. 三个工厂模式，有什么区别

   答案：

   三个工厂模式，各有千秋

   从简单工厂模式——》工厂方法模式，解决了对产品的拓展不符合OCT原则的问题

   从工厂方法模式——》抽象工厂模式，解决了一个过程只能生产一个产品的问题

   但是反而多了一个问题，就是又产生了部分不符合OCT原则的问题，对工厂的拓展符合OCT，但是没错要拓展一个产品，就要修改一次工厂里面的方法

5. 高并发的concurrenthashmap

   推荐使用，
   在jdk8之前是使用分段加锁的一个方式，分成16个桶，每次只加锁其中一个桶，而在jdk8又加入了红黑树和CAS算法来实现。
   每次只会锁目前一个segment，用synchronized+CAS，效率更高了，并发度更高。

   问题：concurrenthashmap，有一个线程进入这一个桶，进行put方法，他还能再进入吗？

   妈呀，我竟然说不可以，前面都采用synchronized了，synchronized是可重入锁啊！！！后面纠正过来了，但是。。

6. spring容器启动过程

   spring我配置文件在哪里读取？


