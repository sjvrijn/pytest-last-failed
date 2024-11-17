# pytest-last-failed
A GitHub Action to enable running pytest --last-failed in your CI

## Introduction
This composite GitHub Action makes use of GitHub's [Cache action] to enable
pytest's `--last-failed` functionality for your CI workflow to speed up testing
in PRs, _without_ compromising on having the full test suite running and passing
before a merge.

## Usage
This action requires that Python is already setup and that `pytest` is already
installed in the environment.

Then, if your workflow currently contains a `pytest` step as follows:
```
  - name: pytest
    run: pytest --my --pytest --arguments
```

you can replace it with:
```
  - name: pytest
    uses: sjvrijn/pytest-last-failed@v3
    with:
      # Additional arguments to be passed to pytest
      pytest-args: '--my --pytest --arguments'

      # Whether to perform a full run of the test-suite once any previously failed
      # tests pass again. This may catch any unintended side effects of fixing the
      # previously failed tests, at the cost of running the entire suite again.
      # Default: false
      run-all-on-success: ''
```

That's all! Note that this means you don't have to add the `--last-failed`
argument yourself, that's taken care of by using this action.

## Explanation
For any first run, it will always run the entire test suite. On subsequent runs,
if the pytest cache indicates any last-failed tests, it will run `pytest
--last-failed` instead.

If that run succeded, i.e., all previously failed tests now pass again, the
`run-all-on-success` flag can be set to perform another run of the whole test
suite to check for any unintended side effects of the implemented fixes.

In pseudo-code:
```python
if not cache.has_last_failed_entries():
    pytest --cache-clear $PYTEST-ARGS
else:
    result = $( pytest --last-failed $PYTEST-ARGS )

    if result == 'success' and $RUN-ALL-ON-SUCCESS:
        pytest --cache-clear $PYTEST-ARGS
```


[Cache action]: https://github.com/actions/cache
