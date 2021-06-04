```java
interface Iterable<T> {
    // 返回迭代器
	Iterator<T> iterator();
    default void forEach(Consumer<? super T> action){
        // 元素为null,抛出空指针异常
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
}
```

```java
interface Collection<E> extends Iterable<E> {
    //Integer.MAX_VALUE大小限制
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    void clear();
}
```

```java
public interface Queue<E> extends Collection<E> {
    boolean add(E e);
    boolean offer(E e);
    E remove();
    E poll();
    E element();
    E peek();
}
```

```java
public abstract class AbstractQueue<E> 
    extends AbstractCollection<E> 
    implements Queue<E> {
    
    public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue null");
    }
    
    public E remove() {
        E x = poll();
        if (x != null) 
            return x;
        else 
            throw new NoSuchElementException();
    }
    
    public E element() {
        E x = peek();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }
    
    public void clear() {
        while (poll() != null);
    }
    
    public boolean addAll(Collection<? extends E> c) {
        if (c == null)
            throw new NullPointerException();
        if (c == this) 
            throw new IllegalArgumentException();
        boolean modified = false;
        for (E e: c)
            if (add(e))
                modified = true;
        return modified;
    }
}
```

```java
public interface BlockingQueue<E> extends Queue<E> {
    boolean add(E e);
    boolean offer(E e);
    void put(E e) throws InterruptedException;
    boolean offer(E e, long timeOut, TimeUnit unit) throws InterruptedException;
    E take() throw InterruptedException;
    E poll(long timeOut, TimeUnit unit) throws InterruptedException;
    // 剩余容量
    int remainingCapacity();
    boolean remove(Object o);
    boolean contains(Object o);
    // Removes all available elements from this queue and adds them to the given collection
    int drainTo(Collection<? super E> c);
    int drainTo(Collection<? super E> c, int maxElements);
}
```

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E> 
    implements BlockingQueue<E>, java.io.Serializable {
    final Object[] items;
    int takeIndex;
    int putIndex;
    int count;
    final ReentrantLock lock;
    private final Condition notEmpty;
    private final Condition notFull;
    transient Itrs itrs = null;
    
    public ArrayBlockingQueue(int capacity) {
        this(capacity, false);
    }
    
    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull = lock.newCondition();
    }
    
    public ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c) {
        this(capacity, fair);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            int i = 0;
            try {
                for(E e: c) {
                    checkNotNull(e);
                    items[i++] = e;
                }
            } catch (ArrayIndexOutOfBoundsException ex) {
                throw new IllegalArgumentException();
            }
            count = i;
            putIndex = (i == capacity) ? 0 : i;
        } finally {
            lock.unlock();
        }
    }
    
    final int dec(int i) {
        return ((i == 0) ? items.length : i) - 1;
    }
    final E itemAt(int i) {
        return (E)items[i];
    }
    private void enqueue(E x) {
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
        notEmpty.signal();
    }
    private E dequeue() {
        final Object[] items = this.items;
        E x = (E)items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeue();
        notFull.signal();
        return x;
    }
    void removeAt(final int removeIndex) {
        final Object[] items = this.items;
        if (removeIndex == takeIndex) {
            item[takeIndex] = null;
            if (++takeIndex == items.length) 
                takeIndex = 0;
            count--;
            if (itrs != null)
                itrs.elementDequeue();
        } else {
            final int putIndex = this.putIndex;
            for (int i = removeIndex;;) {
                int next = i + 1;
                if (next == items.len)
                    next = 0;
                if (next != putIndex) {
                    items[i] = items[next];
                    i = next;
                } else {
                    items[i] = null;
                    this.putIndex = i;
                    break;
                }
            }
            count--;
            if (itrs != null)
                itrs.removeAt(removeIndex);
        }
        notFull.signal();
    }
    
    public boolean offer(E e) {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count == items.length)
                return false;
            else {
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock()
        }
    }
    
    public void put(E e) throws InterruptedException {
        checkNotNUll(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
    
    public boolean offer(E e, long timeout, TimeUnit unit) throw InterruptedException {
        checkNotNull(e);
        long nanos = unti.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
                if (nanos <= 0)
                    return false;
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }
    
    public E poll() {
        final ReenTrantLock lock = this.lock;
        lock.lock();
        try {
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }
    
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    
    public E poll(long timeout, TimeUnit unit) throws InterruptedException {
        final ReentrantLock lock = this.lock;
        long nanos = unit.toNanos(timeout);
        lock.lockInterruptibly();
        try {
            while (count == 0) {
                if (nanos <= 0)
                    return null;
            	nanos = notEmpty.awaitNanos(nanos)
            }
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    
    public E peek() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return itemAt(takeIndex);
        } finally {
            lock.unlock();
        }
    }
    
    public int size() {
        final ReentrantLock lock = this.lock;
  		try {
            return count;
        } finally {
            lock.unlock();
        }
    }
    
    public int remainCapacity() {
        final ReentrantLock lock = this.lock;
        try {
            return items.length - count;
        } finally {
            lock.unlock();
        }
    }
    
    public boolean remove(Object o) {
        if (o == null) return false;
        final object[] = items;
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count > 0) {
                final int putIndex = this.putIndex;
                int i = takeIndex;
                do {
                    if (0.equals(items[i])) {
                        removeAt(i);
                        return true;
                    }
                    if (++i == items.length)
                        i = 0;
                } while (i != putIndex);
            }
            return false;
        } finally {
            lock.unlock();
        }
    }
    
    public boolean contains(Object o) {
        if (o == null) return false;
        final Object[] items = this.items;
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count > 0) {
                final int putIndex = this.putIndex;
            	int i = this.takeIndex;
            	do {
                	if (o.equals(items[i]))
                    	return true;
                	if (++i == item.lenght)
                    	i = 0;
            	} while (i != putIndex);
            } 
            return false;
        } finally {
            lock.unlock();
        }
    }
    
    public Object[] toArray() {
        Object[] a;
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            a = new Oject[count];
            int n = items.length - takeIndex;
            if (count <= n)
                System.arraycopy(items, takeIndex, a, 0, count);
            else {
                System.arraycopy(items, takeIndex, a, 0, n);
                System.arraycopy(items, 0, a, n, count - n);
            }
        } finally {
            lock.unlock()
        }
        return a;
    }
    
    public String toString() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            int k = this.count;
            if (k == 0) return "[]";
            final Object[] items = this.items;
            StringBuilder sb = new StringBuilder();
            sb.append("[");
            for (int i = takeIndex; ; ) {
                Object e = items[i];
                sb.append(e == this ? "(this colllection)" : e);
                if (--k == 0)
                    return sb.append("]").toString();
                sb.append(",").append(" ");
                if (++i == items.length)
                    
            }
        } finally {
            lock.unlock();
        }
    }
    
    private class Itr implements Iterator<E> {
        private int cursor;
        private E nextItem;
        private int nextIndex;
        private E lastItem;
        private int lastRet;
        private int prevTakeIndex;
        private int prevCycles;
        private static final int NONE = -1;
        private static final int REMOVED = -2;
        private static final int DETACHED = -3;
        Itr() {
            lastRet = NONE;
            final ReentrantLock lock = ArrayBlockingQueue.this.lock;
            lock.lock();
            try {
                if (count == 0) {
                    cursor = NONE;
                    nextIndex = NODE;
                } else {
                    final int tabkeIndex = ArrayBlockingQueue.this.takeIndex;
                }
            } finally {}
        }
    }
}
```

双端队列

```java
public interface Deque<E> extends Queue<E> {
    void addFirst(E e);
    void addLast(E e);
    boolean offerFirst(E e);
    boolean offerLast(E e);
    E removeFirst();
    E removeLast();
    E pollFirst();
    E pollLast();
    E getFirst();
    E getLast();
    E peekFirst();
    E peekLast();
    boolean removeFirstOccurrence(Object o);
    boolean removeLastOccurrence(Object o);
    boolean add(E e);
    boolean offer(E e);
    E remove();
    E poll();
    E element();
    E peek();
    void push(E e);
    E pop();
}
```

```java
public interface BlockingDeque<E> extends BlockingQueue<E>, Deque<E> {
    void addFirst(E e);
    void addLast(E e);
    boolean offerFirst(E e);
    boolean offerLast(E e);
    void putFirst(E e) throws InterruptedException;
    void putLast(E e) throws InterruptedException;
    boolean offerFirst(E e, long timeout, TimeUnit unit) throws InterruptedException;
    boolean offerLast(E e, long timeout, TimeUnit unit) throws InterruptedException;
    E takeFirst() throws InterruptedException;
    E takeLast() throws InterruptedException;
    E pollFirst(long timeout, TimeUnit unit) throws InterruptedException;
    E pollLast(long timeout, TimeUnit unit) throw InterruptedException;
    boolean removeFirstOccurrence(Object o);
}
```

