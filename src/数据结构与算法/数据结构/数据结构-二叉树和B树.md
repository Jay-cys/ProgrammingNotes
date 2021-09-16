数的优点=数组的优点+链表的优点


# 二叉树遍历算法

```.cpp
#ifndef BINARY_TREE_HPP
#define BINARY_TREE_HPP
#include <iostream>
#include <queue>

template<typename T>
class TreeNode
{
public:
    TreeNode()
    {
        leftChild = nullptr;
        rightChild = nullptr;
    }
    T data;
    TreeNode<T>* leftChild;
    TreeNode<T>* rightChild;
};

template<typename T>
class BinaryTree
{
public:
    //二叉树的遍历操作
    void InOrder(TreeNode<T>* currentNode);//中序遍历：左子树-节点-右子树
    void PreOrder(TreeNode<T>* currentNode);//前序遍历 节点-左子树-右子树
    void PostOrder(TreeNode<T>* currentNode);//后续遍历 左子树-右子树-节点
    void LevelOrder(TreeNode<T>* currentNode);//层序遍历, 一层一层显示value
    
    void showNodeValue(TreeNode<T>* currentNode);
    
    TreeNode<T>* root;   
};

template <typename T>
void BinaryTree<T>::InOrder(TreeNode<T>* currentNode)
{
    if (currentNode)
    {
        //左子树递归
        InOrder(currentNode->leftChild);
        //显示当前节点
        showNodeValue(currentNode);
        //右子树递归
        InOrder(currentNode->rightChild);
    }
}

template <typename T>
void BinaryTree<T>::PreOrder(TreeNode<T>* currentNode)
{
    if (currentNode)
    {
        //显示当前节点
        showNodeValue(currentNode);
        //左子树递归
        PreOrder(currentNode->leftChild);
        //右子树递归
        PreOrder(currentNode->rightChild);
    }
}

template <typename T>
void BinaryTree<T>::PostOrder(TreeNode<T>* currentNode)
{
    if (currentNode)
    {
        //左子树递归
        PostOrder(currentNode->leftChild);
        //右子树递归
        PostOrder(currentNode->rightChild);
        //显示当前节点
        showNodeValue(currentNode);
    }
}

template <typename T>
void BinaryTree<T>::LevelOrder(TreeNode<T>* currentNode)
{
    std::queue<TreeNode<T>*> q;//保存未显示的节点
    while (currentNode)
    {
        showNodeValue(currentNode);
        //显示当前节点后，将其左右子树加入队列
        if (currentNode->leftChild)
            q.push(currentNode->leftChild);
        if (currentNode->rightChild)
            q.push(currentNode->rightChild);

        if (q.empty())
            return;

        currentNode = q.front();//设置下次循环从队列去除一个新的节点
        q.pop();//删除队列第一个

    }

}

template <typename T>
void BinaryTree<T>::showNodeValue(TreeNode<T>* currentNode)
{
    std::cout<<currentNode->data;
}
#endif
```


# 二叉查找树

BST: binary search tree

- 每一个节点有一个值，不允许重复
- 左子树键值小于根节点键值
- 右子树键值大于根节点键值

```cpp
#ifndef BST_H
#define BST_H
#include <iostream>

template<typename T> class BST;//前置声明

template<typename T>
class Element
{
public:
    T key;//将键值单独一个类放置，方便扩展数据结构
};

template<typename T>
class BstNode
{
    friend class BST<T>;//友元类，可以访问私有成员
public:
    Element<T> data;
private:
    BstNode* leftChild;
    BstNode* rightChild;
};

template<typename T>
class BST
{
public:
    BST(BstNode<T>* init = 0)
    {
        root = init;
    }
    bool insertKey(const Element<T>& ele); //插入数据
    BstNode<T>* search(const Element<T>& ele);
    BstNode<T>* search(BstNode<T>* start, const Element<T>& ele);//递归查找数据
    BstNode<T>* forSearch(const Element<T>& ele);//迭代查找数据
    void display()
    {
        if (root)
            display(root);
        else
            std::cout<<"no nodes"<<std::endl;
    }
    void display(BstNode<T>* node);//显示节点数据
private:
    BstNode<T>* root;
};

template<typename T>
void BST<T>::display(BstNode<T>* node)
{
    if (!node)
    {
        return;
    }
    //显示当前节点及其左右子树的key
    std::cout<<"data.key: "<<node->data.key<<std::endl;
    if (node->leftChild)
    {
        display(node->leftChild);
    }
    if (node->rightChild)
    {
        display(node->rightChild);
    }
}

template<typename T>
bool BST<T>::insertKey(const Element<T>& ele)
{
    BstNode<T>* p = root;
    BstNode<T>* q = nullptr; //q为p的父节点
    //插入之前需要先查找，找到合适的位置
    while (p)
    {
        q = p; //p改变之前，q指向当前节点
        if (ele.key == p->data.key)
        {
            return false;//值重复，失败返回
        }
        else if (ele.key < p->data.key)
        {
            //小于，继续查找当前节点左子树
            p = p->leftChild;
        }
        else
        {
            //大于，继续查找当前节点右子树
            p = p->rightChild;
        }
    }
    //循环结束，合适的位置就是q
    p = new BstNode<T>;//创建新节点
    p->leftChild = nullptr;
    p->rightChild = nullptr;
    p->data = ele;
    if (!root)
        root = p; //没有根节点，p就是根节点
    else if (ele.key < q->data.key)
    {
        q->leftChild = p;
    }
    else
    {
        q->rightChild = p;
    }

    return true;
}

template<typename T>
BstNode<T>* BST<T>::search(const Element<T>& ele)
{
    if (root)
        return search(root, ele);
    else
        return nullptr;
}

template<typename T>
BstNode<T>* BST<T>::search(BstNode<T>* start, const Element<T>& ele)
{
    if (!start)
        return nullptr;

    if (start->data.key == ele.key)
    {
        return start;
    }
    else if (start->data.key > ele.key)
    {
        return search(start->leftChild, ele);
    }
    else
    {
        return search(start->rightChild, ele);
    }
}

template<typename T>
BstNode<T>* BST<T>::forSearch(const Element<T>& ele)
{
    for (BstNode<T>* n = root; n != nullptr; )
    {
        if (ele.key == n->data.key)
        {
            return n;
        }
        else if (ele.key < n->data.key)
        {
            n = n->leftChild;
        }
        else
        {
            n = n->rightChild;
        }
    }
     return nullptr;
}
#endif // BST_H
```


# 红黑树：高级二叉查找树

C++标准库中set，multiset，map，multimap等都是红黑树的实现。
红黑树特征：节点有颜色，插入和删除节点是要遵守红黑规则：

- 每一个节点不是红色就是黑色
- 根节点是黑色
- 如果节点是红色，则它的子节点必须是黑的。即红色不能相邻
- 从根到叶子节点的每条路经，必须包含相同数目的黑色节点

如果当前二叉树不符合红黑规则，有以下两种修正方式：

- 改变节点的颜色
- 旋转


## 旋转

旋转分为单旋转和双旋转，应用于下面场景：

![snipaste_20181118_201243.png](.assets/1577678880087-bc7b6840-601c-41cc-8597-dc8d4f81f381.png)


## 代码

红黑树代码讲解：[https://www.bilibili.com/video/av31763085/?p=28](https://www.bilibili.com/video/av31763085/?p=28)
头文件：[redblacktree.h](https://www.yuque.com/attachments/yuque/0/2020/txt/690827/1582507801061-58d564ab-caf4-4147-9cf4-656a76b76bb3.txt?_lake_card=%7B%22uid%22%3A%221577678899557-0%22%2C%22src%22%3A%22https%3A%2F%2Fwww.yuque.com%2Fattachments%2Fyuque%2F0%2F2020%2Ftxt%2F690827%2F1582507801061-58d564ab-caf4-4147-9cf4-656a76b76bb3.txt%22%2C%22name%22%3A%22redblacktree.h%22%2C%22size%22%3A6666%2C%22type%22%3A%22text%2Fplain%22%2C%22ext%22%3A%22txt%22%2C%22progress%22%3A%7B%22percent%22%3A99%7D%2C%22status%22%3A%22done%22%2C%22percent%22%3A0%2C%22id%22%3A%22srUVA%22%2C%22card%22%3A%22file%22%7D)

```cpp
#ifndef REDBLACKTREE_H
#define REDBLACKTREE_H
#include "except.h"

template <class T>
class RedBlackTree;

template <typename T>
class RedBlackNode;

//红黑树
template <typename T>
class RedBlackTree
{
public:
    enum {RED, BLACK};
    typedef RedBlackNode<T> Node;

    RedBlackTree(const T& negInf);
    ~RedBlackTree();

    void insert(const T& x);
    void rotateWithLeftChild(Node* & K2) const; //单旋转，向右
    void rotateWithRightChild(Node* & k1) const; //单旋转，向左
    void doubleRotateWithLeftChild(Node* & K2) const; //双旋转
    void doubleRotateWithRightChild(Node* & k1) const;//双旋转
    void handleReorient(const T& item); //平滑处理
    RedBlackNode<T>* rotate(const T& item, Node* parent); //旋转函数

    bool isEmpty();
    void makeEmpty();
    void deleteNodes(Node* start);//删除从start开始往后所有节点

    bool find(const T& x, Node* node);
    bool findMin(T& value);
    bool findMax(T& value);

    //private:  //just for test
    Node* header; //指向root根节点的指针
    Node* nullNode; //无子节点

    Node* current;
    Node* parent; //父节点
    Node* grand; //祖父节点
    Node* great; //曾祖父节点
};

//红黑树节点
template <typename T>
class RedBlackNode
{
public:
    friend class RedBlackTree<T>;
    RedBlackNode(const T& theElement = T(),
                 RedBlackNode* lt = nullptr,
                 RedBlackNode* rt = nullptr,
                 int c = RedBlackTree<T>::BLACK)
    {
        element = theElement;
        left = lt;
        right = rt;
        color = c;
    }
    //private:
    T element;
    RedBlackNode* left;
    RedBlackNode* right;
    int color; //颜色
};

template <typename T>
RedBlackTree<T>::RedBlackTree(const T& neginf)
{
    nullNode = new Node();
    nullNode->left = nullNode->right = nullNode;

    //构造函数指定header为负无穷大
    header = new Node(neginf);
    header->left = header->right = nullNode;
}

template <typename T>
RedBlackTree<T>::~RedBlackTree()
{
    makeEmpty();
    delete nullNode;
    delete header;
}

template <typename T>
void RedBlackTree<T>::insert(const T& x)
{
    current = parent = grand = header;
    nullNode->element = x;
    //查找位置
    while (current->element != x)
    {
        great = grand;
        grand = parent;
        parent = current;

        current = x < current->element ? current->left : current->right;

        //检查：如果当前节点，左右儿子节点都是红色，需要处理
        if (current->left->color == RED && current->right->color == RED)
        {
            handleReorient(x);
        }
    }

    if (current != nullNode)
        throw  DuplicateItemException();

    current = new Node(x, nullNode, nullNode);
    if (x < parent->element)
        parent->left = current;
    else
        parent->right = current;

    //自动平滑当前树，变为红黑树(重点)
    handleReorient(x);
}

//向右旋转
template <typename T>
void RedBlackTree<T>::rotateWithLeftChild(Node* & k2) const
{
    Node* k1 = k2->left;
    k2->left = k1->right;
    k1->right = k2;
    k2 = k1;
}

//向左旋转
template <typename T>
void RedBlackTree<T>::rotateWithRightChild(Node* & k1) const
{
    Node* k2 = k1->right;
    k1->right = k2->left;
    k2->left = k1;
    k1 = k2;
}

template <typename T>
void RedBlackTree<T>::doubleRotateWithLeftChild(Node* & k2) const
{
    rotateWithRightChild(k2->left);
    rotateWithLeftChild(k2);
}

template <typename T>
void RedBlackTree<T>::doubleRotateWithRightChild(Node* & k1) const
{
    rotateWithLeftChild(k1->right);
    rotateWithRightChild(k1);
}

template <typename T>
void RedBlackTree<T>::handleReorient(const T& item)
{
    //变色
    current->color = RED;
    current->left->color = BLACK;
    current->right->color = BLACK;

    if (parent->color == RED)
    {
        grand->color = RED;
        if ((item < grand->element) != (item < parent->element))
        {
            parent = rotate(item, grand);//单旋转
        }
        current = rotate(item, great);//双旋转
        current->color = BLACK;
    }
    header->right->color = BLACK;
}

/*
 * 左子树向右转--LL
 * 左子树向左转--LR
 * 右子树向右转--RL
 * 右子树向左转--RR
 */
template <typename T>
RedBlackNode<T>* RedBlackTree<T>::rotate(const T& item, Node* parent)
{
    if (item < parent->element)
    {
        item < parent->left->element ? rotateWithLeftChild(parent->left)
                                     : rotateWithRightChild(parent->left);
        return parent->left;
    }
    else
    {
        item < parent->right->element ? rotateWithLeftChild(parent->right)
                                      : rotateWithRightChild(parent->right);
        return parent->right;
    }
}

template <typename T>
bool RedBlackTree<T>::isEmpty()
{
    return header->right == nullNode;
}

template <typename T>
void RedBlackTree<T>::makeEmpty()
{
    deleteNodes(header->right);
    header->right = nullNode;
}

template <typename T>
void RedBlackTree<T>::deleteNodes(Node* start)
{
    //使用递归
    if (start != start->left)
    {
        deleteNodes(start->left);
        deleteNodes(start->right);
        delete start;
    }
}

template <typename T>
bool RedBlackTree<T>::find(const T& x, Node* node)
{
    if(isEmpty())
        return false;

    nullNode->element = x;
    Node* start = header->right;
    for(;;)
    {
        if (x < start->element)
        {
            start = start->left;
        }
        else if (x > start->element)
        {
            start = start->right;
        }
        else if (start != nullNode)
        {
            node->element = start->element;
            return true;
        }
        else
            return false;
    }
}

template <typename T>
bool RedBlackTree<T>::findMin(T& value)
{
    if(isEmpty())
        return false;

    //一直向左查找最小
    Node* node = header->right;
    while (node->left != nullNode)
    {
        node = node->left;
    }

    value = node->element;
    return true;
}

template <typename T>
bool RedBlackTree<T>::findMax(T& value)
{
    if(isEmpty())
        return false;

    //一直向左查找最小
    Node* node = header->right;
    while (node->right != nullNode)
    {
        node = node->right;
    }

    value = node->element;
    return true;
}
#endif // REDBLACKTREE_H
```


# B树

## B树用途
B树和红黑树类似，单可以有多个子节点，对减少磁盘IO操作次数有显著作用。  
B树主要用途如下：

- 实现文件系统
- 实现数据库系统的数据文件和索引文件

## B数基本概念
![image.png](.assets/1577679065434-8f176c54-fae4-43e5-821e-89b32b5c4b40.png)

