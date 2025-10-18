# Including pony-odbc

Let's go ahead and create a new pony project.

```shell
red@panic:~/projects$ mkdir psql-demo
red@panic:~/projects$ cd psql-demo
red@panic:~/projects/psql-demo$ corral init
red@panic:~/projects/psql-demo$ corral add github.com/redvers/pony-odbc.git --version 0.3.0
red@panic:~/projects/psql-demo$ corral fetch
git cloning github.com/redvers/pony-odbc.git into /home/red/projects/psql-demo/_repos/github_com_redvers_pony_odbc_git
git checking out @0.3.0 into /home/red/projects/psql-demo/_corral/github_com_redvers_pony_odbc
red@panic:~/projects/psql-demo$
```

Let's create a very minimal Makefile

```make
all:
  corral run -- ponyc -d
  ./psql-demo
```

... and our initial main.pony

```pony
use "pony-odbc"
use "lib:odbc" // For unixODBC. For iODBC, use "lib:iodbc"

actor Main
  let env: Env

  new create(env': Env) =>
    env = env'
```

Now go ahead and run make, and run ldd to double-check that the library linked correctly:

```shell
red@panic:~/projects/psql-demo$ make
corral run -- ponyc -d
  exit: Exited(0)
  out:
  err: Building builtin -> /home/red/.local/share/ponyup/ponyc-release-0.59.0-x86_64-linux-ubuntu24.04/packages/builtin
Building . -> /home/red/projects/psql-demo
Building pony-odbc -> /home/red/projects/psql-demo/_corral/github_com_redvers_pony_odbc/pony-odbc
Building debug -> /home/red/.local/share/ponyup/ponyc-release-0.59.0-x86_64-linux-ubuntu24.04/packages/debug
Building ffi -> /home/red/projects/psql-demo/_corral/github_com_redvers_pony_odbc/pony-odbc/ffi
Building collections -> /home/red/.local/share/ponyup/ponyc-release-0.59.0-x86_64-linux-ubuntu24.04/packages/collections
Building pony_test -> /home/red/.local/share/ponyup/ponyc-release-0.59.0-x86_64-linux-ubuntu24.04/packages/pony_test
Building time -> /home/red/.local/share/ponyup/ponyc-release-0.59.0-x86_64-linux-ubuntu24.04/packages/time
Building random -> /home/red/.local/share/ponyup/ponyc-release-0.59.0-x86_64-linux-ubuntu24.04/packages/random
Generating
 Reachability
 Selector painting
 Data prototypes
 Data types
 Function prototypes
 Functions
 Descriptors
Verifying
Writing ./psql-demo.o
Linking ./psql-demo

./psql-demo
red@panic:~/projects/psql-demo$ ldd ./psql-demo
        linux-vdso.so.1 (0x00007e36767fb000)
        libodbc.so.2 => /lib/x86_64-linux-gnu/libodbc.so.2 (0x00007e3676748000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007e367665f000)
        libatomic.so.1 => /lib/x86_64-linux-gnu/libatomic.so.1 (0x00007e3676654000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007e3676626000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007e3676400000)
        /lib64/ld-linux-x86-64.so.2 (0x00007e36767fd000)
        libltdl.so.7 => /lib/x86_64-linux-gnu/libltdl.so.7 (0x00007e3676619000)
red@panic:~/projects/psql-demo$
```

Note that libodbc.so.2 is linked into our executable.
