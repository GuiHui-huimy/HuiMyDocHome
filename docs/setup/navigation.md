# Navigation Menus

This chapter describes the navigation menu management interface in the LabPress administration panel. It explains how header and footer menus are stored, configured, and rendered, and how they can be extended through the plugin system.

The content is intended for site administrators who need to manage the public navigation structure of a LabPress site. It assumes basic familiarity with the admin panel and the general site configuration described in [Site Settings](site-settings.md).

## 1. Overview

LabPress supports two independent menu locations:

- **Header navigation** – displayed in the main navigation bar on every public page.
- **Footer links** – displayed in the footer area.

Both locations are managed from a single admin page: **Settings > Menu Settings** (or **Settings > Navigation Menus** depending on the current translation).

Menu items are stored in the `nav_menu` table and can be created, edited, reordered, and removed directly from the admin panel. The menu system is designed to be lightweight and does not rely on a separate configuration file.

> **截图占位**：此处可插入 **Settings > Menu Settings** 页面完整截图，展示头部菜单和页脚链接两个区域，建议尺寸 1280×800。

## 2. Database Table Structure

All navigation data is stored in the `nav_menu` table. Each row represents a single menu item.

| Column | Type | Description |
|--------|------|-------------|
| `id` | `int unsigned` | Primary key, auto-increment |
| `title` | `varchar(100)` | Display text of the menu item |
| `url` | `varchar(500)` | Target URL or internal path |
| `target` | `varchar(20)` | Link behavior: `_self` or `_blank` |
| `sort_order` | `int` | Order in which the item appears |
| `is_active` | `tinyint(1)` | Whether the item is visible: `1` for yes, `0` for no |
| `location` | `varchar(20)` | Menu location: `header` or `footer` |
| `parent_id` | `int unsigned` | Optional parent item for hierarchical menus; currently reserved |

The `parent_id` column is present for future support of nested menus. The current implementation does not use this column, and all items are rendered as a flat list.

## 3. Access and Permissions

The menu settings page is available under **Settings > Menu Settings** in the admin sidebar.

Access to this page requires the `all` permission. The permission check is implemented at the beginning of the view file:

```php
if (!hasPermission('all')) {
    echo '<div class="card"><p>' . __('no_permission', 'admin') . '</p></div>';
    return;
}
```

Only users with the `all` permission can add, edit, delete, or reorder menu items. The corresponding API endpoint `api/save.php` enforces the same permission for `nav-menus` operations.

## 4. Admin Interface

The navigation menu management page is divided into two separate tables: one for header menus and one for footer links. Each table provides controls for adding, editing, and deleting items.

### 4.1 Header Navigation

The header menu appears in the main navigation bar at the top of every public page. It is used for primary links such as:

- Home
- Tools
- Projects
- Publications
- Research News
- Contact

Each menu item can be configured with a title, URL, target, sort order, and active status.

### 4.2 Footer Links

The footer links appear in the footer section, typically as a vertical list. They are often used for links to external resources, such as:

- Main laboratory site
- Blog
- Community forum
- Citation page
- Partner organizations

Footer links are managed in the same way as header menus and share the same database table.

### 4.3 Adding a Menu Item

To add a new menu item:

1. Open **Settings > Menu Settings**.
2. Locate the appropriate section, either **Header Navigation** or **Footer Links**.
3. Click **Add Header Menu** or **Add Footer Link**.
4. In the modal form, fill in the following fields:

| Field | Description |
|-------|-------------|
| **Title** | The text displayed for the link. |
| **URL** | The destination URL. For internal pages, use paths such as `/tools.php`, `/projects.php`, `/news.php`, `/publications.php`. For external links, use the full URL, for example `https://example.com`. |
| **Open In** | Select `_self` for the same window or `_blank` for a new window. For external links, `_blank` is usually recommended. |
| **Sort** | A numeric value that controls the display order. Lower values appear first. |
| **Enabled** | Select **Yes** or **No** to control visibility. |

5. Click **Save**.

The new item is inserted into the `nav_menu` table and appears on the frontend after the next page load.

### 4.4 Editing a Menu Item

Each row in the menu tables includes an **Edit** button. Clicking this button opens the same modal form with the current values populated.

The edit form includes a hidden field for the item ID. When saved, the existing record is updated in the database.

### 4.5 Deleting a Menu Item

The **Delete** button in each row removes the item after confirmation. The corresponding API request uses the `delete` action for the `nav-menus` type and removes the record by its ID.

> **截图占位**：此处可插入新增或编辑菜单项弹窗截图，展示 Title、URL、Open In、Sort、Enabled 等字段，建议尺寸 800×600。

## 5. Save and Delete Workflow

The admin menu page interacts with the backend through the unified data save endpoint `api/save.php`.

### 5.1 Saving

When a menu item is added or updated, the JavaScript file `admin/assets/js/nav-menus.js` submits a JSON request with `type` set to `nav-menus` and the form data in the `data` object.

The server handles the `nav-menus` case as follows:

```php
case 'nav-menus':
    if ($action === 'delete') {
        $stmt = $pdo->prepare("DELETE FROM nav_menu WHERE id = ?");
        $stmt->execute([$data['id']]);
    } else {
        $title = $data['title'] ?? '';
        $url = $data['url'] ?? '';
        $target = $data['target'] ?? '_self';
        $sort_order = (int)($data['sort_order'] ?? 0);
        $is_active = (int)($data['is_active'] ?? 1);
        $location = $data['location'] ?? 'header';

        if (!empty($data['id'])) {
            $stmt = $pdo->prepare("UPDATE nav_menu SET title=?, url=?, target=?, sort_order=?, is_active=?, location=? WHERE id=?");
            $stmt->execute([$title, $url, $target, $sort_order, $is_active, $location, $data['id']]);
        } else {
            $stmt = $pdo->prepare("INSERT INTO nav_menu (title, url, target, sort_order, is_active, location) VALUES (?,?,?,?,?,?)");
            $stmt->execute([$title, $url, $target, $sort_order, $is_active, $location]);
        }
    }
    break;
```

If the `id` field is present, the record is updated. If it is empty or omitted, a new record is inserted.

### 5.2 Deleting

The delete operation sends a request with `action` set to `delete` and an object containing the `id` field.

The server executes a `DELETE FROM nav_menu WHERE id = ?` statement. There is no recycle bin or undo mechanism, so deletion is permanent and immediate.

## 6. Frontend Rendering

The public frontend renders navigation menus using the `getNavMenus()` function defined in `includes/functions.php`:

```php
function getNavMenus($location = 'header') {
    global $pdo;
    $stmt = $pdo->prepare("SELECT id, title, url, target FROM nav_menu WHERE is_active = 1 AND location = ? ORDER BY sort_order ASC");
    $stmt->execute([$location]);
    return $stmt->fetchAll();
}
```

Only items with `is_active = 1` are returned. The results are ordered by `sort_order` in ascending order.

### 6.1 Header Rendering

The header template `includes/header.php` loads the header menu items and renders them as an unordered list. Each item is wrapped in a `<li>` element and includes the title and URL. The `target` attribute is added when the link is configured to open in a new window.

Before output, each menu item is passed through the `nav_menu_item` filter. This allows plugins such as Dynamic Multi-Language to replace the displayed title with a translated version without modifying the template.

### 6.2 Footer Rendering

The footer template `includes/footer.php` loads the footer menu items using the same function and renders them as a simple list of links. The same `nav_menu_item` filter is applied to footer items.

## 7. Multilingual Support

Menu titles are stored in the original site language in the database. The Dynamic Multi-Language plugin provides a translation interface for these titles.

The translation is implemented through the `nav_menu_item` filter. The plugin registers a filter that:

1. Reads the original menu item ID.
2. Queries the `ml_translations` table for a translation matching the current locale.
3. If a translation is found, the link HTML is regenerated with the translated title.

Translations for header and footer menus can be managed in the Dynamic Multi-Language admin page under the **Nav Menus** tab.

For more details, see [Dynamic Multi-Language Plugin](../i18n/dynamic-multilang.md).

## 8. Hooks and Extensibility

The navigation menu system exposes several hooks that allow plugins to modify behavior.

| Hook | Type | Description |
|------|------|-------------|
| `nav_menu_item` | Filter | Applied to each rendered menu item HTML. Receives the HTML string and the raw menu item array. |
| `before_nav` | Action | Triggered before the main navigation menu is rendered. |
| `after_nav` | Action | Triggered after the main navigation menu is rendered. |
| `admin_prepare_nav-menus` | Filter | Applied to menu data before it is saved. |
| `admin_after_nav-menus_save` | Action | Triggered after a menu item is saved. |

Plugin developers can use these hooks to add custom menu classes, insert additional menu items, or modify the menu data before it is stored.

For a complete reference of available hooks, see [Hooks Reference](../plugins/hooks-reference.md).

## 9. Example: Adding an External Link

The following example shows how to add an external publication link to the footer.

1. Open **Settings > Menu Settings**.
2. In the **Footer Links** section, click **Add Footer Link**.
3. Fill in the form:
   - **Title**: `PubMed`
   - **URL**: `https://pubmed.ncbi.nlm.nih.gov`
   - **Open In**: `New window`
   - **Sort**: `6`
   - **Enabled**: `Yes`
4. Click **Save**.

The new link will appear in the footer on all public pages. If the Dynamic Multi-Language plugin is active, you can optionally provide an English or Chinese translation for the title.

## 10. Next Steps

After configuring the navigation menus, you may want to continue with:

- [Plugin Management](plugins.md) – activate or install additional plugins.
- [Site Settings](site-settings.md) – manage general site identity and homepage content.
- [Content Management](../content/publications.md) – create publications, slides, tools, projects, and news.
- [Internationalization](../i18n/overview.md) – manage multilingual support for menus and content.
- [Hooks Reference](../plugins/hooks-reference.md) – learn about available hooks for menu customization.