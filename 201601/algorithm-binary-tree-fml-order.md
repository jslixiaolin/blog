二叉树是数据结构里经常使用的一种数据结构，需要注意其和树的区别（二叉树的一个节点最多只能有2个子树，而树没这个限制），还有完全二叉树和满二叉树。
创建如下图的一颗二叉树：
![这里写图片描述](http://img.blog.csdn.net/20151014224811773)
#### 一、创建二叉树

```
public class BinaryTree {
  private Node root=null;

  public BinaryTree(){
    root=new Node("A");
  }

  /**
   * 创建一颗二叉树
   * @param root
   */
  public void createBinaryTree(Node root){
    Node nodeB=new Node("B");
    Node nodeC=new Node("C");
    Node nodeD=new Node("D");
    Node nodeE=new Node("E");
    Node nodeF=new Node("F");
    Node nodeG=new Node("G");
    Node nodeH=new Node("H");
    Node nodeI=new Node("I");
    Node nodeM=new Node("M");
    Node nodeN=new Node("N");

    root.leftChild=nodeB;
    root.rightChild=nodeC;
    nodeB.leftChild=nodeD;
    nodeB.rightChild=nodeE;
    nodeD.leftChild=nodeH;
    nodeC.leftChild=nodeF;
    nodeC.rightChild=nodeG;
    nodeE.rightChild=nodeI;
    nodeI.leftChild=nodeM;
    nodeI.rightChild=nodeN;
  }

/**
   * 二叉树结点
   * @author: lixiaolin
   * @CreateDate: 2015-10-14 下午10:18:35
   * @version: 1.0
   */

  private class Node{
    private String data;
    private Node leftChild;
    private Node rightChild;

    public Node(){
    }

    public Node(String data){
      this.data=data;
      this.leftChild=null;
      this.rightChild=null;
    }
  }
```


#### 二、遍历二叉树
遍历是二叉树经常使用的操作。
常用的遍历方式有先序、中序、后序，这个是针对跟节点的访问顺序而言的。
下面先给出递归遍历二叉树的方法

```
/**
   * 先序递归遍历
   * @param root
   */
  public void preOrder(Node root){
    if(root==null){
      return;
    }
    visted(root);
    preOrder(root.leftChild);
    preOrder(root.rightChild);
  }

  /**
   * 中序递归遍历
   * @param root
   */
  public void inOrder(Node root){
    if(root==null){
      return ;
    }
    inOrder(root.leftChild);
    visted(root);
    inOrder(root.rightChild);
  }

  /**
   * 后序递归遍历
   * @param root
   */
  public void postOrder(Node root){
    if(root==null){
      return ;
    }
    postOrder(root.leftChild);
    postOrder(root.rightChild);
    visted(root);
  }

```

#### 三、非递归遍历
先序遍历和中序遍历的思路差不多，都在先遍历完所有的左子树，将之存储到栈中，利用栈的先进后出的特点，碰到有右子树的节点，先将其输出，然后加入栈中。
后序的思路不一样。要保证根结点在左孩子和右孩子访问之后才能访问，因此对于任一结点P，先将其入栈。如果P不存在左孩子和右孩子，则可以直接访问它；或者P存在左孩子或者右孩子，但是其左孩子和右孩子都已被访问过了，则同样可以直接访问该结点。若非上述两种情况，则将P的右孩子和左孩子依次入栈，这样就保证了每次取栈顶元素的时候，左孩子在右孩子前面被访问，左孩子和右孩子都在根结点前面被访问。

```
/**
   * 非递归先序遍历
   * @param root
   */
  public void noRecPreOrder(Node root){
    if(root==null){
      return ;
    }
    Node node=root;
    Stack<Node> stack=new Stack<Node>();
    while(node!=null){
      visted(node);
      stack.push(node);
      node=node.leftChild;
    }
    while(!stack.empty()){
      node=stack.pop();
      node=node.rightChild;
      while(node!=null){
        visted(node);
        stack.push(node);
        node=node.leftChild;
      }
    }

  }

  /**
   * 非递归实现中序遍历
   * @param root
   */
  public void noRecInOrder(Node root){
    if(root==null){
      return;
    }
    Stack<Node> stack=new Stack<Node> ();
    Node node=root;
    while(node!=null){
      stack.push(node);
      node=node.leftChild;
    }

    while(!stack.empty()){
      node=stack.pop();
      visted(node);
      node=node.rightChild;
      while(node!=null){
        stack.push(node);
        node=node.leftChild;
      }
    }
  }

  /**
   * 非递归后序遍历
   * @param root
   */
  public void noRecPostOrder(Node root){
    if(root==null){
      return;
    }
    Stack<Node> stack=new Stack<Node>();
    Node node=root;
    Node preVisted=null;
    stack.push(node);
    while(!stack.isEmpty()){
      node=stack.peek();
      if((node.leftChild==null&&node.rightChild==null)||(preVisted!=null&&(node.leftChild==preVisted||node.rightChild==preVisted))){
        node=stack.pop();
        visted(node);
        preVisted=node;
      }else{
        if(node.rightChild!=null){
          stack.push(node.rightChild);
        }
        if(node.leftChild!=null){
          stack.push(node.leftChild);
        }
      }
    }

  }

```