---
title: 动手安装 Haskell koans 笔记
date: 2016-08-15 15:24:30
tags:
  - koan
  - haskell
---

## 下载源代码
## 编译安装

```
cabal update
cabal build Setup.hs -v
```
### 报错了
```
Package has never been configured. Configuring with default flags. If this
fails, please run configure manually.
/usr/local/bin/ghc --numeric-version
looking for tool ghc-pkg near compiler in /usr/local/bin
found ghc-pkg in /usr/local/bin/ghc-pkg
/usr/local/bin/ghc-pkg --version
/usr/local/bin/ghc --supported-languages
/usr/local/bin/ghc --info
Reading available packages...
Choosing modular solver.
Resolving dependencies...
Could not resolve dependencies:
trying: HaskellKoans-0.1 (user goal)
next goal: testloop (dependency of HaskellKoans-0.1)
Dependency tree exhaustively searched.
Configuring HaskellKoans-0.1...
cabal: At least the following dependencies are missing:
HUnit -any,
attoparsec -any,
hspec -any,
mtl -any,
testloop -any,
text -any
```
### 缺少依赖，接着安装依赖
```
  cabal install HUnit
Resolving dependencies...
Configuring HUnit-1.3.1.1...
Failed to install HUnit-1.3.1.1
Build log ( /root/.cabal/logs/HUnit-1.3.1.1.log ):
cabal: Error: some packages failed to install:
HUnit-1.3.1.1 failed during the configure step. The exception was:
user error ('/usr/local/bin/ghc' exited with an error:
/usr/bin/ld: cannot find -lgmp
collect2: ld returned 1 exit status
)
```
### 根据提示，缺少libgmp.so
```
gcc -lgmp --verbose
Using built-in specs.
Target: x86_64-redhat-linux
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-languages=c,c++,objc,obj-c++,java,fortran,ada --enable-java-awt=gtk --disable-dssi --with-java-home=/usr/lib/jvm/java-1.5.0-gcj-1.5.0.0/jre --enable-libgcj-multifile --enable-java-maintainer-mode --with-ecj-jar=/usr/share/java/eclipse-ecj.jar --disable-libjava-multilib --with-ppl --with-cloog --with-tune=generic --with-arch_32=i686 --build=x86_64-redhat-linux
Thread model: posix
gcc version 4.4.7 20120313 (Red Hat 4.4.7-17) (GCC) 
COMPILER_PATH=/usr/libexec/gcc/x86_64-redhat-linux/4.4.7/:/usr/libexec/gcc/x86_64-redhat-linux/4.4.7/:/usr/libexec/gcc/x86_64-redhat-linux/:/usr/lib/gcc/x86_64-redhat-linux/4.4.7/:/usr/lib/gcc/x86_64-redhat-linux/:/usr/libexec/gcc/x86_64-redhat-linux/4.4.7/:/usr/libexec/gcc/x86_64-redhat-linux/:/usr/lib/gcc/x86_64-redhat-linux/4.4.7/:/usr/lib/gcc/x86_64-redhat-linux/
LIBRARY_PATH=/usr/lib/gcc/x86_64-redhat-linux/4.4.7/:/usr/lib/gcc/x86_64-redhat-linux/4.4.7/:/usr/lib/gcc/x86_64-redhat-linux/4.4.7/../../../../lib64/:/lib/../lib64/:/usr/lib/../lib64/:/usr/lib/gcc/x86_64-redhat-linux/4.4.7/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-v' '-mtune=generic'
 /usr/libexec/gcc/x86_64-redhat-linux/4.4.7/collect2 --eh-frame-hdr --build-id -m elf_x86_64 --hash-style=gnu -dynamic-linker /lib64/ld-linux-x86-64.so.2 /usr/lib/gcc/x86_64-redhat-linux/4.4.7/../../../../lib64/crt1.o /usr/lib/gcc/x86_64-redhat-linux/4.4.7/../../../../lib64/crti.o /usr/lib/gcc/x86_64-redhat-linux/4.4.7/crtbegin.o -L/usr/lib/gcc/x86_64-redhat-linux/4.4.7 -L/usr/lib/gcc/x86_64-redhat-linux/4.4.7 -L/usr/lib/gcc/x86_64-redhat-linux/4.4.7/../../../../lib64 -L/lib/../lib64 -L/usr/lib/../lib64 -L/usr/lib/gcc/x86_64-redhat-linux/4.4.7/../../.. -lgmp -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/x86_64-redhat-linux/4.4.7/crtend.o /usr/lib/gcc/x86_64-redhat-linux/4.4.7/../../../../lib64/crtn.o
/usr/bin/ld: cannot find -lgmp
collect2: ld returned 1 exit status
```
### 试着安装一下
```
yum -y install gmp
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 ...
base                                                                                                                  | 3.7 kB     00:00     
extras                                                                                                                | 3.4 kB     00:00     
nginx                                                                                                                 | 2.9 kB     00:00     
updates                                                                                                               | 3.4 kB     00:00     
Package gmp-4.3.1-10.el6.x86_64 already installed and latest version
Nothing to do
```
### 貌似已经安装了
```
# find / -name libgmp.so*
/opt/gitlab/embedded/lib/engines/libgmp.so
/usr/lib64/libgmp.so.3.5.0
/usr/lib64/libgmp.so.3
/usr/lib64/openssl/engines/libgmp.so
/usr/lib64/libgmp.so.10
```
### 建立一个符号链接试试
```
ln -sv /usr/lib64/libgmp.so.3.5.0 /usr/lib64/libgmp.so
`/usr/lib64/libgmp.so' -> `/usr/lib64/libgmp.so.3.5.0'
```
### 在看看结果
```
 ls -al /usr/lib64/libgmp*
lrwxrwxrwx. 1 root root     26 Aug 15 07:18 /usr/lib64/libgmp.so -> /usr/lib64/libgmp.so.3.5.0
lrwxrwxrwx. 1 root root     11 Jul 26 05:07 /usr/lib64/libgmp.so.10 -> libgmp.so.3
lrwxrwxrwx. 1 root root     15 Jun 21 01:58 /usr/lib64/libgmp.so.3 -> libgmp.so.3.5.0
-rwxr-xr-x. 1 root root 376792 Nov 18  2015 /usr/lib64/libgmp.so.3.5.0
lrwxrwxrwx. 1 root root     17 Jun 21 01:58 /usr/lib64/libgmpxx.so.4 -> libgmpxx.so.4.1.0
-rwxr-xr-x. 1 root root  17992 Nov 18  2015 /usr/lib64/libgmpxx.so.4.1.0
```
### ldconfig 
### 接着安装,竟然可以

```
cabal install HUnit
Resolving dependencies...
Configuring HUnit-1.3.1.1...
Building HUnit-1.3.1.1...
Installed HUnit-1.3.1.1
```
### 然后 testloop 编译不过去了
```
cabal install testloop
Resolving dependencies...
Configuring testloop-0.1.1.0...
Building testloop-0.1.1.0...
Failed to install testloop-0.1.1.0
Build log ( /root/.cabal/logs/testloop-0.1.1.0.log ):
Configuring testloop-0.1.1.0...
Building testloop-0.1.1.0...
Preprocessing library testloop-0.1.1.0...
[1 of 6] Compiling System.TestLoop.Internal.Signal ( src/System/TestLoop/Internal/Signal.hs, dist/build/System/TestLoop/Internal/Signal.o )
[2 of 6] Compiling System.TestLoop.Util ( src/System/TestLoop/Util.hs, dist/build/System/TestLoop/Util.o )
[3 of 6] Compiling System.TestLoop.Internal.Types ( src/System/TestLoop/Internal/Types.hs, dist/build/System/TestLoop/Internal/Types.o )
[4 of 6] Compiling System.TestLoop.Internal.Watcher ( src/System/TestLoop/Internal/Watcher.hs, dist/build/System/TestLoop/Internal/Watcher.o )
[5 of 6] Compiling System.TestLoop.Internal.Cabal ( src/System/TestLoop/Internal/Cabal.hs, dist/build/System/TestLoop/Internal/Cabal.o )
[6 of 6] Compiling System.TestLoop  ( src/System/TestLoop.hs, dist/build/System/TestLoop.o )

src/System/TestLoop.hs:47:23:
    Couldn't match type `FS.FilePath' with `[Char]'
    Expected type: FilePath
      Actual type: FS.FilePath
    In the second argument of `treeExtExists', namely
      `(FS.decodeString path)'
    In a stmt of a 'do' block:
      treeExtExists
        manager
        (FS.decodeString path)
        "hs"
        (reloadTestSuite moduleName modulePath paths)

src/System/TestLoop.hs:49:23:
    Couldn't match type `[Char]' with `FS.FilePath'
    Expected type: FilePath -> IO ()
      Actual type: FS.FilePath -> IO ()
    In the fourth argument of `treeExtExists', namely
      `(reloadTestSuite moduleName modulePath paths)'
    In a stmt of a 'do' block:
      treeExtExists
        manager
        (FS.decodeString path)
        "hs"
        (reloadTestSuite moduleName modulePath paths)
cabal: Error: some packages failed to install:
testloop-0.1.1.0 failed during the building phase. The exception was:
ExitFailure 1
``` 