# 数据结构与算法

### 并查集

并查集：
+ 用数组来存某节点 `i` 的父节点 `parent` ；
+ 最开始父节点都是它本身，即 `i`   指向 `i`；
+ 合并操作：将一个变量的根结点的指向第二个变量的根节点；
+ 查找操作：沿着当前变量的父节点一路向上查找，直到父节点等于本身；
+ 路径压缩：可以考虑在合并的时候，秩优化

##### Leetcode 990 等式方程的可满足性

```
输入：["a==b","b!=a"]
输出：false
```

```java
class Solution {
  public boolean equationsPossible(String[] equations){
    int length = equations.length;
    int[] parent = new int[26];
    for(int i = 0; i < 26; i++) parent[i] = i;
    for(String str : equations){
      if(str.charAt(1) == '='){
        int index1 = str.charAt(0) - 'a';
        int index2 = str.charAt(3) - 'a';
        union(parent, index1, index2);
      }
    }
    for(String str : equations){
      if(str.charAt(1) == '!'){
        int index1 = str.charAt(0) - 'a';
        int index2 = str.charAt(3) - 'a';
        if(find(parent, index1) == find(parent, index2)) return false;
      }
    }
    return true;
  }
  private void union(int[] parent, int index1, int index2){
    parent[find(parent, index1)] = find(parent, index2);
  }
  private int find(int[] parent, int index){
    while(parent[index] != index){
      parent[index] = parent[parent[index]]; // 边找边优化
      index = parent[index]
    }
    return index;
  }
}
```





### 动态规划

#### 求最值

最优化的问题，如果只问结果，不问过程，就是用「动态规划」去求解的

框架：这个问题有什么【状态】？有什么【选择】？

**找到状态和选择 -> 明确 dp 数组/函数的定义 -> 寻找状态之间的关系**。

**1、遍历的过程中，所需的状态必须是已经计算出来的**。

**2、遍历的终点必须是存储结果的那个位置**

最典型的动态规划：斐波那契数列/跳台阶

#### 凑零钱问题

> 给你 `k` 种面值的硬币，面值分别为 `c1, c2 ... ck`，每种硬币的数量无限，再给一个总金额 `amount`，问你**最少**需要几枚硬币凑出这个金额，如果不可能凑出，算法返回 -1 。

#### 最长递增子序列

> LIS问题：子序列不一定是连续的，子串是连续的

```java
public int lengthOfLIS(int[] nums) {
    if(nums.length <= 0) return 0;
    int[] dp = new int[nums.length];
    Arrays.fill(dp, 1);
    int max = 1;
    for(int i = 1; i < nums.length; i++){
        for(int j = 0; j < i; j++){
            if(nums[i] > nums[j]) dp[i] = Math.max(dp[i], dp[j]+1);
        }
        max = Math.max(max, dp[i]);
    }
    return max;
}
```

二分法求解最长递增子序列

```java

```



#### 高楼扔鸡蛋

> 你面前有一栋从 1 到 `N` 共 `N` 层的楼，然后给你 `K` 个鸡蛋（`K` 至少为 1）。现在确定这栋楼存在楼层 `0 <= F <= N`，在这层楼将鸡蛋扔下去，鸡蛋**恰好没摔碎**（高于 `F` 的楼层都会碎，低于 `F` 的楼层都不会碎）。现在问你，**最坏**情况下，你**至少**要扔几次鸡蛋，才能**确定**这个楼层 `F` 呢？

**最坏**：最坏情况是指，鸡蛋的破碎一定发生在搜索区间穷尽的时候；如果第一层楼就破了，那就可以结束测试了！

**至少**：选择查找思路，使扔的次数最小

【状态】当前拥有的鸡蛋数量K和需要测试的楼层数N，随着测试的进行，鸡蛋个数减少，楼层搜索范围会减小

【选择】选择去哪一层楼扔鸡蛋

当选择在 `i`  层扔了鸡蛋：

+ 鸡蛋碎了：鸡蛋个数 K-1，搜索区间【1，N】变成【1，i-1】
+ 鸡蛋没碎：鸡蛋个数不变 K，搜索区间【i+1，N】

基础情况：

+ 楼层为0的时候，无论鸡蛋几个，结果都是0；
+ 楼层为1的时候，0个鸡蛋，结果是0；有鸡蛋>=1 ，结果1；
+ 鸡蛋0个，结果总是0；
+ 鸡蛋1个，结果等于楼层的高度；
+ 表格需要dp[N+1]\[K+1]
+ 结果是dp[N]\[K]

$$
dp[i][j]= 
min	
1≤k≤i

 (max(dp[k−1][j−1],dp[i−k][j])+1)
$$

- 时间复杂度：O(N^2K)*O*(*N*2*K*)，三层 `for` 循环，每层循环都是线性的；
- 空间复杂度：O(NK)*O*(*N**K*)，表格的大小。

```java
public int superEggDrop(int K, int N){
  // dp[i][j] i代表楼层，j代表鸡蛋个数，一共i层楼，j个鸡蛋做最少的实验次数
  int[][] dp = new int[N+1][K+1];
//  for(int i = 0; i <= N; i++){
//    Arrays.fill(dp[i], Integer.MAX_VALUE);
//  }
  for(int j = 0; j <=K; j++){
    // 填写楼层为0，1的时候的结果
    dp[0][j] = 0;
    dp[1][j] = 1;
  }
  dp[1][0] = 0;
  for(int i = 2; i <= N; i++){
    dp[i][0] = 0;
    dp[i][1] = i;
  }
  for(int i = 2; i <= N; i++){
    for(int j = 2; j <= K; j++){
      //int min = Integer.MAX_VALUE;
      // dp[i][j]是把这i层楼都测试一遍得到的最小值
      //for(int k = 1; k <= i; k++){
      //  min = Math.min((Math.max(dp[k-1][j-1], dp[i-k][j])+1), min);
      //}
      //dp[i][j] = min;
      
    	// 在这里可以用二分来找到 dp[k-1][j-1], dp[i-k][j]的交点
        int left = 1, right = i;
        while(left < right){
            int mid = (left + right) >>> 1;
            int breakcount = dp[mid-1][j-1];
            int nonbreakcount = dp[i-mid][j];
            if(nonbreakcount > breakcount) left = mid + 1;
            else right = mid;
        }
        dp[i][j] = Math.max(dp[left-1][j-1], dp[i-left][j]) +1;
    }
  }
  return dp[N][K];
}
```



**第二种思路**

修改 `dp` 数组的定义，**确定当前的鸡蛋个数`k`和最多允许的扔鸡蛋次数`m`，就知道能够确定 `F` 的最高楼层 `N`数**。

例如dp[1]\[7] = 7

**1、无论你在哪层楼扔鸡蛋，鸡蛋只可能摔碎或者没摔碎，碎了的话就测楼下，没碎的话就测楼上**。

**2、无论你上楼还是下楼，测试总数 = 去楼上测试的次数 + 去楼下测试的次数 + 1（当前这层楼）**。 
$$
dp[k][m] = dp[k][m - 1] + dp[k - 1][m - 1] + 1
$$

```java
public int superEgg(int K, int N){
  int[][] dp = new int[K+1][N+1];
  int m = 0;
  while(dp[K][m] < N){
    m++;
    for(int i = 1; i <= K; i++){
      dp[i][m] = dp[i][m-1] + dp[i-1][m-1] +1;
    }
  }
  return m;
}
```



### 贪心算法--动态规划的特殊情况

每次都求最优解，以局部最优得到全局最优

贪心是**动态规划的一种特殊类型**的题目

#### 凑零钱问题

如果用一般的动态规划，逐个尝试所有币值的零钱就好。但是贪心思想要求从最大的零钱开始。



#### 区间问题

碰到区间问题的整体思路就是：**先按照某种顺序排序（左区间或者右区间），然后遍历他们**

##### 区间选点

给定N个闭区间[ai,bi]，请你在数轴上选择尽量少的点，使得每个区间内至少包含一个选出的点。输出选择的点的最小数量。

应用情况：射爆气球（选1个点）。

思路：

+ 按照右区间从小到大排序；选取第一个右区间作为目标点；
+ 遍历，如果当前区间已包含目标点（左区间<=目标点），continue；否则ans+1，把当前区间的右区间作为目标点；

变式：

每个区间选2个点：区别在于左区间<目标点，continue；左区间=目标点，ans+1，更新目标点；左区间>目标点，ans+2，更新目标点。



##### 最多不相交区间

给定N个闭区间，选择若干区间，使得选中的区间之间互不相交（包括端点）。输出可选取区间的最大数量。

应用情况：选课/参加活动，选择数量最多

思路：同上



##### 区间分组

给定N个闭区间，将这些区间分成若干组，使得每组内部的区间两两之间（包括端点）没有交集，并使得组数尽可能小。输出最小组数。

应用情况：公司今天有20场会议，问最少用几个会议室可以安排下

思路：

+ 按照左区间从小到大排序；
+ 判断能够将当前区间，判断能否放到某个现有的组中？左区间>所有已存在组右区间的最小值（小根堆来维护）

```java
import java.util.*;
public class Main{
	public static void main(String[] args){
		Scanner sc = new Scanner(System.in);
    int N = sc.nextInt();
    int[][] list = new int[N][2];
    for(int i = 0; i < N; i++){
      int l = sc.nextInt();
      int r = sc.nextInt();
      list[i] = new int[]{l, r};
    }
    System.out.print(partition(list));
	}
  
  private static int partition(int[][] list){
    Arrays.sort(list, (p1,p2)->p1[0]-p2[0]);
    PriorityQueue<Integer> queue = new PriorityQueue<>();
    for(int[] p : list){
      if(queue.isEmpty() || queue.peek() >= p[0]) queue.offer(p[1]);
      else {
        queue.poll();
        queue.offer(p[1]); 
      }
    }
    return queue.size();
  }
}
```



##### 区间覆盖

给定N个闭区间[ai,bi]以及一个线段区间[s,t]，请你选择尽量少的区间，将指定线段区间完全覆盖。

输出最少区间数，如果无法完全覆盖则输出-1。

思路：

+ 按照左区间从小到大排序；
+ 在所有能覆盖start的区间中选择右区间最大的，然后将start更新为右区间的最大值。



### 二分

二分的应用场景：

+ 单调性
+ 两段性：一部分满足，一部分不满足

模版：

先确定左右区间的取值，注意边界条件

```java
while(left<right){
	int mid = (left+right)>>>1;
  if(???) left = mid+1;
  else right = mid;
}
// 出循环的时候left==right
```

应用：

+ 二分查找：如果nums[mid] == target 直接返回

+ 开方：范围1～target进行二分查找，考虑0，1特例

+ Lc33. 搜索旋转排序数组：先定位旋转点，再对左/右半边进行二分查找；也可以直接进行二分（逻辑会绕一点）

+ Lc34. 搜索一个数的范围：用右边界二分查找的方法，查找target和target-1，得到第一个大于目标值的位置。

+ Lc35. 搜索插入位置： 查找第一个比target大的位置，检查前一个值是不是target，考虑结果位置==0的情况：left > 0 && nums[left-1] == target? left-1:left;

+ Lc74. 搜索二维矩阵：矩阵严格递增，通过坐标转换当作一维数组来二分查找。也可以从右上角开始查找。

+ Lc153. 寻找旋转数组中的最小值：

+ Lc154. 寻找旋转数组中的最小值2：数组中可能有重复的元素，这时候找到的仅仅是最小值，不一定是旋转点。在mid的值等于right的值，right--

+ Lc162. 寻找峰值：二分查找，将mid和mid+1对比，如果mid大，峰值在左边，right = mid；如果mid小，left = mid+1

+ **Lc275. H指数2：** 未排序的H指数题目，可以先排序然后再用二分来查找（或者直接用计数排序法）；已经排序的题目，找到第一个（len-i）> citations[i] ，此时答案是（len-i）

+ Lc287. 寻找重复数：二分查找，对比区间内理论上应该出现的数目，与实际出现的数目

+ Lc300. 最长上升子序列：维护一组堆，由一个top数组来保存堆最小的值。遍历数组，对每个数组，二分查找已存在的堆，如果不能放进堆中（值比所有最小值都大），新开一个堆。最后堆的个数就是子序列的长度。

+ Lc315. 计算右侧小于当前元素的个数：本质上是求有多少（逆序对）

  + 逆序对相关的题目，首先想到归并排序，但是如何在归并的之后重新定位索引

  + 树状数组：

    ```java
    class Solution {
        public List<Integer> countSmaller(int[] nums) {
            if (nums.length == 0) {
                return new ArrayList<>();
            }
    
            //nums数组预处理，求最小值/最大值
            int min = Integer.MAX_VALUE;
          	int max = Integer.MIN_VALUE;
            for (int value : nums) {
                if (value < min) {
                    min = value;
                }
            }
            //减去最小值，保证都是正数，最小1
            for (int i = 0; i < nums.length; i++) {
                nums[i] = nums[i] - min + 1;
              	max = Math.max(max, num[i]);
            }
          
            int[] BITree = new int[max + 1];
            BITree[0] = 0;
            int[] countArr = new int[nums.length];
          	// 从最后一个值开始update 树 
            for (int i = nums.length - 1; i >= 0; i--) {
                int count = getSum(nums[i] - 1, BITree);
                countArr[i] = count;
                update(nums[i], BITree);
            }
            List<Integer> result = new ArrayList<>();
            for (int value : countArr) {
                result.add(value);
            }
            return result;
        }
        //获得a[i]从1，value的和
        private int getSum(int value, int[] BITree) {
            int sum = 0;
            while (value > 0) {
                sum += BITree[value];
                value -= lowbit(value);
            }
            return sum;
        }
    
        //更新树状数组
        private void update(int value, int[] BITree) {
            while (value <= BITree.length - 1) {
                BITree[value] += 1;
                value += lowbit(value);
            }
        }
    
        //求出m的二进制表示的末尾1的位置
        private int lowbit(int x){
            return x & (-x);
        }
    }
    ```

    

  + 插入排序+二分：从最右端开始，右边的元素都是排好序的，所以可以用二分来查找需要插入的位置。

  + **插入排序的思路简单，但是时间复杂度很高。但如果是机考，用这个写简单。**

    ```java
        public List<Integer> countSmaller(int[] nums) {
            List<Integer> ans = new ArrayList<>();
            if(nums.length <= 0) return ans;
            int[] res = new int[nums.length];
            for(int i = nums.length-2; i >=0; i--){
                int left = i+1, right = nums.length;
                while(left < right){
                    int mid = (left + right)>>>1;
                    if(nums[mid] < nums[i]) left = mid +1;
                    else right = mid;
                }
                res[i] = (left-i-1);
                int tmp = nums[i];
                for(int k = i; k < left-1; k++){
                    nums[k] = nums[k+1];
                }
                nums[left-1] = tmp;
            }
            for(int n : res){
                ans.add(n);
            }
            return ans;
        }
    ```

    



### 排序

#### 冒泡/插入 $$n^2$$

```java
public void bubbleSort(int[] nums){
  for(int i = nums.length-1; i >= 0; i--){
    for(int j = 0; j < i; j++){
      if(nums[j] > nums[j+1]) swap(nums, j, j+1);
    }
  }
}
// 标记位置的冒泡排序
public void bubbleSort2(int[] nums){
  int flag = nums.length-1;
  while(flag>0){
    int i = flag;
    flag = 0;
    for(int j = 0; j < i; j++){
      if(nums[j] > nums[j+1]) {
        swap(nums, j ,j+1);
        flag = j+1;
      }
    }
  }
}
private void swap(int[] nums, int i, int j){
  int tmp = nums[i];
  nums[i] = nums[j];
  nums[j] = tmp;
}
```

#### 快排 nlogn-$$n^2$$

快排的思想：递归和分治

```java
public void quickSort(int[] nums, int start, int end){
  if(start==end) return;
  int pivotIndex = partition(nums, start, end);
  if(pivotIndex>start) quickSort(nums, start, pivotIndex-1);
  if(pivotIndex<end) quickSort(nums, pivotIndex+1, end);
}
private void int partition(int[] nums, int start, int end){
  int pivot = nums[start];
  int lt = start + 1, gt = end;
  while(true){
    while(lt<=end && nums[lt] < pivot) lt++;
    while(gt>=start && nums[gt] > pivot) gt--;
    if(lt>gt) break;
    swap(nums, lt, gt);
    lt++;
    gt--;
  }
  swap(nums, start, gt);
  return gt;
}
```

应用场景：

+ TopK 问题：为啥我用这个快排模版就比别人慢很多？？

  

#### 归并 nlogn

```java
mergeSort(nums, 0, len - 1，new int[len]);
public void mergeSort(int[] nums, int start, int end, int[] tmp){
  if(start == end) return;
  int mid = (start + end)>>>1;
  mergeSort(nums, start, mid, tmp);
  mergeSort(nums, mid+1, end, tmp);
  if(nums[mid]<=nums[mid+1]) return;
  mergeTwo(nums, start, mid, end, tmp);
}
private void mergeTwo(int[] nums, int start,int mid, int end, int[] tmp){
  System.arraycopy(nums, start, tmp, start, end-start+1);
  int i = start, j = mid + 1;
  for(int k = start; k <= end; k++){
    if(i == mid+1) nums[k] = tmp[j++];
    else if(j == end+1) nums[k] = tmp[i++];
    else if(nums[i] < nums[j]) nums[k] = tmp[i++];
    else nums[k] = tmp[j++];
  }
}
```

应用：

+ 剑指51：逆序对统计：在mergeTwo的时候统计即可
+ Lc493 ：翻转对：nums[i] > (2* (long)nums[j]) 在merge之前统计，考虑负数，溢出情况
+ lc315：计算右侧小于当前元素的个数：在二分中出现过，可以用树状数组、插入+二分、归并
+ lc327：区间和的个数：
+ Lc148 ：排序链表：



#### 桶排序

- [LeetCode 164. Maximum Gap (hard)](https://github.com/muyids/leetcode/blob/master/algorithms/101-200/164.maximum-gap.md)
- [LeetCode 220. Contains Duplicate III (medium)](https://github.com/muyids/leetcode/blob/master/algorithms/201-300/220.contains-duplicate-iii.md)
- [LeetCode 451. Sort Characters By Frequency (medium)](https://github.com/muyids/leetcode/blob/master/algorithms/401-500/451.sort-characters-by-frequency.md)



### 字符串

#### 大数乘法

```java
// 核心内容！！！
int[] res = new int[len1+len2];
for(int i = len1-1; i >= 0; i--){
    for(int j = len2-1; j >= 0; j--){
        int ans = (arr1[i]-'0') * (arr2[j]-'0') + res[i+j+1];
        res[i+j+1] = ans %10;
        res[i+j] += ans/10;
    }
} 

```



#### KMP

字符串匹配：

```java
// 关键是getNext（）
public int[] getNext(char[] p){
  int len = p.length;
  int[] next = new int[len];
  int k = -1, j = 0;
  next[0] = -1;
  while(j < len-1){
    if(k == -1 || p[j] = p[k]){
      next[++j] = ++k;
    } else {
      k = next[k];
    }
  }
}
public int kmpIndex(String source, String pattern){
  char[] s = source.toCharArray();
  char[] p = pattern.toCharArray();
  int i = 0, j = 0;
  while(i<s.length && j < p.length){
    if(j == -1 || s[i] == p[j]){
      i++;
      j++;
    } else {
      j = next[j];
    }
  }
  if(j == p.length){
    return i-j;
  }
  return -1;
}
```

- LeetCode 38.  外观数列 ：

  + 求第n个数，从头开始模拟生成过程。
  + 遍历上一个字符串，当前i和下一个i+1不同的时候，append一次，遇到结尾len-1的时候append一次

- LeetCode 49. 字母异位词分组：遍历每个单词，对单词的字母重新排序，查找hashmap里是否已经存在。

- LeetCode 151.  翻转字符串里的单词：双指针从尾巴遍历字符串，找到两个空格之间的下标，append一次

- LeetCode 165.  比较版本号： 易错的地方在分割。用3个while，分别遍历1，2共同的长度，1或者2剩余的部分

  ```java
  String[] v1 = version1.split("\\.");
  ```

- LeetCode 929.  独特的电子邮件 ：单独处理一下两个特殊符号就好了

- LeetCode 5.  最长回文子串：中心扩散法：考虑奇数和偶数两种情况

- LeetCode 6. Z字变换：找数学规律

- LeetCode 208. 前缀树：详细看前缀树部分

- [LeetCode 273. Integer to English Words (hard)](https://github.com/muyids/leetcode/blob/master/algorithms/201-300/273.integer-to-english-words.md)



### DFS和回溯

排列组合、棋盘搜索、树形问题

#### 汉诺塔-递归

三塔汉诺塔问题：先把n-1个放到中间塔，最大的一个放到目标塔，再把n-1个从中间塔放到目标塔

可以用递归，也可以动态规划 dp[i] = dp[i-1] *2 + 1;



四塔汉诺塔问题：先在四座塔的情况下，将i个放到中间塔，剩余的n-i个放到目标塔就转换成了三塔问题，最后把i个放回目标塔；

Fourdp[i] = min(fourdp[j] * 2+ dp[i-j] )，j的 取值范围 [0，i )



#### N皇后-回溯



#### 排列组合问题

- LeetCode 46. Permutations 
- LeetCode 47. Permutations II 

- LeetCode 996. Number of Squareful Arrays 

- LeetCode 39. Combination Sum (medium)
- LeetCode 40. Combination Sum II (medium)
- LeetCode 216. Combination Sum III (medium)

- LeetCode 93. Restore IP Addresses (medium)： IP地址这道题，每次截取长度为1，2，3的子串，判断数值是否有效。关键在于判断有效情况，要考虑0开头的字符串。实际上还可以剪枝，但我懒得想了

- LeetCode 131. Palindrome Partitioning (medium)：分割回文串：从头开始分割，然后check

  可以用动态规划来优化：check中每一次都重复检查，可以先用动态规划求出最长回文子串，这样check就是O（1）；



#### 子集问题

- LeetCode 78. Subsets (medium) ：回溯枚举出所有可能情况

- LeetCode 90. Subsets II (medium)：类似上边的组合问题，有重复的值，就需要先排序，然后剪掉重复项；

  

#### 棋盘搜索

- LeetCode 351. Android Unlock Patterns (medium) ：安卓手机的手势解锁，需要经过m-n个数。处理边界情况：n<1 || m <0 || m > n 直接返回0，n大于9的情况，按照9来处理；

  同样是回溯，需要记录哪些数字已经经过了，check部分需要分类讨论：

  + 1-9 ｜｜ 3-7 的情况：中间要跨过5，需要5已经访问过、目标数没有访问才行；
  + 1-7 ｜｜ 2-8 ｜｜ 3-9 ：中间要跨过4，5，6，需要中间数已经访问过、目标数没有访问才行；
  + 1-3 ｜｜ 4-6 ｜｜ 7-9：中间要跨过2，5，8，需要中间数已经访问过、目标数没有访问才行；
  + 其他情况，没有中间数，只要目标数没有访问就可以。

- LeetCode 329. Longest Increasing Path in a Matrix (hard)

- [LeetCode 52. N-Queens II (hard)](https://github.com/muyids/leetcode/blob/master/algorithms/1-100/52.n-queens-ii.md)

- LeetCode 37. Sudoku Solver (hard)

- LeetCode 473. Matchsticks to Square (medium)



#### 遍历图

+ LC 133 克隆图：有点类似复杂链表的复制，利用一个hashmap来保存已经clone过的node
+ 



#### 其他

- [LeetCode 22. Generate Parentheses (medium)](https://github.com/muyids/leetcode/blob/master/algorithms/1-100/22.generate-parentheses.md)
- [LeetCode 131. Palindrome Partitioning (medium)](https://github.com/muyids/leetcode/blob/master/algorithms/101-200/131.palindrome-partitioning.md)
- [LeetCode 306. Additive Number (medium)](https://github.com/muyids/leetcode/blob/master/algorithms/301-400/306.additive-number.md)
- [LeetCode 17. Letter Combinations of a Phone Number (medium)](https://github.com/muyids/leetcode/blob/master/algorithms/1-100/17.letter-combinations-of-a-phone-number.md)





### 单调栈

题目和next相关的，局部最大值/最小值

- [LeetCode 239. Sliding Window Maximum (hard)：维护单调递减队列，队头元素是当前窗口的最大值
- [LeetCode 42. Trapping Rain Water (hard)：维护单调递减栈，遇到大的，计算中间能装雨水的面积
- [LeetCode 84. Largest Rectangle in Histogram (hard)：维护单调递增栈，遇到小的，计算前面的最大面积
- [LeetCode 85. Maximal Rectangle (hard)：动态规划题，用单调栈的话，要怎么理解？相当于对【0-i】行的数组构成的长方形求最大面积，每增加一行，重新构建长方形，重新求一次当前的最大面积。
- [LeetCode 402. Remove K Digits (medium)：从数字字符串中删掉这个数中的k位数字，小的数在前更小，所以维护一个单调递增的栈，一旦遇到比栈顶小的数/或者到结尾了，删除栈中的数字。
- leetcode 496 下一个更大的元素1：
- [LeetCode 503. Next Greater Element II (medium)
- [LeetCode 768. Max Chunks To Make Sorted II (hard)
- [LeetCode 739. Daily Temperatures (medium)
- [LeetCode 901. Online Stock Span (medium)
- [LeetCode 1019. Next Greater Node In Linked List (medium)
- [LeetCode 907. Sum of Subarray Minimums (medium)





### 二叉树

#### 根据前序遍历复原二叉树

用栈和迭代模拟递归：

前面的爹，缺儿子的都入栈，当遇到一个儿子，看儿子的level和栈顶的level匹不匹配。不匹配，栈顶出栈直到匹配。

```java
public TreeNode recoverFromPreorder(String S) {
    if(S.length() <= 0) return null;
    //if(S.length() == 1) return new TreeNode(S.charAt(0)-'0');
    LinkedList<TreeNode> stack = new LinkedList<>();
    int i = -1, j = 0;
    while(j < S.length()){
        if(S.charAt(j) != '-'){
            int k = j;
            while(k < S.length() && S.charAt(k) != '-') k++;
            TreeNode node = new TreeNode(Integer.valueOf(S.substring(j,k)));
            while(!stack.isEmpty() && stack.size() > (j-i-1)) stack.pop();
            if(!stack.isEmpty()) {
                TreeNode parent = stack.peek();
                if(parent.left == null) parent.left = node;
                else parent.right = node;
            }
            stack.push(node);
            i = k-1;
            j = k;
        } else {
            j++;
        }
    }
    return stack.pollLast();
}
```



### Nim游戏

对于一个Nim游戏的局面(a1,a2,...,an)，它是P-position当且仅当a1^a2^...^an=0，其中^表示[异或]运算。

1. P-position，在该状态，上一个移动的玩家(**P**revious player)能够获胜，也就是后手必胜
2. N-position，在该状态，下一个移动的玩家(**N**ext player)能够获胜，也就是先手必胜
3. 对于每一个 P-position，对于任何一个合法的移动下的下一个状态一定是一个 N-position
4. 对于每一个 N-position，一定存在一个合法移动，使得下一个状态是 P-position



（0，0，0）是一个P点，先拿的人会输

（0，0，x）是一个N点，先拿的人会赢

（0，x，x）是一个p点，先拿的人会输

思路：

+ 先对所有堆的值进行异或，如果结果是0，则当前是一个p点，先拿的人会输，返回
+ 结果非0，当前是n点，要把nimsum变成0
+ nimsum依次异或堆的值，找到那个能将nimsum变成0的数字：
  + int tmp = nimsum^arr[i]; 
  + if(arr[i]-tmp > 0) 表明找到这个数





### 字典序

给定整数n和m, 将1到n的这n个整数按字典序排列之后, 求其中的第m个数。
对于n=11, m=4, 按字典序排列依次为1, 10, 11, 2, 3, 4, 5, 6, 7, 8, 9, 因此第4个数是2.

```java
import java.util.*;
public class Main{
    public static void main(String[] args){
        Scanner sc = new Scanner(System.in);
        long N = sc.nextLong();
        long M = sc.nextLong();
        System.out.print(dicM(N, M));
    }
    private static long dicM(long N, long M){
				// 先确定第M个数是什么数字开头的？
      	// 如果是1开头，那么1-N之间有多少个1开头的数字？
      	// cntNum函数就是用来统计一共有多少个pre开头的数字存在
      	// 如果1开头的count个数字大于等于M,那么M确实是1开头，pre*10，M--，进一步检查能不能以10、11、-、19数组开头；
      //如果1开头的数字count小于M，那么1开头的数字不够M，需要进一步检查能不能以2开头，pre++，M-=count
    }
    private static long cntNum(long pre, long N){

    }
}
```



#### N个数两两异或

给定整数m以及n各数字A1,A2,..An，将数列A中所有元素两两异或，共能得到n(n-1)/2个结果，请求出这些结果中大于m的有多少个。

 





### 前缀树Trie

用于检索字符串数据集中的键。

先构造TrieNode：

+ 一个存放子节点的数组；一个final值指定数组的大小；
+ 一个标志位，表示这个节点是不是单词的end；
+ 针对子节点的操作：是否已存在，获取子节点，存储子节点；
+ 针对标志位的操作：查询，设置；
+ 构造函数

再构造Trie：

+ 一个root节点，新构造时为空；
+ 插入字符串（一组子节点）：如果有前缀已存在，就在此基础上继续添加；
+ 查询前缀：遍历前缀字符串的字符，直到结束或者某个字符不存在；
+ 查询字符串（一组子节点，最后一个节点需要是end）：利用查询前缀，如果返回的结果非空（字符都存在），并且是end；
+ 查询是否startWith：利用查询前缀，返回结果是非空的即可，不需要是end。



1. LC 208 ：实现前缀树

2. LC 211：添加与搜索单词：这道题包括了模式匹配，addword的时候是一样的，但是search的时候要处理特殊字符，需要用到递归。

   ```java
   private boolean find(TrieNode curNode, String word){
       for(int i = 0; i < word.length(); i++){
           char ch = word.charAt(i);
           if(ch == '.'){
               for(TrieNode child : curNode.getChild()){
                   if(child != null){
                       if(find(child, word.substring(i+1))) return true;
                   }
               }
               return false;
           } else {
               TrieNode child = curNode.getChild()[ch-'a'];
               if(child != null){
                   curNode = child;
               } else{
                   return false;
               }
           }
       }
       return curNode.getEnd();
   }
   ```

3. 单词搜索：一般情况可以用回溯来处理，数据量更大的时候回溯会超时，所以需要用前缀树来处理



### 线段树 & 树状数组

#### 区间求和问题？

一组序列中一部分连续元素累加的和。

+ 方法一：前缀和：【i，j】 = 【1，j】-【1，i-1】
+ 方法二：树状数组Fenwick树：**适用于区间中的元素是动态更新的情况**。利用倍增思想与二进制特性来降低更新元素的时间复杂度。



### java集合类对比分析

1. 底层数据结构
2. 增删改查方式
3. 初始容量，扩容方式，扩容时机
4. 线程安全与否
5. 是否允许空，是否允许重复，是否有序

**ArrayList**

初始容量：10，底层是object数组

增删慢：在每次添加新的元素时，ArrayList都会检查是否需要进行扩容操作（1.5倍），数据向新数组的重新拷贝；**解决办法**：在构造ArrayList时可以给ArrayList指定一个初始容量，这样就会减少扩容时数据的拷贝问题；当然在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。

不同步：如果多个线程同时访问一个ArrayList实例。同步处理：

```java
List list = Collections.synchronizedList(new ArrayList(...)); 
```

