# su26-ai301-contribution

# Contribution [#216]: [Implement an option to control the opacity of blockgraph structures]

**Contribution Number:** [1 / 2 / 3]  
**Student:** [Robert Asumeng]  
**Issue:** [[GitHub issue link](https://github.com/tqec/tqec/issues/690)]  
**Status:** [Phase I] [Complete]

---

## Why I Chose This Issue

I chose this issue because it aligned very well with me and my coding preferences, I personally prefer Python and the language is in python. Also the issue pertains to a QOL feature and I appreciate good QOL.
---

## Understanding the Issue

### Problem Description

The issue is that the Graphing system does not have the ability to control the opacity of the visual and the creator would like that implemented.

### Expected Behavior

The graphs opacity should be changeable

### Current Behavior

The opacity isn't changeable

### Affected Components

Files in th src/tqec folder 

---

## Reproduction Process

### Environment Setup

Currently, you need to install tqec from source with pip or uv:

With pip:

python -m pip install git+https://github.com/tqec/tqec.git


### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

**Root cause:** `TQECColor.rgba` property (`src/tqec/interop/color.py:70-77`) hardcodes `alpha=1.0` for X/Y/Z/H face colors. `_BaseColladaData._add_materials()` (`src/tqec/interop/collada/read_write.py:455-478`) creates COLLADA `Effect` objects with `transparency=rgba[3]`, which is always 1.0. No opacity parameter flows through the call chain: `BlockGraph.to_dae_file()` → `write_block_graph_to_dae_file()` → `_BaseColladaData.__init__()` → `_add_materials()`.

Only correlation surfaces (`X_CORRELATION`, `Z_CORRELATION`) use `alpha=0.8` — these should remain unchanged.

### Proposed Solution

Thread an `opacity: float` parameter through the DAE export pipeline. Apply it as an alpha override on all non-correlation-surface materials only. Correlation surfaces preserve their intrinsic alpha (0.8) unchanged. This lets users see internal block structure in a single rendered image without washing out correlation surfaces.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** BlockGraph 3D visualization always renders blocks at 100% opacity. Users who want to export a single image showing internal structure (e.g., boundaries, internal pipes) cannot do so because no opacity parameter exists. The fix requires threading an opacity value from the public API through to the COLLADA material definitions, without affecting correlation surface visibility.

**Match:** The codebase already uses `RGBA.with_alpha()` (`src/tqec/interop/color.py:26`) but it's never called from the rendering pipeline. The `TQECColor` enum already demonstrates per-color alpha (correlation surfaces at 0.8 vs blocks at 1.0). This pattern extends naturally: apply a uniform alpha override at the material-creation step.

**Plan:**
1. **`src/tqec/interop/collada/read_write.py`:**
   - Add `opacity: float = 1.0` parameter to `write_block_graph_to_dae_file()`
   - Store `self._opacity: float` in `_BaseColladaData.__init__()`
   - In `_add_materials()`, when creating `Effect` for non-correlation colors, replace alpha with `self._opacity` using `face_color.rgba.with_alpha(self._opacity).as_floats()`
   - Correlation surface colors keep intrinsic alpha unchanged

2. **`src/tqec/computation/block_graph.py`:**
   - Add `opacity: float = 1.0` to `to_dae_file()` signature, pass through
   - Add `opacity: float = 1.0` to `view_as_html()` signature, pass through

3. **`tests/interop/collada/read_write_test.py`:**
   - Test default opacity produces `transparency=1.0` in DAE
   - Test opacity=0.5 scales material transparency
   - Test correlation surface opacity preserved (0.8) when block opacity is 0.5
   - Test out-of-range opacity raises error

**Implement:** `https://github.com/<your-fork>/tqec/tree/fix-issue-#690`

**Review:** Self-review checklist:
- [ ] Follows project code conventions (type hints, docstrings, ruff compliance)
- [ ] No breaking changes to existing API (new parameter with default)
- [ ] Test coverage for new parameter
- [ ] `uv run pytest tests/` passes on all existing tests
- [ ] `uv run ruff check src/` passes

**Evaluate:** Verification steps:
1. `uv run pytest tests/interop/collada/read_write_test.py -v` — all tests pass including new ones
2. Parse generated DAE XML to assert material transparency values are correct
3. Roundtrip test: write graph with opacity → read back → graph structure unchanged
4. Manual visual check with `g.view_as_html(opacity=0.3)` in Jupyter
5. Full test suite: `uv run pytest tests/ -m ""`

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
