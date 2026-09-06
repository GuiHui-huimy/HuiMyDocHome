# Changelog

All notable changes to LabPress will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-08-14

### Added

- Translation function `__()` now supports explicit plugin language loading using the `plugin:` prefix. Plugin authors can call `__('key', 'plugin:plugin-slug')` to load translations directly from a specific plugin, avoiding key collisions with core language packs.

### Changed

- Plugin language files are now loaded only when explicitly requested or during the automatic scan of active plugins. The automatic scan now sanitizes plugin slugs before building file paths.
- Dynamic Multi-Language plugin language keys now use the `ml_` prefix to prevent conflicts with core language keys.

### Fixed

- Fixed an issue where the Dynamic Multi-Language plugin's language keys (such as `plugin_name`, `about_us`, `research_title`) overwrote core language keys, causing incorrect strings to appear in the admin panel.
- Fixed plugin language file loading path when using the explicit `plugin:` module prefix.

### Documentation

- Updated plugin development documentation to include the `plugin:` explicit language loading syntax and key prefix recommendation.

## [1.0.0] - 2026-08-11

### Added

- Initial stable release of LabPress, an open-source academic CMS designed for research laboratories.

#### Core Features

- Dynamic content management – Edit "About Us", research directions, stats, navigation menus directly from the backend.
- Built-in academic modules – Publications, projects, research news, tools, and version management.
- Hook-driven plugin system – Fully extendable via WordPress-like Action/Filter hooks.
- Comprehensive i18n – Front-end and back-end locales are decoupled; dynamic multi-language support for all user-generated content.
- Markdown editor – Integrated EasyMDE with live preview and split-screen editing.
- Lightweight & zero-dependency – Pure PHP + MySQL, no frameworks or Composer required.
- Plugin ecosystem – Install, activate, update, and uninstall plugins directly from the admin panel.

### Installation

- Clone or download the repository.
- Copy `includes/config-sample.php` to `includes/config.php` and fill in database credentials.
- Import `labpressexample.sql` into MySQL.
- Grant write permissions to `uploads/` and `plugins/` directories.
- Access frontend at `/`, backend at `/admin`.

### Important

- The default administrator account included in the example database must be changed immediately after login.
- This release uses GPL v3 license.