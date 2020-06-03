# LinkedHashMap

## afterNodeAccess

如果按访问顺序排列，将访问过后的元素插入到队列的尾端

```java
void afterNodeAccess(Node<K,V> e) { // 将访问过的元素移动到链表的尾部
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) { // 按照访问顺序排列，并且 e 元素不在队列的尾端
        // p == e 需要调整顺序的node节点
        LinkedHashMap.Entry<K,V> p = (LinkedHashMap.Entry<K,V>)e, 
        	// before 前驱节点
        	b = p.before,
        	// after 后继节点
        	a = p.after;
        // p 要被放到最后的，所以它的`after` = null
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        // 设置tail尾节点
        tail = p;
        ++modCount;
    }
}
```