# Reproduce of issues with Pants and remote execution

This is an attempt to reproduce issues that I had with Pants remote execution feature
when upgrading from version `2.15.2` to `2.17.1`. Each Docker Compose file should show
different paths taken to overcome errors. I tried also `2.19.0.dev4` without success.

I believe there are multiple issues:
 - Global remote execution mode is broken since `2.16.0`
 - Env is not added properly to remote exec processes since `2.15.x`
 - Remote execution is not turned on when used with the Global remote exec switch and Environments (without explicitly specifying `environment` parameter on all targets) from `2.16.0`
 - `download_python_binary` is broken in `2.17.x` with a destination directory not being created

For remote storage and execution, [BuildBarn](https://github.com/buildbarn) is being
used. Most of the Docker Compose setup was taken from
[the official repository](https://github.com/buildbarn/bb-deployments), whereas Python
example code was borrowed from [Pants Python example repository](https://github.com/pantsbuild/example-python).

I would recommend running steps from [Cleanup](#cleanup) after testing each case.

## Prerequisites

You need just Docker Compose `1.27.0+` version.

## Pants 2.15.2 with global remote execution

```shell
docker compose -f docker-compose.yaml -f docker-compose.2_15_2_global.yaml up
```

This is the first version that shows issues with remote execution. It fails with below error:

```console
 Engine traceback:
   in select
     ..
   in pants.core.goals.test.run_tests
     `test` goal

 Traceback (most recent call last):
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/core/goals/test.py", line 874, in run_tests
     results = await MultiGet(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 361, in MultiGet
     return await _MultiGet(tuple(__arg0))
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 168, in __await__
     result = yield self.gets
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/goals/pytest_runner.py", line 468, in run_python_tests
     setup = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/goals/pytest_runner.py", line 468, in run_python_tests
     setup = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/core/goals/test.py", line 1027, in get_filtered_environment
     await Get(EnvironmentVars, EnvironmentVarsRequest(test_env_aware.extra_env_vars))
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/platform_rules.py", line 74, in complete_environment_vars
     env_process_result = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
 native_engine.IntrinsicError: For action Digest { hash: Fingerprint<5889c3f98ff4c26e39f8a4cf3cf159bb014a2ab78d6a1d88a51e68500c2840f7>, size_bytes: 139 }: Error from remote execution: InvalidArgument: "Failed to run command: Cannot find executable \"env\" in search paths \"\""
```

It is related to [pants/issues/17262](https://github.com/pantsbuild/pants/issues/17262). On my investigation,
the potential culprit could be [PR #16876](https://github.com/pantsbuild/pants/pull/16876/files#diff-d3d3d6b9c05606ad1357b2202cc2bc77e9efbf4a4a4d7c92f7cea24d98c963a3).
The problem with mentioned changed is [the part below](https://github.com/pantsbuild/pants/blob/main/src/python/pants/engine/internals/platform_rules.py#L74-L82):

```python
    env_process_result = await Get(
        ProcessResult,
        Process(
            ["env", "-0"],
            description=f"Extract environment variables from {description_of_env_source}",
            level=LogLevel.DEBUG,
            cache_scope=env_tgt.executable_search_path_cache_scope(),
        ),
    )
```

According to the `Process` docstring, it is hermetic and doesn't have access `PATH`` env variable.

> This process will be hermetic, meaning that it cannot access files and environment variables
> that are not explicitly populated. For example, $PATH will not be defined by default, unless
> populated through the `env` parameter.

Instead, it should have something similar to:

```python
    search_path = os.pathsep.join(request.search_path)
    result = await Get(
        ProcessResult,
        Process(
            ["env", "-0"],
            description=f"Extract environment variables from {description_of_env_source}",
            level=LogLevel.DEBUG,
            env={"PATH": search_path},
            cache_scope=env_target.executable_search_path_cache_scope(),
        ),
    )
```

This can be workaround by always adding `PATH`` to environment variables in remote execution's runner.
See `config/worker_with_env_vars.jsonnet` for the configuration example.

```jsonnet
      environment_variables: {
        PATH: '/usr/bin:/bin:/usr/local/bin:/opt/homebrew/bin'
      }
```

See [Pants 2.15.2 with global remote execution and hack](#pants-2152-with-global-remote-execution-and-hack).

## Pants 2.15.2 with global remote execution and hack

```shell
docker compose -f docker-compose.yaml -f docker-compose.2_15_2_global_with_hack.yaml up
```

By explicitly adding `PATH` to workers, I managed to get it working. However it fails with error below 
(I am still trying to find a fix for that, it must be related to me setup of MacOS -> Linux emulation), it
is at least executed remotely.

```console
 Traceback (most recent call last):
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.15.2/lib/python3.9/site-packages/pants/engine/process.py", line 289, in fallible_to_exec_result_or_raise
     raise ProcessExecutionFailure(
 pants.engine.process.ProcessExecutionFailure: Process 'Extracting 1 requirement to build requirements.pex from python-default_lockfile.pex: pytest>=7.1.3' failed with exit code 1.
 stdout:

 stderr:
 Failed to resolve requirements from PEX environment @ /worker/build/2cda3899aa1bbdd1/root/.cache/pex_root/unzipped_pexes/6304e9daad96bc47ab0c1433c0a1a806a48b8f90.
 Needed cp39-cp39-manylinux_2_36_x86_64 compatible dependencies for:
  1: iniconfig
     Required by:
       pytest 7.4.3
     But this pex had no ProjectName(raw='iniconfig', normalized='iniconfig') distributions.
  2: packaging
     Required by:
       pytest 7.4.3
     But this pex had no ProjectName(raw='packaging', normalized='packaging') distributions.
  3: pluggy<2.0,>=0.12
     Required by:
       pytest 7.4.3
     But this pex had no ProjectName(raw='pluggy', normalized='pluggy') distributions.
  4: exceptiongroup>=1.0.0rc8; python_version < "3.11"
     Required by:
       pytest 7.4.3
     But this pex had no ProjectName(raw='exceptiongroup', normalized='exceptiongroup') distributions.
  5: tomli>=1.0.0; python_version < "3.11"
     Required by:
       pytest 7.4.3
     But this pex had no ProjectName(raw='tomli', normalized='tomli') distributions.



 Use `--keep-sandboxes=on_failure` to preserve the process chroot for inspection.
```

Most of the following examples will build on top of that fix/hack.

## Pants 2.15.2 with environments and hack

```shell
docker compose -f docker-compose.yaml -f docker-compose.2_15_2_environments.yaml up
```

Similar to [Pants 2.15.2 with global remote execution and hack](#pants-2152-with-global-remote-execution-and-hack).
This is only to demonstrate that one don't need to use `defaults()` to enable remote execution.

## Pants 2.16.0 with environments and hack

```shell
docker compose -f docker-compose.yaml -f docker-compose.2_16_0_environments.yaml up
```

In this version and configuration, remote execution doesn't run and everything is executed locally. Adding
below in the root `BUILD` file, fixes the issue.

```BUILD
__defaults__(all=dict(environment="remote"))
```

To prove it, run below command.

```shell
docker compose -f docker-compose.yaml -f docker-compose.2_16_0_environments_with_defaults.yaml up
```

## Pants 2.17.1 with environments and defaults to remote environment and hack

```shell
docker compose -f docker-compose.yaml -f docker-compose.2_17_1_environments_with_defaults.yaml up
```

Although all previously suggested fixes are applied, I got a new error..

```console
 Engine traceback:
   in select
     ..
   in pants.core.goals.test.run_tests
     `test` goal

 Traceback (most recent call last):
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/core/goals/test.py", line 874, in run_tests
     results = await MultiGet(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 361, in MultiGet
     return await _MultiGet(tuple(__arg0))
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 168, in __await__
     result = yield self.gets
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/goals/pytest_runner.py", line 468, in run_python_tests
     setup = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/util_rules/pex.py", line 800, in create_pex
     result = await Get(BuildPexResult, PexRequest, request)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/goals/pytest_runner.py", line 256, in setup_pytest_for_target
     ) = await MultiGet(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 452, in MultiGet
     return await _MultiGet((__arg0, __arg1, __arg2, __arg3, __arg4, __arg5))
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 168, in __await__
     result = yield self.gets
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/util_rules/pex.py", line 800, in create_pex
     result = await Get(BuildPexResult, PexRequest, request)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/util_rules/pex.py", line 667, in build_pex
     pex_python_setup = await _determine_pex_python_and_platforms(request)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/util_rules/pex.py", line 447, in _determine_pex_python_and_platforms
     python = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/util_rules/pex.py", line 667, in build_pex
     pex_python_setup = await _determine_pex_python_and_platforms(request)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/util_rules/pex.py", line 447, in _determine_pex_python_and_platforms
     python = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/util_rules/pex.py", line 359, in find_interpreter
     result = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/core/util_rules/adhoc_binaries.py", line 56, in get_python_for_scripts
     result = await Get(_PythonBuildStandaloneBinary, _DownloadPythonBuildStandaloneBinaryRequest())
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/core/util_rules/adhoc_binaries.py", line 111, in download_python_binary
     await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
 pants.engine.process.ProcessExecutionFailure: Process 'Install Python for Pants usage' failed with exit code 1.
 stdout:

 stderr:
 cp: cannot create directory '.python-build-standalone/c12164f0e9228ec20704c1aba97eb31b8e2a482d41943d541cc8e3a9e84f7349': No such file or directory
 touch: cannot touch '.python-build-standalone/c12164f0e9228ec20704c1aba97eb31b8e2a482d41943d541cc8e3a9e84f7349/DONE': No such file or directory
```

I captured request created by Pants:

```console
action=Action
{
  command_digest: Some(
    Digest { hash: "1e393ddd736c4f9428f70ab1116c16b0e9e56bcfea621fb6dd592b2cb9a60251", size_bytes: 320 }
  ),
  input_root_digest: Some(
    Digest { hash: "72524d3b899ee1e0cb2ccb8dd338d2361d2d07acddbb3898df87d7af931ed133", size_bytes: 148 }
  ),
  timeout: None,
  do_not_cache: false,
  salt: b"",
  platform: None 
}; 

command=Command
{
  arguments: [
    "/usr/bin/tar",
    "-xvf",
    "cpython-3.9.16+20230116-x86_64-unknown-linux-gnu-install_only.tar.gz"
  ],
  environment_variables: [
    EnvironmentVariable { name: "PANTS_CACHE_KEY_EXECUTION_STRATEGY", value: "remote_execution" },
    EnvironmentVariable { name: "PANTS_CACHE_KEY_TARGET_PLATFORM", value: "linux_x86_64" },
    EnvironmentVariable { name: "PATH", value: "/usr/bin:/bin:/usr/local/bin:/opt/homebrew/bin" }
  ],
  output_files: [],
  output_directories: ["python"],
  output_paths: [],
  platform: Some(
    Platform { 
      properties: [
        Property { name: "OSFamily", value: "Linux" },
        Property { name: "container-image", value: "docker://python:3.9" }
      ]
    }
  ),
  working_directory: "",
  output_node_properties: []
};

execute_request=ExecuteRequest
{
  instance_name: "simple-runner",
  skip_cache_lookup: true,
  action_digest: Some(
    Digest { hash: "5d36f8998349b6f2aef3224e99f9c88f2937ba4b598b17a514318dcf9d9333b6", size_bytes: 142 }
  ),
  execution_policy: None,
  results_cache_policy: None
}
```

```console
action=Action
{ 
  command_digest: Some(
    Digest { hash: "697378a4b114f775f6561e4985ca01499daae01cae2fd36762d0878ebbd1f0ac", size_bytes: 640 }
  ),
  input_root_digest: Some(
    Digest { hash: "c12164f0e9228ec20704c1aba97eb31b8e2a482d41943d541cc8e3a9e84f7349", size_bytes: 81 }
  ),
  timeout: None,
  do_not_cache: false,
  salt: b"",
  platform: None 
};

command=Command
{ 
  arguments: [
    "/usr/bin/bash",
    "-c",
    "if [ ! -f \".python-build-standalone/c12164f0e9228ec20704c1aba97eb31b8e2a482d41943d541cc8e3a9e84f7349/DONE\" ]; then\n    cp -r python \".python-build-standalone/c12164f0e9228ec20704c1aba97eb31b8e2a482d41943d541cc8e3a9e84f7349\"\n    touch \".python-build-standalone/c12164f0e9228ec20704c1aba97eb31b8e2a482d41943d541cc8e3a9e84f7349/DONE\"\nfi\n"
  ],
  environment_variables: [
    EnvironmentVariable { name: "PANTS_CACHE_KEY_EXECUTION_STRATEGY", value: "remote_execution" },
    EnvironmentVariable { name: "PANTS_CACHE_KEY_SALT", value: "6e20423c-345f-440b-9538-bd13cc589234" },
    EnvironmentVariable { name: "PANTS_CACHE_KEY_TARGET_PLATFORM", value: "linux_x86_64" },
    EnvironmentVariable { name: "PATH", value: "/usr/bin:/bin:/usr/local/bin:/opt/homebrew/bin" }
  ],
  output_files: [],
  output_directories: [],
  output_paths: [],
  platform: Some(
    Platform {
      properties: [
        Property { name: "OSFamily", value: "Linux" },
        Property { name: "container-image", value: "docker://python:3.9" }
      ]
    }
  ),
  working_directory: "",
  output_node_properties: []
}; 

execute_request=ExecuteRequest
{
  instance_name: "simple-runner",
  skip_cache_lookup: true,
  action_digest: Some(
    Digest { hash: "08d246f5c7bf24e5a52726e1d998c9919c9d9fca90d00825371df63e3d7ca760", size_bytes: 141 }
  ),
  execution_policy: None,
  results_cache_policy: None
}
```

I believe the issue is around [`installation_root` assignment](https://github.com/pantsbuild/pants/blob/release_2.17.1/src/python/pants/core/util_rules/adhoc_binaries.py#L97) and might be related to [PR #19611](https://github.com/pantsbuild/pants/pull/19611).

```python
    installation_root = f"{PythonBuildStandaloneBinary._SYMLINK_DIRNAME}/{download_result.output_digest.fingerprint}"

    # NB: This is similar to what we do for every Python provider. We should refactor these into
    # some shared code to centralize the behavior.
    installation_script = dedent(
        f"""\
        if [ ! -f "{installation_root}/DONE" ]; then
            cp -r python "{installation_root}"
            touch "{installation_root}/DONE"
        fi
    """
    )
```

I don't see where that directory is being created so there is chance that it should be created first.

In `2.16.0`, [the same but working code](https://github.com/pantsbuild/pants/blob/release_2.16.0/src/python/pants/core/util_rules/adhoc_binaries.py#L86-L100)
looked like this:

```python
    result = await Get(
        ProcessResult,
        Process(
            argv=[tar_binary.path, "-xvf", filename],
            input_digest=python_archive,
            env={"PATH": os.pathsep.join(SEARCH_PATHS)},
            description="Extract Python",
            level=LogLevel.DEBUG,
            output_directories=("python",),
        ),
    )

    return _PythonBuildStandaloneBinary(
        f"{PythonBuildStandaloneBinary.SYMLINK_DIRNAME}/python/bin/python3", result.output_digest
    )
```

It provides directory to the untarred path.

## Pants 2.17.1 with environments and defaults to remote environment and subprocess-envs

```shell
docker compose -f docker-compose.yaml -f docker-compose.2_17_1_environments_with_subprocess_envs.yaml up
```

This is an attempt, suggested on Pants' Slack, to remove BuildBarn's `PATH` hack. Unfortunately, it
fails the same way.

```console
 21:39:08.82 [DEBUG] Completed: run_execute_request
 21:39:08.82 [DEBUG] Completed: Scheduling: Extract environment variables from the remote execution environment
 21:39:08.82 [DEBUG] Completed: pants.backend.python.goals.pytest_runner.setup_pytest_for_target
 21:39:08.82 [DEBUG] Completed: pants.backend.python.goals.pytest_runner.setup_pytest_for_target
 21:39:08.82 [DEBUG] Completed: Run Pytest - (environment:remote, helloworld/greet/greeting_test.py:tests)
 21:39:08.83 [DEBUG] Completed: Run Pytest - (environment:remote, helloworld/translator/translator_test.py:tests)
 21:39:08.83 [DEBUG] Completed: `test` goal
 21:39:08.83 [DEBUG] computed 1 nodes in 0.534182 seconds. there are 214 total nodes.
 21:39:08.83 [ERROR] 1 Exception encountered:

 Engine traceback:
   in select
     ..
   in pants.core.goals.test.run_tests
     `test` goal

 Traceback (most recent call last):
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/core/goals/test.py", line 874, in run_tests
     results = await MultiGet(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 361, in MultiGet
     return await _MultiGet(tuple(__arg0))
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 168, in __await__
     result = yield self.gets
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/goals/pytest_runner.py", line 468, in run_python_tests
     setup = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/backend/python/goals/pytest_runner.py", line 468, in run_python_tests
     setup = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/core/goals/test.py", line 1027, in get_filtered_environment
     await Get(EnvironmentVars, EnvironmentVarsRequest(test_env_aware.extra_env_vars))
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 626, in native_engine_generator_send
     res = rule.send(arg) if err is None else rule.throw(throw or err)
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/platform_rules.py", line 74, in complete_environment_vars
     env_process_result = await Get(
   File "/root/.cache/nce/3d6643e46b53e4cc0b2a0d5c768866226ddce3de1f57f80c4a02d8d39800fa8e/bindings/venvs/2.17.1/lib/python3.9/site-packages/pants/engine/internals/selectors.py", line 118, in __await__
     result = yield self
 native_engine.IntrinsicError: For action Digest { hash: Fingerprint<b1f34b2bd7dc6bfab59b33fe0729a8d2b1b25c7e7f2bd1cb087f2f187c910c48>, size_bytes: 139 }: Error from remote execution: InvalidArgument: "Failed to run command: Cannot find executable \"env\" in search paths \"\""
```

## Cleanup

Run the cleanup step to remove all created Docker volumes.

```shell
docker-compose down -v
```