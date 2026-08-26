

## [Task1](https://github.com/gruns/icecream/issues/246)

* Testing and making a PR on a repo called [icecream](https://github.com/gruns/icecream), which it's slogan is `never use print to debug again`. It's pretty cute.
* The main problem is this issue https://github.com/gruns/icecream/issues/36, which indicates that `print()` is faster than `ci()` I mean duuh, but anyway, I tried to make it faster by doing a simple thing, you can find the details in my Issue + pull request (check it under). The flame graph was provided before and after, with a noticeable performance.

- **Before:** `1.1946s` (16,741.34 calls/sec)
- **After:** `0.7468s` (26,781.97 calls/sec)
- **Result:** ~37.5% reduction in total execution time (~60% increase in throughput)



## Task2

* I think the N+1 Query problem is very easy to fix, even for noob developers. I have putted a huge effort trying to find any project on github that have a DB and got such a problem, but I couldn't. Frameworks can solve this easily.
## Task3


[MySQL CPU Flame Graph]([https://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html?utm_source=chatgpt.com](https://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html?utm_source=chatgpt.com#:~:text=5%2E1%2E%20MySQL))

![[mysql_flame.png]]

The first example is a production MySQL server with unexpectedly high CPU usage. The raw profiler output initially made `add_to_status()` / `calc_sum_of_all_status()` look hot, but the flame graph showed that **most of the CPU time was actually in `JOIN::exec`**. That redirected the investigation toward the SQL execution path and ultimately helped solve the issue.

**What's hot:** `JOIN::exec` and its surrounding call path.

**Bottleneck:** SQL query execution, not the MySQL status-maintenance code that initially looked suspicious.

**My takeaway:** The flame graph is valuable because it shows the _overall distribution_ of samples. Looking at only the first few profiler stacks can give you the wrong idea about what is actually consuming CPU.


---

[ext4 File-System Archive](https://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html?utm_source=chatgpt.com#FileSystems:~:text=5%2E2%2E%20File%20Systems)


![[ext4_flame.png]]

The second example profiles the Linux kernel while an ext4 filesystem is being archived. The graph shows significant kernel CPU time in `sys_newfstatat()` and `sys_getdents()`, which are related to **filesystem metadata work while walking directories**. It also shows `sys_openat()` for opening files and page faults as file contents are brought into userspace.

**What's hot:** `sys_newfstatat()` and `sys_getdents()`.

**Bottleneck:** Filesystem metadata operations involved in traversing the directory tree.

**My takeaway:** A flame graph can reveal that the expensive part of a file-processing workload isn't necessarily moving the file's data. Here, a lot of kernel CPU time is spent finding and inspecting files and directories.


---


[NUMA Rebalancing](https://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html?utm_source=chatgpt.com#:~:text=5%2E5%2E%20NUMA%20Rebalancing)

![[numa_flame.png]]

The third example is particularly interesting. Netflix encountered unexpectedly high **system CPU usage**, around 60%. The initial flame graph looked messy, with many thin "hairs" caused by interrupts, making the important pattern difficult to see. Reversing the merge/visualization order revealed two large towers accounting for **55% of all samples**.

The underlying problem was **NUMA memory rebalancing going wrong in that Linux kernel version**, consuming more than half of the CPU cycles. The workaround was to disable NUMA rebalancing for that kernel version; the issue was later fixed in newer kernels.

**What's hot:** NUMA rebalancing.

**Bottleneck:** The kernel was spending huge amounts of CPU time trying to rebalance memory between NUMA nodes.

**My takeaway:** Sometimes a flame graph initially looks confusing and the bottleneck isn't obvious. Changing the visualization can expose a dominant pattern that was hidden by many small interrupt stacks.



## Task4

repo: https://github.com/pytest-dev/pytest

On my journey to find a good repo to test `pdb` on it, I told my self why not trying to test pytest itself.

It reproducibly fails on:

- Python **3.13.5**
- pytest **9.2.0.dev184+g68308aa28**
- Linux
* pytest 9.0.3



I investigated a real failing test in pytest using Python's `pdb` debugger. The test `test_fixture_doctest_skip_has_line_number` failed because a `DoctestItem` reported `None` for its source line number. I placed a breakpoint in `DoctestItem.reportinfo()` and inspected the live object state. This showed that `reportinfo()` was not causing the problem: the underlying `DocTest` already had `lineno=None`, while its filename, name, and example source were correct. I then traced the object backward through `DoctestModule.collect()` and confirmed that it came from `MockAwareDocTestFinder.find()`. Inspecting pytest's source revealed version-specific doctest line-number compatibility logic for older Python versions and Python 3.12, but no equivalent special branch for Python 3.13. The investigation therefore narrowed the problem to doctest line-number discovery on Python 3.13. I did not claim a final fix because the standard-library `doctest` implementation had not yet been stepped through to prove exactly where the `None` value originated. The main lesson was to use debugger state to trace an incorrect value backward instead of modifying the line where the failure was first observed.

-->> IN DETAILED INVESTIGATION:


Running the repository's tests produced import errors because the installed pytest did not match the source checkout. Examples included:

```
ImportError: cannot import name '_get_prog_name' from '_pytest.config'
ImportError: cannot import name 'RegisteredMarker' from '_pytest.config'
ModuleNotFoundError: No module named '_pytest.approx'
```


## 1. Running the test suite

The installed `anyio` plugin was incompatible with the development version of pytest. It attempted:

```
from _pytest.python import CallSpec2
```

and pytest reported:

```
PytestRemovedIn10Warning:
_pytest.python.CallSpec2 has been renamed to CallSpec.
```

To isolate pytest itself from external plugins, the test suite was run with:

```
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest -q
```

Result:

```
2 failed, 4370 passed, 48 skipped, 15 xfailed, 5 xpassed, 9 warnings
```

The two failures were:

```
testing/test_doctest.py::TestDoctests::test_fixture_doctest_skip_has_line_number
testing/test_helpconfig.py::test_version_verbose
```

`test_version_verbose` passed when run individually:

```
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest     testing/test_helpconfig.py::test_version_verbose -vv
```

Result:

```
1 passed in 0.48s
```

Therefore, it was not selected as the debugger target.

The doctest failure reproduced independently.


![[failed_test.png]]

---

# 2. The selected bug

The selected test was:

```
testing/test_doctest.py::TestDoctests::test_fixture_doctest_skip_has_line_number
```

It was reproduced with:

```
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest     testing/test_doctest.py::TestDoctests::test_fixture_doctest_skip_has_line_number -vv
```

The failure was:

```
E       assert None is not None
```

at:

```
testing/test_doctest.py:895
```

The test creates a temporary file containing a fixture with a doctest:

```
import pytest

@pytest.fixture
def unavailable():
    '''
    >>> getfixture("unavailable")
    '''
    pytest.skip("unavailable")
```

It then performs:

```
items, _reprec = pytester.inline_genitems(p, "--doctest-modules")
assert items[0].reportinfo()[1] is not None
```

The expected result is that the doctest item's report information contains a source line number. Instead, the value is `None`.

---

# 3. First breakpoint: `DoctestItem.reportinfo()`

The production method was found with:

```
grep -Rni "def reportinfo" src/_pytest/doctest.py
```

The method was:

```
def reportinfo(self) -> tuple[os.PathLike[str] | str, int | None, str]:
    return self.path, self.dtest.lineno, f"[doctest] {self.name}"
```

A temporary breakpoint was added:

```
def reportinfo(self) -> tuple[os.PathLike[str] | str, int | None, str]:
    breakpoint()
    return self.path, self.dtest.lineno, f"[doctest] {self.name}"
```

The test was run with capture disabled:

```
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest -s     testing/test_doctest.py::TestDoctests::test_fixture_doctest_skip_has_line_number -vv
```

At the debugger:

```
(Pdb) p self
<DoctestItem test_fixture_doctest_skip.unavailable>

(Pdb) p self.dtest
<DocTest test_fixture_doctest_skip.unavailable from /tmp/pytest-of-ahmadquraan/pytest-3/test_fixture_doctest_skip_has_line_number0/test_fixture_doctest_skip.py:None (1 example)>

(Pdb) p self.dtest.lineno
None
```

### Important discovery

`reportinfo()` was **not** creating the bad value.

The underlying `DocTest` already had:

```
lineno = None
```

before `reportinfo()` returned it.

---

# 4. Inspecting the `DocTest`

Additional debugger inspection:

```
(Pdb) p self.dtest.filename
'/tmp/pytest-of-ahmadquraan/pytest-3/test_fixture_doctest_skip_has_line_number0/test_fixture_doctest_skip.py'

(Pdb) p self.dtest.name
'test_fixture_doctest_skip.unavailable'

(Pdb) p self.dtest.examples
[<doctest.Example object at ...>]

(Pdb) p self.dtest.examples[0]
<doctest.Example object at ...>

(Pdb) p self.dtest.examples[0].source
'getfixture("unavailable")
'
```

The state was:

|Field|Value|
|---|---|
|`dtest.filename`|Correct temporary file|
|`dtest.name`|Correct doctest name|
|`dtest.examples`|One example|
|Example source|`getfixture("unavailable")`|
|`dtest.lineno`|`**None**`|

This showed that the doctest was being discovered correctly, but its overall line number was missing.

---

# 5. Tracing the value backward

The relevant collection code was:

```
for test in finder.find(module, module.__name__):
    if test.examples:  # skip empty doctests
        yield DoctestItem.from_parent(
            self, name=test.name, runner=runner, dtest=test
        )
```

This means pytest obtains the `DocTest` from:

```
finder.find(module, module.__name__)
```

and then passes it directly to `DoctestItem`.

The investigation therefore moved backward from:

```
DoctestItem.reportinfo()
```

to:

```
DoctestModule.collect()
```

and then to:

```
MockAwareDocTestFinder.find()
```

---

# 7. Inspecting pytest's compatibility code

`src/_pytest/doctest.py` contains a custom:

```
class MockAwareDocTestFinder(doctest.DocTestFinder):
```

It contains version-specific `_find_lineno()` handling.

The code checks:

```
is_find_lineno_broken = (
    py_ver_info_minor < (3, 11)
    or (
        py_ver_info_minor == (3, 11) and sys.version_info.micro < 9
    )
    or (
        py_ver_info_minor == (3, 12) and sys.version_info.micro < 3
    )
)
```

There is also a special Python 3.12 branch:

```
elif py_ver_info_minor == (3, 12):

    def _find_lineno(self, obj, source_lines):
        if isinstance(obj, FixtureFunctionDefinition):
            obj = inspect.unwrap(obj)

        return super()._find_lineno(
            obj,
            source_lines,
        )
```

The investigation environment is Python:

```
3.13.5
```

Therefore neither the older-Python branch nor the Python 3.12 branch applies.

This was a strong clue because the failing test specifically involves a pytest fixture containing a doctest and expects the doctest line number to be available.

---

# 8. Second debugger breakpoint: `DoctestModule.collect()`

A temporary breakpoint was added immediately before doctest discovery:

```
breakpoint()
for test in finder.find(module, module.__name__):
```

The first attempt without `-s` failed because pytest's output capture prevented `pdb` from reading stdin:

```
OSError: pytest: reading from stdin while output is captured!
Consider using `-s`.
```

The debugger was therefore run with:

```
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest -s     testing/test_doctest.py::TestDoctests::test_fixture_doctest_skip_has_line_number -vv
```

At the breakpoint:

```
(Pdb) p sys.version_info
sys.version_info(major=3, minor=13, micro=5, releaselevel='final', serial=0)

(Pdb) p finder
<_pytest.doctest.DoctestModule.collect.<locals>.MockAwareDocTestFinder object at ...>
```

Stepping with `n` showed execution passing through:

```
for test in finder.find(module, module.__name__):
```

and reaching:

```
yield DoctestItem.from_parent(
    self, name=test.name, runner=runner, dtest=test
)
```

This confirmed that `finder.find()` successfully returned the `DocTest`, and the object was then passed into `DoctestItem`.


---
# 10. Conclusion

The investigation proved that:

1. The failure is reproducible.
    
2. It occurs on Python 3.13.5.
    
3. `DoctestItem.reportinfo()` returns `self.dtest.lineno`.
    
4. The `DocTest` already contains `lineno=None`.
    
5. The filename, name, and doctest example are valid.
    
6. The `DocTest` comes from pytest's `MockAwareDocTestFinder.find()`.
    
7. Pytest contains version-specific compatibility code for doctest line-number discovery.
    
8. The current source has special handling for Python 3.11 and 3.12, but not a Python 3.13-specific `_find_lineno()` branch.
    

The strongest hypothesis is that this is related to doctest line-number discovery for pytest fixtures on Python 3.13.

However, the investigation **did not fully prove the ultimate root cause** because the standard-library `doctest.DocTestFinder._find_lineno()` implementation was not stepped through to the exact point where `lineno` becomes `None`.

Therefore, this journal intentionally does **not** claim that a final fix was made.

