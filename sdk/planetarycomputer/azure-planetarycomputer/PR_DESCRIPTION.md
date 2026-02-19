# Description

Regenerates the `azure-planetarycomputer` SDK (v1.0.0b1) from the latest TypeSpec definition at commit [19973db](https://github.com/Azure/azure-rest-api-specs/commit/19973dbec9712fd97de1268ef14566bc7b50a629) using `@azure-tools/typespec-python@0.60.1`.

**This PR does not introduce breaking changes.** All public model classes, enum types, operation methods, and their signatures remain unchanged. The API version remains `2025-04-30-preview`.

## Changes

### Code generation updates (13 files)
- Regenerated from TypeSpec spec `specification/orbital/Microsoft.PlanetaryComputer` (API version `2025-04-30-preview`, unchanged)
- Updated emitter from previous version to `@azure-tools/typespec-python@0.60.1`
- Bumped `azure-core` dependency minimum from `>=1.35.0` to `>=1.37.0`

### Improvements from updated emitter
- **Docstring formatting** — multi-line docstrings consolidated to proper single-paragraph style; trailing periods added to all enum value descriptions for Sphinx consistency
- **Expanded `model_base.py`** — additional deserialization/serialization utilities (+143 lines)
- **Improved code generation metadata** — `_metadata.json` now includes per-service API version tracking
- **Generated pylint suppressions** — emitter now generates `# pylint: disable=too-many-locals` on operations that require it, reducing manual post-generation fixes

### Manual post-generation fixes
- Added pylint suppressions for `import-error` (MutableMapping, 3 files), `unused-import` (_deserialize_xml, 2 files), `no-value-for-parameter` (model_base.py), and `too-many-branches,too-many-statements` (inline on one function)
- Added `CONTRIBUTING.md` documenting the post-generation workflow and all required manual fixes

### Test fixes and re-recordings (8 files)
- **Added `test_10a_create_thumbnail_asset`** to `test_00` (sync + async) — creates a minimal 16×9 PNG as a collection asset with key `"thumbnail"`, ensuring the thumbnail exists before downstream SAS tests read it
- **Removed `pytest.skip`** from `test_03`/`test_04` SAS thumbnail tests (sync + async) — these were masking a missing test dependency, not a real skip condition
- **Added retry logic** to `test_04_signed_href_can_download_asset` — 5 attempts with 15s delay to handle SAS delegation key timing in live suite runs
- **Fixed conftest sanitizer** for `EnvironmentVariableLoader` interaction — added secondary regex sanitizer to catch hash suffixes after env var replacement transforms `naip-atl-2-<hash>` → `naip-atl-<hash>`
- **Removed `test_04a_create_thumbnail_asset`** from `test_07` (sync + async) — thumbnail creation moved to `test_00` where it logically belongs (after collection creation)
- **Re-recorded all 16 test files** (8 sync + 8 async) with updated sanitizers

### Local validation results
- **Static analysis:** Black, Pylint (10.00/10), Sphinx, MyPy, Pyright — all pass (0 errors)
- **Tests (playback):** 202 passed, 0 failed, 0 skipped

## Breaking Changes

**None.** This regeneration does not change the public API surface:
- No model classes were renamed, added, or removed
- No enum types or values were changed
- No operation methods were renamed or had signature changes
- No parameters were renamed or retyped

The only user-visible change is the bumped `azure-core>=1.37.0` minimum dependency.

# All SDK Contribution checklist:
- [x] **The pull request does not introduce [breaking changes]**
- [ ] **CHANGELOG is updated for new features, bug fixes or other significant changes.**
- [x] **I have read the [contribution guidelines](https://github.com/Azure/azure-sdk-for-python/blob/main/CONTRIBUTING.md).**

## General Guidelines and Best Practices
- [x] Title of the pull request is clear and informative.
- [x] There are a small number of commits, each of which have an informative message. This means that previously merged commits do not appear in the history of the PR. For more information on cleaning up the commits in your PR, [see this page](https://github.com/Azure/azure-powershell/blob/master/documentation/development-docs/cleaning-up-commits.md).

### [Testing Guidelines](https://github.com/Azure/azure-sdk-for-python/blob/main/CONTRIBUTING.md##building-and-testing)
- [x] Pull request includes test coverage for the included changes.
