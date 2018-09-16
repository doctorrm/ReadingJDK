# Vector类

<h2>开门见山</h2>

下图是Vector类的依赖，Vector和List等都是集合类中的容器类，既然属于集合类，那就肯定拥有集合类的普遍功能，比如存储，增删改查。但也避免不了和其它集合类一样的问题，包括容器扩容，多线程操作安全以及与其它容器类的活生生的性能对比。而以上的种种，尽在Vector的源码中，下面我们幽径通幽：
![](../IMAGES/vector.png)

<h2>混沌中的现形——构造方法</h2>

    public Vector(int initialCapacity, int capacityIncrement) {                  
        super();
        if (initialCapacity < 0)//参数负值校验
            throw new IllegalArgumentException("Illegal Capacity: " +
                    initialCapacity);
        this.elementData = new Object[initialCapacity];//创建容器
        this.capacityIncrement = capacityIncrement;
    }
之所以我们只看这个构造方法，是因为其它所有构造器到头来都是调用的这个构造方法。Vector初始化会创建一个指定大小的Object对象数组，如果你传入初始化大小参数为0，那因为Vector不会自动给你扩容，所以你要存数据的时候就会报错，因为溢出了。
正因为是Object超类数组，所以Vector可以存储任意类型的数据，那有人要问，如果没有明确指定类型，那我们需不需要用泛型来指定呢？答案是不需要，就算你指定了Integer的泛型，你还是可以存String类型的数据，因为数据是用Object数组来存的。
而扩容参数capacityIncrement只是赋值给全局变量做个标记，这有点像懒加载的味道，也就是说只有在当前进行add的操作会导致容量溢出时才会使用到capacityIncrement，这种设计让构造Vector的时候节省了不少力气，这种全局标记也方便被使用。
   
<h2>容器的操作——增删改查</h2>
有意思的东西来了，对操作而言，所有的写操作方法都被加上了synchronized悲观锁，保证每次在Vector增、删、改节点的时候，通过悲观锁来保证同步。因此Vector是线程安全的！
增：
add（）和insertElement（）方法，不仅可以往数组最后加一个数据，也可以加多个数据，还指定位置插入数据。在每次往Vector加数据前都会判断是否将导致容量溢出，如果会就调用下面的扩容方法实现扩容，注意，只有增操作可以触发扩容。

     private void grow(int minCapacity) {
            // 对数组溢出非常敏感的逻辑
            int oldCapacity = elementData.length;
            int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                    capacityIncrement : oldCapacity);
            if (newCapacity - minCapacity < 0)
                newCapacity = minCapacity;
            if (newCapacity - MAX_ARRAY_SIZE > 0)
                newCapacity = hugeCapacity(minCapacity);
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
        
扩容之后，指定下一个索引值为指定数据值即可。

注意：每个容器都是有容量极限的，Vector存储毫不例外。如下，Vector最大容量是整型整数的最大值-8，大概42亿。那为什么减8呢？原因是有些虚拟机会在编译时往Vector数组所在内存空间的前面加上一个字节头部信息，所以要减去8以作兼容避免内存溢出。

    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
