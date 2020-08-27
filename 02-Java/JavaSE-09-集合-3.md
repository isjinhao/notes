## 红黑树

### 二叉搜索树

由于红黑树是二叉查找树的平衡模式，所以在了解红黑树之前，咱们先来看下二叉查找树。

二叉查找树（Binary Search Tree），也称有序二叉树（ordered binary tree），排序二叉树（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

- 若任意结点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
- 若任意结点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
- 任意结点的左、右子树也分别为二叉查找树。
- 没有键值相等的结点（no duplicate nodes）。

因为，一棵由n个结点，随机构造的二叉查找树的高度为lgn，一般操作的执行时间为O（lgn）。但二叉树若退化成了一棵具有n个结点的线性链后，则此些操作最坏情况运行时间为O（n）。



### 性质

红黑树是一颗平衡二叉树，平衡二叉树的概念：每课树的左右子树高度差都不超过1。

1. 每个结点都有颜色，红色或者黑色
2. 根结点是黑色的
3. 每个叶结点是黑色的（设叶结点是NIL，但它不是一个空节点，是一个哨兵）
4. 如果一个结点是红色的，则它的两个子结点都是黑色的
5. 对每个结点，从该结点到其所有后代结点的简单路径上，均包含相同数目的黑色结点

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/e0a29782-c0a9-4736-aada-c31acb92131f"></div>

图中的NIL其实可以用一个哨兵（T.nil）替代，但在画图时往往其实不画NIL结点。



### 旋转

对红黑树的增加和删除会破坏红黑树原本的性质，旋转是用来使其恢复到红黑树的手段。

#### 左旋

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/69c29b47-b61e-47ae-ab11-e8a327ad3694"></div>

```java
// LEFT-ROTATE(T, x)    设x是图中的pivot
y = x.right
x.right = y.left
if y.left != T.nil
	y.left.p = x
y.p = x.p
if x.p == T.nil
	T.root = y
else if x == x.p.left		// 判断应该放在P的左子树还是右子树上
	x.p.left = y
else
    x.p.right = y
y.left = x
x.p = y
```

#### 右旋

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/b2c71757-8b47-4b5f-8b88-c5d7fb2570a6"></div>

```java
// RIGHT-ROTATE(T, x)    设x是图中的pivot
y = x.left
x.left = y.right
if y.right != T.nil
	y.right.p = x
y.p = x.p
if x.p == T.nil
	T.root = y
else if x == x.p.right		// 判断应该放在P的左子树还是右子树上
	x.p.right = y
else 
    x.p.lrft = y
y.right = x
x.p = y
```



### 二叉查找树的插入

如果要在二叉查找树中插入一个结点，首先要查找到结点插入的位置，然后进行插入，假设插入的结点为z的话，插入的伪代码如下：

```java
// TREE-INSERT(T, z)
y = T.nil
x = T.root
while x != NIL
	y = x
	if z.key < x.key
		x = x.left
	else
        x = x.right
z.p = y
if y == NIL
	T.root = z              // Tree T was empty
else if z.key < y.key
	y.left = z
else
    y.right = z
```



### 红黑树的插入

如果我们插入的是黑色节点，会违反了性质五，需要进行大规模调整，如果我们插入的是红色节点，那就只有在要插入节点的父节点也是红色的时候违反性质四或者是当插入的节点是根节点时，违反性质二，所以，我们把要插入的节点的颜色变成红色。仍然设被插入结点是z：

```java
// RB-INSERT(T, z)
y = T.nil
x = T.root
while x != T.nil
	y = x
	if z.key < x.key
		x = x.left
	else
		x = x.right
z.p = y
if y == T.nil
	T.root = z
else if z.key < y.key
	y.left = z
else 
    y.right = z
z.left = T.nil
z.right = T.nil
z.color = RED
RB-INSERT-FIXUP(T, z)
```

我们把上面这段红黑树的插入代码，跟我们之前看到的二叉查找树的插入代码，可以看出，RB-INSERT(T, z)前面的16行代码基本就是二叉查找树的插入代码，然后第17-18行代码把z的左孩子、右孩子都赋为叶结点nil，第19行再把z结点着为红色，最后为保证红黑性质在插入操作后依然保持，调用一个辅助程序RB-INSERT-FIXUP来对结点进行重新着色，并旋转。  **换言之**：

- 如果插入的是根结点，因为原树是空树，此情况只会违反性质2，所以直接把此结点涂为黑色。  
- 如果插入的结点的父结点是黑色，由于此不会违反性质2和性质4，红黑树没有被破坏，所以此时也是什么也不做。  

但当遇到下述3种情况时需要修复：

- 插入修复情况1：如果当前结点的父结点是红色且祖父结点的另一个子结点（叔叔结点）是红色
- 插入修复情况2：当前结点的父结点是红色，叔叔结点是黑色，当前结点是其父结点的右子
- 插入修复情况3：当前结点的父结点是红色，叔叔结点是黑色，当前结点是其父结点的左子

#### 修复

```java
RB-INSERT-FIXUP(T, z)
while z.p.color == RED
	if z.p == z.p.p.left
		y = z.p.p.right			// 指向叔叔结点
		if y.color == RED
			z.p.color = BLACK
			y.color = BLACK
			z.p.p.color = RED
		else if z == z.p.right
			z = z.p
			LEFT-ROTATE(T, z)
        z.p.color = BLACK
        z.p.p.color = RED
        RIGHT-ROTATE(T, z.p.p)
    else // 与 if 对称，等下只分析if，不分析else
		y = z.p.p.left
		if y.color == RED
			z.p.color = BLACK
			z.p.p.color = RED
		else if z == z.p.left
			z = z.p
			RIGHT-ROTATE(T, z)
        z.p.color = BLACK
        z.p.p.color = RED
        LEFT-ROTATE(T, z.p.p)
T.root.color = BLACK
```

下面，咱们来分别处理上述3种插入修复情况。

**插入修复情况1：当前结点的父结点是红色且祖父结点的另一个子结点（叔叔结点）是红色。**  

上诉代码中第5行至第8行是处理这种情况：将父结点和叔叔结点涂黑，将爷爷结点涂红。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/63d84e52-f4ca-4400-897a-1db3ae80e5ec"></div>
<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/a0275212-0d5d-4963-bddc-6b45b3bf3f5e"></div>

**插入修复情况2：当前结点的父结点是红色,叔叔结点是黑色，当前结点是其父结点的右子**  

上诉代码中第9行至第11行是处理这种情况：当前结点的父结点做为新的当前结点，以新当前结点为支点左旋。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/6ae43f67-710e-4049-8a59-5eaf312d7e1c"></div>
<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/a4dd3504-6874-4770-8986-9af3d706b15f"></div>

**插入修复情况3：当前结点的父结点是红色,叔叔结点是黑色，当前结点是其父结点的左子** 

上述代码中的12-14行处理此种情况：父结点变为黑色，祖父结点变为红色，在祖父结点为支点右旋。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/b6985972-466e-49d9-83fd-7e4de59d11c4"></div>
<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/84e1edc6-6bc1-4658-82c7-3d6dc78324b6"></div>

### 二叉查找树的后继

```java
// TREE-SUCCESSOR(x)
if x.right != T.nil
	return TREE-MINIMUM(x.right)
y = x.p
while y != T.nil and x == y.right
	x = y
	y = y.p
return y
```

$x$是待查找结点，右子树不为空，右子树的最小值所在的节点是后继。解释第`4-8`行代码需要证明一个定理：

> 对于一棵二叉搜索树T，其关键字各不相同，如果T中一个节点$x$的右子树为空，且$x$有一个后继$y$，那么$y$一定是$x$的最底层祖先，并且其左孩子也是$x$的祖先。（每个结点都是其自身的祖先）

#### 证明

对于给定结点$x$，若其后继$y$存在，则$y > x$。

1. 考虑结点$x$，对于$x$的左子树，显然其中任意结点值都小于$x$，所以$y$必定不在其左子树中。
2. $x$的右子树，其中任意结点值都大于$x$,但是根据题设，其右子树为空。

所以，$y$必定为$x$的祖先或其祖先的右子树。

又因为$y$是其中大于$x$且最小的一个，则$y$不可能是其祖先的右子树，那么我们可以将范围缩小至$y$必定为$x$的某一祖先，又根据$y>x$，则$x$必定在$y$的左子树中，即$y$的左孩子也是$x$的祖先（$x$也是$x$的祖先）

对于所有满足条件的，假设有$p_0,p_1 \dots p_n$共$n+1$个，且$p_0 < p_1 < p_2 < \dots < p_n$。显然，$x$的前驱结点$y$必定是其中的最小一个，即$y=p_0$。又因为$y$是$x$的祖先，则$y$必然是$x$的最底层祖先。

#### 结论

这个定理实际的意义是，对于二叉搜索树中的一个节点（$x$），如果其不存在右子树且还有后继（$y$），则$y$是$x$祖先节点中有左子树的最底层祖先。如下图中13的后继是15。

<div align="center"><img width="70%" src="http://blogfileqiniu.isjinhao.site/15aa8eff-11ca-4da1-85bf-335305cd9ac3"></div>

### 二叉查找树的前驱

前驱的代码和后继对应。

```java
TREE-PREDECESSOR(x)
if x.left != null
	return TREE-MAXNUM(x.left);
y = x.p
while y != null and x == y.left
	x = y
	y = y.p
return y
```

`3-7`行代码的意思是，对于二叉搜索树中的一个节点（$x$），如果其不存在左子树且还有前驱（$y$），则$y$是$x$祖先节点中有右子树的最底层祖先。如图中17的前驱是15。



### 二叉查找树的删除

讨论删除之前需要证明一个定理：

> 如果一个二叉搜索树中的一个节点有两个孩子，那么它的后继没有左孩子，它的前驱没有右孩子。

**证明**

如果一棵二叉搜索树中的一个结点有两个孩子，那么它的后继为它的右子树中的最小值，所以它的后继没有左孩子，它的前驱为它的左子树中的最大值，所以它的前驱没有右孩子。

**删除时有三种情况：**

1. 如果被删除节点（$z$）没有孩子节点，直接删除，修改父节点相应指针指向空。
2. 如果$z$只有一个孩子，把孩子提到树中$z$所在的位置，并修改$z$的父节点，用$z$的孩子来替换。
3. 如果$z$有两个孩子，那么找$z$的后继$y$（一定在$z$的右子树中）。
   1. 如果$y$是$z$的右孩子，用$y$替换$z$，同时留下$y$的右孩子。
   2. 如果$y$不是$z$的右孩子，有之上定理可知，$y$是没有左孩子的，此时用$y$的右孩子替换$y$，用$y$替换$z$，不留下$y$的右孩子。

#### TRANSPLANT

```java
// TRANSPLANT(T, u, v)
if u.p == T.nil
	T.root = v
else if u == u.p.left
	u.p.left = v
else
	u.p.right = v
if v != T.nil
	v.p = u.p
```

`TRANSPLANT`的功能是在树$T$中用一棵以$v$为根的子树来替换一棵以$u$为根的子树。

- `2-3`行：当$u$是树根的时候，直接让$T$的根指向$v$。
- `4-5`行：当$u$是一个左孩子的时候，将$v$放在$u$的左孩子的位置。
- `6`行：当$u$是一个右孩子的时候，将$v$放在$u$的右孩子的位置。
- `7-8`行：更新$v$的父节点。

#### TREE-DELETE

```java
TREE-DELETE(T, z)
if z.left == T.nil
	TRANSPLANT(T, z, z.right)
else if z.right == T.nil
	TRANSPLANT(T, z, z.left)
else 
    y = TREE-MINIMUM(z.right)
	if y.p != z
		TRANSPLANT(T, y, y.right)
        y.right = z.right
        y.right.p = y
    TRANSPLANT(T, z, y)
    y.left = z.left
    y.left.p = y
```

1. `2-5`行：如果$z$没有左孩子，那么用其右孩子替换$z$。如果$z$没有右孩子，那么用其左孩子替换$z$。
2. `12-14`行：如果$y$是$z$的右孩子，用$y$替换$z$，同时留下$z$的左孩子。
3. `7-11`行：如果如果$z$有两个孩子，且$y$不是$z$的右孩子，用$y$的右孩子替换$y$，用$y$替换$z$。

<div align="center"><img width="100%" src="http://blogfileqiniu.isjinhao.site/18039267-fd06-46a8-960b-3e969c28d4e7"></div>

### 红黑树的删除

红黑树的删除和二叉查找树的删除有类似的结构。

#### RB-TRANSPLANT

```java
// RB-TRANSPLANT(T, u, v)
if u.p == T.nil
	T.root = v
else if u == u.p.left
	u.p.left = v
else
	u.p.right = v
v.p = u.p
```

过程RB-TRANSPLANT与TRANSPLANT有一点不同，即第8行无条件赋值，因为红黑树中的哨兵不是空。

#### RB-DELETE

```java
// RB-DELETE(T, z)
y = z
y-original-color = y.color			// 删除的结点的颜色
if z.left == T.nil					// 左子树为空，右子树直接上去
	x = z.right						// x指向被删除结点的右孩子
	RB-TRANSPLANT(T, z, z.right)
else if z.right == T.nil			// 右子树为空，右子树直接上去
	x = z.left
	RB-TRANSPLANT(T, z, z.left)
else
	y = TREE-MINIMUM(z.right)		// 右子树中最小的
	y-original-color = y.color		// 删除的节点的颜色
	x = y.right
	if y.p == z
		x.p = y
	else
		RB-TRANSPLANT(T, y, y.right)			// 和二叉搜索树删除时第2、3点一致
		y.right = z.right
		y.right.p = y
	RB-TRANSPLANT(T, z, y)
	y.left = z.left
	y.left.p = y
	y.color = z.color
if y-original-color == BLACK			//当删除的是黑色时需要调整，红色不需要
	RB-DELETE-FIXUP(T, x)
```

我们从被删结点后来顶替它的那个结点开始调整，并认为它有额外的一重黑色。这里额外一重黑色是什么意思呢，我们不是把红黑树的结点加上除红与黑的另一种颜色，这里只是一种假设，我们认为我们当前指向它，因此空有额外一种黑色，可以认为它的黑色是从它的父结点被删除后继承给它的，它现在可以容纳两种颜色，如果它原来是红色，那么现在是红+黑，如果原来是黑色，那么它现在的颜色是黑+黑。有了这重额外的黑色，原红黑树性质5就能保持不变。现在只要恢复其它性质就可以了，做法还是尽量向根移动和穷举所有可能性。

```java
// RB-DELETE-FIXUP(T, x)
while x != T.root and x.color == BLACK
	if x == x.p.left
		w = x.p.right  // 兄弟结点
		if w.color == RED
			w.color = BLACK
			x.p.color = RED
			LEFT-ROTATE(T, x.p)
            w = x.p.right
        if w.left.color == BLACK and w.right.color == BLACK
        	w.color = RED
        	x = x.p
        else if w.right.color == BLACK
        	w.left.color = BLACK
        	w.color= RED
        	RIGHT-RATATE(T, w)
            w = x.p.right
        w.color = x.p.color
        x.p.color = BLACK
        w.right.color = BLACK
        LEFT-ROTATE(T, x.p)
        x = T.root
	else
		same as then clause with "right" and "left" exchanged
x.color = BLACK
```

（y是被删除的结点，x是怼上去的结点，即x占据了y的位置）

如果y的颜色是RED，则y不可能为红黑树的根，所以不管x的颜色是什么，都不会影响到红黑树的性质。所以只考虑y的颜色为BLACK的情况。

在y的颜色是BLACK的情况下，如果x的颜色为RED的话，删除y之后，结点y所在的分支的黑高就会减1，所以，只需要将x的颜色变为BLACK，则该分支的黑高会加1，则会保持住红黑树的颜色性质。

**所以最终要考虑的情况就是：**y颜色为BLACK，x的颜色为BLACK的情况。因把y删除后，x顶替y的位置，y所在分支的黑高减1，所以，假设x节点的颜色为BLACK-BLACK，简称BB，也就是原来y的BLACK增加到x上了，这样就保证了该分支的黑高不变，接下来要做的就是调整x所在的分支，使红黑树的性质保持不变，又分为下面的几种情况（只考虑x为左孩子的情况，右孩子的情况是对称的）

**删除修复情况1：当前结点是黑+黑且兄弟结点为红色(此时父结点和兄弟结点的子结点为黑)。**

解法：把父结点染成红色，把兄弟结点染成黑色，左旋，之后重新进入算法。此变换后原红黑树性质5不变，而把问题转化为兄弟结点为黑色的情况(注：变化前，原本就未违反性质5，只是为了**把问题转化为兄弟结点为黑色的情况**)。  即第5行至第9行。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/e10c53c8-d006-4b1b-9f40-7aca52707b66"></div>
<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/a668eb94-49be-4310-b64f-f0c663fe04e6"></div>

**删除修复情况2：当前结点是黑加黑且兄弟是黑色且兄弟结点的两个子结点全为黑色。**

解法：把当前结点和兄弟结点中抽取一重黑色追加到父结点上，把父结点当成新的当前结点，重新进入算法。（此变换后性质5不变），即第10-12行代码操作。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/2a6dd071-187e-468d-b83f-d46c584f6f30"></div>
<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/65b9b69a-53ae-46bc-8dae-d17b33ae3cfd"></div>

**删除修复情况3：当前结点颜色是黑+黑，兄弟结点是黑色，兄弟的左子是红色，右子是黑色。**

解法：把兄弟结点染红，兄弟左子结点染黑，之后再以兄弟结点为支点解右旋，之后重新进入算法。此是把当前的情况转化为情况4，而性质5得以保持，即第13-17行代码：

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/67881a31-213e-4500-ba21-60edfba97d85"></div>
<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/487c48c9-f085-409e-b3df-5810d49123b1"></div>

**删除修复情况4：当前结点颜色是黑-黑色，它的兄弟结点是黑色，但是兄弟结点的右子是红色，兄弟结点左子的颜色任意。**  

解法：把兄弟结点染成当前结点父结点的颜色，把当前结点父结点染成黑色，兄弟结点右子染成黑色，之后以当前结点的父结点为支点进行左旋，此时算法结束，红黑树所有性质调整正确，即第18-22行代码，如下所示：

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/96a2ab00-d796-4cbf-9979-1710db3f287b"></div>
<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/330536ac-eeba-40e4-9fd1-ca230961671e"></div>

## TreeMap分析

```java
public class TreeMap<K,V> 
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

在此分析一下Map的体系结构：

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/6ef892e1-3e35-47ff-8878-55d4e2c80266"></div>

AbstractMap是一个实现Map接口的抽象类，它实现了一个Map的骨架，方便开发者来实现一个自己的Map。如果开发者需要实现一个自己的不可变的Map，只需要继承它再实现entrySet方法。如果开发者想需要实现一个自己的可变的Map，在不可变Map的基础上还需要实现put方法，并且在其的iterator上实现remove方法。

SortedMap是一个键有序的Map，存在Comparator时使用其规则，否则使用Comparable的规则。

NavigableMap是一个导航Map，它含有的方法可以帮助寻找一个键的上一个键，下一个键，或者按键值逆序等。



### 旋转

#### 左旋

```java
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> r = p.right;
        p.right = r.left;
        if (r.left != null)
            r.left.parent = p;
        r.parent = p.parent;
        if (p.parent == null)
            root = r;
        else if (p.parent.left == p)
            p.parent.left = r;
        else
            p.parent.right = r;
        r.left = p;
        p.parent = r;
    }
}
```

#### 右旋

```java
private void rotateRight(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> l = p.left;
        p.left = l.right;
        if (l.right != null) l.right.parent = p;
        l.parent = p.parent;
        if (p.parent == null)
            root = l;
        else if (p.parent.right == p)
            p.parent.right = l;
        else p.parent.left = l;
        l.right = p;
        p.parent = l;
    }
}
```



### 增加

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        // 如果实体不支持自然排序，有没有传入排序器，报异常。此行代码只做类型检查
        compare(key, key); // type (and possibly null) check

        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)	// key < t.key
                t = t.left;
            else if (cmp > 0)	// key > t.key
                t = t.right;
            else
                return t.setValue(value);  // key和t.key相等时覆盖
        } while (t != null);
    }
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // t为空时会走到此步，parent为最后一个经过的有数据的节点
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;		// 返回的是旧的 value
}
```

```java
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;

    while (x != null && x != root && x.parent.color == RED) {
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            // y是x的叔叔结点
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            
            // 当前结点的父结点是红色且祖父结点的另一个子结点（叔叔结点）是红色
            // 将父结点和叔叔结点涂黑，将爷爷结点涂红
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
                
            // 当前结点的父结点是红色,叔叔结点是黑色，当前结点是其父结点的右子
            // 当前结点的父结点做为新的当前结点，以新当前结点为支点左旋
            } else {
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                
                // 当前结点的父结点是红色，叔叔结点是黑色，当前结点是其父结点的左子
                // 父结点变为黑色，祖父结点变为红色，在祖父结点为支点右旋
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            // 当前结点的父结点是红色且祖父结点的另一个子结点（叔叔结点）是红色
            // 将父结点和叔叔结点涂黑，将爷爷结点涂红
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
                
            // 当前结点的父结点是红色,叔叔结点是黑色，当前结点是其父结点的左子
            // 当前结点的父结点做为新的当前结点，以新当前结点为支点右旋
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                
                // 当前结点的父结点是红色,叔叔结点是黑色，当前结点是其父结点的左子
                // 父结点变为黑色，祖父结点变为红色，在祖父结点为支点左旋
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    root.color = BLACK;
}

```



### 删除

```java
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;
    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;		// 返回旧的value
}

private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // If strictly internal, copy successor's element to p and then make p
    // point to successor.
    if (p.left != null && p.right != null) {
        // s 是后继
        Entry<K,V> s = successor(p);
        
        // p 拷贝到 s
        p.key = s.key;
        p.value = s.value;
        
        // p 现在指向需要被删除的结点
        p = s;
    } // p has 2 children

    // 到了这里，无论原始的p有无孩子，p都是被即将被删除的结点
    // Start fixup at replacement node, if it exists.
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    // replacement指向有数据的子树，设为R子树，此子树会替代原始的子树
    if (replacement != null) {	// 有孩子
        // Link replacement to parent
        // R子树的父亲指向即将被删除的结点的父亲
        replacement.parent = p.parent;
        // 如果即将被删除的结点没有父亲（有孩子）
        if (p.parent == null)
            root = replacement;
       	// 把R子树挂上去
        else if (p == p.parent.left)
            p.parent.left  = replacement;
       	// 把R子树挂上去
        else
            p.parent.right = replacement;
		
        // 到了这步，p已经被替换掉了。将其的连接结点都置空
        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // Fix replacement
        // R子树上去后，如果是删除的结点是黑色，需要调整
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    // 和第39行的区别是：第39行的情况p是一个根结点，而此种情况就p这一个结点
    } else if (p.parent == null) { // return if we are the only node.
        root = null;
    // 没有孩子，有父亲，是个叶结点
    } else { //  No children. Use self as phantom replacement and unlink.
        if (p.color == BLACK)
            // 转下去分析修复
            fixAfterDeletion(p);

        if (p.parent != null) {
            if (p == p.parent.left)	
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}

```

```java
// x 是上一步的R子树
private void fixAfterDeletion(Entry<K,V> x) {
    
    // 真正要修复的情况是：被删除的结点是黑色且升上去的结点也是黑色（此时黑高会减1）
    // 算法的目的是让x指向的那棵树是红结点且整棵树不违反红黑树的性质
    while (x != root && colorOf(x) == BLACK) {
        
        // 总共有四种违反红黑树性质的情况。
        // 若被删除的结点树红色，直接删除，无任何影响
        // 若被删除的结点是黑色，但是上来的结点是红色，直接将其染黑就行了
        if (x == leftOf(parentOf(x))) {
            
            // 当前结点是黑+黑且兄弟结点为红色（此时父结点和兄弟结点的子结点为黑）
            Entry<K,V> sib = rightOf(parentOf(x));

            // 把父结点染成红色，把兄弟结点染成黑色，左旋
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                // 旋转不改变x的指向
                rotateLeft(parentOf(x));
                sib = rightOf(parentOf(x));
            }
            
            // ---到了这步，x仍然指向这一轮迭代初的结点，sib会转为黑色
            
			// 兄弟结点的两个子结点全为黑色，兄弟结点染红
            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);
                // 走到此步，退出循环，x此时为红色
                x = parentOf(x);
            } else {
                // 兄弟结点是黑色，兄弟的左子是红色，右子是黑色
                // 把兄弟结点染红，兄弟左子结点染黑，之后再以兄弟结点为支点解右旋
                if (colorOf(rightOf(sib)) == BLACK) {
                    setColor(leftOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateRight(sib);
                    // 调整兄弟结点的指向
                    sib = rightOf(parentOf(x));
                }
                // 到了这步，兄弟结点是黑色，兄弟结点的右子是红色
                // 把兄弟结点染成当前结点父结点的颜色，把当前结点父结点染成黑色，
                // 兄弟结点右子染成黑色，之后以当前结点的父结点为支点进行左旋
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(rightOf(sib), BLACK);
                rotateLeft(parentOf(x));
                x = root;
            }
        } else { // symmetric
            Entry<K,V> sib = leftOf(parentOf(x));

            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateRight(parentOf(x));
                sib = leftOf(parentOf(x));
            }

            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateLeft(sib);
                    sib = leftOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(leftOf(sib), BLACK);
                rotateRight(parentOf(x));
                x = root;
            }
        }
    }
    setColor(x, BLACK);
}
```



## 2-3树

红黑树是目前在工程应用中被证实综合性能最好的平衡树，但是为什么它就是那么好的？实际上红黑树吸收了AVL树和2-3树的特点。对于AVL树，是历史上第一个被创造的平衡二叉树，也是一颗绝对符合平衡二叉树性质的树。红黑树吸收了它的旋转思想来调整红黑树的状态。对于2-3树，红黑树其实就等价于等价于2-3树。

<div align="center"><img width="80%" src="http://blogfileqiniu.isjinhao.site/02fc799e-0209-4a6c-9e59-d22e20abfd70" /></div>

我们下面就分析2-3树。



### **树的定义**

2-3 树要么为空要么如下：

- 对于 2- 节点，和普通的 BST 节点一样，有一个数据域和两个子节点指针，两个子节点要么为空，要么也是一个2-3树，当前节点的数据的值要大于左子树中所有节点的数据，要小于右子树中所有节点的数据。
- 对于 3- 节点，有两个数据域 a 和 b 和三个子节点指针，左子树中所有的节点数据要小于a，中子树中所有节点数据要大于 a 而小于 b ，右子树中所有节点数据要大于 b 。

性质：

- 对于每一个结点有 1 或者 2 个关键码。
- 当节点有一个关键码的时，节点有 2 个子树。
- 当节点有 2 个关键码时，节点有 3 个子树。
- 所有叶子点都在树的同一层。



### 2-3树查找

2-3 树的查找类似二叉搜索树的查找过程，根据键值的比较来决定查找的方向。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/7a9975b2-b1e5-4e85-a3d8-373699586c8d" /></div>



### 2-3树插入

对于非空树插入主要分为 4 种情况：

- 向2-节点中插入新节点
- 向一棵只含3-节点的树中插入新节点
- 向一个父节点为2-节点的3-节点中插入新节点
- 向一个父节点为3-节点的3-节点中插入新节点

#### 向2-节点中插入新节点

操作步骤：如果未命中查找结束于一个2-节点，直接将2-节点替换为一个3-节点，并将要插入的键保存在其中。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/5dc4ca5b-97ff-4c4b-9ce7-65a8770fa19a" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/ce866562-a6c5-4760-acd1-d7881d6ddca6" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/8272d852-7b3c-43e9-a3f5-34201028cac7" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/1280f365-c4a0-4471-92f3-38f2b5b2c2b6" /></div>

#### 向一棵只含3-节点的树中插入新节点

操作步骤：先临时将新键存入唯一的3-节点中，使其成为一个4-节点，再将它转化为一颗由3个2- 节点组成的2-3 树，分解后树高会增加1。

<div align="center"><img width="35%" src="http://blogfileqiniu.isjinhao.site/46bb30b2-72c8-4561-8ab4-c0e02df8da4f" /></div>

#### 向一个父节点为2-节点的3-节点中插入新节点

操作步骤：先构造一个临时的4-节点并将其分解，分解时将中键移动到父节点中（中键移动后，其父节点中的位置由键的大小确定）

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/62715daf-f79a-4d48-8a8d-b5b18a33a2a4" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/a2ac891e-2393-442e-aa26-baf4e11dcd45" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/6b339d72-ff4b-4fb4-8c8a-03a5fab7c8a5" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/d2c3c783-d561-429a-9403-760a5e980462" /></div>

#### 向一个父节点为3-节点的3-节点中插入新节点

操作步骤：插入节点后一直向上分解构造的临时4-节点并将中键移动到更高层双亲节点，直到遇到一个2-节点并将其替换为一个不需要继续分解的3-节点，或是到达树根（3-节点）。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/54d169e0-f7b9-4f24-a91e-4f8dc47e4fc9" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/9e3c92ae-f6f3-4185-ac81-d5b4c5db196f" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/0830f682-8b41-4e99-ba94-bcf75bb977b8" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/d8909957-3511-4455-9d1d-583823f93eb2" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/d9479828-9eff-452c-989e-d6bf513c802a" /></div>

#### 分解根节点

操作步骤：如果从插入节点到根节点的路径上全是3-节点（包含根节点在内），根节点将最终被替换为一个临时的4-节点，将临时的4-节点分解为3个2-节点，分解后树高会增加1。

<div align="center"><img width="50%" src="http://blogfileqiniu.isjinhao.site/5f9b6cdd-7c80-4393-a1a7-95ee731442b6" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/be37d611-d780-422a-a4fd-27f090d424c7" /></div>



### 2-3树删除

删除之前，先要对2-3树进行一次命中的查找，查找成功才可以进行删除操作。删除节点大概分为3种情形

- 删除非叶子节点。
- 删除不为2-节点的叶子节点。
- 删除为2-节点的叶子节点。



#### 删除非叶子节点

操作步骤：使用中序遍历下的直接后继节点key来覆盖当前待删除节点key，再删除用来覆盖的后继节点key。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/ee6d3217-afbf-4b0b-b8c9-ec7de920a6c7" /></div>



#### 删除不为2-节点的叶子节点

操作步骤：删除不为2-节点的叶子节点，直接删除节点即可。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/c79107d2-9eb8-40e8-8d34-5eff0bb9e4ac" /></div>



#### 删除节点为2-节点，父节点为2-节点，兄弟节点为3-节点

操作步骤：当前待删除节点的父节点是2-节点、兄弟节点是3-节点，将父节点移动到当前待删除节点位置，再将兄弟节点中最接近当前位置的key移动到父节点中。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/750377ce-94ff-4afb-9108-d7cfc23faabf" /></div>





#### 删除节点为2-节点，父节点为2-节点，兄弟节点为2-节点

操作步骤：当前待删除节点的父节点是2-节点、兄弟节点也是2-节点，先通过移动兄弟节点的中序遍历直接后驱到兄弟节点，以使兄弟节点变为3-节点。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/109d04d5-7cca-4cd4-b7f5-36678cff9c2b" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/bf9bef8c-180a-439b-9912-b970209ecfbe" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/98055b64-8231-4e9c-a2bb-8a6032591433" /></div>

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/35273406-7c2b-4e28-a09c-67b891df51a2" /></div>

#### 删除节点为2-节点，父节点为3-节点

操作步骤：当前待删除节点的父节点是3-节点，拆分父节点使其成为2-节点，再将再将父节点中最接近的一个拆分key与中孩子合并，将合并后的节点作为当前节点。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/b7c34935-b732-4d83-a4e9-fd1ad03840b1" /></div>

#### 2-3树为满二叉树，删除叶子节点

操作步骤：若2-3树是一颗满二叉树，将2-3树层树减少，并将当前删除节点的兄弟节点合并到父节点中，同时将父节点的所有兄弟节点合并到父节点的父节点中，如果生成了4-节点，再分解4-节点。

<div align="center"><img width="60%" src="http://blogfileqiniu.isjinhao.site/755f55bc-b118-4852-b7c2-79242eb51955" /></div>



### 如何理解红黑树

对红黑树的每个结点，从该结点到其所有后代结点的简单路径上，均包含相同数目的黑色结点。也就是说其黑节点是一个绝对平衡的二叉树。所以整个树是不是平衡其实是看红色节点是不是产生了倾斜，但是有规定红节点的子节点是黑节点，确保整棵树不会偏的最狠，即使是红黑节点交叉的情况，双方的最大高度差值也是2倍，不会降为线性。为什么红节点的子节点全都是黑节点呢，因为如果不是，会产生红节点的儿子是红节点，红儿子的节点还是红节点的情况，最终一路红到底，破坏了性质5。

分析我们前面的代码会发现红黑树在插入删除的时候不需要进行递归，所以对红黑树的插入删除操作可以在O(1)的复杂度内完成。而AVL树最差的情况是O(lgn)。



## 部分转至

- https://www.itcodemonkey.com/article/13495.html