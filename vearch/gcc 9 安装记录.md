gcc 9 安装记录

下载gcc 9.3.0 代码 https://github.com/gcc-mirror/gcc/tree/releases/gcc-9.3.0

```
yum install zip
zip gcc-releases-gcc-9.3.0.zip
cd gcc-releases-gcc-9.3.0
./configure
```

报错

```
configure: error: Building GCC requires GMP 4.2+, MPFR 2.4.0+ and MPC 0.8.0+.
Try the --with-gmp, --with-mpfr and/or --with-mpc options to specify
their locations.  Source code for these libraries can be found at
their respective hosting sites as well as at
ftp://gcc.gnu.org/pub/gcc/infrastructure/.  See also
http://gcc.gnu.org/install/prerequisites.html for additional info.  If
you obtained GMP, MPFR and/or MPC from a vendor distribution package,
make sure that you have installed both the libraries and the header
files.  They may be located in separate packages.
```

安装 GMP 4.2+, MPFR 2.4.0+ and MPC 0.8.0+

在ftp://gcc.gnu.org/pub/gcc/infrastructure/下载

```
yum install -y zip bzip2 flex  gcc make gcc++
```

把下面的下载下来放在gcc-releases-gcc-9.3.0文件夹中执行`./contrib/download_prerequisites`, 或者直接执行./contrib/download_prerequisites会自动下载这些包

```
gmp-6.1.0.tar.bz2
isl-0.18.tar.bz2
mpc-1.0.3.tar.gz
mpfr-3.1.4.tar.bz2
```

```
# run
./contrib/download_prerequisites
./configure --disable-multilib --prefix=/home/gcc
make -j8
make install
```

