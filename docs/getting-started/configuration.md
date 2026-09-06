
This chapter describes the configuration layers of LabPress and explains how system-level settings are stored, accessed, and extended. It is intended for administrators and developers who need to understand the underlying configuration architecture rather than perform step-by-step setup tasks. For operational steps such as creating a database or changing the default password, refer to the [Installation Guide](installation.md) or [Quick Start](quick-start.md).

## 1. Configuration Overview

LabPress separates configuration into two distinct layers:

| Layer                     | Storage                      | Purpose                                                                                      |
| ------------------------- | ---------------------------- | -------------------------------------------------------------------------------------------- |
| Environment configuration | `includes/config.php`        | Database connection, time zone, and PHP-level constants                                      |
| Site configuration        | `site_config` database table | Business settings such as site name, language, homepage content, and plugin-specific options |

This separation keeps sensitive environment details out of the database while allowing site administrators to modify business settings dynamically from the admin panel.

## 2. Environment Configuration

The file `includes/config.php` is the only configuration file required by LabPress. It defines four database constants and sets the default time zone.

```php
<?php
session_start();

define('DB_HOST', 'localhost');
define('DB_NAME', 'labpress');
define('DB_USER', 'labpress_user');
define('DB_PASS', 'your_strong_password');
#PDO 
try {
    $pdo = new PDO(
        "mysql:host=" . DB_HOST . ";dbname=" . DB_NAME . ";charset=utf8mb4",
        DB_USER,
        DB_PASS,
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES => false
        ]
    );
} catch (PDOException $e) {

    die("Database Connect Error: " . $e->getMessage());

}

// TimeZone
date_default_timezone_set('Asia/Shanghai');

```

The PDO connection logic that follows should be treated as part of the core and normally does not need modification.

Because `config.php` contains sensitive credentials, it must not be committed to a public repository. Keep a separate local copy or rely on `.gitignore` to exclude it.

## 3. PHP Runtime Settings

Several PHP configuration directives directly affect LabPress behavior. These settings are usually controlled through `php.ini`, `.user.ini`, or your hosting panel.

| Directive | Recommended Value | Impact |
|-----------|-------------------|--------|
| `upload_max_filesize` | `10M` or higher | Controls the maximum image and plugin ZIP upload size |
| `post_max_size` | `20M` or higher | Must be larger than `upload_max_filesize`; affects form submissions |
| `max_execution_time` | `60` or higher | Long imports or plugin installation may require more time |
| `memory_limit` | `128M` or higher | Prevents memory exhaustion during Markdown rendering or large data operations |
| `display_errors` | `Off` in production | Prevents leakage of file paths and internal errors |
| `log_errors` | `On` | Writes errors to a log for debugging without exposing them to users |
| `file_uploads` | `On` | Required for image and plugin uploads |

The plugin installation page includes a system environment panel that displays the current values for several of these settings. If an upload or installation fails, verify that these values meet the recommendations.

## 4. Site Configuration Storage

All dynamic site configuration is stored in the `site_config` table. Each row contains a `config_key` and a `config_value`. The table uses a primary key on `config_key`, so each setting exists only once.

The core provides two primary functions for working with this table:

- `LabPress::config($key, $default = '')` – retrieves a setting by key, with an optional fallback.
- `LabPress::setConfig($key, $value)` – inserts or updates a setting.

`LabPress::config()` uses a static cache. The first call loads the complete configuration through `getSiteConfig()`, after which subsequent calls return cached values without querying the database again. This improves performance on pages that read many settings.

If you modify a setting directly in the database, the cache must be refreshed by starting a new PHP request. In normal operation, settings are changed through the admin panel, which writes to the database and becomes available on the next page load.

## 5. Core Configuration Keys

The following keys are used by the LabPress core. They are managed through the **Settings > General Settings** page unless otherwise noted.

| Key                 | Description                                                                    |
| ------------------- | ------------------------------------------------------------------------------ |
| `site_name`         | Short site name displayed in the footer and the admin sidebar                  |
| `site_title`        | Full site title used in browser title bars                                     |
| `title_format`      | Format for generating page titles; supports `{page}` and `{site}`              |
| `language`          | Default language locale for the admin panel and fallback for the frontend      |
| `about_us`          | HTML content for the homepage “About Us” section                               |
| `research_title`    | Heading for the research directions section                                    |
| `research_subtitle` | Subheading for the research directions section                                 |
| `research_cards`    | JSON-encoded array of research direction cards                                 |
| `stat_items`        | JSON-encoded array of statistics items, each with `icon`, `label`, and `value` |
| `ml_source_locale`  | Source language locale used by the Dynamic Multi-Language plugin               |

Plugins may create additional keys. For example, the Dynamic Multi-Language plugin stores its source locale in `ml_source_locale` and uses `LabPress::setConfig()` to save it.

## 6. Plugin System Configuration

Plugins are stored as individual directories inside `plugins/`. The `plugins` table tracks the registration status of each plugin:

| Column | Description |
|--------|-------------|
| `slug` | Directory name; must match the folder name exactly |
| `name` | Human-readable plugin name parsed from the plugin header |
| `description` | Short description parsed from the plugin header |
| `version` | Plugin version |
| `author` | Plugin author |
| `status` | `1` for active, `0` for inactive |

When the plugin list page is opened, LabPress can scan the `plugins/` directory and insert any new plugins into the `plugins` table with a status of `0`. This is done manually with the **Scan for new plugins** button.

Active plugins are loaded once during initialization through `loadPlugins()`. Each active plugin’s `plugin.php` file is required, and then the `init` action hook is fired. Plugins use this hook to register filters, actions, and admin pages.

Plugin language files are loaded from the plugin’s `languages/` directory. The core can load plugin translations automatically into the global translation array, or plugins can request explicit loading by calling `__('key', 'plugin:plugin-slug')`.

Detailed plugin development and management are covered in [Plugin Management](../setup/plugins.md) and [Plugin Development](../plugins/intro.md).

## 7. Internationalization Framework

LabPress supports multilingual sites through two independent systems.

The core language packs are stored in `languages/{locale}/`. Each locale directory contains a `lang.config` file with metadata and one or more PHP files that return arrays of translation keys. The `__()` function loads the appropriate module on demand and applies the `translate` filter to the final value.

The Dynamic Multi-Language plugin extends this system to database-stored content. It registers filters on `translate` and other hooks to replace dynamic strings such as navigation titles, slide descriptions, and site configuration fields. Translations are stored in the `ml_translations` table with a unique constraint on `table_name`, `record_id`, `field_name`, and `locale`.

The admin panel always uses the site default language, while the frontend language is selected based on the URL `lang` parameter, a cookie, or the default language setting.

For a full explanation, see the [Internationalization Overview](../i18n/overview.md).

## 8. Frontend Resource Loading

LabPress uses a minimal set of frontend dependencies. Some libraries are bundled locally to avoid external requests, while others may be loaded from CDNs for convenience.

Local resources include:

- EasyMDE editor styles and scripts under `admin/assets/`
- Isotope filtering library under `assets/js/`
- Custom CSS and JavaScript files under `assets/`

The Font Awesome icon library is referenced from a public CDN in several core templates. If your deployment requires a fully offline environment, you should download the Font Awesome assets and update the relevant template links.

The exact resource locations are visible in the page source of the frontend and admin panel. Changing these paths requires editing the corresponding PHP templates directly.

## 9. Next Steps

After reviewing the configuration layers described here, you can proceed to the detailed guides for individual functional areas:

- [Site Settings](../setup/site-settings.md) – complete reference for the general settings page.
- [Navigation Menus](../setup/navigation.md) – configuration of header and footer menus.
- [Plugin Management](../setup/plugins.md) – plugin installation, activation, and lifecycle.
- [Content Management](../content/publications.md) – working with publications, slides, tools, projects, and news.
- [Internationalization](../i18n/overview.md) – in-depth discussion of language packs and dynamic translations.
- [Plugin Development](../plugins/intro.md) – how to build custom plugins and use the hook system.