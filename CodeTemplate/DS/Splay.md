## 伸展树

伸展树是一个非常灵活的数据结构，能够动态维护有序序列，支持非常多的操作。

其主要的两个操作数旋转与伸展。

旋转和一般的平衡树的旋转相同，不过为了保证树的高度，实行双层伸展的策略。具体单层伸展退化的情况见邓俊辉数据结构教材。

下面是两个函数的模板。

```cpp
// 将x和其父节点进行旋转
void rotate(int x) {
    int y = tr[x].p, z = tr[y].p;
    int k = tr[y].s[1] == x;
    tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
    tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[y].s[k]].p = y;
    tr[x].s[k ^ 1] = y, tr[y].p = x;
    pushup(y);  // 利用孩子节点的信息更新父节点的信息，必须先pushup(y)，再pushup(x)
    pushup(x);
}

// 伸展操作，将x伸展到k的孩子节点位置，一般k为0或者根节点。
// 当k为零的时候，相当于将x旋转到根节点的位置
// 当k为根的时候，可以得到两个区间之间的子树
void splay(int x, int k) {
    while (tr[x].p != k) {

        int y = tr[x].p,  z = tr[y].p;
        if (z != k) {
            if ((tr[y].s[1] == x) ^ (tr[z].s[1] == y)) rotate(x);
            else rotate(y);
        }
        rotate(x);
    }
    if (!k) root = x;
}
```

伸展树其它的操作几乎都可以由这两个操作来实现。

#### 插入

插入操作比较容易，找到插入的位置，然后将节点插入即可。在插入之后调用`splay`操作将新插入的节点旋转到根的位置，保证树的高度。

```c++
    // 插入操作
    void _insert(int k) {
        int u = root, p = 0;
        while (u) {
            p = u;
            if (data[u].k == k) break;
            else u = data[u].s[k > data[u].k];
        }
        if (u) {
            data[u].cnt ++;
            pushup(u);
        } else {
            u = ++idx;
            if (p) data[p].s[k > data[p].k] = u;
            data[u].init(k, p);
        }
        splay(u, 0);
    }
```

#### 删除

有很多策略，下面是一种：

1. 找到要删除的节点
2. 将其旋转到根
3. 找到其前驱`l`(左子树最右侧的一个节点)和后继(右子树最左侧的一个节点)`r`。将`l`旋转到根，将`r`旋转到`l`的孩子节点。`splay(l, 0), splay(r, l)`。这个时候，`r`的左孩子就是要删除的节点。将`r`的左孩子置为空，并更新信息即可。

#### 在第k个节点之后插入一段

将第`k`个节点旋转到根，再讲第`k+1`个节点旋转到根的右孩子，再讲插入的一段使用递归地方法构建为一棵树。

插入到`k+1`节点的左孩子即可。

#### 删除第k个节点之后的长度为l的段

将第`k`个节点旋转到根，将第`k + l + 1`个节点旋转到根的右孩子，这个时候第`k + l + 1`个节点的左孩子就是要删除的那一段，直接将其左孩子值为空即可。

#### Splay合并

如果将两个完全没有关系的Splay进行合并的话，可以使用启发式的方法，将`size`较小的树合并到较大的树上，时间复杂度可以控制在插入的节点个数为`n log(n)`级别。

#### 区间更新

这里可以使用类似线段树的懒标记的方法。假设需要将第`l`到第`r`个节点之间的区间添加一些操作。

这个时候先将第`l - 1`个节点旋转到根，再将第`r + 1`个节点旋转到根的右孩子。这个时候就可以直接得到要操作的区间了。再按照线段树的维护懒标记的方法进行维护即可。

在节点维护区间信息，并且具有懒标记的情况下，注意懒标记下传的时刻。

当我们需要将节点`x`旋转到根的时候，我们需要保证从根节点到节点`x`这一条路径上的节点，其懒标记都传到其孩子节点。而且只需要这一个路径传了就行，在向上旋转的时候就可以保证信息被正确地维护了。所以在查找节点`x`的时候需要`pushdown`路径上经过的所有节点的懒标记。

终极代码见[维护数列](Splay_3.cpp)。实现了Splay的大部分操作以及线段树维护区间的功能。

小技巧：

1. 在建树的时候，为了能够实现方便得到任意一个区间，可以在一开始在树中加入两个哨兵节点。哨兵节点的信息如何设定需要保证在pushup的时候的正确的信息。同时空节点的信息设定也需要注意这一点。
2. `Splay`可以**维护任意类型**的序列，有序无序取决于插入的时候的定义。旋转操作最核心的地方在于其可以保证树的**中序遍历**不变。通过伸展操作可以得到**代表任意一个区间的子树**，在其上操作即可。