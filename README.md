# Netty NPE In GraalVM 25 and up

**Steps to reproduce**

* Clone this repo
* Set `JAVA_HOME` to some JDK gradle supports (I use JDK21)
* Set `GRAALVM_HOME` to a [GraalVM early access build](https://github.com/graalvm/oracle-graalvm-ea-builds) for JDK 25 or 26
    * I reproduced this issue with `jdk-25.0.0-ea.26` and `jdk-26.0.0-ea.03`
* Run `./gradlew nativeRun`
* Once the server started, in a different terminal run `echo "Hello" | nc localhost 8080` (or `netcat` if you don't have bsd-netcat)

Output from the server:

```
Starting server on port 8080
java.lang.NullPointerException
        at io.netty.util.internal.PlatformDependent0.safeConstructPutInt(PlatformDependent0.java:681)
        at io.netty.util.internal.PlatformDependent.safeConstructPutInt(PlatformDependent.java:569)
        at io.netty.util.internal.ReferenceCountUpdater.setInitialValue(ReferenceCountUpdater.java:67)
        at io.netty.buffer.AdaptivePoolingAllocator$Chunk.<init>(AdaptivePoolingAllocator.java:884)
        at io.netty.buffer.AdaptivePoolingAllocator$Magazine.newChunkAllocation(AdaptivePoolingAllocator.java:805)
        at io.netty.buffer.AdaptivePoolingAllocator$Magazine.allocate(AdaptivePoolingAllocator.java:703)
        at io.netty.buffer.AdaptivePoolingAllocator$Magazine.tryAllocate(AdaptivePoolingAllocator.java:574)
        at io.netty.buffer.AdaptivePoolingAllocator.allocate(AdaptivePoolingAllocator.java:230)
        at io.netty.buffer.AdaptivePoolingAllocator.allocate(AdaptivePoolingAllocator.java:217)
        at io.netty.buffer.AdaptiveByteBufAllocator.newDirectBuffer(AdaptiveByteBufAllocator.java:70)
        at io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:188)
        at io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:179)
        at io.netty.buffer.AbstractByteBufAllocator.ioBuffer(AbstractByteBufAllocator.java:140)
        at io.netty.channel.DefaultMaxMessagesRecvByteBufAllocator$MaxMessageHandle.allocate(DefaultMaxMessagesRecvByteBufAllocator.java:120)
        at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:151)
        at io.netty.channel.nio.AbstractNioChannel$AbstractNioUnsafe.handle(AbstractNioChannel.java:445)
        at io.netty.channel.nio.NioIoHandler$DefaultNioRegistration.handle(NioIoHandler.java:381)
        at io.netty.channel.nio.NioIoHandler.processSelectedKey(NioIoHandler.java:575)
        at io.netty.channel.nio.NioIoHandler.processSelectedKeysPlain(NioIoHandler.java:520)
        at io.netty.channel.nio.NioIoHandler.processSelectedKeys(NioIoHandler.java:493)
        at io.netty.channel.nio.NioIoHandler.run(NioIoHandler.java:468)
        at io.netty.channel.SingleThreadIoEventLoop.runIo(SingleThreadIoEventLoop.java:206)
        at io.netty.channel.SingleThreadIoEventLoop.run(SingleThreadIoEventLoop.java:177)
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:1073)
        at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
        at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
        at java.base@26/java.lang.Thread.runWith(Thread.java:1487)
        at java.base@26/java.lang.Thread.run(Thread.java:1474)
        at org.graalvm.nativeimage.builder/com.oracle.svm.core.thread.PlatformThreads.threadStartRoutine(PlatformThreads.java:832)
        at org.graalvm.nativeimage.builder/com.oracle.svm.core.thread.PlatformThreads.threadStartRoutine(PlatformThreads.java:808)
```
