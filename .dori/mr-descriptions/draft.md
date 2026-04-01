## Summary

Improve the NeMo Automodel installation guide based on findings from live GPU testing on a Brev instance. Five issues were identified and fixed: ambiguous CUDA prerequisite, missing PATH guidance for PyPI installs, undocumented Docker container version lag, broken editable install due to missing pip upgrade, and shell syntax errors in the Docker+Mount section.

- **Prerequisites** -- clarify CUDA driver vs toolkit requirement
- **Install methods** -- fix editable install instructions, add PATH and version notes
- **Docker** -- restructure Docker+Mount section for correct shell syntax

**4 files changed**, 527 insertions(+), 16 deletions(-).

### What changed

| Area | Files | Description |
|------|-------|-------------|
| Install guide | `docs/guides/installation.md` | 5 improvements based on live GPU testing |
| Test reports | `.dori/reports/` (3 files) | Walkthrough log, SSH log, and test report from Brev validation |

<details>
<summary><h3>Commit summary</h3></summary>

### Commits

<!-- COMMIT_TABLE_START -->
| SHA | Description |
|-----|-------------|
| `320d4983` | docs(installation): improve install guide based on GPU testing |
| `cdbc039a` | test: add brev-test reports for installation guide validation |
<!-- COMMIT_TABLE_END -->

<!-- COMMIT_DETAILS_START -->
**Commit `320d4983` -- docs(installation): improve install guide based on GPU testing:**
- Clarify CUDA prerequisite: "CUDA driver" (not toolkit/nvcc), verify with `nvidia-smi`
- Add `:::{note}` for PATH warnings when pip installs scripts to `~/.local/bin`
- Add note about Docker container version potentially differing from PyPI release
- Add `pip3 install --upgrade pip setuptools` before editable install, with PEP 660 explanation and `uv` alternative
- Restructure Docker+Mount section into 3 numbered steps with separate code blocks, removing inline comments that broke shell continuation

**Commit `cdbc039a` -- test: add brev-test reports for installation guide validation:**
- Test report with 5 installation methods validated (PyPI, Editable, GitHub Source, Docker Container, Extras)
- Walkthrough log documenting step-by-step execution and observations
- SSH log with full command output from remote GPU instance
<!-- COMMIT_DETAILS_END -->

</details>

<details>
<summary><h3>Test summary</h3></summary>

**Report**: `.dori/reports/2026-04-01-0318-installation-test-report.md`

| # | Test | Result |
|---|------|--------|
| 1 | Install via PyPI | **Pass** |
| 2 | Install in Developer Mode (Editable) | **Pass** (after fix) |
| 3 | Install via GitHub (Source) | **Pass** |
| 4 | Install via NeMo Docker Container | **Pass** |
| 5 | Install Extras | **Pass** |

**Result**: 5/5 passed

**Environment**: Brev GPU instance (NVIDIA A10G, CUDA 12.6, Ubuntu 22.04)

</details>

---

Checklist
---

- [ ] Links to a ticket
- [x] Has an assignee
- [ ] Has reviewers
- [x] Has tests
- [ ] Pipeline passes
- [ ] All discussions resolved
- [x] Documentation updated

---

*This PR was created with AI assistance (DORI documentation framework). All changes were validated through live GPU testing.*
