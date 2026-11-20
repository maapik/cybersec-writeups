[← Назад к оглавлению](../../README.md)

# PhishNet — Криминалистический анализ фишингового письма (`.eml`)

| Параметр | Значение |
| :--- | :--- |
| **Платформа** | Hack The Box (Sherlocks) |
| **Категория** | Email Forensics / Phishing Analysis |
| **Инструменты** | Linux CLI (`less`, `base64`, `sha256sum`, `7z`), MITRE ATT&CK |

## Executive Summary

Бухгалтерия получила срочный запрос на оплату от якобы известного поставщика. Письмо содержит подозрительную ссылку и ZIP-вложение с вредоносным скриптом с двойным расширением, закодированное в Base64 внутри `.eml`.

Поскольку `.eml` является обычным текстовым файлом, первичный разбор заголовков и тела удобно провести напрямую в Linux CLI (`less`).

---

## Вопросы и разбор артефактов

Начальные заголовки письма:

<img width="1082" height="556" alt="Заголовки .eml письма" src="https://github.com/user-attachments/assets/b45b1abd-d2fa-4560-8387-b0f2d3bb2a5c" />

1. **IP-адрес отправителя:** `45.67.89.10`
2. **Почтовый сервер-ретранслятор (Relay):** `203.0.113.25`
3. **Email отправителя (`From`):** `finance@business-finance.com`
4. **Адрес для ответа (`Reply-To`):** `support@business-finance.com`
5. **Результат проверки SPF:** `pass`

---

Основной текст письма:

<img width="1095" height="542" alt="Тело письма" src="https://github.com/user-attachments/assets/bc9fc806-004e-4bae-b045-a614f01057ad" />

6. **Домен в фишинговой ссылке:** `secure.business-finance.com`
7. **Название компании в письме:** `Business Finance Ltd.`
8. **Имя вложения:** `Invoice_2025_Payment.zip`

---

### Вопрос 9
**Какой SHA-256 хэш у вложения?**

Извлекаем Base64-блок вложения и считаем хэш без сохранения исполняемого кода на диск:
```bash
echo UEsDBBQAAAAIABh/WloXPY4qcxITALvMGQAYAAAAaW52b2ljZV9kb2N1bWVudC5wZGYuYmF0zL3ZzuzIsR18LQN+h62DPujWX0e7 | base64 -d | sha256sum
```

**Ответ:** `8379C41239E9AF845B2AB6C27A7509AE8804D7D73E455C800A551B22BA25BB4A`

---

### Вопрос 10
**Как называется вредоносный файл внутри ZIP-архива?**

Находим закодированное вложение в теле `.eml`:

<img width="965" height="135" alt="Base64 блок вложения" src="https://github.com/user-attachments/assets/3608274f-ab55-4ab8-b6fb-86ec3e9ebb53" />

Декодируем его в архив:
```bash
echo UEsDBBQAAAAIABh/WloXPY4qcxITALvMGQAYAAAAaW52b2ljZV9kb2N1bWVudC5wZGYuYmF0zL3ZzuzIsR18LQN+h62DPujWX0e7 | base64 -d > "Invoice.zip"
```

При инспекции архива через `7zip` виден файл с двойным расширением, маскирующийся под PDF:

<img width="623" height="63" alt="Проверка архива 7zip" src="https://github.com/user-attachments/assets/a1351f44-cc91-4080-95c9-08f976f77d47" />

<img width="746" height="423" alt="Содержимое архива" src="https://github.com/user-attachments/assets/96ee89e6-4e94-4faa-a199-772e3e5ccf9c" />

**Ответ:** `invoice_document.pdf.bat`

---

### Вопрос 11
**Какая техника MITRE ATT&CK связана с этой атакой?**

<img width="1289" height="528" alt="Техника MITRE ATT&CK" src="https://github.com/user-attachments/assets/45c8abea-bee1-41d6-8c63-fe9557584485" />

**Ответ:** `T1566.001` *(Phishing: Spearphishing Attachment)*

---

## Индикаторы компрометации (IOCs) и Kill Chain

- **Имя файла:** `invoice_document.pdf.bat`
- **SHA-256:** `8379C41239E9AF845B2AB6C27A7509AE8804D7D73E455C800A551B22BA25BB4A`

1. **Delivery:** отправка письма с подменёнными заголовками и ZIP-вложением.
2. **User Execution:** пользователь распаковывает архив и запускает файл-приманку, обманувшись двойным расширением `.pdf.bat`.
3. **Execution:** `.bat`-скрипт выполняется в контексте ОС жертвы.
