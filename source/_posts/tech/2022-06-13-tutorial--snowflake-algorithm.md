---
title: 雪花算法原理与Java实现
date: 2022-06-13
tags: 
  - tutorial
  - snowflake algorithm
categories: technology
keywords: 'snowflake java'
---

# 1. 是什么
推特公司创建的一种在分布式系统中生成全局唯一ID的算法。

# 2. 原理
一个由雪花算法生成的ID由64个bit组成，其中：
- 第1位：占用1个bit，其值始终为0，表示是正整数。
- 第2-42位：占用41个bit，表示时间戳（即所选epoch以来的毫秒数），共可使用69年。
- 第43-52位：占用10个bit，表示不同实例。可以按需拆分使用，如2bit表示机房id，8bit表示机器ID，这样可以部署到2^2个机房，每个机房部署2^8台机器。
- 第43-64位：占用12个bit，记录同一毫秒内产生的不同ID，即一毫秒内同一机器可产生的ID数为2^12=4096个。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/238da66ce25e45ff8322c9fdc8e6c8b0~tplv-k3u1fbpfcp-watermark.image?)

# 3. 时钟回拨
机器时间是动态调整的，如果出现时间倒退，那么可能会出现生成重复ID的问题：机器M1在时间A生成了ID"0 A M1 1"，绝对时间来到了B，但由于各种原因，机器M1发生了时钟回退，时间被设置回了A，此时使用雪花算法生成ID，是存在再次生成ID"0 A M1 1"的可能的。

# 4. 优点及缺点
1. 优点：纯内存操作，速度极快
2. 缺点：TPS有上限，每秒最多只能生成2^12 * 1000 = 4096 * 1000个ID；另外还存在时钟回拨问题。

# 5. Java语言实现

[SnowflakeWorker:](https://github.com/hellooo-stack/hello-commons/blob/master/src/main/java/site/hellooo/commons/idgenerator/SnowflakeWorker.java)
```java
package site.hellooo.commons.idgenerator;

import java.util.Optional;

public class SnowflakeWorker {

//    64位比特位分布：按官方约定分，即：1个保留位、41位时间戳、10位工作机器id、12位序列号
//    其中，10位工作机器id(workId)分5位数据中心id(dataCenterId) + 5位机器id(machineId)
    private static final int SEQUENCE_BITS = 12;
    private static final int WORKER_ID_BITS = 10;
    private static final int TIMESTAMP_BITS = 41;
    private static final int REMAINING_BITS = 1;

//    41位时间戳从2022-07-01 00:00:00 开始计算
//    2022-07-01 00:00:00
    private static final long START_TIMESTAMP = 1656604800000L;

    private long dataCenterId;
    private long dataCenterIdBits;
    private long machineId;
    private long machineIdBits;

    private final long machineIdShift;
    private final long dataCenterIdShift;
    private final long timestampShift;

//    当前bit下的最大值
    private final long SEQUENCE_MASK = ~(-1L << SEQUENCE_BITS);
    private long sequence;
    private long lastTimestamp = -1L;


    private SnowflakeWorker(SnowflakeWorkerBuilder builder) {
        this.dataCenterId = builder.dataCenterId;
        this.dataCenterIdBits = builder.dataCenterIdBits;
        this.machineId = builder.machineId;
        this.machineIdBits = builder.machineIdBits;

        this.machineIdShift = SEQUENCE_BITS;
        this.dataCenterIdShift = machineIdShift + machineIdBits;
        this.timestampShift = dataCenterIdShift + WORKER_ID_BITS;
    }

    public synchronized long nextId() {
        long currentTimestamp = System.currentTimeMillis();
        boolean throwException = false;

        if (currentTimestamp == lastTimestamp) {
//            如果sequence溢出，那么0 & 最大值还是0
//            如果sequence不溢出，那么sequence & 最大值 还是sequence
            sequence = (sequence + 1) & SEQUENCE_MASK;
            if (sequence == 0) {
//                如果当前时间戳序列号已满，那么时间戳++，序号从0开始算
                currentTimestamp = tillNextMills(lastTimestamp);
            }
        } else if (currentTimestamp > lastTimestamp) {
            sequence = 0;
        } else {
            throwException = true;
        }

        if (throwException) {
            throw new IllegalArgumentException("Fatal: current timestamp is smaller that last timestamp, currentTimestamp=" + currentTimestamp + ", lastTimestamp=" + lastTimestamp);
        }

        lastTimestamp = currentTimestamp;

        return ((currentTimestamp - START_TIMESTAMP) << timestampShift)
                | (dataCenterId << dataCenterIdShift)
                | (machineId << machineIdShift)
                | sequence;
    }

    private long tillNextMills(long lastTimestamp) {
        long timestamp = System.currentTimeMillis();
        while (timestamp <= lastTimestamp) {
            timestamp = System.currentTimeMillis();
        }
        return timestamp;
    }

    public static final class SnowflakeWorkerBuilder {
        private static final long DEFAULT_DATA_CENTER_ID_BITS = 5;
        private static final long DEFAULT_MACHINE_ID_BITS = 5;

        private long dataCenterId;
        private long dataCenterIdBits = DEFAULT_DATA_CENTER_ID_BITS;
        private long machineId;
        private long machineIdBits = DEFAULT_MACHINE_ID_BITS;

        public SnowflakeWorker build() {
            Optional.of(this)
                    .filter(builder -> builder.dataCenterIdBits + builder.machineIdBits == WORKER_ID_BITS)
                    .orElseThrow(() -> new IllegalArgumentException("Fatal: dataCenterIdBits plus machineIdBits should be " + WORKER_ID_BITS + ", please check!"));

            return new SnowflakeWorker(this);
        }

        public SnowflakeWorkerBuilder dataCenterId(long dataCenterId) {
            this.dataCenterId = dataCenterId;
            long maxDataCenterId = ~(-1L << dataCenterIdBits);

            Optional.of(dataCenterId)
                    .filter(id -> id <= maxDataCenterId)
                    .orElseThrow(() -> new IllegalArgumentException("Fatal: dataCenterId should less than or equals " + maxDataCenterId + " but got " + dataCenterId + ", please check!"));

            return this;
        }

        public SnowflakeWorkerBuilder dataCenterIdBits(long dataCenterIdBits) {
            this.dataCenterIdBits = dataCenterIdBits;
            return this;
        }

        public SnowflakeWorkerBuilder machineId(long machineId) {
            this.machineId = machineId;
            long maxMachineId = ~(-1L << machineIdBits);

            Optional.of(machineId)
                    .filter(id -> id <= maxMachineId)
                    .orElseThrow(() -> new IllegalArgumentException("Fatal: maxMachineId should less than or equals " + maxMachineId + " but got " + machineId + ", please check!"));

            return this;
        }

        public SnowflakeWorkerBuilder machineIdBits(long machineIdBits) {
            this.machineIdBits = machineIdBits;
            return this;
        }
    }
}
```

[SnowflakeWorkerTest:](https://github.com/hellooo-stack/hellooo-commons/blob/master/src/test/java/site/hellooo/commons/idgenerator/SnowflakeWorkerTest.java)
```java
package site.hellooo.commons.idgenerator;

import org.junit.Test;

import java.util.HashSet;
import java.util.Set;

public class SnowflakeWorkerTest {
    @Test
    public void generating100000TimesAndUnique() {
        SnowflakeWorker snowflakeWorker = new SnowflakeWorker.SnowflakeWorkerBuilder()
                .dataCenterId(1)
                .machineId(21)
                .build();

        long start = System.currentTimeMillis();
        Set<Long> idSet = new HashSet<>();
        for (int i = 0; i < 100000; i++) {
            long id = snowflakeWorker.nextId();
            if (idSet.contains(id)) {
                System.out.println("num: " + idSet.size());
                System.out.println(id);
                throw new RuntimeException("id generated by snowflake duplicated");
            } else {
                idSet.add(id);
            }
        }
        long timing = System.currentTimeMillis() - start;
        System.out.println("computed num: " + idSet.size() + " timing: " + timing);
    }
}
```
