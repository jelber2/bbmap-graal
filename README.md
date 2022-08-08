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
```

## Edit -graal.sh versions of randomgenome.sh and bbduk.sh

make a copy of randgenome.sh and name it randomgenome-graal.sh

line 59 of randomgenome-graal.sh
should look like this

```bash
local CMD="java -XX:+UnlockExperimentalVMOptions -XX:+EnableJVMCI -XX:+UseJVMCICompiler $EA $EOOM $z $z2 $JNI -cp $CP jgi.BBDuk $@"
```

pretty much adding `-XX:+UnlockExperimentalVMOptions -XX:+EnableJVMCI -XX:+UseJVMCICompiler ` before `$EA`, same thing for bbduk-graal.sh, except line 366


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
https://www.rust-lang.org/tools/install

```sh
cargo install hyperfine
```

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
