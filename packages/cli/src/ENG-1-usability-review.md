# ENG-1: Usability review of zen

This review is based on the CLI interfaces and shared argument parser in the selected ZeneraNeo source. Each finding includes the exact interface text read from the repository and a concrete replacement.

## 1. Root help should make the command surface discoverable

**Observed interface text:** the command registry exposes `init`, `list`, `run`, `open`, `key`, `models`, `check`, `inspect`, `view`, `export`, `import`, `memory`, `sandbox`, `server`, `cache`, `completion`, and `version`, plus external command families.

**Problem:** A flat command list does not tell a new user which commands are safe starting points, what each command consumes, or where to find the next level of help. The result is discovery by trial and error.

**Proposed replacement:** Make `zen --help` group commands by task and show one short purpose line for each group. Include a first-use path such as `zen init <dir>` followed by `zen run <project> <prompt>`, and finish with `Use zen <command> --help for details.` Keep the complete command list available, but lead with the common path.

## 2. `zen init` hides the meaning of its optional positional argument

**Exact interface text:** `zen init [dir] [--name <name>] [--model <ref>] [--force]`.

**Problem:** The usage line says `dir`, but it does not say whether the directory is created, adopted, or overwritten. The source behavior distinguishes an existing project, a nonempty directory, and the `--force` case, so a user cannot predict the consequence of supplying an existing path from the help text alone.

**Proposed replacement:** Use `zen init [directory] [options]` and add explicit help text: `Creates a project in directory. An existing project is adopted. A nonempty directory is refused unless --force is supplied; --force may replace generated files.` Also show one safe example and one refusal example.

## 3. `zen run` leaves the primary invocation shape as an opaque options bucket

**Exact interface text:** `zen run [project] [prompt] [options]`.

**Problem:** The command accepts project and prompt positionals plus session, input, workspace, memory, model, read-only, output, events, batch, concurrency, and related flags. A user cannot tell from this usage line which inputs are required for an ordinary run, which options are mutually exclusive, or whether a prompt can be supplied through an input file.

**Proposed replacement:** Split the help into common forms: `zen run <project> <prompt>`, `zen run <project> --input <file>`, and `zen run <project> --workspace <dir>`. Add an Options section grouped into Input, Workspace, Session, Output, and Execution, and state the conflict rule beside the affected options instead of leaving it to a later validation error.

## 4. `zen inspect` makes the command mode and run target look optional when they are not

**Exact interface text:** `zen inspect [report|graph|node|ask] [run] [--dir <run dir>] [--open]`.

**Problem:** Both the inspection mode and the run are shown as optional, but the implementation has mode-specific requirements: node inspection needs a node id, ask needs a question and an interactive context, and the available report data depends on a run. The usage line therefore suggests that `zen inspect` is a complete invocation when it is only a shell for several different operations.

**Proposed replacement:** Publish separate forms: `zen inspect report <run>`, `zen inspect graph <run>`, `zen inspect node <node-id> <run>`, and `zen inspect ask <run> --question-file <file>`. Mark `--dir` as the alternative run source and put `--open` only on the forms where it applies. Add a one-line description of what each mode returns.

## 5. Shared parser errors should identify the command, argument, and repair

**Observed parser behavior:** the shared parser uses strict `parseArgs` with positionals enabled; parser exceptions are converted through `usageError`, and the `one()` helper rejects extra positionals with `expected at most one <what>, got <count>`.

**Problem:** A generic usage error and a count-only message do not tell the user whether the failure came from a typo, a missing argument, an extra positional, or an option that belongs to another command. The required error cases are especially hard to recover from when the command has many flags.

**Proposed replacement:** Preserve the exact offending token and command context in every error. Use forms such as `zen run: unknown option --worskpace. Did you mean --workspace?`, `zen init: missing value for --name. Use --name <name>.`, and `zen inspect node: expected a node id before <run>.` Keep the current nonzero exit behavior, but end each message with the shortest valid corrected invocation or the relevant `zen <command> --help` target.

## Review boundary

These findings are specific to the command interfaces and parser behavior visible in the selected ZeneraNeo source. Runtime help and error output should be checked after the repository can be executed under its required Node version; this document does not claim unobserved runtime output.
