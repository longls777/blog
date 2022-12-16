---
title: leetcode652
date: 2022-09-06 21:06:44
tags: DFS
categories: Algorithm Problem
math: true
---

#### [652. 寻找重复的子树](https://leetcode.cn/problems/find-duplicate-subtrees/)

给定一棵二叉树 `root`，返回所有**重复的子树**。

对于同一类的重复子树，你只需要返回其中任意**一棵**的根结点即可。

如果两棵树具有**相同的结构**和**相同的结点值**，则它们是**重复**的。

 

**示例 1：**

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/e1.jpg)

```
输入：root = [1,2,3,4,null,2,4,null,null,4]
输出：[[2,4],[4]]
```

**示例 2：**

![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/e2.jpg)

```
输入：root = [2,1,1]
输出：[[1]]
```

**示例 3：**

**![img](http://longls777.oss-cn-beijing.aliyuncs.com/img/e33.jpg)**

```
输入：root = [2,2,2,3,null,3,null]
输出：[[2,3],[3]]
```

 

**提示：**

- 树中的结点数在`[1,10^4]`范围内。
- `-200 <= Node.val <= 200`

#### 题解

```java
class Solution {
    Map<String, Integer> map = new HashMap<>();
    List<TreeNode> ans = new ArrayList<>();
    public List<TreeNode> findDuplicateSubtrees(TreeNode root) {
        dfs(root);
        return ans;
    }
    String dfs(TreeNode root) {
        if (root == null) return " ";
        StringBuilder sb = new StringBuilder();
        sb.append(root.val).append("_");
        sb.append(dfs(root.left)).append(dfs(root.right));
        String key = sb.toString();
        map.put(key, map.getOrDefault(key, 0) + 1);
        if (map.get(key) == 2) ans.add(root);
        return key;
    }
}
```



> 作者：AC_OIer
> 链接：https://leetcode.cn/problems/find-duplicate-subtrees/solution/by-ac_oier-ly58/
> 来源：力扣（LeetCode）
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。