# Configuration workflow

Read this file only when `config.yaml` is missing, `configured` is not `1`, the user explicitly requests reconfiguration, or a configuration repair is required.

## Bootstrap

1. If `config.yaml` is missing, copy the structure from `config.example.yaml` without changing the example file.
2. Validate `schema_version`. Version `1` is supported by Skill version 1.2.
3. Preserve valid existing values during repair or schema migration.
4. Never store credentials, tokens, passwords, private repository content, or transient discovery results.

## Full first-use or reconfiguration flow

Ask only the questions relevant to the selected branch.

### 1. Documentation decision

Ask whether a successful installation should create an explanation document.

- If no, set `documentation.enabled: false`; do not ask for a directory, template, or retry policy. Continue to project scanning.
- If yes, set `documentation.enabled: true` and continue.

### 2. Directory and existing-template decision

In one interaction, collect:

- the directory where successful-installation documents should be written;
- whether the user wants to reuse an existing format.

Validate that the directory exists or can be created with the user's authorization.

### 3. Template branch

If reusing an existing format, ask the user to choose:

- `directory_samples`: infer a dynamic template from files already in the documentation directory;
- `explicit_file`: use a specific template file supplied or selected by the user.

If not reusing an existing format, ask the user to choose:

- `ai_generated`: generate a complete recommended template for confirmation;
- `collaborative`: develop the template with the user through focused questions.

For `ai_generated` and `collaborative`, use the installed Obsidian-related Skills that are materially relevant. Save the confirmed template as a real file before configuration can finish, then store its path in `documentation.template.file`.

### 4. Directory-sample behavior

For `directory_samples`:

- examine Markdown files in the configured directory only;
- do not recurse unless the user later changes `recursive`;
- read at most `sample_limit` files;
- prefer samples matching the current project type;
- otherwise use the most common structure;
- infer only formatting and structure, never copy project-specific or sensitive content.

If no usable sample exists, ask whether to switch to `ai_generated` or `collaborative`. Do not switch automatically.

### 5. Project scanning depth

Ask for one level:

- `light`: scan relevant task titles, summaries, project identifiers, manifests, and directly relevant configuration or documentation.
- `medium`: perform the light scan, then read at most `recent_task_limit` related recent tasks and at most `turn_limit_per_task` recent turns from each.

Default limits are 10 tasks and 10 turns per task. Filter by title or summary relevance before reading task content; never scan all conversations indiscriminately.

### 6. Confirmation and commit

Show a concise summary of all selected values. After user confirmation:

1. Write the completed configuration to `config.yaml`.
2. Validate conditional requirements and referenced paths.
3. Set `configured: 1` only after all required template files and directories are valid.
4. Resume the task that triggered first-use configuration.

## Conditional validation

- `documentation.enabled: false`: ignore directory, template, and documentation retry fields.
- `documentation.enabled: true`: require a valid directory and template mode.
- `explicit_file`, `ai_generated`, or `collaborative`: require a readable template file.
- `directory_samples`: require `sample_limit >= 1`; `file` may be null.
- `project_scan.level`: must be `light` or `medium`.
- `medium`: require positive task and turn limits.
- `documentation.retry.policy`: must be `ask`, `auto_safe`, or `never`; automatic retries must have a finite non-negative limit.

## Repair and reconfiguration

On each use, validate only the fields required for the current task.

1. If one path or field is invalid, auto-detect a replacement.
2. If exactly one reliable replacement exists, update that field and preserve the rest.
3. If multiple candidates or no candidate exists, ask the user to repair that field.
4. Do not rerun the full wizard for one invalid field.
5. Set `configured: 0` and run the full wizard only when the configuration is unreadable, the schema is unsupported, or multiple critical fields are invalid.
6. An explicit user request to reconfigure always runs the full wizard.

## Distribution and upgrades

- Ship `config.example.yaml`.
- Generate `config.yaml` locally on first use.
- Never include a configured `config.yaml` in a public package.
- Never overwrite `config.yaml` during a Skill update.
- If a new Skill version adds fields, merge only missing defaults and preserve user choices.
