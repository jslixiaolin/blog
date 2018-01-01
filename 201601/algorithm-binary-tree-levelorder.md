# 非递归实现二叉树的层次遍历

非递归的层次遍历其实很简单。利用了队列先进先出的特点。
先将根节点入队。如果队列不为空，那么获得队首元素，对其访问。如果它的左子树不为空，那么加入队列，如果它的右子树不为空，那么加入队列。

```
/**
   * 层次遍历
   * @param root
   */
  public void levelOrder(Node root){
    if(root==null){
      return;
    }
    Queue<Node> queue=new LinkedList<Node>();
    Node node=root;
    queue.add(node);
    while(!queue.isEmpty()){
      node=queue.poll();
      visted(node);
      if(node.leftChild!=null){
        queue.add(node.leftChild);
      }
      if(node.rightChild!=null){
        queue.add(node.rightChild);
      }
    }
  }
```
