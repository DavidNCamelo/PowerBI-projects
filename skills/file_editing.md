# PBIP File Editing Skill

## Purpose

This skill defines rules for creating and modifying Power BI Project (PBIP) files while preserving the exact file structure and formatting expected by Power BI and by manual PBIP editing workflows.

## Core principle

PBIP files must be modified minimally.

When modifying an existing PBIP project:

1. Preserve the existing structure whenever possible.
2. Do not reformat the entire file unless explicitly requested.
3. Do not introduce unnecessary whitespace, indentation changes, or line-ending changes.
4. Preserve property ordering unless the PBIP format requires otherwise.
5. Modify only the portions necessary to fulfill the requested change.

## JSON formatting rules

PBIP projects contain JSON files that must be treated as structured configuration files, not as arbitrary generated text.

### Trailing whitespace

Never add trailing spaces or tabs to any line.

For every modified JSON file:

* Remove trailing spaces.
* Remove trailing tabs.
* Do not introduce whitespace-only lines.
* Do not add blank lines at the end of the file.

### End of file

The final JSON character must be the closing character of the JSON document, normally `}` or `]`.

Do not append an additional blank line after the JSON document.

Do not generate:

```text
}
 
```

or:

```text
}


```

The desired representation is:

```text
}
```

When writing the file programmatically, explicitly normalize the resulting content before saving.

Conceptually:

```text
content = remove_trailing_whitespace(content)
content = remove_trailing_blank_lines(content)
```

Then write the normalized content to disk.

### Do not blindly reserialize JSON

Avoid reading a PBIP JSON file and serializing it again solely to make a modification.

Blind JSON serialization can unintentionally:

* change indentation;
* change property ordering;
* change escaping;
* change line endings;
* add or remove whitespace;
* produce large unrelated diffs.

Prefer a minimal textual modification when the requested change can be safely performed that way.

If JSON serialization is required, preserve the project's existing formatting conventions as closely as possible.

## TMDL formatting rules

The semantic model of a PBIP project is stored as TMDL files (`.tmdl`) under `<Name>.SemanticModel/definition/` (for example `model.tmdl`, `database.tmdl`, `relationships.tmdl`, `expressions.tmdl`, `tables/*.tmdl`, `cultures/*.tmdl`, `roles/*.tmdl`).

TMDL is an indentation-sensitive text format. It must be edited as structured text, never regenerated wholesale.

### Known Power BI Desktop behavior

Power BI Desktop writes every `.tmdl` file with two blank lines after the last line of content. This is the canonical representation for this project. Files that do not end this way will be rewritten by Desktop on the next save, producing a permanent, unrelated diff in Git. Therefore, Claude must reproduce this ending exactly.

### End of file

For every `.tmdl` file that Claude creates or modifies:

* The last line of real content (a property, a `ref`, a closing fence, etc.) must be terminated by a line break.
* It must be followed by exactly two blank lines.
* Those blank lines must be truly empty: no spaces, no tabs.
* Do not leave zero, one, or three or more blank lines at the end.

Desired representation (the two empty lines after the last content line are intentional):

```text
	column Amount
		dataType: double
		sourceColumn: Amount


```

Not:

```text
	column Amount
		dataType: double
		sourceColumn: Amount
```

Conceptually:

```text
content = remove_trailing_whitespace_per_line(content)
content = remove_trailing_blank_lines(content)
content = content + EOL + EOL + EOL
```

where `EOL` is the file's existing line ending (LF or CRLF). Then write the normalized content to disk.

Before applying this rule to a new project, confirm it against a `.tmdl` file freshly saved by Power BI Desktop (inspect the final bytes) and match that file exactly.

### Scope of normalization

* This ending applies to every `.tmdl` file Claude creates or modifies, including newly created tables, roles, and cultures.
* Do not touch the ending of `.tmdl` files that were not part of the request. Desktop-saved files already have the correct ending, and editing them adds unrelated diffs.
* If an existing `.tmdl` file does not end this way (for example, it was previously normalized by a tool), fix it only if it is being modified anyway; otherwise flag it to the user.
* This rule applies to `.tmdl` only. JSON files in the project keep the no-blank-lines-at-EOF rule.

### Indentation

TMDL uses indentation to define the object hierarchy.

* Preserve the indentation character already used by the file (Desktop typically writes tabs). Never convert tabs to spaces or the reverse.
* Never change the indentation level of existing lines unless the requested change requires moving the object.
* New objects and properties must use the same indentation character and depth conventions as their siblings.
* Do not reindent multi-line expressions (DAX, Power Query M) beyond what the change requires.

### Trailing whitespace

* Never add trailing spaces or tabs to any line.
* Do not introduce whitespace-only lines.
* Do not strip whitespace that is part of a multi-line string literal inside a DAX or M expression unless it is clearly outside the literal; if unsure, leave the line as is.

### Multi-line expressions

* Preserve the delimiters of multi-line expressions (triple-backtick fences ```` ``` ````) exactly as they appear.
* Preserve the existing placement of `=` and the expression start (inline vs. block).
* Edit the inside of an expression minimally; do not reformat the whole measure, column, or partition expression.

### Object and property preservation

* Do not change `lineageTag`, `sourceLineageTag`, `queryGroup`, or `annotation` values unless the request requires it.
* Do not reorder tables, columns, measures, hierarchies, or properties. New objects go at the end of their sibling group unless the request says otherwise.
* Do not rename objects or files unless explicitly requested. If a table, column, or measure is renamed, update every dependent reference in the project (relationships, `ref` entries in `model.tmdl`, `sortByColumn`, hierarchies, expressions, and report visuals).
* Do not remove or alter `ref table` entries in `model.tmdl` unless adding or removing a table.
* New objects that need a `lineageTag` must receive a newly generated, unique GUID. Never copy an existing one.
* Preserve quoting conventions: names with spaces or special characters stay in single quotes exactly as in the original.

### Line endings and encoding

* Preserve the file's existing line endings (LF or CRLF) and encoding (UTF-8, with or without BOM).
* Do not mix line endings within a file.

## Validation

After modifying a PBIP JSON file:

1. Validate that the file is syntactically valid JSON.
2. Verify that no trailing spaces or tabs exist.
3. Verify that no blank lines exist at EOF.
4. Verify that only the intended content changed.
5. Prefer a diff against the original file when modifying an existing project.

After modifying a `.tmdl` file:

1. Verify that indentation is consistent with the original (same character, correct depth).
2. Verify that no trailing spaces or tabs exist.
3. Verify that the file ends with the last content line followed by exactly two empty lines (no spaces or tabs on them).
4. Verify that multi-line expression fences are balanced.
5. Verify that no `lineageTag` was duplicated or unintentionally changed, and that references to renamed objects were updated.
6. Verify that only the intended content changed, preferably with a diff against the original file.

A final byte-level check should conceptually verify:

```text
last_character ∈ {valid JSON closing character}   (JSON)
ends with: content + EOL + EOL + EOL              (TMDL)
```

and that no whitespace-only lines exist after the last content line (JSON: none at all; TMDL: exactly two empty lines).

## PBIP preservation rules

When modifying PBIP projects:

* Do not convert `.pbip` projects into `.pbix`.
* Do not remove PBIP metadata.
* Do not rename files unless explicitly requested.
* Do not change IDs, GUIDs, names, or references unless required by the requested modification.
* Preserve relationships between PBIP files.
* Preserve existing folder structure.
* Preserve encoding and line-ending conventions whenever possible.
* Avoid unrelated formatting changes.

## Git compatibility

PBIP files are commonly version-controlled.

Therefore, modifications should produce the smallest possible Git diff.

Before considering a modification complete, check for:

* unexpected whitespace-only changes;
* changes to line endings;
* changes to indentation;
* reordered JSON properties or reordered TMDL objects;
* unexpected final blank lines in JSON, or a `.tmdl` ending that is not exactly two empty lines;
* unrelated changes to other PBIP files.

A change that is semantically correct but produces a large unrelated diff should be treated as undesirable and reviewed before completion.

## Priority

When there is a conflict between convenience and preservation of the existing PBIP representation, prioritize:

1. Valid PBIP structure.
2. Preservation of existing content.
3. Minimal diff.
4. Preservation of formatting.
5. Correct EOF representation: no trailing whitespace anywhere; no blank lines at EOF for JSON; exactly two empty lines at EOF for `.tmdl`.
6. Only then apply general formatting preferences.