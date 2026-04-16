---
name: saiadmin6
description: Guide for developing SaiAdmin6.x plugins, encompassing both the Webman backend and Vue/Nuxt frontend. Use this skill when the user wants to create a new plugin, understand the plugin structure, or add features to an existing SaiAdmin6.x plugin.
---

# SaiAdmin Plugin Development

This skill provides guidance on the standard structure and development workflow for SaiAdmin6.x plugins, based on the `saiadmin6.x` architecture.

## Overview

A standard SaiAdmin plugin consists of two main parts:
1.  **Backend**: Located in `server/plugin/<plugin_name>`. See [Backend Rules](references/backend.md).
2.  **Frontend**: Located in `saiadmin-artd/src/views/plugin/<plugin_name>`. See [Frontend Rules](references/frontend.md).

## Database Standards

All plugin tables must follow standardized structure. See [Database Standards](references/database.md) for:
- Primary key requirements
- Required standard fields (status, created_by, updated_by, timestamps)
- Complete table creation examples
- Naming conventions

## Creating a New Plugin

### Step 1: Create Backend (Automatic)

Use the built-in SaiAdmin command to generate the backend structure:

1.  Open terminal in `server/` directory.
2.  Run: `php webman sai:plugin <your_plugin_name>`

This command automatically creates:
- `server/plugin/<your_plugin>` directory
- `config/` files (app, route, middleware, etc.)
- `app/` directory with `admin/controller`, `api/controller`, etc.
- A default `IndexController`.

### Step 2: Create Frontend (Manual)
1.  Create directory `saiadmin-artd/src/views/plugin/<your_plugin>`.
2.  Create `api/` directory for API definitions.
3.  Create view directories as needed.

## Development Standards

For detailed development standards and code patterns, please refer to:

-   **[Database Standards](references/database.md)**: Covers table structure, required fields, and naming conventions.
-   **[Backend Development Rules](references/backend.md)**: Covers Controllers, Models, Validators, and directory structure.
-   **[Frontend Development Rules](references/frontend.md)**: Covers API definitions, List Pages, Dialogs, and directory structure.
