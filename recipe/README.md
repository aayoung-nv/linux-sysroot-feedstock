# glibc 2.35 sysroots

This recipe is intended for a dedicated `v2.35` maintenance branch, following
this feedstock's existing version branches. It adds an optional glibc 2.35
build target; it does not propose changing conda-forge's default glibc baseline.

Ubuntu Jammy supplies glibc 2.35 for x86-64 and aarch64. The recipe repackages
`libc6`, `libc6-dev`, `libc-bin`, and `locales`, plus `linux-libc-dev` in the
separate kernel-header output. Ubuntu's multiarch paths are normalized to the
existing Conda compiler sysroot layout. Upstream copyright notices are included.
The system loader and runtime libc remain in use when applications execute.

## Refresh sources

From this directory:

```sh
python update.py --glibc-version 2.35-0ubuntu3.15 --kernel-version 5.15.0-191.201
```

The updater reads both Jammy update indices, requires the specified package
versions, and writes URLs/SHA-256 checksums for both architectures. Increment
`build_number` when updating packages within the same glibc version. Keep the
sysroot and kernel-header outputs synchronized. If a revision has left Ubuntu's
rolling archive, update to its replacement and rerun the tests.

## Validate

Rerender with `conda smithy rerender`. Use the generated `build-locally.py`
configurations to build each target, including both `with_features` variants:

```sh
python build-locally.py --config linux_64_cross_target_platformlinux-64target_machinex86_64
python build-locally.py --config linux_64_cross_target_platformlinux-aarch64target_machineaarch64
```

Existing file-content tests cover both architectures. On x86-64, additional
Conda GCC/G++ 11.4 tests compile and execute a call to `_dl_find_object` (added in
glibc 2.35) and a C++ thread/exception probe. The tests check the compiler sysroot,
normal ELF interpreter, and the maximum required GLIBC symbol version.

ARM package-content checks can run on an x86 builder because the outputs are
cross-compilation sysroots. They do not establish ARM runtime compatibility.
Native ARM validation is still required before claiming that qualification.
