# DockerfileFuzzer (Concept, no code available yet)

Generates random `Dockerfile` for finding [real bugs of the storage drivers](https://github.com/AkihiroSuda/docker-issues/). (AUFS/Overlay/BtrFS..)

This is just a concept.
I'm planning to implement this, but not working on at the moment.

## Concept
### Motivation
[Storage drivers are bug-prone](https://github.com/AkihiroSuda/docker-issues/).
But it's often hard to reproduce these bugs.
DockerfileFuzzer will generate a `Dockerfile` which can (almost) deterministically reproduce bugs.

Obviously, DockerfileFuzzer is __less__ effective than syscall-level fuzzers (e.g. [Trinity](http://codemonkey.org.uk/projects/trinity/)) in point of the state explorability.
Actually I'm even not sure DockerfileFuzzer can find bugs.
But it is attractive in point of its reproducibility that is clear to everyone without any tool-specific knowledge.

### Test Oracle

1. Clone random 100 github repositories that contain `Dockerfile`. This repositorie set is used as a "corpus".
2. Generate a `Dockerfile` using the corpus with some kind of genetic algorithm.
3. Invoke `docker build` for the generated `Dockerfile` on multiple Docker Machines (dm1: AUFS, dm2: Overlay, ..).
   This `docker build` will __fail__ after some build steps. But this failure is expected and does not matter.
4. Compare the number of successful build steps on each of the Docker Machines. Test Oracle: If the number of successful build steps varies, there should be some bug.
5. Go to 2.


### Genetic Algorithm

* Chromosome
 * Bit vector for selecting lines in `Dockerfile` corpus set
 * Bit vector for CPUs attached to the build container?

* Static Fitness
 * +: the number of non-`apt` operations appeared in `Dockerfile`
 * -: `apt` operations for CentOS images

* Dynamic Fitness
 * +: the number of successful build steps
 * +: Kernel gcov? (requires `CONFIG_GCOV_KERNEL` and  `CONFIG_GCOV_PROFILE_ALL`)
 * -: time took for build

* Ideas
 * Can we inject random `rm`, `mv`, `cp` for exploration?  


## Related Work (including non-fuzzing ones)
 
 * American fuzzy lop: a brute-force fuzzer coupled with an exceedingly simple but rock-solid instrumentation-guided genetic algorithm ([code](http://lcamtuf.coredump.cx/afl/))
 * Go-fuzz: a coverage-guided fuzzing solution for testing of Go packages ([code](https://github.com/dvyukov/go-fuzz))
 * fsfuzzer: a filesystem fuzzer tool that does stress tests of various filesystems ([code](https://github.com/sughodke/fsfuzzer))
 * Trinity: a Linux system call fuzz tester ([code](http://codemonkey.org.uk/projects/trinity/))
 * syzkaller: a distributed, unsupervised, coverage-guided Linux syscall fuzzer ([code](https://github.com/google/syzkaller))
 * Stress Testing The FreeBSD Kernel ([Denmark LinuxForum'06 slide](https://people.freebsd.org/~pho/linuxforum06/linuxforum06.pdf), [code](https://people.freebsd.org/~pho/stress/))
 * SibylFS: formal specification and oracle-based testing for POSIX and real-world file systems ([SOSP'15 paper](http://unikernel.org/files/2015-sibylfs-sosp.pdf), [code](http://sybilfs.io/))
 * EXPLODE: a Lightweight, General System for Finding Serious Storage System Errors ([OSDI'06 paper](http://usenix.org/legacy/event/osdi06/tech/full_papers/yang_junfeng/yang_junfeng.pdf), [code] (http://sf.net/projects/explode))
 * Fuzzing ([Chaos Communication Congress'05 slide](https://events.ccc.de/congress/2005/fahrplan/attachments/683-slides_fuzzing.pdf))
