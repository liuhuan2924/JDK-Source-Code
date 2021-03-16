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

