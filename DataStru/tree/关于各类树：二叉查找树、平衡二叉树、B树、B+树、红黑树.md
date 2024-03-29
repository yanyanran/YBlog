## 关于各类树：二叉查找树、平衡二叉树、B树、B+树、红黑树

#### 1、二叉查找树（BST）

**左结点小于root、右结点大于root**的一种排序树，也叫二叉搜索树。

顾名思义这种树的优点在于比平常的树查找速度更快，查找/插入/删除的时间复杂度都是***O（logN）***。

但是BST的***极端情况是当树变成了像线性链表一样的结构时，时间复杂度就变成了O（N）***。为了解决这种情况的出现就出现了二叉平衡树：



#### 2、二叉平衡树（AVL树）

AVL树全称为：二叉平衡搜索树，是一种*自平衡*的树。

AVL除了规定左结点小于root、右结点大于root之外，还**规定左右子树高度差不能大于1**，这样就保证了它不会成为线性的链表。

AVL树稳定，操作的时间复杂度都是O（logN）。但是由于AVL树需要维持自身的平衡，所以插入和删除结点操作的时候需要对结点进行频繁转换。

因为AVL树每个结点只能存放一个元素，并且每个元素只有两个子结点，所以查找操作时就需要***多次磁盘IO***。

> ***数据存放在磁盘中，每次查找都是将磁盘中的一页数据加入内存**；
>
> ***树的每一层节点存放在一页中，不同层数据存放在不同页***。

那么在多层查询的情况下就需要多次磁盘IO。为了解决这个问题就出现了B树：



#### 3、B树

B树叫做：平衡树（B-树），是一种多路平衡树。数据库的索引技术里就大量使用着B和B+树的数据结构。

和AVL树一样它也是一种自平衡的树，在插入和删除操作时也需要对结点进行旋转等操作。

可以理解为：对于B树而言它由AVL的“瘦高”变成了“**矮胖**”，它***将所有的叶子结点都放在了同一层（且所有节点的关键字都从小到大排列）***，这样在多层次遍历的时候就可以用最简单的解决方法相对减少磁盘IO的次数：

![](https://github.com/yanyanran/pictures/blob/main/B%E6%A0%91.png?raw=true)

- ##### B树的查找规则

  以上图举例：假如我们要查找元素E。

  首先第一步会与root比较，发现E<M，根据左小右大原则将目标放在左子树上。

  接下来拿到了关键字D和G，D<E<G所以直接找到DG的中间结点拿到E和F，通过对比发现E=E，从而返回关键字和指针信息（没有返回null）。

- ##### B树的插入规则

  对于B数的插入时结点的拆分我们规定：***关键字数s <= m - 1***

  那么对于我们想要定义一个5阶树（m=5的平衡五路查找树），也就是**当结点关键字数>4时就需要拆分了**

  举个例子来看构建一棵5阶B数树的过程：

  ![](https://github.com/yanyanran/pictures/blob/main/B%E6%95%B0%E7%9A%84%E6%8F%92%E5%85%A5.jpg?raw=true)

- ##### B树的删除规则

  对于每个结点的关键字数除了前面说的必须“>=路数-1”的上限之外，还有**下限：*s >=(m/2)-1***，小于下限时就要对结点进行***合并***操作

  也就是说，比如对于一个5路查找B树来说，当关键字数<2时就要进行结点合并

  还是拿上面那个树举例子，比如对于上面最后的定B树我们想要删除关键字28，我们发现删除关键字28后该结点关键字为1<2,代表需要合并结点了：

  ![](https://github.com/yanyanran/pictures/blob/main/B%E6%A0%91%E7%9A%84%E5%88%A0%E9%99%A4.jpg?raw=true)



> ***在数据库应用中，B树每个节点存储的数据量大约为4K。这是因为考虑到磁盘数据存储是采用块的形式存储的，每个块的大小为4K，每次对磁盘进行IO数据读取时，同一个磁盘块的数据会被一次性读取出来，所以每一次磁盘IO都可以读取到B树中一个节点的全部数据。***



可以看出**B树在节点空间利用率上改进，在每个节点保存了更多数据、减少了树的高度从而提升了查找性能**

但是B树也有不完善的点：

> 1. **查找不稳定**（最好的情况是在root就查到了，最坏的就是要到叶子结点才查到）
> 2. **遍历麻烦**（因为要用中序遍历所以也会进行一定数量的磁盘IO）

为了解决这些问题，B+树出现了：



#### 4、B+树

- ##### 关于数据存储：

### B+树中的非叶子结点用于保存数据索引，而真正的数据保存在叶子结点中。

> **插播一条：**
>
> **叶子结点是*没有子结点的结点*啊！！（有人今天考数据结构把叶子结点搞错成只有一个子结点的结点了可恶）**

由于非叶子结点不存放数据，所以每一层可以容纳更多元素，也就代表着：一页磁盘可存更多元素，因此**磁盘IO次数也会减少**。

可以看出来，因为所有数据都必须要到叶子结点才能获取到，所以每次数据查询的次数都一样，**查找速度稳定**。

- ##### 关于数据序列：

B+树中叶子结点中的数据和B树一样，都是**从小到大有序排列**。

- ##### 关于遍历速度：

B+树遍历整棵树只需要遍历所有的叶子结点即可，而B树需要对每一层都遍历，显然相比之下**B+树的全节点遍历更快**。