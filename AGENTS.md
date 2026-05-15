# AGENTS.md

## Scope

This repository is a SilverBullet space/library, not an application codebase. Treat
Markdown filenames, folders, tags, and SilverBullet directives as part of the
user-facing system.

## Repository Layout

- `settings.md`: SilverBullet `#meta` page for key bindings and general action
  buttons. It currently defines history navigation shortcuts and an inbox
  button.
- `taskbridge.md`: SilverBullet `#meta` page for the Taskwarrior bridge sync
  action button and its result notifications.
- `taskbridge/behavior-spec.md`: behavior reference for Taskwarrior bridge sync
  outcomes and structured error messages.
- `style.md`: SilverBullet `#meta` page with `space-style` CSS. Keep styling
  scoped and minimal.
- `Page Templates/Quick Note.md`: page template for quick notes. It creates
  pages under `Inbox/YYYY-MM-DD HH:MM:SS`.
- `VirtualPages/projects.md`: virtual page definition for `projects:`.
- `VirtualPages/unlinked_documents.md`: virtual page definition for `udocs:`.

## Editing Rules

- Preserve `#meta` headings on SilverBullet configuration pages.
- Preserve fenced block languages such as `space-lua` and `space-style`; they
  are executable SilverBullet content, not decorative Markdown.
- Keep YAML frontmatter valid in page templates. Quote values when punctuation
  could make YAML ambiguous.
- Do not rename or reorganize pages unless the user explicitly asks. Page names
  and paths are meaningful in SilverBullet.
- Do not invent personal notes, projects, tasks, or knowledge-base content for
  the user. Only add content the user requested or content needed for repo
  maintenance.
- Prefer small, direct Lua snippets that use SilverBullet APIs already present
  in this space.
- For quick captures and notes, keep the body minimal when the filename already
  carries timestamp metadata.

## Validation

There is no build or test suite in this repository. For changes here:

1. Run `git status --short` before and after edits.
2. Run `git diff --check` before finishing.
3. For `space-lua`, `space-style`, or template changes, inspect the affected
   page in SilverBullet when practical and verify the command, action button, or
   virtual page still loads.

## Git Hygiene

- Keep edits narrowly scoped to the requested page or configuration file.
- Do not commit unless the user asks for a commit.
- Do not add generated files, local caches, or SilverBullet runtime artifacts.
- If unrelated user changes are present, leave them intact and work around them.
