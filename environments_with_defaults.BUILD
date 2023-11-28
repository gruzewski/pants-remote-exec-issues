# Copyright 2020 Pants project contributors.
# Licensed under the Apache License, Version 2.0 (see LICENSE).

# A macro that turns every entry in this directory's requirements.txt into a
# `python_requirement_library` target. Refer to
# https://www.pantsbuild.org/docs/python-third-party-dependencies.

__defaults__(all=dict(environment="remote"))

remote_environment(
    name="remote",
    description="Remote execution farm in Docker Compose",
    platform="linux_x86_64",
    extra_platform_properties=[
        "OSFamily=Linux",
        "container-image=docker://python:3.9"
    ],
    fallback_environment="local",
    python_bootstrap_search_path=[
        "<PATH>",
    ],
)

local_environment(
    name="local",
    description="Local environment",
    compatible_platforms=[
        "linux_x86_64",
    ],
)