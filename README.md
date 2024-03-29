# Threads reuse via `LD_PRELOAD` hook

Save the "exited" threads in a "thread pool" and reuse them when new threads is needed, instead of creating and destroying threads again and again.

At the beginning, it was wriiten as a workaround for [the memory leak of qemu-user when threads are created and destroyed](https://gitlab.com/qemu-project/qemu/-/issues/866).

Archived, because the bug has been fixed and merged to qemu 8.0.

**DO NOT USE IN PRODUCTION ENVIRNOMENT**

## Build

```bash
mkdir build
cd build
cmake ..
make
```

And then there should be a `./libthread_reuse.so`.

## Use

```bash
LD_PRELOAD=path-to-libthread_reuse.so command
```

or if in qemu-user
``` bash
qemu-ARCH -E LD_PRELOAD=path-to-libthread_reuse.so program
```

## How can it be implemented

It hooks some functions of pthread:

- `pthread_create`: (origin) create a new thread; (hooked) if there isn't enough threads in the thread pool, then create a new one, else get a thread from pool and submit the task
- `pthread_join`: (origin) join the thread, after that the thread will be destroyed; (hooked) wait until the thread finish its task, and get the return value of the task function, and tell the real thread that it can come back to the pool
- `pthread_detach`: (origin) ...; (hooked) tell the thread that it can come back to the pool
- `pthread_exit`: (origin) exit the thread calling this function; (hooked) jump out of the task function via a long jump (exactly `siglongjmp`)
- `pthread_kill`: (origin) send a signal to the thread if the signal is not `0`, or return whether the thread is terminated if the signal is `0`; (hooked) send a signal if the signal is not `0`, or return whether the task function is terminated if the signal is `0`

## TODO

- [ ] thread cancellation support
- [ ] handle more signals
- [ ] more comment and document
- [ ] code clean-up and better style
- [ ] more configurable
- [ ] more tests
- [ ] test suite
- [ ] better POSIX compatibility

