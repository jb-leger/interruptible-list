A interruptible list
====================

Build a list from a iterable, interruptible by KeyboardInterrupt (SIGINT), and
monitorable by SIGUSR1 and SIGUSR2.

The function `interruptible_list` act as list() constructor, and build a list from
an iterable, except:
  - When a SIGINT (CTRL-C, KeyboardInterrupt) is received, the construction
    is stopped, and the current list is returned.
  - When a SIGUSR1 is received, the last item is pickled in a file or a callback
    is called on the last item, and the construction of the list continues.
  - When a SIGUSR2 is received, the whole current list is pickled in an file or
    a callback is called on the whole current list, and the construction of the
    list continues.

Examples
--------

All examples are illustrated with a slow version of `range(60)`.

```python
import time

def mygen():
    for i in range(60):
        time.sleep(1)
        yield i
```

 - Basic interruptible list:
    ```python
    from interruptible_list import interruptible_list

    L = interruptible_list(mygen())
    ```

 - With callback on `SIGUSR1` to display the last item on `stderr`
    ```python
    import sys
    from interruptible_list import interruptible_list

    def mycallback(x):
        print(x, file=sys.stderr)

    L = interruptible_list(mygen(), callback_last=mycallback)
    ```

 - With callback on `SIGUSR2` to display the mean on the current list on `stderr`:
    ```python
    import sys
    from interruptible_list import interruptible_list

    def mycallback_mean(thelist):
        mean = sum(thelist)/len(thelist)
        print(mean, file=sys.stderr)

    L = interruptible_list(mygen(), callback_whole=mycallback_mean)
    ```

 - With save on `SIGUSR1` and `SIGUSR2`. Last item (`USR1`) or the whole current list
   (`USR2`) is pickled when the signal is received.
    ```python
    from interruptible_list import interruptible_list

    L = interruptible_list(mygen(), save_last=True, save_whole=True)
    ```
