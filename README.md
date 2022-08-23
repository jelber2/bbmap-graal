# bbmap-graal

## Install Graal for JAVA 17

```sh
cd /nfs/scistore16/itgrp/jelbers/bin/
wget https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-22.2.0/graalvm-ce-java17-linux-amd64-22.2.0.tar.gz
tar xzf graalvm-ce-java17-linux-amd64-22.2.0.tar.gz

export PATH="/nfs/scistore16/itgrp/jelbers/bin/graalvm-ce-java17-22.2.0/bin:$PATH"
export JAVA_HOME=/nfs/scistore16/itgrp/jelbers/bin/graalvm-ce-java17-22.2.0
```

## Install BBMap-38.97

```sh
mkdir -p bbmap-38.97
wget https://sourceforge.net/projects/bbmap/files/BBMap_38.97.tar.gz
tar xzf BBMap_38.97.tar.gz --directory=bbmap-38.97
cd bbmap-38.97
mv bbmap/* .
rm -r bbmap/
```

## Edit -graal.sh versions of randomgenome.sh and bbduk.sh

make a copy of randomgenome.sh and name it randomgenome-graal.sh

same thing for bbduk.sh except bbduk-graal.sh

line 59 of randomgenome-graal.sh
should look like this

```bash
local CMD="java -XX:+UnlockExperimentalVMOptions -XX:+EnableJVMCI -XX:+UseJVMCICompiler $EA $EOOM $z $z2 $JNI -cp $CP jgi.BBDuk $@"
```

pretty much adding `-XX:+UnlockExperimentalVMOptions -XX:+EnableJVMCI -XX:+UseJVMCICompiler ` before `$EA`, same thing for bbduk-graal.sh, except line 366

then make sure the `-graal.sh` versions are executable doing, `chmod u+x bbduk-graal.sh` `chmod randomgenome-graal.sh`

## Benchmarking scripts

test.sh

```bash
#! /bin/bash
~/bin/bbmap-38.97/randomgenome.sh len=1000000000 out=/dev/null ow=t seed=1
```

test-graal.sh

```bash
#! /bin/bash
~/bin/bbmap-38.97/randomgenome-graal.sh len=1000000000 out=/dev/null ow=t seed=1
```

### make input for test2 scripts

```sh
~/bin/bbmap-38.97/randomgenome.sh len=1000000000 out=test.fasta ow=t seed=1
```

### test2 scripts

test2.sh

```bash
#! /bin/bash
~/bin/bbmap-38.97/bbduk.sh k=21 threads=8 ref=resources/truseq.fa.gz in=test.fasta out=/dev/null
```

test2-graal.sh

```bash
#! /bin/bash
~/bin/bbmap-38.97/bbduk-graal.sh k=21 threads=8 ref=resources/truseq.fa.gz in=test.fasta out=/dev/null
```

## Benchmarking with hyperfine

First install Rust following these instructions https://www.rust-lang.org/tools/install

Then install `hyperfine`

```sh
cargo install hyperfine
```

Make single-core scripts executable then benchmark

```sh
chmod u+x test.sh
chmod u+x test-graal.sh

/nfs/scistore16/itgrp/jelbers/.cargo/bin/hyperfine ./test.sh ./test-graal.sh
Benchmark 1: ./test.sh
  Time (mean ± σ):     14.456 s ±  0.102 s    [User: 15.592 s, System: 0.442 s]
  Range (min … max):   14.281 s … 14.669 s    10 runs
 
Benchmark 2: ./test-graal.sh
  Time (mean ± σ):     14.493 s ±  0.108 s    [User: 15.641 s, System: 0.440 s]
  Range (min … max):   14.331 s … 14.739 s    10 runs
 
Summary
  './test.sh' ran
    1.00 ± 0.01 times faster than './test-graal.sh'
```

Make multi-core scripts executable then benchmark

```sh
chmod u+x test2.sh
chmod u+x test2-graal.sh

/nfs/scistore16/itgrp/jelbers/.cargo/bin/hyperfine -i ./test2.sh ./test2-graal.sh
Benchmark 1: ./test2.sh
  Time (mean ± σ):     14.905 s ±  0.221 s    [User: 15.656 s, System: 1.523 s]
  Range (min … max):   14.531 s … 15.316 s    10 runs
 
Benchmark 2: ./test2-graal.sh
  Time (mean ± σ):     13.423 s ±  0.569 s    [User: 13.748 s, System: 1.489 s]
  Range (min … max):   12.806 s … 14.709 s    10 runs
 
Summary
  './test2-graal.sh' ran
    1.11 ± 0.05 times faster than './test2.sh'
```    
    

## Single-core benchmarks between OpenJ9 JIT https://github.com/eclipse-openj9/openj9/releases/tag/openj9-0.33.0

#### note openj9-0.33.0 compiled with Docker container then transferred to a Debian 11 server

test-jdk.sh

```bash
#! /bin/bash
test=`which java`
if [ "$test" =! "/mnt/nfs/clustersw/shared/java/jre/17/bin/java" ]
then
    module load java/17
fi
~/bin/bbmap-38.97/randomgenome.sh len=1000000000 out=/dev/null ow=t seed=1
```

test-openj9.sh

```bash
#! /bin/bash
test=`which java`
if [ "$test" =! "/nfs/scistore16/itgrp/jelbers/bin/openj9-jdk17/bin/java" ]
then
    export PATH="/nfs/scistore16/itgrp/jelbers/bin/openj9-jdk17/bin:$PATH"
fi
~/bin/bbmap-38.97/randomgenome.sh len=1000000000 out=/dev/null ow=t seed=1
```

```sh
/nfs/scistore16/itgrp/jelbers/.cargo/bin/hyperfine ./test-jdk.sh ./test-openj9.sh -i
Benchmark 1: ./test-jdk.sh
  Time (mean ± σ):     10.082 s ±  0.067 s    [User: 10.709 s, System: 0.232 s]
  Range (min … max):    9.970 s … 10.162 s    10 runs
 
Benchmark 2: ./test-openj9.sh
  Time (mean ± σ):     10.031 s ±  0.287 s    [User: 10.638 s, System: 0.252 s]
  Range (min … max):    9.231 s … 10.185 s    10 runs
 
  Warning: Statistical outliers were detected. Consider re-running this benchmark on a quiet PC without any interferences from other programs. It might help to use the '--warmup' or '--prepare' options.
 
Summary
  './test-openj9.sh' ran
    1.01 ± 0.03 times faster than './test-jdk.sh'
```

## Multi-core benchmarks

test2-jdk.sh

```bash
#! /bin/bash
test=`which java`
if [ "$test" =! "/mnt/nfs/clustersw/shared/java/jre/17/bin/java" ]
then
    module load java/17
fi
~/bin/bbmap-38.97/bbduk-graal.sh k=21 threads=8 ref=resources/truseq.fa.gz in=test.fasta out=/dev/null
```

test2-openj9.sh

```bash
#! /bin/bash
test=`which java`
if [ "$test" =! "/nfs/scistore16/itgrp/jelbers/bin/openj9-jdk17/bin/java" ]
then
    export PATH="/nfs/scistore16/itgrp/jelbers/bin/openj9-jdk17/bin:$PATH"
fi
~/bin/bbmap-38.97/bbduk-graal.sh k=21 threads=8 ref=resources/truseq.fa.gz in=test.fasta out=/dev/null
```
    
```sh
/nfs/scistore16/itgrp/jelbers/.cargo/bin/hyperfine ./test2-jdk.sh ./test2-openj9.sh -i
Benchmark 1: ./test2-jdk.sh
  Time (mean ± σ):     11.210 s ±  0.771 s    [User: 18.791 s, System: 1.895 s]
  Range (min … max):    9.948 s … 11.880 s    10 runs
 
  Warning: Statistical outliers were detected. Consider re-running this benchmark on a quiet PC without any interferences from other programs. It might help to use the '--warmup' or '--prepare' options.
 
Benchmark 2: ./test2-openj9.sh
  Time (mean ± σ):     11.492 s ±  0.611 s    [User: 18.985 s, System: 1.866 s]
  Range (min … max):    9.786 s … 11.891 s    10 runs
 
  Warning: Statistical outliers were detected. Consider re-running this benchmark on a quiet PC without any interferences from other programs. It might help to use the '--warmup' or '--prepare' options.
 
Summary
  './test2-jdk.sh' ran
    1.03 ± 0.09 times faster than './test2-openj9.sh'
```
