# LabPress Documentation Overview

Welcome to the LabPress documentation. This guide provides a technical overview of LabPress, a lightweight academic content management system designed for research groups, university laboratories, and individual research teams. You will learn what LabPress is, the design principles behind it, the technology stack that powers it, and how the system architecture is organized to support extensibility, internationalization, and dynamic content management.

## What is LabPress?

LabPress is an open-source Academic CMS written in pure PHP and MySQL, with **no framework dependencies**. It allows labs or small research teams to deploy a professional website capable of presenting research outputs, team members, publications, news, custom tools, and projects within minutes. Unlike generic CMS platforms, LabPress is natively tailored for academic use cases: it ships with built-in modules for managing publications, project portfolios with category filters, research news with Markdown editing, tool versioning, and a fully dynamic homepage that can be edited entirely from the administration panel.

LabPress is not just another website builder. Its core strength lies in a **hook-driven plugin system** inspired by WordPress, which allows developers to extend functionality without modifying any core files. Combined with a decoupled internationalization (i18n) layer that handles both static and database-stored dynamic content, LabPress achieves modularity and multilingual support rarely found in lightweight CMS solutions.

## Design Philosophy

LabPress was born out of the practical needs of academic research groups that require both dynamic content management and multilingual presentation. Every architectural decision follows a few guiding principles:

- **Academic First** – Features such as publication listings, project categorization, research news, and tool showcases are first-class entities with dedicated admin interfaces and database tables.
- **Extensible by Design** – All core functionality is exposed through actions and filters via the `LabPress_Hooks` class. Plugins can intercept, modify, or augment behavior at dozens of defined hook points.
- **Zero Coupling** – The core system has no awareness of any plugin. Even the dynamic multilingual capability is implemented entirely as a plugin, using the `translate` filter to replace strings. Removing all plugins leaves a functional site that falls back to its default language for core features.
- **Separation of Concerns** – The frontend and backend are split at the entry-point level, with independent language contexts, asset loading, and permission checks. This ensures that public-facing performance is not impacted by administrative complexity.
- **Lightweight Yet Complete** – The entire application runs on commodity hosting with PHP 7.4+ and MySQL 5.7+. No Composer, no Node.js build step, and no virtual environment are required. Third-party libraries are kept to a minimum and are either bundled locally or loaded from stable public CDNs.

## Technology Stack

| Layer             | Technology                                                                               | Justification                                                                                                  |
| ----------------- | ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Language**      | PHP 7.4+                                                                                 | Ubiquitous on shared hosting; no framework overhead; compatible with most virtual hosting environments         |
| **Database**      | MySQL 5.7+ / MariaDB 10.3+                                                               | Reliable and widely available; structured data for academic entities                                           |
| **Frontend**      | HTML5, CSS3, vanilla JS                                                                  | Minimal dependencies; EasyMDE and Isotope are bundled locally, while Font Awesome is loaded from a public CDN. |
| **Plugin System** | Custom `LabPress_Hooks` class                                                            | Lightweight implementation of the WordPress hook paradigm                                                      |
| **i18n**          | Custom `__()` function + language files                                                  | Module-based loading, supports nested keys, integrates with the filter system                                  |
| **Markdown**      | Parsedown library (bundled)                                                              | Converts Markdown to HTML for news and project content                                                         |
| **Security**      | PDO prepared statements, bcrypt hashing, session-based authentication, permission checks | Prevents SQL injection and XSS; restricts admin actions based on user roles                                    |

LabPress does not depend on any third-party PHP frameworks, ORMs, or template engines. The codebase follows a pragmatic flat-file structure. Frontend pages are standalone PHP files at the project root, each handling its own data retrieval and rendering. Administrative interfaces are separated into `/admin/views/` and loaded by the backend entry points. Shared logic resides in `/includes/`, with data access provided by PDO wrapper functions.

## Architecture Overview

LabPress follows a straightforward request lifecycle:

1. **Bootstrapping**: `index.php` (or the relevant frontend page) loads `includes/config.php`, initializes the database connection, and triggers the `init` action hook after all plugins have been loaded.
2. **Hook Initialization**: Each enabled plugin registers its callbacks on `init`. Plugins typically add filters for translation, menu items, or slide data, and actions for injecting scripts or registering admin pages. For detailed information, please refer to the Plugin Development section.
3. **Content Rendering**: Frontend pages query the database directly using helper functions. Key output points are wrapped in `applyFilters()` or `__()` calls, allowing plugins to alter content without the core being aware of their existence.
4. **Admin Panel**: The `/admin` directory contains an independent entry point that enforces authentication and loads the appropriate view based on the query string. Admin views are plain PHP files that typically contain their own data retrieval and POST handling logic for simplicity, without a separate controller layer.

The **plugin system** is the backbone of extensibility. It provides:

- **Actions** (`addAction`, `doAction`) – for executing code at specific points (e.g., `footer_scripts`, `admin_before_head_end`).
- **Filters** (`addFilter`, `applyFilters`) – for modifying data (e.g., `translate`, `nav_menu_item`, `slides_data`).
- **Plugin Lifecycle Hooks** – `plugin_activation`, `plugin_deactivation`, and `plugin_uninstalled` are triggered during activation, deactivation, and uninstallation respectively.

For a complete list of all available actions and filters, refer to the **[Hooks Reference](../plugins/hooks-reference.md)**.

A plugin is simply a folder inside `/plugins/` containing a `plugin.php` file with a standardized header comment. The admin panel scans this directory, manages activation states, and supports ZIP upload for installation.

The **internationalization system** works on two levels:

- **Static strings** are stored in language files under `languages/{locale}/`, organized by module. The `__()` function loads these on demand and supports dot-notation for nested arrays.
- **Dynamic content** (e.g., homepage slogan, research direction cards, navigation menus) stored in the database is translated via the `Dynamic Multi-Language` plugin, which registers on the `translate` filter and queries its own `ml_translations` table.

The two systems operate independently but share the same `translate` filter mechanism. The core uses `__()` for all output, and the plugin transparently provides translations when available without modifying the core.

## Directory Structure

```
LabPress/
├── admin/                            # Backend management panel
│   ├── index.php                     # Admin entry point (dashboard loader)
│   ├── login.php                     # Admin login page
│   ├── logout.php                    # Logout handler
│   ├── includes/
│   │   ├── admin-header.php          # Admin sidebar + top bar
│   │   └── admin-footer.php          # Admin footer (JS includes)
│   ├── views/                        # Individual management views
│   │   ├── dashboard.php             # System overview & stats
│   │   ├── publications.php          # Publications CRUD
│   │   ├── slides.php                # Slides management
│   │   ├── tool-list.php             # Tool list management
│   │   ├── tool-detail.php           # Tool detail editor
│   │   ├── tool-versions.php         # Version history manager
│   │   ├── project-categories.php    # Project categories CRUD
│   │   ├── projects.php              # Projects management
│   │   ├── news.php                  # Research news management
│   │   ├── users.php                 # User management
│   │   ├── site-config.php           # General site settings
│   │   ├── nav-menus.php             # Navigation menu manager
│   │   ├── plugins.php               # Installed plugins list
│   │   ├── plugins-install.php       # Plugin installer (ZIP upload)
│   │   └── plugins-market.php        # Plugin marketplace (placeholder)
│   └── assets/
│       ├── css/
│       │   ├── admin.css             # Backend global styles
│       │   └── easymde.min.css       # EasyMDE editor styles (admin)
│       └── js/
│           ├── admin.js              # Sidebar & common admin scripts
│           ├── easymde.min.js        # EasyMDE Markdown editor (admin)
│           ├── plugins-manage.js     # Plugin activation/deactivation
│           ├── plugins-install.js    # Plugin installation logic
│           ├── plugins-market.js     # Plugin market interactions
│           ├── publications.js       # Publications CRUD logic
│           ├── slides.js             # Slides CRUD + image upload
│           ├── tool-list.js          # Tool list CRUD
│           ├── tool-detail.js        # Tool detail loading
│           ├── tool-versions.js      # Version management logic
│           ├── project-categories.js # Category CRUD
│           ├── project-manage.js     # Projects CRUD + EasyMDE
│           ├── news-manage.js        # News CRUD + EasyMDE
│           ├── users.js              # User management logic
│           ├── site-config.js        # Site settings dynamic forms
│           └── nav-menus.js          # Menu CRUD logic
│
├── api/                              # RESTful data endpoints
│   ├── data.php                      # Unified data reader (type-based)
│   ├── save.php                      # Unified data saver (type-based)
│   ├── upload.php                    # Image upload handler
│   ├── users.php                     # User CRUD API
│   ├── auth.php                      # Authentication API (check/login/logout)
│   ├── plugin-install.php            # Plugin ZIP installation endpoint
│   ├── plugin-install-from-url.php   # Remote plugin installation
│   └── plugin-uninstall.php          # Plugin removal endpoint
│
├── assets/                           # Frontend global static resources
│   ├── css/
│   │   ├── index.css                 # Homepage styles
│   │   ├── header.css                # Navigation bar styles
│   │   ├── tools.css                 # Tools list page styles
│   │   ├── tool_detail.css           # Tool detail page styles
│   │   ├── news.css                  # News list & detail styles
│   │   ├── news-detail.css           # News detail specific styles
│   │   ├── projects.css              # Project list page styles
│   │   └── project-detail.css        # Project detail page styles
│   └── js/
│       ├── index.js                  # Homepage carousel & publications loader
│       ├── project.js                # Project list interactions (Isotope)
│       └── isotope.pkgd.min.js       # Isotope filtering library
│
├── includes/                         # Core library & configuration
│   ├── config.php                    # Database connection & site constants (user-created, not in repository)
│   ├── config-sample.php             # Configuration template (copy to config.php)
│   ├── functions.php                 # Core functions (data access, helpers, i18n)
│   ├── hooks.php                     # Plugin hook system (LabPress_Hooks)
│   ├── header.php                    # Frontend header + navigation
│   ├── footer.php                    # Frontend footer
│   └── labpress.php                  # LabPress core utility class (config, pages, etc.)
│
├── languages/                        # Core language packs
│   ├── zh_CN/
│   │   ├── lang.config               # Language metadata (flag, name)
│   │   ├── common.php                # Shared UI strings (nav, footer, buttons)
│   │   ├── index.php                 # Homepage strings
│   │   ├── news.php                  # News module strings
│   │   ├── projects.php              # Projects module strings
│   │   ├── tools.php                 # Tools module strings
│   │   └── publications.php          # Publications module strings
│   └── en_US/
│       ├── lang.config
│       ├── common.php
│       ├── index.php
│       ├── news.php
│       ├── projects.php
│       ├── tools.php
│       └── publications.php
│
├── plugins/                          # Plugin directory
│   ├── FooterInfo/                   # Footer customization plugin
│   │   └── plugin.php
│   ├── dynamic-multilang/            # Dynamic multi-language translation plugin
│   │   ├── plugin.php                # Plugin main file (hooks & filters)
│   │   ├── admin-page.php            # Admin management page
│   │   ├── languages/
│   │   │   └── zh_CN.php             # Plugin UI language pack
│   │   └── assets/
│   │       ├── dm-admin.css          # Admin panel styles
│   │       └── dm-admin.js           # Admin panel interactions
│   └── hello-word/                   # Hello World example plugin
│       └── plugin.php
│
├── images/                           # Repository images used by demo content and slides
│   ├── slide-1771610110.png
│   ├── slide-1771663308.png
│   ├── slide3.jpg
│   ├── proteintrim-logo.png
│   └── proteintrim-screenshot.jpg
│
├── uploads/                          # User uploaded files
│   ├── news/                         # News images
│   ├── projects/                     # Project images
│   ├── general/                      # General purpose uploads
│   └── gallery/                      # Gallery images (e.g., Gromacs, ComputBio, ComputChem)
│
├── ReadImg/                          # Repository images (logo, screenshots for README)
│   ├── labpress-logo.png
│   └── screenshot-home.png
│
├── index.php                         # Frontend homepage (dynamic content)
├── tools.php                         # Tools list page
├── tool-detail.php                   # Tool detail page
├── projects.php                      # Projects list page (with category filter)
├── project-detail.php                # Project detail page
├── news.php                          # News list page
├── news-detail.php                   # News detail page
├── publications.php                  # Publications list page
├── category.php                      # Category filtered project list
│
├── labpressexample.sql               # Example database dump
│
├── .gitignore                        # Git ignore rules
├── LICENSE                           # Open-source license (GPL-3.0)
└── README.md                         # Project overview & setup guide
```

## Documentation Structure

This documentation is organized into several major sections, each targeting a specific aspect of LabPress usage and development:

```
- Home: index.md
- Getting Started:
  - Guide Overview: getting-started/guide-overview.md
  - Installation Guide: getting-started/installation.md
  - Quick Start: getting-started/quick-start.md
  - Basic Configuration: getting-started/configuration.md
- Setup:
  - Site Settings: setup/site-settings.md
  - Navigation Menus: setup/navigation.md
  - Plugin Management: setup/plugins.md
- Content Management:
  - Publications: content/publications.md
  - Slides: content/slides.md
  - Tools: content/tools.md
  - Projects: content/projects.md
  - News: content/news.md
  - Users: content/users.md
- Internationalization:
  - Overview: i18n/overview.md
  - Core Language Packs: i18n/core-lang.md
  - Dynamic Multi-Language Plugin: i18n/dynamic-multilang.md
- Plugin Development:
  - Plugin System Introduction: plugins/intro.md
  - Developing Your First Plugin: plugins/develop.md
  - Hooks Reference: plugins/hooks-reference.md
  - Plugin Examples: plugins/examples.md
- Reference:
  - API Endpoints: reference/api.md
  - Database Schema: reference/database.md
  - Developer Resources: reference/developer-resources.md
- Contributing: contributing.md
- Security: security.md
- Changelog: changelog.md
```

The documentation is structured to guide you from initial installation through advanced plugin development. Each section is self-contained and can be read independently based on your needs.

## How to Use This Documentation

This documentation is organized to support different personas:

- **New Users** should start with the **Getting Started** section, which walks through installation, initial configuration, and basic content creation.
- **Site Administrators** will find detailed guides in the **Setup** and **Content Management** chapters, covering every configurable aspect of the system.
- **Developers** interested in extending LabPress should head directly to the **Plugin Development** section, which explains the hook system, plugin anatomy, and includes a full hooks reference.
- **Translators** and those managing multilingual sites can explore the **Internationalization** chapter to understand how both static and dynamic translations work.
- **Contributors** should read the **Contribution Guidelines** and the **Security Policy** before submitting patches.

Every page aims to be technically precise, providing code examples, database schema snippets, and configuration samples where appropriate. If you encounter any inconsistencies, please open an issue on the GitHub repository.

## Next Steps

Begin your journey with the [Installation Guide](installation.md) to get LabPress running on your server, or jump directly to the [Plugin Development Introduction](../plugins/intro.md) if you are eager to write your first extension.