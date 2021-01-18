---
title: 微基准测试工具-JMH
date: 2020-12-02 15:11:27
tags: "JMH"
categories: "基准测试"
---

## 基准测试简介

### 什么是基准测试

基准测试是指通过设计科学的测试方法、测试工具和测试系统，实现对一类测试对象的某项性能指标进行定量的和可对比的测试。

现代软件常常都把高性能作为目标。那么，何为高性能，性能就是快，更快吗？显然，如果没有一个量化的标准，难以衡量性能的好坏。

不同的基准测试其具体内容和范围也存在很大的不同。如果是专业的性能工程师，更加熟悉的可能是类似SPEC提供的工业标准的系统级测试；而对于大多数 Java 开发者，更熟悉的则是范围相对较小、关注点更加细节的微基准测试（Micro-Benchmark）。何谓 Micro Benchmark 呢？ 简单地说就是在 **method** 层面上的 benchmark，精度可以精确到 **微秒级**。

### 何时需要微基准测试

微基准测试大多是 API 级别的性能测试。

微基准测试的适用场景：

- 如果开发公共类库、中间件，会被其他模块经常调用的 API。
- 对于性能，如响应延迟、吞吐量有严格要求的核心 API。

### Measure, don’t guess!

当我们的代码出现性能问题的时候，我们总是试图做一些小的改动（很可能是随意的改动）希望能对性能有所提升。相反，我们应该建立一个稳定的性能测试环境（包括操作系统，jvm，应用服务器，数据库等），设置一些性能目标，针对这一目标不断的进行测试，直到达到你的预期。和持续测试、持续交付类似，我们也应该进行持续的性能测试。

## JMH 简介

[JMH(即 Java Microbenchmark Harness) (opens new window)](http://openjdk.java.net/projects/code-tools/jmh/)，是目前主流的微基准测试框架。JMH 是由 Hotspot JVM 团队专家开发的，除了支持完整的基准测试过程，包括预热、运行、统计和报告等，还支持 Java 和其他 JVM 语言。更重要的是，它针对 Hotspot JVM 提供了各种特性，以保证基准测试的正确性，整体准确性大大优于其他框架，并且，JMH 还提供了用近乎白盒的方式进行 Profiling 等工作的能力。

### 应用场景

1. 当你已经找出了热点函数，而需要对热点函数进行进一步的优化时，就可以使用 JMH 对优化的效果进行定量的分析。
2. 想定量地知道某个函数需要执行多长时间，以及执行时间和输入 n 的相关性
3. 一个函数有两种不同实现（例如 JSON 序列化/反序列化有 Jackson 和 Gson 实现），不知道哪种实现性能更好

### JMH 概念

- `Iteration` - iteration 是 JMH 进行测试的最小单位，包含一组 invocations。
- `Invocation` - 一次 benchmark 方法调用。
- `Operation` - benchmark 方法中，被测量操作的执行。如果被测试的操作在 benchmark 方法中循环执行，可以使用`@OperationsPerInvocation`表明循环次数，使测试结果为单次 operation 的性能。
- `Warmup` - 在实际进行 benchmark 前先进行预热。因为某个函数被调用多次之后，JIT 会对其进行编译，通过预热可以使测量结果更加接近真实情况。

## JMH 快速入门

### 使用maven模板构建测试项目

```shell
mvn archetype:generate \
  -DinteractiveMode=false \
  -DarchetypeGroupId=org.openjdk.jmh \
  -DarchetypeArtifactId=jmh-java-benchmark-archetype \
  -DarchetypeVersion=1.26 \
  -DgroupId=org.sample \
  -DartifactId=jmh-test \
  -Dversion=1.0
```

使用该maven原型创建出来的项目结构如下：

> 一个包含了JMH相关依赖和[maven-shade-plugin](http://maven.apache.org/plugins/maven-shade-plugin/)插件的pom文件 
>
> 一个使用了`@Benchmark`注解的空的`MyBenchmark`文件

这个时候，虽然我们是什么都还没做，但是我们刚刚创建的微基准测试项目已经可以启动并运行了。使用maven命令打包就能生成一个benchmarks.jar

```
mvn clean install
java -jar target/benchmarks.jar
```

当我们使用以上命令运行这个jar时，我们就可以在控制台上看到一些有趣的内容输出：JMH进入循环、预热JVM，执行`@Benchmark`注解的空方法，并给出每秒操作的数量。

```
# JMH version: 1.26
# VM version: JDK 1.8.0_231, Java HotSpot(TM) 64-Bit Server VM, 25.231-b11
# VM invoker: /Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/bin/java
# VM options: <none>
# Warmup: 5 iterations, 10 s each
# Measurement: 5 iterations, 10 s each
# Timeout: 10 min per iteration
# Threads: 1 thread, will synchronize iterations
# Benchmark mode: Throughput, ops/time
# Benchmark: org.sample.MyBenchmark.testMethod

# Run progress: 0.00% complete, ETA 00:08:20
# Fork: 1 of 5
# Warmup Iteration   1: 3613498334.440 ops/s
# Warmup Iteration   2: 3564493205.447 ops/s
# Warmup Iteration   3: 3441462922.691 ops/s
# Warmup Iteration   4: 3507613901.262 ops/s
# Warmup Iteration   5: 3523187020.502 ops/s
Iteration   1: 2448307504.206 ops/s
Iteration   2: 1630681600.090 ops/s
Iteration   3: 3366861310.240 ops/s
Iteration   4: 3519422063.982 ops/s
Iteration   5: 3491225049.948 ops/s

# Run progress: 20.00% complete, ETA 00:06:41
# Fork: 2 of 5
# Warmup Iteration   1: 3568631032.943 ops/s
# Warmup Iteration   2: 3547153684.337 ops/s
# Warmup Iteration   3: 3593536219.916 ops/s
# Warmup Iteration   4: 3511642798.140 ops/s
# Warmup Iteration   5: 3612965640.138 ops/s
Iteration   1: 1528031676.317 ops/s
Iteration   2: 3538056385.238 ops/s
Iteration   3: 3617833508.560 ops/s
Iteration   4: 3632441119.590 ops/s
Iteration   5: 3513838533.836 ops/s

# Run progress: 40.00% complete, ETA 00:05:01
# Fork: 3 of 5
# Warmup Iteration   1: 3586337433.402 ops/s
# Warmup Iteration   2: 3592639002.694 ops/s
# Warmup Iteration   3: 3559066677.988 ops/s
# Warmup Iteration   4: 3603043794.379 ops/s
# Warmup Iteration   5: 3599842356.723 ops/s
Iteration   1: 3556557276.491 ops/s
Iteration   2: 3622054005.246 ops/s
Iteration   3: 3522844274.002 ops/s
Iteration   4: 3603419140.296 ops/s
Iteration   5: 3612098670.320 ops/s

# Run progress: 60.00% complete, ETA 00:03:20
# Fork: 4 of 5
# Warmup Iteration   1: 3620928591.647 ops/s
# Warmup Iteration   2: 3619157269.184 ops/s
# Warmup Iteration   3: 3618194251.462 ops/s
# Warmup Iteration   4: 3615122084.576 ops/s
# Warmup Iteration   5: 3625714964.134 ops/s
Iteration   1: 3613630491.468 ops/s
Iteration   2: 3590035487.192 ops/s
Iteration   3: 3590938922.626 ops/s
Iteration   4: 3624562485.347 ops/s
Iteration   5: 3594880556.938 ops/s

# Run progress: 80.00% complete, ETA 00:01:40
# Fork: 5 of 5
# Warmup Iteration   1: 3555228727.112 ops/s
# Warmup Iteration   2: 3571258125.822 ops/s
# Warmup Iteration   3: 3575934300.056 ops/s
# Warmup Iteration   4: 3585899277.939 ops/s
# Warmup Iteration   5: 3591348099.383 ops/s
Iteration   1: 3515206949.570 ops/s
Iteration   2: 3615852722.541 ops/s
Iteration   3: 3559577622.713 ops/s
Iteration   4: 3601415006.743 ops/s
Iteration   5: 3599049971.403 ops/s


Result "org.sample.MyBenchmark.testMethod":
  3364352893.396 ±(99.9%) 438208797.840 ops/s [Average]
  (min, avg, max) = (1528031676.317, 3364352893.396, 3632441119.590), stdev = 584996207.857
  CI (99.9%): [2926144095.556, 3802561691.236] (assumes normal distribution)


# Run complete. Total time: 00:08:21

REMEMBER: The numbers below are just data. To gain reusable insights, you need to follow up on
why the numbers are the way they are. Use profilers (see -prof, -lprof), design factorial
experiments, perform baseline and negative tests that provide experimental control, make sure
the benchmarking environment is safe on JVM/OS/HW level, ask for reviews from the domain experts.
Do not assume the numbers tell you what you want them to tell.

Benchmark                Mode  Cnt           Score           Error  Units
MyBenchmark.testMethod  thrpt   25  3364352893.396 ± 438208797.840  ops/s
```

### 现有项目引入maven依赖

因为 JMH 是 JDK9 自带的，如果是 JDK9 之前的版本需要加入如下依赖（目前 JMH 的最新版本为 `1.26）：

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.26</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.26</version>
    <scope>provided</scope>
</dependency>
```

### 测试代码

```
package org.sample;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Threads;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.results.format.ResultFormatType;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@Warmup(iterations = 3)
@Measurement(iterations = 5, time = 5, timeUnit = TimeUnit.SECONDS)
@Threads(10)
@Fork(1)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class StringConcatBenchmark {

    @Benchmark
    public void testStringAdd(Blackhole blackhole) {
        String a = "";
        for (int i = 0; i < 10; i++) {
            a += i;
        }
        blackhole.consume(a);
    }

    @Benchmark
    public void testStringBuilderAdd(Blackhole blackhole) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 10; i++) {
            sb.append(i);
        }
        blackhole.consume(sb);
    }

    @Benchmark
    public void testStringBufferAdd(Blackhole blackhole) {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < 10; i++) {
            sb.append(i);
        }
        blackhole.consume(sb);
    }

    public static void main(String[] args) throws RunnerException {
        Options options = new 		   OptionsBuilder().include(StringConcatBenchmark.class.getSimpleName())
            .output("StringAdd.log").resultFormat(ResultFormatType.JSON).build();
        new Runner(options).run();
    }
}

```

#### 执行 main 方法

执行 main 方法，耐心等待测试结果，最终会生成一个测试报告，内容大致如下:

```
# JMH version: 1.26
# VM version: JDK 11.0.6, Java HotSpot(TM) 64-Bit Server VM, 11.0.6+8-LTS
# VM invoker: /Library/Java/JavaVirtualMachines/jdk-11.0.6.jdk/Contents/Home/bin/java
# VM options: -javaagent:/Applications/IntelliJ IDEA 2.app/Contents/lib/idea_rt.jar=61459:/Applications/IntelliJ IDEA 2.app/Contents/bin -Dfile.encoding=UTF-8
# Warmup: 3 iterations, 10 s each
# Measurement: 5 iterations, 5 s each
# Timeout: 10 min per iteration
# Threads: 10 threads, will synchronize iterations
# Benchmark mode: Average time, time/op
# Benchmark: org.sample.StringConcatBenchmark.testStringAdd

# Run progress: 0.00% complete, ETA 00:02:45
# Fork: 1 of 1
# Warmup Iteration   1: 1133.878 ±(99.9%) 8.908 ns/op
# Warmup Iteration   2: 526.384 ±(99.9%) 4.519 ns/op
# Warmup Iteration   3: 539.472 ±(99.9%) 4.808 ns/op
Iteration   1: 506.017 ±(99.9%) 12.309 ns/op
Iteration   2: 605.922 ±(99.9%) 7.550 ns/op
Iteration   3: 490.174 ±(99.9%) 12.273 ns/op
Iteration   4: 526.695 ±(99.9%) 16.855 ns/op
Iteration   5: 512.370 ±(99.9%) 9.163 ns/op


Result "org.sample.StringConcatBenchmark.testStringAdd":
  528.235 ±(99.9%) 174.681 ns/op [Average]
  (min, avg, max) = (490.174, 528.235, 605.922), stdev = 45.364
  CI (99.9%): [353.554, 702.916] (assumes normal distribution)


# JMH version: 1.26
# VM version: JDK 11.0.6, Java HotSpot(TM) 64-Bit Server VM, 11.0.6+8-LTS
# VM invoker: /Library/Java/JavaVirtualMachines/jdk-11.0.6.jdk/Contents/Home/bin/java
# VM options: -javaagent:/Applications/IntelliJ IDEA 2.app/Contents/lib/idea_rt.jar=61459:/Applications/IntelliJ IDEA 2.app/Contents/bin -Dfile.encoding=UTF-8
# Warmup: 3 iterations, 10 s each
# Measurement: 5 iterations, 5 s each
# Timeout: 10 min per iteration
# Threads: 10 threads, will synchronize iterations
# Benchmark mode: Average time, time/op
# Benchmark: org.sample.StringConcatBenchmark.testStringBufferAdd

# Run progress: 33.33% complete, ETA 00:01:52
# Fork: 1 of 1
# Warmup Iteration   1: 216.710 ±(99.9%) 1.301 ns/op
# Warmup Iteration   2: 216.944 ±(99.9%) 1.575 ns/op
# Warmup Iteration   3: 217.175 ±(99.9%) 2.539 ns/op
Iteration   1: 231.397 ±(99.9%) 1.942 ns/op
Iteration   2: 237.466 ±(99.9%) 1.343 ns/op
Iteration   3: 251.344 ±(99.9%) 1.715 ns/op
Iteration   4: 234.449 ±(99.9%) 5.907 ns/op
Iteration   5: 215.355 ±(99.9%) 3.810 ns/op


Result "org.sample.StringConcatBenchmark.testStringBufferAdd":
  234.002 ±(99.9%) 49.740 ns/op [Average]
  (min, avg, max) = (215.355, 234.002, 251.344), stdev = 12.917
  CI (99.9%): [184.262, 283.743] (assumes normal distribution)


# JMH version: 1.26
# VM version: JDK 11.0.6, Java HotSpot(TM) 64-Bit Server VM, 11.0.6+8-LTS
# VM invoker: /Library/Java/JavaVirtualMachines/jdk-11.0.6.jdk/Contents/Home/bin/java
# VM options: -javaagent:/Applications/IntelliJ IDEA 2.app/Contents/lib/idea_rt.jar=61459:/Applications/IntelliJ IDEA 2.app/Contents/bin -Dfile.encoding=UTF-8
# Warmup: 3 iterations, 10 s each
# Measurement: 5 iterations, 5 s each
# Timeout: 10 min per iteration
# Threads: 10 threads, will synchronize iterations
# Benchmark mode: Average time, time/op
# Benchmark: org.sample.StringConcatBenchmark.testStringBuilderAdd

# Run progress: 66.67% complete, ETA 00:00:56
# Fork: 1 of 1
# Warmup Iteration   1: 307.429 ±(99.9%) 1.923 ns/op
# Warmup Iteration   2: 222.881 ±(99.9%) 2.113 ns/op
# Warmup Iteration   3: 215.555 ±(99.9%) 1.020 ns/op
Iteration   1: 223.996 ±(99.9%) 0.877 ns/op
Iteration   2: 220.913 ±(99.9%) 4.542 ns/op
Iteration   3: 239.446 ±(99.9%) 4.552 ns/op
Iteration   4: 246.228 ±(99.9%) 2.113 ns/op
Iteration   5: 211.977 ±(99.9%) 0.850 ns/op


Result "org.sample.StringConcatBenchmark.testStringBuilderAdd":
  228.512 ±(99.9%) 53.942 ns/op [Average]
  (min, avg, max) = (211.977, 228.512, 246.228), stdev = 14.009
  CI (99.9%): [174.570, 282.454] (assumes normal distribution)


# Run complete. Total time: 00:02:48

REMEMBER: The numbers below are just data. To gain reusable insights, you need to follow up on
why the numbers are the way they are. Use profilers (see -prof, -lprof), design factorial
experiments, perform baseline and negative tests that provide experimental control, make sure
the benchmarking environment is safe on JVM/OS/HW level, ask for reviews from the domain experts.
Do not assume the numbers tell you what you want them to tell.

Benchmark                                    Mode  Cnt    Score     Error  Units
StringAddBenchMarkDemo.testStringAdd         avgt    5  528.235 ± 174.681  ns/op
StringAddBenchMarkDemo.testStringBufferAdd   avgt    5  234.002 ±  49.740  ns/op
StringAddBenchMarkDemo.testStringBuilderAdd  avgt    5  228.512 ±  53.942  ns/op

Benchmark result is saved to jmh-result.json
```

## JMH 常用注解

### @BenchmarkMode

基准测试类型。这里选择的是 `Throughput` 也就是吞吐量。根据源码点进去，每种类型后面都有对应的解释，比较好理解，吞吐量会得到单位时间内可以进行的操作数。

- `Throughput` - 整体吞吐量，例如“1 秒内可以执行多少次调用”。
- `AverageTime` - 调用的平均时间，例如“每次调用平均耗时 xxx 毫秒”。
- `SampleTime` - 随机取样，最后输出取样结果的分布，例如“99%的调用在 xxx 毫秒以内，99.99%的调用在 xxx 毫秒以内”
- `SingleShotTime` - 以上模式都是默认一次 iteration 是 1s，唯有 SingleShotTime 是只运行一次。往往同时把 warmup 次数设为 0，用于测试冷启动时的性能。
- `All` - 所有模式

### @Warmup

上面我们提到了，进行基准测试前需要进行预热。一般我们前几次进行程序测试的时候都会比较慢， 所以要让程序进行几轮预热，保证测试的准确性。其中的参数 iterations 也就非常好理解了，就是预热轮数。

为什么需要预热？因为 JVM 的 JIT 机制的存在，如果某个函数被调用多次之后，JVM 会尝试将其编译成为机器码从而提高执行速度。所以为了让 benchmark 的结果更加接近真实情况就需要进行预热。

### @Measurement

度量，其实就是一些基本的测试参数。

- `iterations` - 进行测试的轮次
- `time` - 每轮进行的时长
- `timeUnit` - 时长单位

都是一些基本的参数，可以根据具体情况调整。一般比较重的东西可以进行大量的测试，放到服务器上运行。

### @Threads

每个进程中的测试线程，这个非常好理解，根据具体情况选择，一般为 cpu 乘以 2,可用于类或者方法上。

### @Fork

进行 fork 的次数。如果 fork 数是 2 的话，则 JMH 会 fork 出两个进程来进行测试。

### @OutputTimeUnit

这个比较简单了，基准测试结果的时间类型。一般选择秒、毫秒、微秒。

### @Benchmark

方法级注解，表示该方法是需要进行 benchmark 的对象，用法和 JUnit 的 @Test 类似。

### @Param

属性级注解，@Param 可以用来指定某项参数的多种情况。特别适合用来测试一个函数在不同的参数输入的情况下的性能。

### @Setup

方法级注解，这个注解的作用就是我们需要在测试之前进行一些准备工作，比如对一些数据的初始化之类的。

### @TearDown

方法级注解，这个注解的作用就是我们需要在测试之后进行一些结束工作，比如关闭线程池，数据库连接等的，主要用于资源的回收等。

### @State

当使用 @Setup 参数的时候，必须在类上加这个参数，不然会提示无法运行。

State 用于声明某个类是一个“状态”，然后接受一个 Scope 参数用来表示该状态的共享范围。 因为很多 benchmark 会需要一些表示状态的类，JMH 允许你把这些类以依赖注入的方式注入到 benchmark 函数里。Scope 主要分为三种。

- `Thread` - 该状态为每个线程独享。
- `Group` - 该状态为同一个组里面所有线程共享。
- `Benchmark` - 该状态在所有线程间共享。

## JMH 可视化

除此以外，如果你想将测试结果以图表的形式可视化，可以试下这些网站：

- `JMH Visual Chart`：[http://deepoove.com/jmh-visual-chart/](https://link.zhihu.com/?target=http%3A//deepoove.com/jmh-visual-chart/)
- `JMH Visualizer`：[https://jmh.morethan.io/](https://link.zhihu.com/?target=https%3A//jmh.morethan.io/)

比如将上面测试例子结果的 json 文件导入，就可以实现可视化：

![](/Users/yanzeyu/Blog/cat_house/source/images/JMH Visual Chart.png)

## 参考资料

- [jmh 官方示例(opens new window)](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/)
- [Java 微基准测试框架 JMH](https://www.xncoding.com/2018/01/07/java/jmh.html)
- https://dunwu.github.io/javatech/test/jmh.html
- https://www.cnblogs.com/Binhua-Liu/p/5617995.html
- https://www.zhihu.com/question/276455629

