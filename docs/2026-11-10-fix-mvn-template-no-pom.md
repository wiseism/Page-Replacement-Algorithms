# 2026-11-10 Java CI 修复（Maven 模板误用 → javac 编译）

## 现象

`Java CI with Maven` workflow 报错：

```
##[error]No file in /home/runner/work/Page-Replacement-Algorithms/... matched to [**/pom.xml]
make sure you have checked out the target repository
```

## 根因

仓库是纯 `javac` + `Makefile` 的教学代码（simpager/fifo/lru/opt 等 .java），没有任何 Maven 工程（pom.xml）。GitHub UI 添加 CI 时误选了 Java-Maven 模板，`setup-java` 的 `cache: maven` 扫描 `**/pom.xml` 失败导致 job 直接失败。

## 修复

`.github/workflows/maven.yml` 重命名为语义正确的构建流程（模板只保留文件名，内容替换）：

- 触发分支：`master`、`maven`；
- `actions/setup-java@v4`（temurin 17，去掉 `cache: maven`）；
- 使用现有 `Makefile`：`make`（javac 编 simpager.java）＋ `make clean`。

## 补充（同日稍后）

首次在 `maven` 分支触发后仍失败：`make: No rule to make target 'simpager.java'` —— maven 分支只剩 Makefile/.github，源码文件（*.java）并不在该分支上；master 分支同样内容的 run 成功。已将 maven 分支重置为与 master 一致（force push）。
