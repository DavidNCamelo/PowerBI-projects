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

## Validation

After modifying a PBIP JSON file:

1. Validate that the file is syntactically valid JSON.
2. Verify that no trailing spaces or tabs exist.
3. Verify that no blank lines exist at EOF.
4. Verify that only the intended content changed.
5. Prefer a diff against the original file when modifying an existing project.

A final byte-level check should conceptually verify:

```text
last_character ∈ {valid JSON closing character}
```

and that the file does not contain additional whitespace-only lines after that character.

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
* reordered JSON properties;
* unexpected final blank lines;
* unrelated changes to other PBIP files.

A change that is semantically correct but produces a large unrelated diff should be treated as undesirable and reviewed before completion.

## Priority

When there is a conflict between convenience and preservation of the existing PBIP representation, prioritize:

1. Valid PBIP structure.
2. Preservation of existing content.
3. Minimal diff.
4. Preservation of formatting.
5. No trailing whitespace or blank lines at EOF.
6. Only then apply general formatting preferences.
