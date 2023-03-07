# 数据结构（三）栈与队列


# 栈

栈是一种操作受限的线性表，只允许在一端插入（push()）和删除（pop()）数据。
特性： 后进者先出，先进者后出
使用场景： 如浏览器的前进、后退功能

## 使用数组实现一个简易的栈结构

```
  class CreateStack {
    private stack: any[] = [];

    public push(item: any) {
      this.stack.push(item);
      return true;
    }

    public pop() {
      if (this.stack.length === 0) {
        return null;
      }
      const tmp = this.stack[this.stack.length - 1];
      this.stack.pop();
      return tmp;
    }
}
```

# 队列

队列和栈非常的相似，最基本的操作也是两个： 入队（enqueue()），放一个数据到队列的尾部，出队（dequeue()）,从队头取出一个数据。
特性： 先进先出。

## 使用数组实现一个简易的队列结构

```
  class CreateStack {
    private stack: any[] = [];
    private headIndex: number = 0；
    public enqueue(item: any) {
      this.stack.push(item);
      return true;
    }

    public dequeue() {
      if (this.stack.length === headIndex) {
        return null;
      }
      const tmp = this.stack[headIndex];
      ++headIndex;
      return tmp;
    }
}
```

