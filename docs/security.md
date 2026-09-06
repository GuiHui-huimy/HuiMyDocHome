# Security

This chapter describes the security measures implemented in LabPress and provides recommendations for running a LabPress site securely in production. It also explains how to report security vulnerabilities.

The information is intended for site administrators and developers. It covers the built-in protections, deployment considerations, and the process for responsible disclosure.

## 1. Overview

LabPress includes several security controls that are designed to protect the application, its data, and its users during normal operation. These controls cover authentication, authorization, database access, output escaping, file uploads, and plugin management.

While LabPress includes these protections, no web application can be considered completely secure without a properly configured hosting environment. Administrators should follow the deployment recommendations in this chapter and keep the system up to date.

## 2. Built-In Security Measures

### 2.1 Password Storage

User passwords are never stored in plain text. LabPress uses PHP’s `password_hash()` function with the bcrypt algorithm to generate password hashes.

During login, the system verifies the supplied password with `password_verify()`. This ensures that passwords are stored in a format that is resistant to brute-force and precomputation attacks if the database is compromised.

### 2.2 Session-Based Authentication

Access to the admin panel is protected by PHP sessions. After a successful login, the user’s identity and permissions are stored in session variables.

All admin entry points verify that a valid session exists before allowing access. If the session is missing or invalid, the user is redirected to the login page or the API returns an authentication error.

The admin login process also supports a `login_redirect_url` filter, but authentication itself is enforced before any admin view is rendered.

### 2.3 Permission System

LabPress uses a role-based permission model for backend operations.

Each administrator account has a `permissions` field stored as a JSON array. Permissions are checked before:

- Rendering admin pages.
- Executing write operations through the API.
- Performing sensitive actions such as plugin management, user management, and site configuration.

The `hasPermission()` function enforces these checks. Users with the `all` permission have full access. Other permissions allow only the corresponding module access.

### 2.4 Database Security

All database queries use PDO prepared statements with parameter binding. This prevents SQL injection by separating SQL logic from user-supplied values.

The database connection is configured with:

- `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`
- `PDO::ATTR_EMULATE_PREPARES => false`

These settings ensure that errors are handled consistently and that prepared statements are not emulated in a way that could weaken their safety.

### 2.5 Output Escaping

User-generated content and data retrieved from the database are escaped before being rendered in HTML output.

The templates use `htmlspecialchars()` to escape dynamic values. This reduces the risk of cross-site scripting (XSS) when displaying content such as news titles, project names, menu items, and configuration values.

Markdown rendering is performed by the bundled Parsedown library, and the resulting HTML is filtered before it is passed to the template layer.

### 2.6 File Upload Restrictions

The image upload endpoint `api/upload.php` accepts only a defined set of image MIME types:

- `image/jpeg`
- `image/png`
- `image/gif`
- `image/webp`

Uploaded files are saved in the `uploads/` directory under a subdirectory determined by the content type. The upload operation is restricted to authenticated users.

Plugin ZIP uploads are restricted to users with the `all` permission, and the installation process checks for the presence of a `plugin.php` file inside the uploaded archive.

### 2.7 Plugin Management Controls

Plugin management actions are restricted to users with the `all` permission. These actions include:

- Plugin installation from ZIP
- Plugin installation from a remote URL
- Plugin activation and deactivation
- Plugin uninstallation

The plugin uninstall process validates the plugin slug using a strict pattern that only permits letters, numbers, underscores, and hyphens. This prevents path traversal through the plugin identifier.

### 2.8 Configuration Protection

The database configuration file `includes/config.php` contains sensitive credentials. It is not intended to be committed to version control.

The repository includes a `.gitignore` rule that excludes `config.php`. Administrators should verify that this file is not exposed publicly and that the web server is configured to deny direct access to PHP files inside the `includes/` directory.

## 3. Production Deployment Recommendations

### 3.1 Use HTTPS

Enable HTTPS on the production server. HTTPS encrypts traffic between the browser and the server, protecting login credentials and session cookies from interception.

Most hosting panels, including BaoTa, provide tools for obtaining and installing free SSL certificates.

### 3.2 Change Default Credentials

The sample database includes a default administrator account. The credentials are publicly documented and must be changed immediately after installation.

Administrators should create strong, unique passwords and avoid reusing passwords across systems.

### 3.3 Restrict Access to Sensitive Directories

The following directories should not be publicly accessible through the web server:

- `includes/`
- `plugins/` – except through the normal PHP loading process
- `api/` – if direct access is not required, restrict it to application-level routing

The web server should be configured to deny direct access to PHP files in these directories when appropriate.

### 3.4 Set Correct File Permissions

Directories that require write access from the PHP process should be configured with the minimum necessary permissions.

Typical recommended permissions:

```bash
chmod -R 755 uploads/
chmod -R 755 plugins/
chmod -R 755 images/
```

If the web server runs under a separate user such as `www-data`, ownership should be adjusted accordingly.

### 3.5 Keep the System Updated

Check the GitHub repository regularly for new releases and security updates. Apply updates in a timely manner.

Before performing an update, back up the database and the web directory. If the update requires manual file replacement, follow the release notes carefully.

### 3.6 Backup Regularly

Maintain regular backups of both the database and the `uploads/` directory. Backups should be stored in a secure location outside the web root.

The most important data is stored in the MySQL database. Use the backup tools provided by your hosting panel or automated scripts to create periodic database dumps.

### 3.7 Use Trusted Plugins Only

Plugins can execute PHP code and therefore have full access to the server environment. Only install plugins from trusted sources.

Before activating a new plugin, review its code or ensure that it comes from a repository or author you trust.

## 4. Reporting a Vulnerability

If you discover a security vulnerability in LabPress, please report it privately. Do not open a public issue or disclose the details until the maintainers have had a chance to address the problem.

The full reporting process is described in the `SECURITY.md` file in the repository root.

When reporting a vulnerability, include:

- A clear description of the vulnerability.
- The affected version or commit.
- Steps to reproduce the issue.
- The potential impact.
- Any suggested remediation, if available.

The maintainers will acknowledge the report and work with the reporter to validate and address the issue.

## 5. Known Security Boundaries

LabPress is designed for use by research groups and small laboratory teams. It is not intended to be exposed to untrusted, anonymous user registration or content submission.

The admin panel is restricted to trusted administrators. The public frontend is primarily read-oriented and does not allow public user registration or comment submission by default.

When deploying LabPress in a shared or public environment, treat the admin panel as the primary security boundary and restrict access to it using additional controls such as IP allowlists or VPN access if necessary.

## 6. Next Steps

After reviewing the security documentation, you may want to:

- Read the [Installation Guide](../getting-started/installation.md) for secure installation steps.
- Review the [Basic Configuration](../getting-started/configuration.md) for environment settings.
- Check the [Users Management](../content/users.md) guide for account and permission best practices.
- Review the [Plugin Management](../setup/plugins.md) guide for plugin safety.
- Consult the [Developer Resources](../reference/developer-resources.md) if you are working with the codebase.