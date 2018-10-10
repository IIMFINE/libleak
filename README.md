# libleak

`libleak` detects memory leak by hooking memory functions (e.g. `malloc`)
by LD_PRELOAD.

There is no need to modify or re-compile the target program, and you can
enable/disable the detection during target running.

There is very little impact on performance.

In fact `libleak` can not identify memory leak, while it just takes the
memory as leak if it lives longer than a threshold. The threshold is 60
second by default, but you should set it according to your scenarios.


# licence

GPLv2

# OS-machine

- GNU/Linux only by now. But FreeBSD should be OK with some code changing

- x86_64 is only tested by now. But others should be OK.


# build from source

The development package of `libelf` and `libunwind` is need before building.

They can be installed in Debian/Ubuntu by:

    $ sudo apt install libelf-dev libunwind-dev

or in CentOS by:

    $ sudo yum install elfutils-libelf-devel libunwind-devel

Then go:

    $ git clone --recursive https://github.com/WuBingzheng/libleak.git
    $ cd libleak
    $ make


# usage

### basic

0. `libelf` and `libunwind` are need by `libleak`. They can be installed in Debian/Ubuntu by:

        $ sudo apt install libelf libunwind

   or in CentOS by:

        $ sudo yum install elfutils-libelf libunwind

1. [Download](https://github.com/WuBingzheng/libleak/releases) or build the shared-object `libleak.so`.

2. Run the target program:

        $ LD_PRELOAD=/path/of/libleak.so ./a.out

3. Then you will read output in `/tmp/libleak.$pid` in time.

### set expire threshold

As said above, you should set expire threshold according to your scenarios.

For example, if you are debugging an HTTP server with keepalive, and there are
connections last for more than 5 minutes, you should set the threshold to 300
second to cover it.
If your program is expected to free every memory in 1 second, you should set
the threshold to 2 to get report in time.

The threshold is set by environment variable `LEAK_EXPIRE` (in second, default is 60):

    $ LD_PRELOAD=/path/of/libleak.so LEAK_EXPIRE=300 ./a.out

Besides, the threshold will be increased if any memory block is freed after
expiration, with `LEAK_AUTO_EXPIRE` enabled (default is disabled):

    $ LD_PRELOAD=/path/of/libleak.so LEAK_AUTO_EXPIRE=1 ./a.out

### enable/disable detection during running

`libleak` begins the detection at the very beginning of target process running
by default. However you can enable/disable the detection during running by
setting `LEAK_PID_CHECK` and `LEAK_PID_FILE`:

    $ LD_PRELOAD=/path/of/libleak.so LEAK_PID_CHECK=10 ./a.out

`LEAK_PID_CHECK` set the interval (in second, default is 0) to check `LEAK_PID_FILE`.

`LEAK_PID_FILE` (default is `/tmp/libleak.enabled`) contains target pids:
one pid each line, no empty line, no comment line.
You can add or delete pids to/from this file during running.

### for multi-thread program

`libleak` is multi-thread safe.

### for multi-process program

Log file will be created for each process.

Besides you can choose which processes to be detect by `LEAK_PID_FILE`
and `LEAK_PID_CHECK` said above.

### set the log

The log file is set by `LEAK_LOG_FILE` (default is `/tmp/libleak.$pid`).