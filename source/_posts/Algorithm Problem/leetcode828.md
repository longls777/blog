---
title: leetcode828
date: 2022-09-6 21:02:39
tags: 字符串
categories: Algorithm Problem
math: true
---

#### [828. 统计子串中的唯一字符](https://leetcode.cn/problems/count-unique-characters-of-all-substrings-of-a-given-string/)

我们定义了一个函数 `countUniqueChars(s)` 来统计字符串 `s` 中的唯一字符，并返回唯一字符的个数。

例如：`s = "LEETCODE"` ，则其中 `"L"`, `"T"`,`"C"`,`"O"`,`"D"` 都是唯一字符，因为它们只出现一次，所以 `countUniqueChars(s) = 5` 。

本题将会给你一个字符串 `s` ，我们需要返回 `countUniqueChars(t)` 的总和，其中 `t` 是 `s` 的子字符串。输入用例保证返回值为 32 位整数。

注意，某些子字符串可能是重复的，但你统计时也必须算上这些重复的子字符串（也就是说，你必须统计 `s` 的所有子字符串中的唯一字符）。

**示例 1：**

```
输入: s = "ABC"
输出: 10
解释: 所有可能的子串为："A","B","C","AB","BC" 和 "ABC"。
     其中，每一个子串都由独特字符构成。
     所以其长度总和为：1 + 1 + 1 + 2 + 2 + 3 = 10
```

**示例 2：**

```
输入: s = "ABA"
输出: 8
解释: 除了 countUniqueChars("ABA") = 1 之外，其余与示例 1 相同。
```

**示例 3：**

```
输入：s = "LEETCODE"
输出：92
```

**提示：**

- `1 <= s.length <= 10^5`
- `s` 只包含大写英文字符

#### 题解

```java
class Solution {
    public int uniqueLetterString(String s) {
        char[] cs = s.toCharArray();
        int n = cs.length, ans = 0;
        int[] l = new int[n], r = new int[n];
        int[] idx = new int[26];
        Arrays.fill(idx, -1);
        for (int i = 0; i < n; i++) {
            int u = cs[i] - 'A';
            l[i] = idx[u];
            idx[u] = i;
        }
        Arrays.fill(idx, n);
        for (int i = n - 1; i >= 0; i--) {
            int u = cs[i] - 'A';
            r[i] = idx[u];
            idx[u] = i;
        }
        for (int i = 0; i < n; i++) ans += (i - l[i]) * (r[i] - i);
        return ans;
    }
}
```

原问题为求所有子数组的唯一字符数量和，其可等价为求每个 s[i]s[i] 对答案的贡献，即每个 s[i]s[i] 可作为多少个子数组的唯一元素。

假定我们能预处理出两数组 l 和 r 分别代表 s[i]s[i] 作为子数组唯一字符时，其所能到达的最远两端：

l[i] = a 代表下标 aa 为 s[i]s[i] 能够作为子数组唯一字符时的最远左边界，即为 s[i]s[i] 左边第一个与 s[i]s[i] 值相同的位置（若不存在，则为 a = -1a=−1）
r[i] = b 代表跳表 bb 为 s[i]s[i] 能够作为子数组唯一字符时的最远右边界，即为 s[i]s[i] 右边第一个与 s[i]s[i] 值相同的位置（若不存在，则为 b = nb=n）
子数组左端点个数为 (i - a)(i−a) 个，右端点个数为 (b - i)(b−i) 个，根据乘法原理可知，子数组个数为两者乘积。

预处理 l 和 r 只需要使用遍历计数即可。

> 作者：AC_OIer
> 链接：https://leetcode.cn/problems/count-unique-characters-of-all-substrings-of-a-given-string/solution/by-ac_oier-922k/
> 来源：力扣（LeetCode）
> 