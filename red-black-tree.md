本文参考 Java 中 java.util.TreeMap，描述红黑树的插入与删除。

参考文献：

- [王争《数据结构与算法之美》红黑树（下）](https://time.geekbang.org/column/article/68976) 需要注册极客时间账号，本专栏可免费阅读 2 篇
- [CarpenterLee《JCFInternals》TreeSet and TreeMap](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/5-TreeSet%20and%20TreeMap.md) JCF 为 Java Collections Framework，系统讲述 JCF 内部实现
- [yangecnu《寒江独钓博客》浅谈算法和数据结构: 九 平衡查找树之红黑树](https://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html) 里面左旋右旋的 gif 不错，但是不知道原始出处

本文不使用图片，不使用实际示例来抽象的说明红黑树的插入与删除，并根据调整结束时的 Case 反推全部 Case。

## 说明

一些缩写：

- `X`，当前关注节点
- `P`，当前关注节点 `X` 的父节点（parent）
- `G`，当前关注节点 `X` 的祖父节点（grant-parents）
- `U`，当前关注节点 `X` 的叔叔节点（uncle）
- `S`，当前关注节点 `X` 的兄弟节点（sibling）

颜色示例：
- `🅧`，实心，表示黑色
- `Ⓧ`，空心，表示红色
- `🇽`，虚线，表示颜色未知或者不关心
- `⬤`，实心圆圈，表示需要额外算一个黑色

## 插入

插入的节点，总是红色。如果插在黑色节点之后，则不需要处理。插在红色节点之后，最终出路只有一条，对应 Case 3。

### 插入 Case 3

这里仅考虑 `P` 为 `G` 左节点情况，`P` 为 `G` 右节点时情况类似。

对于 Case 3，把 `G` 右旋，交换 `P` 与 `G` 颜色。此时，无论 `X`，`P` 右节点 `p`（根据红黑树定义，应为黑色），还是 `U`，没有改变调整之前的黑色节点数目，但是不再有连续红色节点，**调整结束**。


```
         🅖                   🅟
       ↙   ↘                ↙   ↘
     Ⓟ      🅤     ⇒      Ⓧ      Ⓖ⤸
   ↙   ↘                        ↙   ↘
  Ⓧ     p                     p     🅤

```

（此 Case 中，`G` 到 `P` 黑色节点数与 `G` 到 `U` 黑色节点数不一致，易知由其他 Case 演变而来。）

对应源码如下：

```java
    setColor(parentOf(x), BLACK);
    setColor(parentOf(parentOf(x)), RED);
    rotateRight(parentOf(parentOf(x)));
    // 此时不再符合循环条件 x.parent.color == RED
```

不符合此 Case 的情况有：

- `U` 为红色，对应 Case 1；
- `X` 为 `P` 的右节点，对应 Case 2。

### 插入 Case 1

对于 Case 1，把 `P`，`U` 标记为黑色，把 `G` 标记为红色，然后**循环**处理 `G`。这种调整，与 `X` 为 `P` 左节点或右节点无关。

```
         🅖                   Ⓖ
       ↙   ↘                ↙   ↘
     Ⓟ      Ⓤ     ⇒      🅟      🅤
     ⇣                    ⇣
     Ⓧ                   Ⓧ
```

对应源码如下：

```java
    Entry<K,V> y = rightOf(parentOf(parentOf(x)));
    if (colorOf(y) == RED) {
        setColor(parentOf(x), BLACK);
        setColor(y, BLACK);
        setColor(parentOf(parentOf(x)), RED);
        x = parentOf(parentOf(x));
    }
```

### 插入 Case 2

对于 Case 2，把 `X` 的父节点 `P` 左旋，关注节点变为 `P`，转换成 Case 3。

```
         🅖                       🅖
       ↙   ↘                    ↙   ↘
     Ⓟ      🅤     ⇒          Ⓧ      🅤
   ↙   ↘                    ↙   ↘
 p      Ⓧ                ⤹Ⓟ     x2
      ↙   ↘              ↙  ↘
     x1   x2            p    x1
```

（此 Case 中，`G` 到 `P` 黑色节点数与 `G` 到 `U` 黑色节点数不一致，易知由其他 Case 演变而来。）

对应源码如下：

```java
    if (x == rightOf(parentOf(x))) {
        x = parentOf(x);
        rotateLeft(x);
    }
```

## 删除

删除一个节点，调整则复杂许多。主要分为两种情况：

- 删除的节点只有左节点或右节点，使用其子节点代替；
- 删除的节点同时有左右节点，找到后续节点（位于右侧的最左节点），代替待删除的节点，然后删除这个后续节点。

如果删除的节点为红色，此时不影响红黑树的平衡性，不需要调整。如果删除的节点为黑色，则需要平衡处理，最终出路只有一条，对应 Case 4。

### 删除 Case 4

这里仅考虑 `X` 为 `P` 左节点情况，`X` 为 `P` 右节点时情况类似。

对于 Case 4，左旋 `P`，`P` 与 `S` 颜色互换，`S` 右节点 `R` 从红色变为黑色，去掉 `X` 多余的黑色，**调整结束**。

```            
     🇵                        🇸        
   ↙   ↘                      ↙  ↘      
⬤🅧     🅢        ⇒        ⤹🅟    🅡       
       ↙   ↘              ↙   ↘          
     🇱     Ⓡ          🅧      🇱           
```

对应源码如下：

```java
    setColor(sib, colorOf(parentOf(x)));
    setColor(parentOf(x), BLACK);
    setColor(rightOf(sib), BLACK);
    rotateLeft(parentOf(x));
    x = root;
```

此时，分部根节点到各子节点黑色节点数目如下：

|  | 调整前 | 调整后 |
| --- | --- | --- |
| X | ? + 2 | ? + 2 |
| L之前 | ? + 1 | ? + 1 |
| R | ? + 1 | ? + 1 |

不符合此情况的有：

- `S` 为红色，对应 Case 1；
- `R` 为黑色，又分两种：
	- `L` 为黑色，对应 Case 2；
	- `L` 为红色，对应 Case 3。

### 删除 Case 1

对于 Case 1，如果 `S` 为红色，则 `P`、`L` 与 `R` 均为黑色。此时，左旋 `P`，`P` 设置为红色，`S` 设置为黑色。调整之后，`X` 的兄弟节点从 `S` 节点变为其侄子节点 `L`，继续 Case 2 或 Case 3 或 Case 4。

```            
     🅟                        🅢        
   ↙   ↘                      ↙  ↘      
⬤🅧     Ⓢ        ⇒        ⤹Ⓟ    🅡       
       ↙   ↘              ↙   ↘          
     🅛     🅡        ⬤🅧      🅛           
```

对应源码如下：

```java
    Entry<K,V> sib = rightOf(parentOf(x));

    if (colorOf(sib) == RED) {
        setColor(sib, BLACK);
        setColor(parentOf(x), RED);
        rotateLeft(parentOf(x));
        sib = rightOf(parentOf(x));
    }
```

此时，分部根节点到各子节点黑色节点数目如下：

|  | 调整前 | 调整后 |
| --- | --- | --- |
| X | 3 | 3 |
| L | 2 | 2 |
| R | 2 | 2 |

### 删除 Case 2

对于 Case 2，`S` 设置为红色，关注节点从 `X`变为 `P`，继续循环。如果 `P` 为红色，循环结束，把 `P` 设为黑色即可。

```
     🇵                      ⬤🇵        
   ↙   ↘                     ↙  ↘      
⬤🅧     🅢        ⇒        🅧    Ⓢ      
       ↙   ↘                    ↙   ↘           
     🅛     🅡                 🅛     🅡 
```

对应源码如下：

```java
    if (colorOf(leftOf(sib))  == BLACK &&
        colorOf(rightOf(sib)) == BLACK) {
        setColor(sib, RED);
        x = parentOf(x);
    } 
```

此时，分部根节点到各子节点黑色节点数目如下：

|  | 调整前 | 调整后 |
| --- | --- | --- |
| X | ? + 2 | ? + 2 |
| L | ? + 2 | ? + 2 |
| R | ? + 2 | ? + 2 |

### 删除 Case 3

对于 Case 3，右旋 `S`，`L` 设为黑色，`S` 设为红色，然后继续 Case 4。

```
     🇵                       🇵        
   ↙   ↘                     ↙   ↘      
⬤🅧     🅢        ⇒       ⬤🅧    🅛     
       ↙   ↘                     ↙   ↘           
     Ⓛ     🅡                  l1     Ⓢ⤸
    ↙  ↘                            ↙    ↘ 
   l1  l2                          l2     🅡
```

对应源码如下：

```java
    if (colorOf(rightOf(sib)) == BLACK) {
        setColor(leftOf(sib), BLACK);
        setColor(sib, RED);
        rotateRight(sib);
        sib = rightOf(parentOf(x));
    }
```

此时，分部根节点到各子节点黑色节点数目如下：

|  | 调整前 | 调整后 |
| --- | --- | --- |
| X | ？+ 2 | ？+ 2 |
| l1前 | ? + 1 | ? + 1 |
| l2前 | ? + 1 | ? + 1 |
| R | ? + 2 | ? + 2 |