---
name: global-output
description: Shared output policy and file naming rules for object reference files
---

Shared output policy for `references/object-*.md`

---

# OUTPUT MODES
Use mode names `single-file` and `multi-file`:
- `single-file`: One artifact with the full object in GeneXus syntax: `.main.gx`
- `multi-file`: One artifact per syntax part: `.main.gx` plus optional native artifacts

---

# FORMAT MATRIX
- `<name>.<type>.main.gx`
	* Mandatory in both modes
	* `single-file`: Complete object definition
	* `multi-file`: Only GeneXus-specific definition
- `<name>.<type>.layout.xml`
	* Optional layout definition in XML syntax
- `<name>.<type>.properties.toml`
	* Optional object properties in TOML syntax
- `<name>.<type>.documentation.md`
	* Optional object documentation in Markdown syntax

---

# SAVE POLICY
When saving a target object, resolve mode in this order:
1. Detect existing artifacts for the target object using canonical names:
	* `<name>.<type>.main.gx`
	* `<name>.<type>.layout.xml`
	* `<name>.<type>.properties.toml`
	* `<name>.<type>.documentation.md`
2. If artifacts exist:
	* Infer current mode from existing files
	* Preserve that mode by default
3. If user explicitly requests a different mode:
	* Switch to requested mode
	* Restructure artifacts to that mode before final save
4. If no artifact exists for the target object:
	* Use explicitly requested mode, if provided
	* Otherwise default to `single-file`

Mode inference rules:
- For `single-file`, only `.main.gx` exists and split artifacts are absent
- For `multi-file`, both `.main.gx` and at least one split artifact exist

---

# RESTRUCTURE
Rules for file-system restructuring:
1. `single-file` → `multi-file`
	* Extract native artifact parts only when sections exist or are requested
	* Keep only GeneXus-specific content in `<name>.<type>.main.gx` after successful split
2. `multi-file` → `single-file`
	* Consolidate all available parts into `<name>.<type>.main.gx` using target object `SYNTAX` section
	* Remove obsolete split artifacts after successful consolidation

---

# EXAMPLES
- Single-file object:
	* `Customer.transaction.main.gx` (includes source, properties, and documentation)
- Multi-file object:
	* `Customer.transaction.main.gx`
	* `Customer.transaction.properties.toml`
	* `Customer.transaction.documentation.md`

---

# CONSTRAINTS
- Output policy governs artifact selection and naming only
- Output policy never overrides section completeness rules from object `SYNTAX` section
- Object-specific reference files may define exceptions to this policy
- Enforce canonical naming only from `FORMAT MATRIX` section
- Generate `multi-file` artifacts only when requested or required
- Keep one mode per target object after save; never keep both `single-file` and `multi-file` artifacts
