---
title: leetcode300
date: 2022-10-24 20:00:00
tags: DP
categories: Algorithm Problem
math: true
---

#### [300. 最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

**示例 1：**

```
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
```

**示例 2：**

```
输入：nums = [0,1,0,3,2,3]
输出：4
```

**示例 3：**

```
输入：nums = [7,7,7,7,7,7,7]
输出：1
```

 

**提示：**

- `1 <= nums.length <= 2500`
- `-104 <= nums[i] <= 104`

 

**进阶：**

- 你能将算法的时间复杂度降低到 `O(n log(n))` 吗?

#### 题解

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        if(n <= 1) return n;

        int[] tail = new int[n];
        int end = 0;
        tail[end] = nums[0];

        for(int i = 1; i < n; i++){
            if(nums[i] > tail[end]){
                tail[++end] = nums[i];
            }else{
                int l = 0, r = end;
                while(l < r){
                    int m = (l + r) >> 1;
                    if(tail[m] < nums[i])
                        l = m + 1;
                    else
                        r = m;
                }
                tail[l] = nums[i];
            }
        }

        return ++end;
    }
}
```

关键是构建tail数组，厘清两种情况



> 作者：LeetCode-Solution
> 链接：https://leetcode.cn/problems/longest-increasing-subsequence/solution/zui-chang-shang-sheng-zi-xu-lie-by-leetcode-soluti/
> 来源：力扣（LeetCode）
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

