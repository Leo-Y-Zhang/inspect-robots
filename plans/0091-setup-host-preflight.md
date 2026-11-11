# 0091: Run the host preflight after setup

Part of #50. Builds on the read-only probes in plan 0090.

## Design

Extract doctor's version/display/policy aggregation into a private shared
helper in `_doctor_probes.py`. Both commands use that helper and the existing
device probe; doctor retains its device-check phase before construction and
its existing error exit status. Setup never invokes doctor's conformance
phase because that constructs an embodiment.

After saving the config and printing missing runtime dependencies, setup
reloads the selected file and probes its saved policy, embodiment and args.
Component environment overrides do not replace the names setup just wrote.
Display checks still receive the wizard's environment, and device checks
receive its injectable sysfs root. `--config` and `INSPECT_ROBOTS_CONFIG`
continue to select the file through the existing CLI/config path handling.

Print findings with their original severity, code, message and remediation
command in one yellow block before `Next:`. Print nothing when clean, matching
the runtime-dependency block. Keep exit code 0 after a successful write,
including for error findings or unexpected preflight exceptions: setup has
completed its configuration job, and its existing dependency warnings already
follow this rule. Doctor remains the command for a diagnostic exit status.
Unexpected check failures give a command to rerun doctor on the saved file.

The probes perform metadata/sysfs/path reads and short-timeout TCP connects.
They never construct components, open cameras or CAN interfaces, send policy
requests, reset, step, or energise hardware. Setup's interactive-only guard
and existing prompt flow remain unchanged.

## Validation

Extend `tests/test_setup.py` with the same host-inventory and TCP-connector
stubs as the doctor tests. Cover all five findings together, saved-config
selection through the default path, environment and CLI flag, output ordering,
TTY/NO_COLOR behaviour, silent clean checks, and unexpected check failures.
Assert no extra prompts, no component construction, the one-second timeout,
and successful exit/config retention. Run the full contribution gates with
100% line and branch coverage.
