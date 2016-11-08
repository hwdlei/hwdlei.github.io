---
layout: post
title: 蚂蚁金服在线笔试题 - 链表两两反转
date: 2016-10-20
categories: 笔试面试
tags: [笔试,算法与数据结构]
description: 通告一下，我已不再每天写千字文，准备采用以下的方法进行练习，由于文章篇幅较长，链接较多，建议到简书或博客进行阅读。

---

## 问题
```
 // 一个链表：a->b->c->d->e
 // 每两个元素进行反转：b->a->d->c->e
 // 输入链表头指针
 // 输出：反转后的链表头指针
 // 要求：不新建节点
```

## 解答
```java
public class Main {

	public static class Node {
		public final String name;
		private Node next;
		public Node(String name) {
			this.name = name;
		}
		public void setNext(Node next) {
			this.next = next;
		}
		public Node getNext() {
			return this.next;
		}
		@Override
		public String toString() {
			return this.name;
		}
	}

	public static Node revertList(Node node) {
		if (node == null) {
			return null;
		}
		// 存储结果的头节点
		Node result = node;
		// 当前节点的上一个节点
		Node lastNode = null;
		// 当前节点的上上一个节点
		Node oldNode = null;
		int i = 1;
		while (node != null) {
			if (i % 2 != 0) {
			    // 奇数位，只需平移指针
				oldNode = lastNode;
				lastNode = node;
				node = node.getNext();
			} else {
			    // 偶数位考虑元素反转
				if (oldNode == null) {
				    // 前二个元素反转，无需考虑oldNode，但要注意result赋值
					lastNode.setNext(node.getNext());
					node.setNext(lastNode);
					result = node;
				} else {
				    // 3，4... 反转
					oldNode.setNext(node);
					lastNode.setNext(node.getNext());
					node.setNext(lastNode);
				}
				// 这里lastNode已经和Node反转了，所以只需更新oldNode和Node
				oldNode = node;
				node = lastNode.getNext();
			}
			i++;
		}
		return result;
	}

	public static void main(String[] args) {
	
		Node a = new Node("a");
		Node b = new Node("b");
		a.setNext(b);
		Node c = new Node("c");
		b.setNext(c);
		Node d = new Node("d");
		c.setNext(d);
		Node e = new Node("e");
		d.setNext(e);

		Node result = revertList(a);

		while (result != null) {
			System.out.print(result.name + "->");
			result = result.getNext();
		}
	}
}

```
