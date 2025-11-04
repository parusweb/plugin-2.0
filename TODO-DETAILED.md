# ⏳ TODO - Детальный план оставшихся модулей

## 📋 Общая статистика

- ✅ Готово: **6 модулей** (15%)
- ⏳ Осталось: **33 модуля** (85%)
- 📦 Общий объем: **39 модулей**

---

## 🎯 ПРИОРИТЕТ 1: Модули отображения (3 модуля)

### 7️⃣ display/calculators.php
**Зависит от:** core-product-calculations

**Что переносить из functions.php:**
- Строки 400-1100: Блок `add_action('wp_footer', function() { ... })`
- Вывод HTML калькуляторов (площадь, размеры, множитель, погонные метры, фальшбалки)
- Блоки: `#calc-area`, `#calc-dim`, `#calc-multiplier`, `#calc-running-meter`, `#calc-square-meter`

**Структура модуля:**
```php
// БЛОК 1: Калькулятор площади
add_action('wp_footer', 'parusweb_render_area_calculator');

// БЛОК 2: Калькулятор размеров (старый)
add_action('wp_footer', 'parusweb_render_dimensions_calculator');

// БЛОК 3: Калькулятор множителя
add_action('wp_footer', 'parusweb_render_multiplier_calculator');

// БЛОК 4: Калькулятор погонных метров
add_action('wp_footer', 'parusweb_render_running_meter_calculator');

// БЛОК 5: Калькулятор квадратных метров
add_action('wp_footer', 'parusweb_render_square_meter_calculator');

// БЛОК 6: Калькулятор фальшбалок
add_action('wp_footer', 'parusweb_render_falsebalk_calculator');

// БЛОК 7: Калькулятор реечных перегородок
add_action('wp_footer', 'parusweb_render_partition_slat_calculator');
```

**Что убрать:**
- Все `console.log()`
- Комментарии `// ВАЖНО:`, `// КРИТИЧНО:`
- Повторяющиеся проверки `if (!is_product()) return;`

---

### 8️⃣ display/product-info.php
**Зависит от:** core-category-helpers

**Что переносить:**
- Строки 50-100: Вывод информации о площади упаковки
- Строки 150-200: Бейджи и метки товаров
- Блоки показа единиц измерения

**Структура:**
```php
// БЛОК 1: Информация о площади
add_action('woocommerce_before_add_to_cart_button', 'parusweb_display_area_info');

// БЛОК 2: Бейджи типа товара
add_action('woocommerce_after_shop_loop_item_title', 'parusweb_display_product_badges');

// БЛОК 3: Единицы измерения
add_filter('woocommerce_product_add_to_cart_text', 'parusweb_modify_add_to_cart_text');
```

---

### 9️⃣ display/non-cash-price.php
**Зависит от:** нет

**Что переносить:**
- Строки 3400-3450: Блок `add_action('wp_footer', 'add_non_cash_price_js')`
- JavaScript вывода цены с наценкой 10%

**Структура:**
```php
// БЛОК 1: Вывод цены по безналу
add_action('wp_footer', 'parusweb_render_non_cash_price', 999);

function parusweb_render_non_cash_price() {
    if (!is_product()) return;
    // JavaScript код увеличения цены на 10%
}
```

---

## 🛒 ПРИОРИТЕТ 2: Модули корзины (2 модуля)

### 🔟 cart/cart-functionality.php
**Зависит от:** core-product-calculations

**Что переносить:**
- Строки 1200-1400: `add_filter('woocommerce_add_cart_item_data')`
- Строки 1500-1600: `add_filter('woocommerce_add_to_cart_quantity')`
- Строки 1650-1700: `add_action('woocommerce_add_to_cart')`
- Логика добавления данных калькуляторов в корзину

**Структура:**
```php
// БЛОК 1: Добавление данных калькуляторов в корзину
add_filter('woocommerce_add_cart_item_data', 'parusweb_add_calculator_data_to_cart', 10, 3);

// БЛОК 2: Установка правильного количества
add_filter('woocommerce_add_to_cart_quantity', 'parusweb_adjust_cart_quantity', 10, 2);

// БЛОК 3: Корректировка после добавления
add_action('woocommerce_add_to_cart', 'parusweb_correct_cart_quantity', 10, 6);

// БЛОК 4: JavaScript для карточек товаров
add_action('wp_footer', 'parusweb_card_purchase_script');
```

---

### 1️⃣1️⃣ cart/cart-display.php
**Зависит от:** cart-functionality

**Что переносить:**
- Строки 1420-1550: `add_filter('woocommerce_get_item_data')`
- Строки 1800-1900: Форматирование цен в корзине
- Строки 1950-2050: Отображение метаданных товаров

**Структура:**
```php
// БЛОК 1: Отображение данных калькулятора в корзине
add_filter('woocommerce_get_item_data', 'parusweb_display_calculator_data_in_cart', 10, 2);

// БЛОК 2: Форматирование цены в корзине
add_filter('woocommerce_cart_item_price', 'parusweb_format_cart_item_price', 10, 3);

// БЛОК 3: Форматирование итоговой суммы
add_filter('woocommerce_cart_item_subtotal', 'parusweb_format_cart_item_subtotal', 10, 3);

// БЛОК 4: Мини-корзина
add_filter('woocommerce_widget_cart_item_quantity', 'parusweb_format_mini_cart_quantity', 10, 3);

// БЛОК 5: Удаление цен из названий услуг
add_filter('woocommerce_get_item_data', 'parusweb_remove_price_from_service_name', 15, 2);
```

---

## 📦 ПРИОРИТЕТ 3: Модули заказов (1 модуль)

### 1️⃣2️⃣ orders/order-processing.php
**Зависит от:** cart-functionality

**Что переносить:**
- Строки 1720-1850: `add_action('woocommerce_checkout_create_order_line_item')`
- Строки 2100-2200: `add_action('woocommerce_checkout_update_order_meta')`
- Строки 2250-2350: `add_action('woocommerce_admin_order_data_after_shipping_address')`

**Структура:**
```php
// БЛОК 1: Сохранение данных калькулятора в заказ
add_action('woocommerce_checkout_create_order_line_item', 'parusweb_save_calculator_to_order', 10, 4);

// БЛОК 2: Сохранение услуг покраски
add_action('woocommerce_checkout_create_order_line_item', 'parusweb_save_painting_to_order', 10, 4);

// БЛОК 3: Сохранение метаданных заказа
add_action('woocommerce_checkout_update_order_meta', 'parusweb_save_order_meta');

// БЛОК 4: Отображение в админке заказа
add_action('woocommerce_admin_order_data_after_shipping_address', 'parusweb_display_order_calculator_data');

// БЛОК 5: Форматирование метаданных
add_filter('woocommerce_order_item_display_meta_key', 'parusweb_format_order_meta_key', 10, 3);
add_filter('woocommerce_order_item_display_meta_value', 'parusweb_format_order_meta_value', 10, 3);
```

---

## ⚙️ ПРИОРИТЕТ 4: Модули админки (4 модуля)

### 1️⃣3️⃣ admin/product-meta.php
**Только для админки**

**Что переносить:**
- Строки 180-280: `add_action('woocommerce_product_options_pricing')` - Множитель цены
- Строки 300-450: Настройки калькулятора размеров
- Строки 480-550: `add_action('woocommerce_process_product_meta')`

**Структура:**
```php
// БЛОК 1: Поля множителя цены
add_action('woocommerce_product_options_pricing', 'parusweb_add_price_multiplier_field');

// БЛОК 2: Поля настроек калькулятора
add_action('woocommerce_product_options_general_product_data', 'parusweb_add_calculator_settings');

// БЛОК 3: Сохранение метаполей
add_action('woocommerce_process_product_meta', 'parusweb_save_product_meta');
```

---

### 1️⃣4️⃣ admin/category-meta.php
**Только для админки**

**Что переносить:**
- Строки 560-620: `add_action('product_cat_add_form_fields')` - Множитель категории
- Строки 630-680: `add_action('product_cat_edit_form_fields')`
- Строки 690-750: Сохранение метаполей категории
- Строки 3200-3350: Поля фасок для категорий

**Структура:**
```php
// БЛОК 1: Множитель цены для категории
add_action('product_cat_add_form_fields', 'parusweb_add_category_multiplier_field');
add_action('product_cat_edit_form_fields', 'parusweb_edit_category_multiplier_field', 10, 2);

// БЛОК 2: Поля типов фасок
add_action('product_cat_edit_form_fields', 'parusweb_add_faska_fields', 10, 2);

// БЛОК 3: Сохранение
add_action('created_product_cat', 'parusweb_save_category_meta');
add_action('edited_product_cat', 'parusweb_save_category_meta', 10, 2);
```

---

### 1️⃣5️⃣ admin/falsebalk-meta.php
**Только для админки**

**Что переносить:**
- Строки 3450-3700: `add_action('woocommerce_product_options_general_product_data', 'add_falsebalk_shapes_fields')`
- Настройка форм фальшбалок (Г, П, О)
- Сохранение данных форм

**Структура:**
```php
// БЛОК 1: Метабокс форм фальшбалок
add_action('woocommerce_product_options_general_product_data', 'parusweb_add_falsebalk_shapes_fields');

// БЛОК 2: Сохранение форм
add_action('woocommerce_process_product_meta', 'parusweb_save_falsebalk_shapes_fields');
```

---

### 1️⃣6️⃣ admin/shtaketnik-meta.php
**Только для админки**

**Что переносить:**
- Строки 3760-3850: Метаполя цен для форм верха штакетника
- `_shape_price_round`, `_shape_price_triangle`, `_shape_price_flat`

**Структура:**
```php
// БЛОК 1: Поля цен форм верха
add_action('woocommerce_product_options_pricing', 'parusweb_add_shtaketnik_shape_prices');

// БЛОК 2: Сохранение
add_action('woocommerce_process_product_meta', 'parusweb_save_shtaketnik_shape_prices');
```

---

## ⭐ ПРИОРИТЕТ 5: Специализированные модули (4 модуля)

### 1️⃣7️⃣ features/painting-services.php
**Зависит от:** core-category-helpers

**Что переносить:**
- Строки 1900-2100: ACF поля услуг покраски
- Строки 2150-2250: Функция `get_available_painting_services_by_material()`
- Строки 2300-2400: Интеграция с калькуляторами

**Структура:**
```php
// БЛОК 1: Регистрация ACF полей
add_action('acf/init', 'parusweb_register_painting_acf_fields');

// БЛОК 2: Страница настроек
add_action('acf/init', 'parusweb_create_painting_options_page');

// БЛОК 3: Получение услуг
function parusweb_get_painting_services($product_id) { }

// БЛОК 4: Предзаполнение услуг по умолчанию
function parusweb_populate_default_painting_services() { }
```

---

### 1️⃣8️⃣ features/liter-products.php
**Зависит от:** core-category-helpers

**Что переносить:**
- Строки 2800-2900: Функция выбора объема (тары) для ЛКМ
- Строки 2950-3050: Добавление в корзину с объемом
- Строки 3100-3200: Пересчет цены × объем со скидкой

**Структура:**
```php
// БЛОК 1: Вывод селекта объема
add_action('woocommerce_before_add_to_cart_button', 'parusweb_render_tara_select');

// БЛОК 2: Добавление объема в корзину
add_filter('woocommerce_add_cart_item_data', 'parusweb_add_tara_to_cart', 10, 3);

// БЛОК 3: Отображение в корзине
add_filter('woocommerce_get_item_data', 'parusweb_display_tara_in_cart', 10, 2);

// БЛОК 4: Пересчет цены
add_action('woocommerce_before_calculate_totals', 'parusweb_recalculate_tara_price');

// БЛОК 5: JavaScript
add_action('wp_footer', 'parusweb_tara_update_script');
```

---

### 1️⃣9️⃣ features/delivery-calculator.php
**Зависит от:** нет

**Что переносить:**
- Строки 2400-2800: Калькулятор доставки с Яндекс.Картами
- AJAX handlers для расчета стоимости
- Метод доставки WooCommerce

**Структура:**
```php
// БЛОК 1: Подключение скриптов
add_action('wp_enqueue_scripts', 'parusweb_enqueue_delivery_scripts');

// БЛОК 2: Вывод калькулятора
add_action('woocommerce_before_checkout_billing_form', 'parusweb_render_delivery_calculator');

// БЛОК 3: AJAX обработчики
add_action('wp_ajax_set_delivery_cost', 'parusweb_set_delivery_cost');
add_action('wp_ajax_nopriv_set_delivery_cost', 'parusweb_set_delivery_cost');

// БЛОК 4: Метод доставки
add_action('woocommerce_shipping_init', 'parusweb_init_delivery_method');
add_filter('woocommerce_shipping_methods', 'parusweb_add_delivery_method');

// БЛОК 5: Сохранение в заказ
add_action('woocommerce_checkout_update_order_meta', 'parusweb_save_delivery_info');
add_action('woocommerce_admin_order_data_after_shipping_address', 'parusweb_display_delivery_info');
```

---

### 2️⃣0️⃣ features/non-cash-price.php
(Уже описан в пункте 9)

---

## 🔌 ПРИОРИТЕТ 6: Интеграции (3 модуля)

### 2️⃣1️⃣ integrations/acf-fields.php
**Что переносить:**
- Все блоки регистрации ACF полей (если не входят в другие модули)

### 2️⃣2️⃣ integrations/facet-filters.php
**Что переносить:**
- Строки 3250-3320: Замена текста в FacetWP
- Строки 3350-3420: Заголовки фильтров

### 2️⃣3️⃣ integrations/mega-menu-attributes.php
**Что переносить:**
- Строки 3100-3200: Атрибуты товаров в мега-меню

---

## 👤 ПРИОРИТЕТ 7: Личный кабинет (3 модуля)

### 2️⃣4️⃣ account/account-customization.php
**Что переносить:**
- Кастомизация меню ЛК
- Переименование пунктов
- Плитки на дашборде

### 2️⃣5️⃣ account/company-fields.php
**Что переносить:**
- Поля юрлиц и ИП
- Checkout поля
- Регистрация

### 2️⃣6️⃣ account/inn-lookup.php
**Что переносить:**
- AJAX обработчик DaData
- JavaScript автозаполнения

---

## 🔧 ПРИОРИТЕТ 8: Утилиты (3 модуля)

### 2️⃣7️⃣ utilities/ajax-handlers.php
### 2️⃣8️⃣ utilities/shortcodes.php
### 2️⃣9️⃣ utilities/misc-functions.php

---

## 📜 ПРИОРИТЕТ 9: JavaScript (2 модуля)

### 3️⃣0️⃣ scripts/calculator-scripts.php
### 3️⃣1️⃣ scripts/price-update.php

---

## 🎨 ПРИОРИТЕТ 10: CSS и Assets (8 файлов)

### 3️⃣2️⃣ assets/css/admin.css
### 3️⃣3️⃣ assets/css/frontend.css
### 3️⃣4️⃣ assets/js/admin.js
### 3️⃣5️⃣ assets/js/frontend.js
### 3️⃣6️⃣ assets/js/calculator.js
### 3️⃣7️⃣ assets/js/delivery-calc.js
### 3️⃣8️⃣ README.md
### 3️⃣9️⃣ languages/parusweb-functions.pot

---

## 🎯 Рекомендуемый порядок работы

1. ✅ **display/calculators.php** - самый объемный, но критичный
2. ✅ **cart/cart-functionality.php** - важен для работы калькуляторов
3. ✅ **cart/cart-display.php** - отображение данных
4. ✅ **orders/order-processing.php** - сохранение заказов
5. ✅ **admin/product-meta.php** - метаполя товаров
6. ✅ **features/painting-services.php** - популярная функция
7. ✅ **scripts/calculator-scripts.php** - JavaScript логика
8. ✅ **scripts/price-update.php** - автообновление цен

Остальные модули - по мере необходимости.

---

## 💡 Подсказки при создании модулей

### Шаблон нового модуля:
```php
<?php
/**
 * ============================================================================
 * МОДУЛЬ: [НАЗВАНИЕ]
 * ============================================================================
 * 
 * [Описание функционала]
 * 
 * @package ParusWeb_Functions
 * @subpackage [Группа]
 * @version 2.0.0
 */

if (!defined('ABSPATH')) exit;

// ============================================================================
// БЛОК 1: [НАЗВАНИЕ БЛОКА]
// ============================================================================

// Код...

// ============================================================================
// БЛОК 2: [НАЗВАНИЕ БЛОКА]
// ============================================================================

// Код...
```

### Что убирать:
- ❌ `console.log()`, `error_log()` (кроме критичных ошибок)
- ❌ Комментарии `// ВАЖНО:`, `// TODO:`, `//паттерн для...`
- ❌ Лишние пустые строки (оставить max 1)
- ❌ Закомментированный код
- ❌ Повторяющиеся проверки

### Что оставлять:
- ✅ Заголовки блоков с `// ===`
- ✅ Короткие пояснения сложной логики
- ✅ Описания параметров функций (если не очевидно)

---

**Следующий модуль для работы:** display/calculators.php
