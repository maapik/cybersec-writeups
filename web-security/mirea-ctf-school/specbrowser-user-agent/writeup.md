[← Назад к оглавлению](../../../README.md)

# AdminBrowser — Обход фильтрации по User-Agent

| Параметр | Значение |
| :--- | :--- |
| **Платформа** | МИРЭА CTF School |
| **Категория** | Web (HTTP Headers / User-Agent Spoofing) |

## Описание

Закрытая страница отображает заглушку `"To render this page, you need to use adminbrowser"` и требует «специальный браузер».

## Решение

1. Открываем **DevTools (`F12`) → Network** и обновляем страницу: ответ приходит с кодом `200`, сервер проверяет заголовок `User-Agent`.
2. Открываем вкладку **Network conditions** в DevTools, снимаем галочку *Use browser default* в разделе **User agent**.
3. Указываем `Custom` значение `adminbrowser` (или `Mozilla/5.0 (AdminBrowser)`).
4. Обновляем страницу — сервер принимает заголовок и отдаёт контент с флагом.
