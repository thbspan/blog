## ByteBufAllocator

主要用于分配 `ByteBuf`，并根据需要添加内存泄漏检测功能

- 分配内存
- 内存泄漏检测功能
  - toLeakAwareBuffer 返回一个包装类

## AbstractByteBufAllocator

`ByteBufAllocator`抽象的实现类，提供公共方法，子类包括

- PooledByteBufAllocator
- UnpooledByteBufAllocator

