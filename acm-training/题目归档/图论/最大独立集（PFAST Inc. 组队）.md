# 最大独立集（PFAST Inc. 组队）

- **来源**：Codeforces 经典题（Petya 组队，n≤16 求无矛盾最大队伍）
- **难度**：Codeforces 1500-1600 / CCPC 区域赛铜牌档
- **知识点**：图论 · 最大独立集（MIS）+ 状态压缩枚举
- **归档日期**：2026-08-09

## 一句话破题
志愿者是图的顶点，矛盾关系是边，求"人数最多的内部无边的点集" = **最大独立集**。n ≤ 16 是明示：状压枚举 $2^n$。

## 知识点溯源
**核心洞察**：最大独立集 = 选最多的点，使任意两个被选点之间没有边（两两不相邻）。
**识别标志**：① n ≤ 20 却要求"无冲突最大组合"；② 输入是"若干对冲突/矛盾/讨厌关系"；③ 让你选"互相都不冲突"的最大子集；④ 典型的 $2^n$ 枚举信号。
**模板**：
```java
int best = 0, bestMask = 0;
for (int mask = 0; mask < (1 << n); mask++) {
    int size = Integer.bitCount(mask);
    if (size <= best) continue;
    if (isValid(mask)) { best = size; bestMask = mask; }
}
```
**为什么这么做**：MIS 是 NP-hard，无多项式精确解。题目把 n 压到 16，就是**允许甚至期待** $O(2^n·n^2)$ 暴力。

## 解题推导
- 建模：每个志愿者一顶点，每条矛盾关系一无向边 → 求最大独立集。
- 迷惑信息：*"彼佳本人可能不在队里"*（纯干扰，彼佳无特殊性）；*"区分大小写"*（直接 HashMap/TreeSet 天然处理）；*"姓名唯一"*（方便做字符串→下标映射）。
- 关键跳跃：把"矛盾关系"翻译成"图的边"，把"无矛盾队伍"翻译成"独立集"。
- 边界：m=0 时答案 = 全部 n 人；n=16 时 $1<<n$ 在 int 安全范围。

### 为什么"删度数最大点"贪心是错的（本类题最大坑）
贪心反复删当前度数最大的点，**不保证最优**。反例（n=8）：
```
X-A  X-B  X-C
A-D  B-E  C-F
Y-D  Y-E  Y-F
```
最优解 4 人 `{A,B,C,Y}`；贪心先删 X（度3）再删 Y（度3），只剩 3 人。**"度数大 ≠ 该删"**——X 挡了 A,B,C，Y 挡了 D,E,F，其实删 X 一个就能保留 Y+A+B+C=4 人。这类删点堆贪心在构造数据下必被卡。

## 完整 Java 解法
```java
import java.util.*;
import java.io.*;

public class Main {
    static Scanner sc = new Scanner(System.in);

    private static void solve() throws IOException {
        int n = sc.nextInt();
        int m = sc.nextInt();
        String[] names = new String[n];
        Map<String, Integer> idx = new HashMap<>();
        for (int i = 0; i < n; i++) {
            names[i] = sc.next();
            idx.put(names[i], i);
        }

        boolean[][] conflict = new boolean[n][n];
        for (int i = 0; i < m; i++) {
            int a = idx.get(sc.next());
            int b = idx.get(sc.next());
            conflict[a][b] = conflict[b][a] = true;
        }

        int bestMask = 0, bestSize = 0;
        for (int mask = 0; mask < (1 << n); mask++) {
            int size = Integer.bitCount(mask);
            if (size <= bestSize) continue;
            if (isValid(mask, n, conflict)) { bestMask = mask; bestSize = size; }
        }

        List<String> team = new ArrayList<>();
        for (int i = 0; i < n; i++)
            if ((bestMask & (1 << i)) != 0) team.add(names[i]);
        Collections.sort(team);                       // 字典序输出

        System.out.println(bestSize);
        for (String s : team) System.out.println(s);
    }

    static boolean isValid(int mask, int n, boolean[][] conflict) {
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) == 0) continue;
            for (int j = i + 1; j < n; j++)
                if ((mask & (1 << j)) != 0 && conflict[i][j]) return false;
        }
        return true;
    }

    public static void main(String[] args) throws Exception { solve(); }
}
```
**复杂度**：时间 $O(2^n·n^2)$，空间 $O(n^2)$。

## 举一反三
| 题号 | 与本题差异 | 一句话提示 |
|------|-----------|-----------|
| 二分图最大独立集 | 图是二分图 | MIS = \|V\| − 最小顶点覆盖 = \|V\| − 最大匹配（König 定理）|
| 最大团问题 | 选互相都有关系的最大组 | 最大团 = 补图最大独立集，Bron–Kerbosch 算法 |
| 加权最大独立集 | 每人体重不同 | 仍是状压，比较 weigh 总和 |
| 计数最大独立集个数 | 问方案数 | 状压 DP，dp[mask] 带方案数 |
| n 变 1e5 | 无法暴力 | 一般图只能近似；二分图用匹配 |

## 注意点 / 错题标记
- **核心坑**：用"删度数最大点"的贪心做最大独立集 → 不保证最优，必被构造数据卡。**见到独立集/最大组合，n≤20 直接状压**。
- 输出要**字典序**，Java 用 `Collections.sort`（字符串天然区分大小写）。
- 这题先被贪心思路带偏，应标记为本类题"贪心陷阱"复习点。