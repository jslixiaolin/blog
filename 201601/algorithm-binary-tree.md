# 二叉树的常用操作
二叉树就是每个结点最多有两个子树的树形存储结构。

## 一、求二叉树高度
```
/**
   * 求二叉树高度
   * @param root
   * @return
   */
  public int getHeight(Node root){
    if(root==null){
      return 0;
    }
    int l=getHeight(root.leftChild);
    int r=getHeight(root.rightChild);
    return l>r?l+1:r+1;
  }
```

## 二、求二叉树结点总数
```
/**
   * 求二叉树结点总数
   * @param root
   * @return
   */
  public int getSize(Node root){
    if(root==null){
      return 0;
    }
    return 1+getSize(root.leftChild)+getSize(root.rightChild);

  }
```

## 三、递归交换左右子树
```
/**
   * 交换左右子树
   * @param root
   */
  public void swapLeftAndRight(Node root){
    if(root==null){
      return;
    }
    Node temp=new Node();
    temp=root.leftChild;
    root.leftChild=root.rightChild;
    root.rightChild=temp;
    swapLeftAndRight(root.leftChild);
    swapLeftAndRight(root.rightChild);
  }
```

## 四、非递归交换左右子树
```
/**
   * 非递归交换左右子树
   * @param root
   */
  public void swapLeftAndRightNoRec(Node root){
    if(root==null){
      return;
    }
    Node node=root;
    Queue<Node> queue=new LinkedList<Node>();
    queue.add(node);
    while(!queue.isEmpty()){
      node=queue.poll();
      Node temp=new Node();
      temp=node.leftChild;
      node.leftChild=node.rightChild;
      node.rightChild=temp;
      if(node.leftChild!=null){
        queue.add(node.leftChild);
      }
      if(node.rightChild!=null){
        queue.add(node.rightChild);
      }
    }
  }
```

## 五、在二叉树中查找某个节点

```
/**
   * 递归查找二叉树中是否存在某个节点
   * @param root
   * @param cNode
   * @return
   */
  public boolean checkNode(Node root,Node cNode){
    if(root==null){
      return false;
    }
    else if(root==cNode){
      return true;
    }
    else{
      boolean existsFlag=false;
      if(root.leftChild!=null){
        existsFlag=checkNode(root.leftChild, cNode);
      }
      if(!existsFlag&&root.rightChild!=null){
        existsFlag=checkNode(root.rightChild, cNode);
      }
      return existsFlag;
    }
  }
```

## 六、查找两个结点的最近父节点

```
public Node getNearsetFarther(Node root,Node node1,Node node2){
    //如果node2的node1的子树中
    if(checkNode(node1, node2)){
      return node1;
    }
    //如果node1在node2的子树中
    if(checkNode(node2, node1)){
      return node2;
    }
    boolean oneInLeft,oneInRight,twoInLeft,twoInRight;
    oneInLeft=checkNode(root.leftChild, node1);
    oneInRight=checkNode(root.rightChild, node1);
    twoInLeft=checkNode(root.leftChild,node2);
    twoInRight=checkNode(root.rightChild, node2);

    //node1和node2不在同一边
    if((oneInLeft&&twoInRight)||(oneInRight&&oneInLeft)){
      return root;
    }
    //node1和node2都在左边
    if(oneInLeft&&twoInLeft){
      return getNearsetFarther(root.leftChild, node1, node2);
    }
    //node1和node2都在右边
    if(oneInRight&&twoInRight){
      return getNearsetFarther(root.rightChild, node1, node2);
    }
    else{
      return null;
    }
  }
```