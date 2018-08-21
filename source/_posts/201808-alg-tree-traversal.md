---
title: 二叉树的前序、中序、后序遍历（java实现）
date: 2018-08-08 22:44:02
categories: 算法
tags:
    - 树遍历
    - 树算法
    - 算法
---

雄关漫道真如铁！而今迈步从头越。 =。=

# 0

![](http://p8vrqzrnj.bkt.clouddn.com/treetraversal.png)

- 前序遍历：4 2 1 3 6 5 7 8 10
- 中序遍历：1 2 3 4 5 6 7 8 10
- 后序遍历：1 3 2 5 10 8 7 6 4
- 层序遍历：4 2 6 1 3 5 7 8 10

# 前序遍历

## Recursive

``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    List<Integer> res = new ArrayList<Integer>();
    public List<Integer> preorderTraversal(TreeNode root) {
        if (root != null) {
            res.add(root.val);
            preorderTraversal(root.left);
            preorderTraversal(root.right);
        }
        return res;
    }
}
```

## Iterative

``` java
class Solution {
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        Stack<TreeNode> stack = new Stack<TreeNode>();
        
        while (root != null || !stack.isEmpty()) {
            while(root != null) {
                res.add(root.val);
                stack.push(root);
                root = root.left;
            }
            root = stack.pop();
            root = root.right;
        }
        return res;
    }
}
```

# 中序遍历

## Recursive

``` java
class Solution {
    List<Integer> res = new ArrayList<Integer>();
    public List<Integer> inorderTraversal(TreeNode root) {
        if (root != null) {
            inorderTraversal(root.left);
            res.add(root.val);
            inorderTraversal(root.right);
        }
        return res;
    }
}
```

## Iterative

``` java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        Stack<TreeNode> stack = new Stack<TreeNode>();
        while (root != null || !stack.isEmpty()) {
            while (root != null) {
                stack.push(root);
                root = root.left;
            }
            root = stack.pop();
            res.add(root.val);
            root = root.right;
        }
        return res;
    }
}
```

# 后续遍历

## Recursive

``` java
class Solution {
    List<Integer> res = new ArrayList<Integer>();
    public List<Integer> postorderTraversal(TreeNode root) {
        if (root != null) {
            postorderTraversal(root.left);
            postorderTraversal(root.right);
            res.add(root.val);
        }
        return res;
    }
}
```

## Iterative

``` java
class Solution {
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        Stack<TreeNode> stack = new Stack<TreeNode>();
        TreeNode lastAdd = null;
        while (!stack.isEmpty() || root != null) {
            if (root != null) {
                stack.push(root);
                root = root.left;
            } else {
                TreeNode node = stack.peek();
                if (node.right != null && node.right != lastAdd) {
                    root = node.right;
                } else {
                    res.add(node.val);
                    lastAdd = stack.pop();
                }
            }
        }
        return res;
    }
}
```

# TODO

多叉树的遍历