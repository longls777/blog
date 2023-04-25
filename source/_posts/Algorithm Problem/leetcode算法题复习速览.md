---
title: LeetCode算法题复习速览
date: 2023-3-21 13:42:00
tags: 
categories: Algorithm Problem
math: true
---

> https://github.com/labuladong/fucking-algorithm
>
> https://labuladong.gitee.io/algo/

## 链表

- ##### 双指针

  - [21.合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)
  - [86. 分隔链表](https://leetcode.cn/problems/partition-list/)
  - [23. 合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)
  - [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)
  - [19. 删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
  - [876. 链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)
  - [160. 相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)
  - [141.环形链表](https://leetcode.cn/problems/linked-list-cycle/)
  - [142. 环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/) 
  - [83. 删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)

- [25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/description/)






## 数组

- ##### 双指针

  - [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)
  - [27. 移除元素](https://leetcode.cn/problems/remove-element/)
  - [283. 移动零](https://leetcode.cn/problems/move-zeroes/)
  - [167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)
  - [344. 反转字符串](https://leetcode.cn/problems/reverse-string/)
  - [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)(dp also)

- [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/description/)

- [15. 三数之和](https://leetcode.cn/problems/3sum/description/)

- [1. 两数之和](https://leetcode.cn/problems/two-sum/description/)



## 二叉树

- [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/description/)
- [144. 二叉树的前序遍历](https://leetcode.cn/problems/binary-tree-preorder-traversal/description/)
- [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/description/)
- [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/description/)





## 动态规划

- [322. 零钱兑换](https://leetcode.cn/problems/coin-change/description/)
- [53. 最大子数组和](https://leetcode.cn/problems/maximum-subarray/description/)



## 回溯

- [51. N 皇后](https://leetcode.cn/problems/n-queens/description/)
- [52. N 皇后 II](https://leetcode.cn/problems/n-queens-ii/description/)

- [46. 全排列](https://leetcode.cn/problems/permutations/description/)
- [47. 全排列 II](https://leetcode.cn/problems/permutations-ii/description/)
- [77. 组合](https://leetcode.cn/problems/combinations/description/)
- [39. 组合总和](https://leetcode.cn/problems/combination-sum/description/)
- [40. 组合总和 II](https://leetcode.cn/problems/combination-sum-ii/description/)
- [216. 组合总和 III](https://leetcode.cn/problems/combination-sum-iii/description/)
- [78. 子集](https://leetcode.cn/problems/subsets/description/)
- [90. 子集 II](https://leetcode.cn/problems/subsets-ii/description/)



## BFS

- [111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/description/)
- [752. 打开转盘锁](https://leetcode.cn/problems/open-the-lock/description/)



## 搜索

```java
//搜索单个数
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // 直接返回
            return mid;
        }
    }
    // 直接返回
    return -1;
}
//搜索左区间
int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    // 搜索区间为 [left, right]
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            // 搜索区间变为 [mid+1, right]
            left = mid + 1;
        } else if (nums[mid] > target) {
            // 搜索区间变为 [left, mid-1]
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 收缩右侧边界
            right = mid - 1;
        }
    }
    // 判断 target 是否存在于 nums 中
    // 此时 target 比所有数都大，返回 -1
    if (left == nums.length) return -1;
    // 判断一下 nums[left] 是不是 target
    return nums[left] == target ? left : -1;
}
//搜索右区间
int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 这里改成收缩左侧边界即可
            left = mid + 1;
        }
    }
    // 最后改成返回 left - 1
    if (left - 1 < 0) return -1;
    return nums[left - 1] == target ? (left - 1) : -1;
}
```

- [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/description/)
- [704. 二分查找](https://leetcode.cn/problems/binary-search/description/)
- [33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/description/)







## 滑动窗口

- [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/description/)
- [76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/description/)
- [438. 找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/description/)



## 设计

- [146. LRU 缓存](https://leetcode.cn/problems/lru-cache/description/)



## 排序

- [912. 排序数组](https://leetcode.cn/problems/sort-an-array/description/)


