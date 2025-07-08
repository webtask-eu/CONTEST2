# Изменения в обработке паролей

## Проблема
Пароли с специальными символами (например, `nX76<hv3sMRk`) неправильно обрабатывались из-за чрезмерной санитизации в WordPress.

## Решение

### 1. Замена sanitize_text_field на wp_unslash
**Изменено в файлах:**
- `contest_plugin/contests/public/class-contest-ajax.php`
- `contest_plugin/contests/includes/class-api-handler.php`
- `monitoring_plugin/public/class-monitoring-ajax.php`
- `monitoring_plugin/includes/class-api-handler.php`

**До:**
```php
$password = sanitize_text_field($_POST['password']);
```

**После:**
```php
$password = wp_unslash($_POST['password']);
```

### 2. Изменение отображения паролей в HTML-формах
**Изменено в файлах:**
- `contest_plugin/contests/templates/parts/registration-form.php`
- `contest_plugin/contests/admin/class-admin-pages.php`
- `monitoring_plugin/templates/parts/registration-form.php`
- `monitoring_plugin/admin/class-admin-pages.php`

**Для полей ввода:**
```php
// Экранируем только кавычки для корректной работы HTML
value="<?php echo $is_edit_mode ? str_replace('"', '&quot;', $account->password) : ''; ?>"
```

**Для отображения в таблицах:**
```php
// Экранируем только кавычки, не экранируем другие символы
<?php echo htmlspecialchars($account->password, ENT_NOQUOTES, 'UTF-8'); ?>
```

## Результат

### ✅ Что работает сейчас:
- Пароли с символами `<`, `>`, `&`, `"`, `'`, `[`, `]`, `{`, `}` корректно обрабатываются
- Пароли сохраняются в базе данных без изменений
- Пароли правильно передаются в MT4 API
- Формы редактирования показывают исходные пароли

### ⚠️ Что сохранилось:
- Автоматическое удаление пробелов из паролей (может быть полезно для копирования/вставки)
- Проверка минимальной длины пароля (6 символов)
- Предупреждения о типах паролей (торговый vs инвесторский)

### 🧪 Тестирование
Создан файл `test-password-handling.php` для тестирования различных паролей с специальными символами.

## Пример использования

**Пароль:** `nX76<hv3sMRk`

1. **Ввод в форму:** сохраняется как есть
2. **Сохранение в БД:** `nX76<hv3sMRk` (без изменений)
3. **Передача в API:** корректно URL-кодируется как `nX76%3Chv3sMRk`
4. **Декодирование на сервере:** восстанавливается в `nX76<hv3sMRk`

## Безопасность
- Используется `wp_unslash()` только для удаления слешей, добавленных WordPress
- HTML-вывод по-прежнему безопасен благодаря точечному экранированию
- Подготовленные SQL-запросы предотвращают SQL-инъекции 