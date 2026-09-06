# Contributing

This chapter explains how to contribute to the LabPress project. It covers reporting issues, proposing changes, setting up a development environment, following code conventions, and submitting pull requests.

The goal is to make the contribution process clear and predictable while maintaining the quality and consistency of the project.

## 1. Overview

LabPress is an open-source project released under the GNU General Public License v3.0. Contributions are welcome from anyone, including bug reports, feature suggestions, documentation improvements, and code changes.

The project is hosted on GitHub:

```
https://github.com/GuiHui-huimy/LabPress
```

All contributions should be submitted through GitHub Issues and Pull Requests.

## 2. Code of Conduct

All contributors are expected to follow the project’s Code of Conduct. The full text is available in the `CODE_OF_CONDUCT.md` file in the repository root.

The Code of Conduct establishes basic expectations for respectful communication and professional behavior. Please read it before participating in discussions or submitting contributions.

## 3. Reporting Issues

Issues are used to report bugs, request features, or suggest improvements.

### 3.1 Bug Reports

When reporting a bug, include as much relevant information as possible:

- LabPress version or commit hash.
- PHP version and MySQL version.
- Web server and hosting environment, if relevant.
- Steps to reproduce the problem.
- Expected behavior and actual behavior.
- Any relevant error messages or screenshots.

A clear bug report helps maintainers understand and fix the problem faster.

### 3.2 Feature Requests

Feature requests should describe the problem that the feature would solve rather than only the desired solution. Include:

- The use case or scenario.
- Why the current behavior is insufficient.
- How the proposed change would improve the system.
- Any relevant examples or references.

### 3.3 Security Issues

Do not report security vulnerabilities through public issues. Instead, follow the process described in `SECURITY.md`. Security reports are handled privately until a fix is ready.

## 4. Development Setup

Contributors who want to make code changes should set up a local development environment.

### 4.1 Requirements

Ensure the following are installed:

- PHP 7.4 or higher
- MySQL 5.7+ or MariaDB 10.3+
- PHP extensions: `pdo`, `pdo_mysql`, `json`, `mbstring`, `session`, `fileinfo`, `zip`
- Git

### 4.2 Clone the Repository

```bash
git clone https://github.com/GuiHui-huimy/LabPress.git
cd LabPress
```

### 4.3 Create a Local Database

```bash
mysql -u root -p -e "CREATE DATABASE labpress_dev CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;"
mysql -u root -p labpress_dev < labpressexample.sql
```

### 4.4 Configure the Database Connection

Edit `includes/config.php` and set the database credentials for the local environment.

Keep the local configuration private. Do not commit changes to `config.php` if they contain real credentials.

### 4.5 Start the Local Server

Use the PHP built-in server:

```bash
php -S 127.0.0.1:8080
```

Then open:

```
http://127.0.0.1:8080
```

The admin panel is available at:

```
http://127.0.0.1:8080/admin
```

Default administrator credentials are provided by the sample database. Change them after first login.

## 5. Branching and Workflow

Contributions should be made through feature branches and pull requests.

### 5.1 Fork the Repository

If you do not have write access to the main repository, create a fork on GitHub and clone your fork.

### 5.2 Create a Feature Branch

Use a short, descriptive branch name:

```bash
git checkout -b fix-news-permission
```

Examples:

- `fix-upload-validation`
- `add-plugin-hook`
- `update-readme`

### 5.3 Make Focused Changes

Keep each pull request focused on one change or one logical group of changes. Avoid mixing unrelated modifications in the same branch.

### 5.4 Commit Messages

Use clear and descriptive commit messages. The first line should summarize the change. If necessary, add a blank line followed by a more detailed explanation.

Example:

```
Fix permission check in user management API

The PUT method now verifies that only users with the all
permission can modify other users. Self-service password
changes remain available for all authenticated users.
```

## 6. Code Standards

LabPress is written in plain PHP, JavaScript, and CSS without external frameworks. Contributors should follow the existing style conventions.

### 6.1 PHP

- Use prepared statements for all database queries.
- Escape output with `htmlspecialchars()` where user-visible data is rendered.
- Use the short array syntax `[]`.
- Prefer clear function names and avoid abbreviations unless they are obvious.
- Follow the existing code organization, placing shared functions in `includes/functions.php` and admin views in `admin/views/`.
- Do not add third-party PHP dependencies unless absolutely necessary.

### 6.2 JavaScript

- Use plain JavaScript without framework dependencies.
- Keep admin JavaScript files in `admin/assets/js/` and frontend JavaScript in `assets/js/`.
- Use descriptive variable names and avoid polluting the global namespace unless consistent with existing code.

### 6.3 CSS

- Follow the existing pattern of page-specific stylesheets.
- Use CSS variables where the current theme supports them.
- Avoid inline styles in templates unless the style is generated dynamically.

### 6.4 Language Files

When adding new interface strings:

- Add the key to all existing language files.
- Use descriptive keys and consistent naming.
- For plugin language files, use a unique prefix to avoid conflicts with core keys.

### 6.5 Database Changes

If a change requires a new table or column:

- Update the `labpressexample.sql` file so that new installations include the change.
- Document the change in the relevant documentation page.
- For plugin-specific tables, prefer creating tables during plugin activation rather than modifying the core SQL dump.

## 7. Testing

Before submitting a pull request, perform basic testing.

### 7.1 Syntax Checks

Run PHP syntax checks on all changed files:

```bash
php -l path/to/changed-file.php
```

To check all PHP files:

```bash
find . -name "*.php" -exec php -l {} \;
```

### 7.2 Manual Testing

Manually verify the affected functionality. For example:

- If the change affects login, test login and logout.
- If the change affects content management, test creating and editing the relevant content type.
- If the change affects plugins, test activation, deactivation, and uninstallation.
- If the change affects translations, test both the admin and frontend language contexts.

### 7.3 No Automated Test Suite

The project does not currently have a formal automated test suite. Manual testing is therefore the primary verification method for contributions.

## 8. Documentation

Documentation is an important part of the project. When adding or changing features, update the relevant documentation pages under `docs/`.

Documentation changes can be submitted as pull requests independently of code changes.

When writing documentation:

- Use clear, concise English.
- Follow the existing chapter and section structure.
- Include code examples and screenshots where appropriate.
- Keep technical descriptions accurate and aligned with the actual implementation.

## 9. Pull Requests

Once your changes are ready, submit a pull request to the main repository.

### 9.1 Pull Request Checklist

Before submitting:

- Ensure the branch is up to date with the main branch.
- Run PHP syntax checks.
- Test the change manually.
- Update related documentation if necessary.
- Avoid committing `config.php` with real credentials.
- Do not include unrelated changes in the same pull request.

### 9.2 Pull Request Description

In the pull request description, explain:

- What the change does.
- Why the change is needed.
- How the change was tested.
- Any potential side effects or areas that need review.

### 9.3 Review Process

Maintainers will review the pull request and may request changes. After approval, the changes will be merged into the main branch.

Response times may vary. Keep the discussion professional and focused on the technical content.

## 10. License

By contributing to LabPress, you agree that your contributions will be licensed under the same open-source license as the project. The project is released under the GNU General Public License v3.0.

See the `LICENSE` file for the full license text.

## 11. Next Steps

After reading this guide, you may want to:

- Review the [Developer Resources](developer-resources.md) chapter for more detailed technical context.
- Read the [Plugin Development](../plugins/intro.md) section if you plan to build a plugin.
- Check the [Hooks Reference](../plugins/hooks-reference.md) before adding new hook-related functionality.
- Explore the [API Reference](api.md) and [Database Schema](database.md) for technical details.