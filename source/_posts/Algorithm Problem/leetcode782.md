---
title: leetcode782 变为棋盘
date: 2022-08-23 20:39:39
tags: 矩阵
categories: Algorithm Problem
math: true
---

#### [782. 变为棋盘](https://leetcode.cn/problems/transform-to-chessboard/)

一个 n x n 的二维网络 board 仅由 0 和 1 组成 。每次移动，你能任意交换两列或是两行的位置。

返回 将这个矩阵变为  “棋盘”  所需的最小移动次数 。如果不存在可行的变换，输出 -1。

“棋盘” 是指任意一格的上下左右四个方向的值均与本身不同的矩阵。

 

示例 1:

![示例1](http://longls777.oss-cn-beijing.aliyuncs.com/img/chessboard1-grid.jpg)

输入：board = [[0,1,1,0],[0,1,1,0],[1,0,0,1],[1,0,0,1]]

输出：2

解释：一种可行的变换方式如下，从左到右：第一次移动交换了第一列和第二列。第二次移动交换了第二行和第三行。



示例 2:

![示例2](http://longls777.oss-cn-beijing.aliyuncs.com/img/chessboard2-grid.jpg)

输入：board = [[0, 1], [1, 0]]

输出：0

解释：注意左上角的格值为0时也是合法的棋盘，也是合法的棋盘.



示例 3:

![示例3](http://longls777.oss-cn-beijing.aliyuncs.com/img/chessboard3-grid.jpg)

输入：board = [[1, 0], [1, 0]]

输出：-1

解释：任意的变换都不能使这个输入变为合法的棋盘。



提示：

- n == board.length
- n == board[i].length
- 2 <= n <= 30
- board[i] [j]将只包含 0或 1

#### 题解

```java
class Solution {
        public int movesToChessboard(int[][] board) {
        if (!check(board)) {
            return -1;
        }
        int[] col = new int[board.length];
        for (int i = 0; i < board.length; i++) {
            col[i] = board[i][0];
        }
        return getSwapCount(board[0]) + getSwapCount(col);
    }

    /**
     * 检查合法性 分别检查行和列
     *
     * @param board 数组
     * @return
     */
    public boolean check(int[][] board) {
        return checkFirstRow(board) &&
                checkFirstCol(board) &&
                checkRow(board) &&
                checkCol(board);
    }

    public boolean checkFirstRow(int[][] board) {
        int rowOneCnt = 0;
        int rowZeroCnt = 0;
        int[] first = board[0];
        for (int num : first) {
            if (num == 0) {
                rowZeroCnt++;
            } else {
                rowOneCnt++;
            }
        }
        return rowOneCnt == rowZeroCnt || Math.abs(rowOneCnt - rowZeroCnt) == 1;
    }

    public boolean checkFirstCol(int[][] board) {
        int oneCnt = 0, zeroCnt = 0;
        for (int i = 0; i < board.length; i++) {
            if (board[i][0] == 0) {
                zeroCnt++;
            } else {
                oneCnt++;
            }
        }
        return oneCnt == zeroCnt || Math.abs(oneCnt - zeroCnt) == 1;
    }

    public boolean checkRow(int[][] board) {
        //第一行当做哨兵 其他的行要么和第一行相等 要么和第一行相反
        //如：第一行0110 后续的行只能是 0110、1001
        int[] sentinel = board[0];
        int sameCnt = 0, oppositeCnt = 0;
        for (int[] cur : board) {
            //相同
            if (sentinel[0] == cur[0]) {
                for (int i = 0; i < sentinel.length; i++) {
                    if (sentinel[i] != cur[i]) {
                        return false;
                    }
                }
                sameCnt++;
            } else {
                //相反
                for (int i = 0; i < sentinel.length; i++) {
                    if (sentinel[i] + cur[i] != 1) {
                        return false;
                    }
                }
                oppositeCnt++;
            }
        }
        return sameCnt == oppositeCnt || Math.abs(sameCnt - oppositeCnt) == 1;
    }

    public boolean checkCol(int[][] board) {
        //第一列当做哨兵 其他的列要么和第一列相等 要么和第一列相反
        //如：第一列0110 后续的列只能是 0110、1001
        int sameCnt = 0, oppositeCnt = 0;
        int[] sentinel = new int[board.length];
        for (int j = 0; j < board.length; j++) {
            sentinel[j] = board[j][0];
        }
        for (int j = 0; j < board.length; j++) {
            if (board[0][j] == sentinel[0]) {
                for (int i = 0; i < sentinel.length; i++) {
                    if (sentinel[i] != board[i][j]) {
                        return false;
                    }
                }
                sameCnt++;
            } else {
                for (int i = 0; i < sentinel.length; i++) {
                    if (sentinel[i] + board[i][j] != 1) {
                        return false;
                    }
                }
                oppositeCnt++;
            }
        }
        return sameCnt == oppositeCnt || Math.abs(sameCnt - oppositeCnt) == 1;
    }

    private int getSwapCount(int[] sentinel) {
        //假设都是10101010...
        int preNum = 1;
        int errorCnt = 0;
        for (int i : sentinel) {
            //统计有多少错位
            if (i != preNum) {
                errorCnt++;
            }
            preNum = preNum == 1 ? 0 : 1;
        }
        //数组是偶数个还是奇数个
        if (sentinel.length % 2 == 0) {
            //偶数个 可以是01010101 或者 10101010
            return Math.min(sentinel.length - errorCnt, errorCnt) / 2;
        } else {
            //奇数个 取决于1多还是0多 1多则是1 0 1 0 1 0 1 0 1 、0多则是0 1 0 1 0 1 0 1 0
            //错位是偶数则为1多
            if (errorCnt % 2 == 0) {
                return errorCnt / 2;
            } else {
                return (sentinel.length - errorCnt) / 2;
            }
        }
    }

}
```

首先判断是否可以转换为合法的矩阵，重点是：

行列交换是独立的，交换列不改变行的纵向对应关系。例如n[i] [j]与n[i+x] [j]的对应关系不变，由于最终矩阵变为[0,1,0,1...] [1,0,1,0...]或者[1,0,1,0...] [0,1,0,1...]，由此倒推得出，矩阵中一定只包含两种不同的行(列)，要么与第一行(列)相同，要么相反，假如第一行为0110，则其他行必定为0110或1001，且相同与相反的数量相差1，交换列也不改变行中数字的数量，则每一行的1的数量要么与0相同，要么相差1，列也同样具有这些规律。

然后计算移动次数，由于上述规则，只要第一行和第一列移动到位，则整个矩阵就合法了。移动时注意行(列)的总数为奇数还是偶数，奇数只有一种移动方式，即数量多的类型在前，0多为010101...，1多为101010...

