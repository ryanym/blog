+++
title = "Binary Tree Interview Questions"
date = "2020-03-04T18:07:04-05:00"
description = ""
tags = [
  "binary tree", 
  "tree",
  "graph",
]
categories = [ 
  "leetcode", 
  "interview prep",
]
draft  = false

+++

## Binary Tree

A binary tree is a special type of graph which cannot have any cycles. It is very different from binary search tree in term of properties and usage. Binary tree is an efficient way of representing hiearchical data, and its advanced forms are widely used in implementing database indices, sorting algorithms, encoders and decision-making process.

Binary Tree problems are great to test one's ability to **think recursively**, since recursion one of the fundamentals of CS. Therefore, when working with trees, we should always **treat every node as the root node** to generalize the solution.

A binary tree can be represented as below:

```java
public class TreeNode {
  int val;
  TreeNode left;
  TreeNode right;
  TreeNode(int x) { val = x; }
}
```

## Different ways of tree travesal

### Pre-order

In pre-order traversal, root node is visisted first, then the child nodes. 

[144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)

Recursive

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<Integer>();
        dfs(result, root);
        return result;
        
    }

    private void dfs(List<Integer> result, TreeNode node) {
        if (node != null) {
            result.add(node.val);
            dfs(result, node.left);
            dfs(result, node.right);
        }
    }
}
```

Iterative

```java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<Integer>();
        Deque<TreeNode> stack = new LinkedList<>();
        stack.push(root);

        while (!stack.isEmpty()) {
            TreeNode node = stack.pop();
            if (node != null) {
                result.add(node.val);
                stack.push(node.right);
                stack.push(node.left);
            }
        }
        return result;
    }
}
```



### In-order

In in-order traversal, tree nodes are traversed from left to right regardless of their levels

[94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

Recursive 

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<Integer>();
        dfs(result, root);
        return result;
    }

    private void dfs(List<Integer> result, TreeNode node){
        if (node != null) {
            dfs(result, node.left);
            result.add(node.val);
            dfs(result, node.right);
        }
    }
}		
```

Iterative

This is a little bit tricky as we need to visted the left-most node first. To do so, we use a stack to keep track of the parent nodes as we go down the left-most path of the tree. The first leaf node at the end of the path is our left-most node. We rewind the history by pop node off the stack to visit it's direct parent. After that we visit the right child of this subtree to finish the in-order traversal of the subtree. 

When we are at the right child of the subtree, we treat this node as a root of new subtree and repeat the process until the stack is empty.

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<Integer>();
        Deque<TreeNode> stack = new LinkedList<>();

        TreeNode current = root;
        // (current != null) because we don't have anything in the stack at the beginning
        while (!stack.isEmpty() || current != null) {
            // push nodes to the stack until we hit the left-most node of the tree
            if (current != null) {
                stack.push(current);
                current = current.left;
            } else {
                current = stack.pop();
                result.add(current.val);
                // treat the right child of the subtree as a root of new subtree then repeat the inner while loop
                current = current.right;
            }
        }
        return result;
    }
}
```





### Post-order

In post-order traversal, child nodes are visted first from left to right, then parent node.

[145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

Recursive

```java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<Integer>();
        dfs(result, root);
        return result;
    }

    private void dfs(List<Integer> result, TreeNode node) {
        if (node != null) {
            dfs(result, node.left);
            dfs(result, node.right);
            result.add(node.val);
        }
    }
}
```

Iterative

This is similar to iterative approach we took for in-order traversal. However instead of a `ArrayList` we use a `LinkedList` since we could insert in the front of the list in O(1) time.  

Why do we need to insert to the front of the list?  We're always given the root of the tree which is at the very top and we need the root node to apear last in the output. Once we have this in mind, we just "traverse" the tree in the opposite order of pre-order, we go from parent  to right child then left child and because we are inserting to the front of the list, the output would be reversed, i.e left -> right -> parent. 

Now we have the idea of the base case, we just have to extend the mechanism to the entire tree. We know that every node in the tree is a root of a subtree which we could apply the base case to. Therefore, we just need to travere the entire tree in a parent -> right -> left order using a stack. 

```java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        LinkedList<Integer> result = new LinkedList<Integer>();
        Deque<TreeNode> stack = new LinkedList<>();
        TreeNode current = root;
        while (!stack.isEmpty() || current != null) {
            if (current != null) {
                stack.push(current);
                // always append at the front of the list 
                result.addFirst(current.val);
                // visit and push the right node first
                current = current.right;
            } else {
                // when current node is null, it means we are at the end of right-most leaf, 
                // we then pop node off the stack to get the parent of the subtree
                current = stack.pop();
                // now visit the left node of the subtree, and treat it as the root of a new subtree
                current = current.left;
            }
        }
        return result;
    }
}
```



### Level-order

In level-order traversal, tree is traversed from top to bottom then left to right

[102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

Level-order traversal is essentially a breadth first search, it's very different from other traversal as in we don't treat each node as a "subtree", thus we can't apply any recursive concepts to solve this problem. An easier way of solving this is just look at the tree as a directed graph with no cycles and apply regular BFS techniques to visit all nodes.

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        if (root == null) return result;
        
        queue.add(root);
        
        while(!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();
            for(int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
            }
            result.add(level);
        }
        return result;
    }
}
```

 [107. Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)

This question is same as the last one, except we just need to output the levels in the reverse order. We could either reverse the output of last question with anther O(n) pass or we could reverse the order as we insert levels to the list using `LinkedList`

```java
class Solution {
    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        LinkedList<List<Integer>> result = new LinkedList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        if (root == null) return result;
        queue.add(root);

        while (!queue.isEmpty()) {
            int size = queue.size();
            List<Integer> level = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);

                if (node.left != null) queue.add(node.left);
                if (node.right != null) queue.add(node.right);
            }

            result.addFirst(level);
        }

        return result;
    }
}
```



## Tree Serialization and Deserialization

 [297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

```java
public class Codec {

    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        List<String> output = new ArrayList<>();
        serHelper(output, root);
        return String.join(",", output);
    }

    private void serHelper(List<String> output, TreeNode node) {
        if (node != null) {
            output.add(String.valueOf(node.val));
            serHelper(output, node.left);
            serHelper(output, node.right);
        } else {
            output.add("#");
        }
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        Queue<String> nodes = new LinkedList<>(Arrays.asList(data.split(",")));
        return deHelper(nodes);
    }

    private TreeNode deHelper(Queue<String> nodes) {

        String valStr = nodes.poll();
        if (valStr.equals("#")) return null;
        TreeNode node = new TreeNode(Integer.parseInt(valStr));
        node.left = deHelper(nodes);
        node.right = deHelper(nodes);
        return node;
    }
}
```

## Practice Questions

 [103. Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)

This can be solved using level order traversal we did above, only tweak needed is to zigzag between the levels.

<details>
  <summary>Solution</summary>

  ```java
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        LinkedList<TreeNode> queue = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) return res;
        queue.add(root);
        boolean leftToRight = true;
        while (!queue.isEmpty()) {
            int size = queue.size();
            LinkedList<Integer> level = new LinkedList<>();
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                if (leftToRight) {
                    level.add(node.val);
                } else {
                    level.addFirst(node.val);
                }

                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
            }
            res.add(level);
            leftToRight = !leftToRight;
        }

        return res;
        
    }
}
  ```
</details>

 [124. Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

For problems where you have to find the maximum or minimum property of a tree, I find it easier to come up with a solution with a helper method to do the recrusive call and keep the original method as the driver code to initialize variables. This way we have more freedom in terms of what to return in the recursive function.

In this particular problem, there are two key properties for maximum path sum:

1. a subtree on the max sum path must contribute either its left path sum or its right path sum to the path, i.e `root.val + Math.max(leftSum, rightSum)`
2. max sum path contains a leftSum, root and rightSum, meaning every node could be the root of the max sum path. Therefore to find the max path sum, we need to traverse the tree bottom up and keep update the max sum at every node. 

<details>
  <summary>Solution</summary>

  ```java
class Solution {
    private int res;
    public int maxPathSum(TreeNode root) {
        res = Integer.MIN_VALUE;
        helper(root);
        return res;
    }

    private int helper(TreeNode root) {
        if (root == null) {
            return Integer.MIN_VALUE;
        } else {
            int left = Math.max(0, helper(root.left));
            int right = Math.max(0, helper(root.right));
            res = Math.max(res, root.val + left + right);

            return root.val + Math.max(left, right);
        }
    }
}
  ```
</details>

[543. Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)

Diameter in the problem is defined as distance from one tree node to another. Since binary tree is a directed acyclic graph, there is one and only one path from one node to another, and their lowest common ancestor is the root of the path. 

Once we understand this part, this problem turns out to be very similar to the previous problem we solved: instead of the max sum path we now look for the max length path.

<details>
  <summary>Solution</summary>  

  ```java
class Solution {
    private int res;
    public int diameterOfBinaryTree(TreeNode root) {
        res = 0;
        helper(root);

        return res;
    }
    
    private int helper(TreeNode root) {
        if (root == null) {
            return 0;
        } else {
            int leftPath = helper(root.left);
            int rightPath = helper(root.right);
            res = Math.max(res, leftPath + rightPath);
            return 1 + Math.max(leftPath, rightPath);
        }
    }

}
  ```
</details>

