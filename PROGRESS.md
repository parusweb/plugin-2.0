# ✅ PROGRESS - Готовые модули

Этот файл содержит **полный код** всех уже созданных файлов.

## 📦 Готовые файлы (6 штук)

### 1️⃣ functions.php (тема)
**Путь:** `parusweb-child-theme/functions.php`

```php
<?php
/**
 * ============================================================================
 * PARUSWEB CHILD THEME - ГЛАВНЫЙ ФАЙЛ FUNCTIONS.PHP
 * ============================================================================
 * 
 * Этот файл служит точкой входа для всех функций темы.
 * Основная логика вынесена в плагин ParusWeb Functions.
 * 
 * @package ParusWeb-Child
 * @version 2.0.0
 */

if (!defined('ABSPATH')) exit;

// ============================================================================
// ПОДКЛЮЧЕНИЕ ПЛАГИНА PARUSWEB FUNCTIONS
// ============================================================================

// Загрузка Briks компонентов
include get_stylesheet_directory() . '/inc/briks-loader.php';

// ============================================================================
// БАЗОВАЯ НАСТРОЙКА ТЕМЫ
// ============================================================================

/**
 * Поддержка WebP изображений
 */
add_filter('mime_types', function($mimes) {
    $mimes['webp'] = 'image/webp';
    return $mimes;
});

/**
 * Удаление префикса "Архивы" из заголовков
 */
add_filter('wpseo_title', function($title) {
    return preg_replace('/^\s*Архивы[:\s\-\—]*/u', '', $title);
}, 10);

/**
 * Замена текста "Subtotal" на "Стоимость"
 */
add_filter('gettext', function($translated, $text, $domain) {
    if ($domain === 'woocommerce') {
        if ($text === 'Subtotal' || $text === 'Подытог') {
            return 'Стоимость';
        }
    }
    return $translated;
}, 10, 3);

// ============================================================================
// ИНТЕГРАЦИЯ С PARUSWEB FUNCTIONS PLUGIN
// ============================================================================

/**
 * Проверка активации плагина ParusWeb Functions
 */
function parusweb_check_plugin() {
    if (!class_exists('ParusWeb_Functions')) {
        add_action('admin_notices', function() {
            ?>
            <div class="notice notice-error">
                <p><strong>ParusWeb Child Theme:</strong> Требуется активация плагина "ParusWeb Functions" для корректной работы темы.</p>
            </div>
            <?php
        });
        return false;
    }
    return true;
}
add_action('after_setup_theme', 'parusweb_check_plugin');

// ============================================================================
// ПОДКЛЮЧЕНИЕ ДОПОЛНИТЕЛЬНЫХ МОДУЛЕЙ ТЕМЫ
// ============================================================================

// Схемы покраски (связано с ACF и калькуляторами)
require_once get_stylesheet_directory() . '/inc/pm-paint-schemes.php';

// Описания покраски
require_once get_stylesheet_directory() . '/inc/paint-description.php';

// ============================================================================
// СОВМЕСТИМОСТЬ С LEGACY КОДОМ
// ============================================================================

/**
 * Эти функции могут использоваться в шаблонах темы
 * Оставлены для обратной совместимости
 */

if (!function_exists('get_price_multiplier')) {
    function get_price_multiplier($product_id) {
        $product_multiplier = get_post_meta($product_id, '_price_multiplier', true);
        if (!empty($product_multiplier) && is_numeric($product_multiplier)) {
            return floatval($product_multiplier);
        }
        
        $product_categories = wp_get_post_terms($product_id, 'product_cat', ['fields' => 'ids']);
        if (!is_wp_error($product_categories) && !empty($product_categories)) {
            foreach ($product_categories as $cat_id) {
                $cat_multiplier = get_term_meta($cat_id, 'category_price_multiplier', true);
                if (!empty($cat_multiplier) && is_numeric($cat_multiplier)) {
                    return floatval($cat_multiplier);
                }
            }
        }
        
        return 1.0;
    }
}

if (!function_exists('extract_area_with_qty')) {
    function extract_area_with_qty($title, $product_id = null) {
        // Реализация в плагине: modules/core/product-calculations.php
        // Здесь для совместимости вызываем функцию из плагина
        if (function_exists('parusweb_extract_area_with_qty')) {
            return parusweb_extract_area_with_qty($title, $product_id);
        }
        return null;
    }
}
```

---

### 2️⃣ parusweb-functions.php (главный файл плагина)
**Путь:** `parusweb-functions/parusweb-functions.php`

```php
<?php
/**
 * Plugin Name: ParusWeb Functions
 * Plugin URI: https://parusweb.ru
 * Description: Модульная система для расширения WooCommerce
 * Version: 2.0.0
 * Author: ParusWeb
 * Author URI: https://parusweb.ru
 * Text Domain: parusweb-functions
 * Domain Path: /languages
 * Requires PHP: 7.4
 * Requires at least: 5.8
 * WC requires at least: 5.0
 * WC tested up to: 9.0
 * 
 * @package ParusWeb_Functions
 */

if (!defined('ABSPATH')) exit;

// ============================================================================
// КОНСТАНТЫ ПЛАГИНА
// ============================================================================

define('PARUSWEB_VERSION', '2.0.0');
define('PARUSWEB_PLUGIN_DIR', plugin_dir_path(__FILE__));
define('PARUSWEB_PLUGIN_URL', plugin_dir_url(__FILE__));
define('PARUSWEB_MODULES_DIR', PARUSWEB_PLUGIN_DIR . 'modules/');

// ============================================================================
// ОСНОВНОЙ КЛАСС ПЛАГИНА
// ============================================================================

class ParusWeb_Functions {
    
    private static $instance = null;
    private $active_modules = [];
    private $available_modules = [];
    
    public static function instance() {
        if (is_null(self::$instance)) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    private function __construct() {
        $this->define_modules();
        $this->load_active_modules();
        $this->init_hooks();
    }
    
    private function init_hooks() {
        add_action('admin_menu', [$this, 'add_admin_menu']);
        add_action('admin_init', [$this, 'register_settings']);
        add_action('admin_enqueue_scripts', [$this, 'enqueue_admin_scripts']);
        add_action('wp_enqueue_scripts', [$this, 'enqueue_frontend_scripts']);
    }
    
    private function define_modules() {
        $this->available_modules = [
            
            // Критические модули
            'core-category-helpers' => [
                'name' => '🔧 Ядро: Проверка категорий',
                'description' => 'Базовые функции проверки категорий товаров (КРИТИЧЕСКИЙ)',
                'file' => 'core/category-helpers.php',
                'dependencies' => [],
                'critical' => true,
                'admin_only' => false,
                'group' => 'core'
            ],
            
            'core-product-calculations' => [
                'name' => '🔧 Ядро: Расчеты товаров',
                'description' => 'Расчет площади, цен, множителей (КРИТИЧЕСКИЙ)',
                'file' => 'core/product-calculations.php',
                'dependencies' => ['core-category-helpers'],
                'critical' => true,
                'admin_only' => false,
                'group' => 'core'
            ],
            
            // Модули отображения
            'display-price-formatting' => [
                'name' => '💰 Отображение: Форматирование цен',
                'description' => 'Форматирование и вывод цен для разных типов товаров',
                'file' => 'display/price-formatting.php',
                'dependencies' => ['core-product-calculations'],
                'admin_only' => false,
                'group' => 'display'
            ],
            
            'display-calculators' => [
                'name' => '🧮 Отображение: Калькуляторы',
                'description' => 'Вывод калькуляторов на странице товара',
                'file' => 'display/calculators.php',
                'dependencies' => ['core-product-calculations'],
                'admin_only' => false,
                'group' => 'display'
            ],
            
            // ... остальные 30+ модулей ...
            // (см. PROJECT-BRIEF.md для полного списка)
        ];
    }
    
    private function load_active_modules() {
        $enabled = get_option('parusweb_enabled_modules', array_keys($this->available_modules));
        
        foreach ($this->available_modules as $id => $module) {
            if (!empty($module['critical'])) {
                $this->load_module($id);
            }
        }
        
        foreach ($enabled as $module_id) {
            if (!isset($this->available_modules[$module_id])) continue;
            if (!empty($this->available_modules[$module_id]['critical'])) continue;
            
            $this->load_module($module_id);
        }
    }
    
    private function load_module($module_id) {
        if (in_array($module_id, $this->active_modules)) return true;
        if (!isset($this->available_modules[$module_id])) return false;
        
        $module = $this->available_modules[$module_id];
        
        if (!$this->check_dependencies($module_id)) return false;
        if ($module['admin_only'] && !is_admin()) return false;
        
        $module_file = PARUSWEB_MODULES_DIR . $module['file'];
        if (file_exists($module_file)) {
            require_once $module_file;
            $this->active_modules[] = $module_id;
            return true;
        }
        
        return false;
    }
    
    private function check_dependencies($module_id) {
        $module = $this->available_modules[$module_id];
        $enabled = get_option('parusweb_enabled_modules', array_keys($this->available_modules));
        
        foreach ($module['dependencies'] as $dependency) {
            if (!empty($this->available_modules[$dependency]['critical'])) continue;
            if (!in_array($dependency, $enabled)) return false;
        }
        
        return true;
    }
    
    public function get_active_modules() {
        return $this->active_modules;
    }
    
    public function add_admin_menu() {
        add_options_page(
            'ParusWeb Модули',
            'ParusWeb Модули',
            'manage_options',
            'parusweb-modules',
            [$this, 'render_admin_page']
        );
    }
    
    public function register_settings() {
        register_setting('parusweb_modules', 'parusweb_enabled_modules');
    }
    
    public function enqueue_admin_scripts($hook) {
        if ($hook !== 'settings_page_parusweb-modules') return;
        
        wp_enqueue_style('parusweb-admin', PARUSWEB_PLUGIN_URL . 'assets/css/admin.css', [], PARUSWEB_VERSION);
        wp_enqueue_script('parusweb-admin', PARUSWEB_PLUGIN_URL . 'assets/js/admin.js', ['jquery'], PARUSWEB_VERSION, true);
    }
    
    public function enqueue_frontend_scripts() {
        if (!is_product() && !is_cart() && !is_checkout()) return;
        
        wp_enqueue_style('parusweb-frontend', PARUSWEB_PLUGIN_URL . 'assets/css/frontend.css', [], PARUSWEB_VERSION);
        wp_enqueue_script('parusweb-frontend', PARUSWEB_PLUGIN_URL . 'assets/js/frontend.js', ['jquery'], PARUSWEB_VERSION, true);
    }
    
    public function render_admin_page() {
        if (!current_user_can('manage_options')) return;
        
        if (isset($_POST['parusweb_save_modules']) && check_admin_referer('parusweb_modules_save')) {
            $enabled = isset($_POST['parusweb_modules']) ? array_map('sanitize_text_field', $_POST['parusweb_modules']) : [];
            
            foreach ($this->available_modules as $id => $module) {
                if (!empty($module['critical']) && !in_array($id, $enabled)) {
                    $enabled[] = $id;
                }
            }
            
            update_option('parusweb_enabled_modules', $enabled);
            echo '<div class="notice notice-success"><p>✓ Настройки сохранены!</p></div>';
        }
        
        include PARUSWEB_PLUGIN_DIR . 'templates/admin-page.php';
    }
}

function parusweb_functions_init() {
    return ParusWeb_Functions::instance();
}

add_action('plugins_loaded', 'parusweb_functions_init');

function parusweb() {
    return ParusWeb_Functions::instance();
}

function parusweb_is_module_active($module_id) {
    return in_array($module_id, parusweb()->get_active_modules());
}
```

---

### 3️⃣ templates/admin-page.php
**Путь:** `parusweb-functions/templates/admin-page.php`

```php
<?php
/**
 * Шаблон страницы управления модулями
 */
if (!defined('ABSPATH')) exit;
?>

<div class="wrap parusweb-modules-page">
    <h1>⚙️ ParusWeb Functions - Управление модулями</h1>
    
    <div class="notice notice-info">
        <p><strong>ℹ️ Информация:</strong></p>
        <ul style="margin: 10px 0;">
            <li>🔧 <strong>Критические модули</strong> не могут быть отключены</li>
            <li>🔗 При отключении модуля автоматически отключаются зависимые</li>
            <li>🔄 После сохранения обновите страницу</li>
        </ul>
    </div>
    
    <form method="post">
        <?php wp_nonce_field('parusweb_modules_save'); ?>
        
        <table class="wp-list-table widefat fixed striped">
            <thead>
                <tr>
                    <th width="40">Вкл.</th>
                    <th width="30%">Модуль</th>
                    <th width="40%">Описание</th>
                    <th width="20%">Зависимости</th>
                    <th width="10%">Статус</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach ($this->available_modules as $module_id => $module): ?>
                    <tr>
                        <td>
                            <input type="checkbox" 
                                   name="parusweb_modules[]" 
                                   value="<?php echo esc_attr($module_id); ?>"
                                   <?php checked(in_array($module_id, $enabled_modules)); ?>
                                   <?php disabled(!empty($module['critical'])); ?>>
                        </td>
                        <td><strong><?php echo esc_html($module['name']); ?></strong></td>
                        <td><?php echo esc_html($module['description']); ?></td>
                        <td>
                            <?php if (!empty($module['dependencies'])): ?>
                                <?php foreach ($module['dependencies'] as $dep): ?>
                                    <span class="dependency-badge"><?php echo esc_html($dep); ?></span>
                                <?php endforeach; ?>
                            <?php else: ?>
                                <span class="no-deps">—</span>
                            <?php endif; ?>
                        </td>
                        <td>
                            <?php if (in_array($module_id, $this->active_modules)): ?>
                                <span style="color:#46b450;">✓ Загружен</span>
                            <?php else: ?>
                                <span style="color:#999;">− Отключен</span>
                            <?php endif; ?>
                        </td>
                    </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
        
        <p class="submit">
            <input type="submit" name="parusweb_save_modules" class="button button-primary" value="💾 Сохранить">
        </p>
    </form>
</div>
```

---

### 4️⃣ modules/core/category-helpers.php (ПОЛНЫЙ КОД - см. предыдущие сообщения)

### 5️⃣ modules/core/product-calculations.php (ПОЛНЫЙ КОД - см. предыдущие сообщения)

### 6️⃣ modules/display/price-formatting.php (ПОЛНЫЙ КОД - см. предыдущие сообщения)

---

## 📊 Статистика

- ✅ **Готово:** 6 файлов
- ⏳ **Осталось:** 33 файла
- 📦 **Строк кода:** ~800 (из ~3500 в оригинале)
- 🎯 **Прогресс:** ~15%

## 🔄 Следующие шаги

Продолжить с модуля: **display-calculators.php**
