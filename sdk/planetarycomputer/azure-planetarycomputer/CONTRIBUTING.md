# Contributing to azure-planetarycomputer

This guide covers the manual steps required after regenerating the SDK from TypeSpec, and how to validate your changes locally before pushing to CI.

For general Azure SDK for Python contribution guidance, see the [top-level CONTRIBUTING.md](https://github.com/Azure/azure-sdk-for-python/blob/main/CONTRIBUTING.md).

---

## Regenerating the SDK from TypeSpec

### 1. Update `tsp-location.yaml`

Edit [tsp-location.yaml](tsp-location.yaml) to point to the desired spec commit and directory:

```yaml
directory: specification/orbital/Microsoft.PlanetaryComputer
commit: <new-commit-sha>
repo: Azure/azure-rest-api-specs
```

### 2. Run code generation

From the package root (`sdk/planetarycomputer/azure-planetarycomputer`):

```bash
npx tsp-client update
```

---

## Known Manual Fixes on Generated Code

The TypeSpec code generator does not emit certain comments or formatting that CI checks require. The following fixes must be **manually applied** after every regeneration.

### Pylint Suppressions

#### `azure/planetarycomputer/operations/_operations.py`

| Location | Suppression | Reason |
|---|---|---|
| Line 1 (file-level) | `# pylint: disable=too-many-lines,too-many-locals,too-many-branches,too-many-statements` | Generated request builders and operations exceed pylint thresholds |
| `from collections.abc import MutableMapping` | `# pylint: disable=import-error` | Not resolvable in the pylint virtualenv |
| `from .._utils.model_base import (` | `# pylint: disable=unused-import` | `_deserialize_xml` is imported but not always used; the suppression must go on the `from` line (Black reformats the import into multi-line) |

#### `azure/planetarycomputer/aio/operations/_operations.py`

| Location | Suppression | Reason |
|---|---|---|
| Line 1 (file-level) | `# pylint: disable=too-many-lines,too-many-locals` | Generated async operations exceed pylint thresholds |
| `from collections.abc import MutableMapping` | `# pylint: disable=import-error` | Not resolvable in the pylint virtualenv |
| `from ..._utils.model_base import (` | `# pylint: disable=unused-import` | Same as sync - suppression goes on the `from` line |

#### `azure/planetarycomputer/_utils/model_base.py`

| Location | Suppression | Reason |
|---|---|---|
| `from collections.abc import MutableMapping` | `# pylint: disable=import-error` | Not resolvable in the pylint virtualenv |
| `return super().__new__(cls)` (in `Model.__new__`) | `# pylint: disable=no-value-for-parameter` | False positive - pylint cannot resolve the MRO for `__new__` |

> **Important:** After adding pylint suppressions, run `tox -e black` first - Black may reformat single-line imports into multi-line, which moves your `# pylint: disable` comments. If Black reformats the `_deserialize_xml` import into multiple lines, the `# pylint: disable=unused-import` comment must be on the `from ... import (` line, **not** on the closing `)` or the individual name line.

### Sphinx Docstring Fixes

The generated docstring for `DataOperations.get_interval_legend` (in both sync and async `_operations.py`) contains two formatting issues that cause Sphinx warnings (treated as errors):

1. **Missing newline after JSON code block:** The closing `]` of the `.. code-block:: json` runs directly into the text `This example defines two intervals:`. Add a blank line between them.
2. **Bullet continuation indentation:** The second bullet's continuation line (`color.`) must be indented to align with the bullet content (use 10 spaces instead of 8) so Sphinx doesn't report "Bullet list ends without a blank line; unexpected unindent."

### Sample Updates

If the TypeSpec renames or removes API operations, the hand-written samples under `samples/` and `samples/async/` must be updated to match. For example, `list_collections` was renamed to `get_collections` - all sample files referencing the old name need to be updated. MyPy and Pyright (which also check samples) will catch these.

---

## Local Validation

Run the following checks **from the package root** before pushing. All commands use the shared tox config:

```bash
cd sdk/planetarycomputer/azure-planetarycomputer
```

### Formatting (Black)

```bash
tox -e black -c ../../../eng/tox/tox.ini --root .
```

### Linting (Pylint)

```bash
tox -e pylint -c ../../../eng/tox/tox.ini --root .
```

### Type Checking (MyPy)

```bash
tox -e mypy -c ../../../eng/tox/tox.ini --root .
```

### Type Checking (Pyright)

```bash
tox -e pyright -c ../../../eng/tox/tox.ini --root .
```

### Documentation (Sphinx)

```bash
tox -e sphinx -c ../../../eng/tox/tox.ini --root .
```

### API Stub Generation

```bash
tox -e apistub -c ../../../eng/tox/tox.ini --root .
```

> **Tip:** Running `black`, `pylint`, `mypy`, `pyright`, and `sphinx` locally catches the vast majority of CI failures before you push.

---

## Quick-Reference Checklist

After running `npx tsp-client update`:

- [ ] Restore `tests/` and `samples/` - `git checkout -- tests/ samples/`
- [ ] Add file-level pylint suppressions to both `_operations.py` files
- [ ] Add inline `# pylint: disable=import-error` to `MutableMapping` imports (3 files)
- [ ] Add `# pylint: disable=unused-import` on the `from ... import (` line for `_deserialize_xml` (2 files)
- [ ] Add inline `# pylint: disable=no-value-for-parameter` to `Model.__new__` in `model_base.py`
- [ ] Run `tox -e black` - formatting (may reformat imports; re-check pylint comment placement)
- [ ] Run `tox -e pylint` - linting (should score 10.00/10)
- [ ] Fix Sphinx docstring issues in `get_interval_legend` (both sync and async `_operations.py`)
- [ ] Run `tox -e sphinx` - documentation
- [ ] Run `tox -e mypy` and `tox -e pyright` - type checking (will catch renamed/removed APIs in samples)
- [ ] Update samples if any operations were renamed or removed
- [ ] Update `CHANGELOG.md` with a release date if preparing a release
