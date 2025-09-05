<!DOCTYPE html>
<html lang="ru">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes, maximum-scale=3.0">
    <title>Бюджетный калькулятор</title>
    
    <!-- PWA Meta Tags -->
    <meta name="theme-color" content="#4caf50">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <meta name="apple-mobile-web-app-title" content="Бюджет">
    <meta name="description" content="Удобный калькулятор для управления личным бюджетом и отслеживания трат">
    
    <!-- Icons -->
    <link rel="icon" type="image/png" sizes="32x32" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>💰</text></svg>">
    <link rel="apple-touch-icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>💰</text></svg>">
    
    <!-- Manifest -->
    <link rel="manifest" href="data:application/manifest+json,{
        &quot;name&quot;: &quot;Бюджетный калькулятор&quot;,
        &quot;short_name&quot;: &quot;Бюджет&quot;,
        &quot;start_url&quot;: &quot;/&quot;,
        &quot;display&quot;: &quot;standalone&quot;,
        &quot;background_color&quot;: &quot;#667eea&quot;,
        &quot;theme_color&quot;: &quot;#4caf50&quot;,
        &quot;orientation&quot;: &quot;portrait-primary&quot;,
        &quot;categories&quot;: [&quot;finance&quot;, &quot;productivity&quot;],
        &quot;icons&quot;: [
            {
                &quot;src&quot;: &quot;data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 192 192'><rect width='192' height='192' fill='%234caf50' rx='20'/><text x='96' y='120' font-size='100' text-anchor='middle' fill='white'>💰</text></svg>&quot;,
                &quot;sizes&quot;: &quot;192x192&quot;,
                &quot;type&quot;: &quot;image/svg+xml&quot;
            },
            {
                &quot;src&quot;: &quot;data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 512 512'><rect width='512' height='512' fill='%234caf50' rx='50'/><text x='256' y='320' font-size='280' text-anchor='middle' fill='white'>💰</text></svg>&quot;,
                &quot;sizes&quot;: &quot;512x512&quot;,
                &quot;type&quot;: &quot;image/svg+xml&quot;
            }
        ]
    }">
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Helvetica Neue', Arial, sans-serif;
            margin: 0;
            padding: 10px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            font-size: 16px;
            line-height: 1.5;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: #fff;
            padding: 15px;
            border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
        }

        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 20px;
            font-size: clamp(1.5rem, 4vw, 2.2rem);
        }

        h2 {
            color: #444;
            border-bottom: 2px solid #4caf50;
            padding-bottom: 5px;
            font-size: clamp(1.2rem, 3vw, 1.5rem);
        }

        h3 {
            font-size: clamp(1rem, 2.5vw, 1.2rem);
        }

        .pwa-install-banner {
            background: linear-gradient(135deg, #4caf50, #45a049);
            color: white;
            padding: 15px;
            border-radius: 8px;
            margin: 15px 0;
            text-align: center;
            display: none;
            animation: slideIn 0.3s ease-out;
        }

        .pwa-install-banner button {
            background: rgba(255,255,255,0.2);
            border: 1px solid rgba(255,255,255,0.3);
            color: white;
            padding: 8px 16px;
            border-radius: 4px;
            cursor: pointer;
            margin: 0 5px;
        }

        .connection-status {
            padding: 8px 12px;
            border-radius: 6px;
            font-size: clamp(0.7rem, 2vw, 0.8rem);
            margin: 10px 0;
            text-align: center;
            transition: all 0.3s ease;
        }

        .connection-status.online {
            background: #e8f5e8;
            color: #2e7d2e;
        }

        .connection-status.offline {
            background: #ffe8e8;
            color: #d32f2f;
        }

        .tabs {
            display: flex;
            margin-bottom: 20px;
            background: #f1f1f1;
            border-radius: 8px;
            overflow: hidden;
            flex-wrap: wrap;
        }

        .tab {
            flex: 1;
            padding: 12px 8px;
            text-align: center;
            background: #f1f1f1;
            cursor: pointer;
            border: none;
            font-size: clamp(0.8rem, 2.5vw, 1rem);
            transition: all 0.3s;
            min-width: 80px;
        }

        .tab.active {
            background: #4caf50;
            color: white;
        }

        .tab:hover:not(.active) {
            background: #e8e8e8;
        }

        label {
            display: block;
            margin-bottom: 5px;
            color: #333;
            font-weight: 500;
            font-size: clamp(0.9rem, 2vw, 1rem);
        }

        input, select {
            width: 100%;
            padding: 12px;
            margin-bottom: 15px;
            border: 2px solid #ddd;
            border-radius: 6px;
            font-size: clamp(14px, 3vw, 16px);
            transition: border-color 0.3s;
            -webkit-appearance: none;
            -moz-appearance: none;
            appearance: none;
        }

        input:focus, select:focus {
            outline: none;
            border-color: #4caf50;
        }

        .input-group {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 10px;
            align-items: end;
            margin-bottom: 15px;
        }

        button {
            padding: 12px 20px;
            background: linear-gradient(135deg, #4caf50, #45a049);
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            margin: 5px;
            font-size: clamp(14px, 2.5vw, 16px);
            font-weight: 500;
            transition: all 0.3s;
            min-height: 44px;
        }

        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(76, 175, 80, 0.3);
        }

        button:active {
            transform: translateY(0);
        }

        button.danger {
            background: linear-gradient(135deg, #f44336, #e53935);
        }

        button.danger:hover {
            box-shadow: 0 4px 12px rgba(244, 67, 54, 0.3);
        }

        button.secondary {
            background: linear-gradient(135deg, #2196f3, #1976d2);
        }

        button.secondary:hover {
            box-shadow: 0 4px 12px rgba(33, 150, 243, 0.3);
        }

        button.warning {
            background: linear-gradient(135deg, #ff9800, #f57c00);
        }

        button.warning:hover {
            box-shadow: 0 4px 12px rgba(255, 152, 0, 0.3);
        }

        button.small {
            padding: 8px 12px;
            font-size: clamp(12px, 2vw, 14px);
            margin: 2px;
            min-height: 36px;
        }

        .collapsible {
            width: 100%;
            text-align: left;
            margin: 10px 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: clamp(0.9rem, 2.5vw, 1rem);
        }

        .collapsible::after {
            content: '▼';
            transition: transform 0.3s;
        }

        .collapsible.active::after {
            transform: rotate(180deg);
        }

        .collapsible-content {
            padding: 0;
            overflow: hidden;
            max-height: 0;
            transition: max-height 0.3s ease;
            background: #f9f9f9;
            border-radius: 6px;
            margin-bottom: 15px;
        }

        .collapsible-content.active {
            max-height: 1500px;
            padding: 15px;
        }

        .summary {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 10px;
            margin: 20px 0;
        }

        .summary-card {
            background: linear-gradient(135deg, #e3f2fd, #f3e5f5);
            padding: 15px;
            border-radius: 8px;
            text-align: center;
            border-left: 4px solid #4caf50;
            min-height: 80px;
            display: flex;
            flex-direction: column;
            justify-content: center;
        }

        .summary-card h3 {
            margin: 0 0 5px 0;
            color: #666;
            font-size: clamp(0.8rem, 2vw, 0.9rem);
        }

        .summary-card .amount {
            font-size: clamp(1.2rem, 4vw, 1.5rem);
            font-weight: bold;
            color: #333;
        }

        .expense-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            margin: 5px 0;
            background: #f8f9fa;
            border-radius: 6px;
            border-left: 4px solid #4caf50;
            flex-wrap: wrap;
            gap: 10px;
        }

        .expense-info {
            flex: 1;
            min-width: 200px;
        }

        .expense-amount {
            font-weight: bold;
            color: #4caf50;
            font-size: clamp(1rem, 3vw, 1.2rem);
        }

        .expense-description {
            color: #666;
            margin: 2px 0;
            font-size: clamp(0.8rem, 2.5vw, 0.9rem);
        }

        .expense-date {
            font-size: clamp(0.7rem, 2vw, 0.8rem);
            color: #999;
        }

        .expense-category, .expense-currency {
            padding: 2px 6px;
            border-radius: 12px;
            font-size: clamp(0.7rem, 2vw, 0.8rem);
            margin: 2px;
            display: inline-block;
        }

        .expense-category {
            background: #e1f5fe;
            color: #0277bd;
        }

        .expense-currency {
            background: #f3e5f5;
            color: #7b1fa2;
        }

        .expense-actions {
            display: flex;
            gap: 5px;
            flex-wrap: wrap;
        }

        .hidden {
            display: none;
        }

        .alert {
            position: fixed;
            top: 20px;
            right: 20px;
            padding: 15px 20px;
            border-radius: 6px;
            font-weight: 500;
            z-index: 1000;
            max-width: 300px;
            font-size: clamp(0.8rem, 2.5vw, 0.9rem);
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }

        .alert.success {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }

        .alert.warning {
            background: #fff3cd;
            color: #856404;
            border: 1px solid #ffeeba;
        }

        .alert.danger {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }

        .alert.info {
            background: #d1ecf1;
            color: #0c5460;
            border: 1px solid #bee5eb;
        }

        .progress-bar {
            width: 100%;
            height: 12px;
            background: #e0e0e0;
            border-radius: 6px;
            overflow: hidden;
            margin: 10px 0;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #4caf50, #8bc34a);
            transition: width 0.3s ease;
        }

        .export-import {
            display: flex;
            gap: 10px;
            margin: 20px 0;
            flex-wrap: wrap;
        }

        .modal {
            display: none;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.5);
            padding: 20px;
            box-sizing: border-box;
        }

        .modal-content {
            background-color: #fefefe;
            margin: auto;
            padding: 20px;
            border-radius: 12px;
            width: 100%;
            max-width: 500px;
            max-height: 90vh;
            overflow-y: auto;
            box-shadow: 0 4px 20px rgba(0,0,0,0.3);
            position: relative;
            top: 50%;
            transform: translateY(-50%);
        }

        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .close {
            color: #aaa;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
            line-height: 1;
            padding: 5px;
        }

        .close:hover,
        .close:focus {
            color: #000;
            text-decoration: none;
            cursor: pointer;
        }

        .category-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            margin: 5px 0;
            background: #f8f9fa;
            border-radius: 6px;
            border-left: 4px solid #2196f3;
            flex-wrap: wrap;
            gap: 10px;
        }

        .backup-status, .exchange-rate-status {
            padding: 8px 12px;
            border-radius: 6px;
            font-size: clamp(0.7rem, 2vw, 0.8rem);
            margin: 10px 0;
            text-align: center;
        }

        .backup-status {
            background: #e8f5e8;
            color: #2e7d2e;
        }

        .exchange-rate-status {
            background: #e3f2fd;
            color: #1976d2;
        }

        .currency-converter {
            background: #f5f5f5;
            padding: 15px;
            border-radius: 6px;
            margin: 15px 0;
        }

        .charts-container {
            display: grid;
            grid-template-columns: 1fr;
            gap: 20px;
            margin: 20px 0;
        }

        .chart-card {
            background: #fff;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        .chart-card h3 {
            margin-top: 0;
            color: #333;
            text-align: center;
        }

        .chart-container {
            position: relative;
            height: 250px;
        }

        .formatted-number {
            font-family: -apple-system, BlinkMacSystemFont, 'SF Mono', 'Monaco', 'Cascadia Code', 'Roboto Mono', 'Courier New', monospace;
        }

        /* Мобильные устройства */
        @media (max-width: 480px) {
            body {
                padding: 5px;
            }
            
            .container {
                padding: 10px;
                border-radius: 8px;
            }

            .tabs {
                flex-direction: column;
            }

            .tab {
                flex: none;
                width: 100%;
                padding: 15px;
            }

            .input-group {
                grid-template-columns: 1fr;
            }

            .export-import {
                flex-direction: column;
            }

            .summary {
                grid-template-columns: 1fr;
            }

            .expense-item {
                flex-direction: column;
                align-items: stretch;
                text-align: center;
            }

            .expense-actions {
                justify-content: center;
                margin-top: 10px;
            }

            .modal-content {
                padding: 15px;
                margin: 10px;
                width: calc(100% - 20px);
                max-height: 90vh;
            }

            .chart-container {
                height: 200px;
            }

            .alert {
                right: 10px;
                left: 10px;
                top: 10px;
                max-width: none;
            }
        }

        /* Планшеты */
        @media (min-width: 481px) and (max-width: 768px) {
            .container {
                padding: 20px;
            }

            .tabs {
                flex-wrap: nowrap;
            }

            .input-group {
                grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            }

            .summary {
                grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
            }

            .charts-container {
                grid-template-columns: 1fr;
            }
        }

        /* Большие планшеты и маленькие ноутбуки */
        @media (min-width: 769px) and (max-width: 1024px) {
            .charts-container {
                grid-template-columns: 1fr 1fr;
            }

            .summary {
                grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
            }
        }

        /* Большие экраны */
        @media (min-width: 1025px) {
            .container {
                padding: 25px;
            }

            .charts-container {
                grid-template-columns: 1fr 1fr;
            }

            .chart-container {
                height: 300px;
            }
        }

        /* Высокие экраны */
        @media (min-height: 800px) {
            .modal-content {
                max-height: 80vh;
            }
        }

        /* Ландшафтная ориентация на мобильных */
        @media (max-width: 768px) and (orientation: landscape) {
            .tabs {
                flex-direction: row;
                flex-wrap: wrap;
            }

            .tab {
                flex: 1;
                min-width: 120px;
            }

            .input-group {
                grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
            }
        }

        /* Темная тема для устройств с поддержкой */
        @media (prefers-color-scheme: dark) {
            body {
                background: linear-gradient(135deg, #2c3e50 0%, #34495e 100%);
            }

            .container {
                background: #2c3e50;
                color: #ecf0f1;
            }

            h1, h2, h3 {
                color: #ecf0f1;
            }

            input, select {
                background: #34495e;
                color: #ecf0f1;
                border-color: #555;
            }

            .summary-card {
                background: linear-gradient(135deg, #34495e, #2c3e50);
                color: #ecf0f1;
            }

            .expense-item {
                background: #34495e;
            }

            .modal-content {
                background: #2c3e50;
                color: #ecf0f1;
            }
        }

        /* Анимации для лучшего UX */
        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(-20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .alert {
            animation: slideIn 0.3s ease-out;
        }

        /* Улучшения для касания */
        button, .tab, .collapsible {
            touch-action: manipulation;
        }

        /* Предотвращение масштабирования при фокусе на iOS */
        @media screen and (max-width: 768px) {
            input[type="text"],
            input[type="number"],
            input[type="date"],
            select {
                font-size: 16px;
            }
        }

        /* PWA специфичные стили */
        @media (display-mode: standalone) {
            body {
                padding-top: env(safe-area-inset-top);
                padding-bottom: env(safe-area-inset-bottom);
            }
        }
    </style>
</head>

<body>
    <div class="container">
        <h1>💰Калькулятор Бюджета</h1>
        
        <!-- PWA Install Banner -->
        <div class="pwa-install-banner" id="installBanner">
            <div>📱 Установите приложение для удобного доступа</div>
            <div style="margin-top: 10px;">
                <button id="installButton">Установить</button>
                <button id="dismissButton">Не сейчас</button>
            </div>
        </div>

        <!-- Connection Status -->
        <div class="connection-status online" id="connectionStatus">
            🌐 Подключение активно
        </div>
        
        <div class="backup-status" id="backupStatus">
            ✅ Автосохранение активно
        </div>

        <div class="exchange-rate-status" id="exchangeRateStatus">
            🔄 Загрузка курсов валют...
        </div>
        
        <div class="tabs">
            <button class="tab active" onclick="switchTab('main')">📊 Основной</button>
            <button class="tab" onclick="switchTab('vacation')">🏖️ Отпуск</button>
            <button class="tab" onclick="switchTab('analytics')">📈 Аналитика</button>
            <button class="tab" onclick="switchTab('settings')">⚙️ Настройки</button>
        </div>

        <!-- Settings Tab -->
        <div id="settingsTab" class="tab-content hidden">
            <h2>Управление категориями</h2>
            <div class="input-group">
                <div>
                    <label for="newCategoryName">Название категории:</label>
                    <input type="text" id="newCategoryName" placeholder="Новая категория">
                </div>
                <div>
                    <label for="newCategoryIcon">Иконка:</label>
                    <input type="text" id="newCategoryIcon" placeholder="🏠" maxlength="2">
                </div>
                <div>
                    <button onclick="addCustomCategory()">Добавить</button>
                </div>
            </div>
            
            <div id="categoriesList"></div>

            <h2>Валюты</h2>
            <div class="input-group">
                <div>
                    <label for="baseCurrency">Основная валюта:</label>
                    <select id="baseCurrency" onchange="setBaseCurrency()">
                        <option value="RUB">₽ Рубль</option>
                        <option value="USD">$ Доллар</option>
                        <option value="EUR">€ Евро</option>
                        <option value="OMR">OMR Риалы</option>
                    </select>
                </div>
                <div>
                    <button onclick="updateExchangeRates()">🔄 Обновить курсы</button>
                </div>
            </div>

            <div class="currency-converter">
                <h3>Конвертер валют</h3>
                <div class="input-group">
                    <div>
                        <input type="number" id="convertAmount" placeholder="Сумма" step="0.01">
                    </div>
                    <div>
                        <select id="fromCurrency">
                            <option value="RUB">₽ Рубль</option>
                            <option value="USD">$ Доллар</option>
                            <option value="EUR">€ Евро</option>
                            <option value="OMR">OMR Риалы</option>
                        </select>
                    </div>
                    <div>
                        <select id="toCurrency">
                            <option value="USD">$ Доллар</option>
                            <option value="RUB">₽ Рубль</option>
                            <option value="EUR">€ Евро</option>
                            <option value="OMR">OMR Риалы</option>
                        </select>
                    </div>
                    <div>
                        <button onclick="convertCurrency()">Конвертировать</button>
                    </div>
                </div>
                <div id="conversionResult"></div>
            </div>

            <h2>Резервное копирование</h2>
            <div class="export-import">
                <button class="secondary" onclick="createBackup()">💾 Создать резервную копию</button>
                <button class="warning" onclick="restoreBackup()">📥 Восстановить из копии</button>
                <button class="secondary" onclick="exportToDoc()">📄 Экспорт в DOC</button>
            </div>

            <h2>PWA Settings</h2>
            <div class="export-import">
                <button class="secondary" onclick="clearAppCache()">🗑️ Очистить кеш</button>
                <button class="secondary" onclick="showInstallPrompt()">📱 Установить приложение</button>
            </div>
        </div>

        <!-- Main Tab -->
        <div id="mainTab" class="tab-content">
            <button class="collapsible" onclick="toggleSection('budgetSetup')">
                Настройка бюджета
            </button>
            <div class="collapsible-content" id="budgetSetup">
                <div class="input-group">
                    <div>
                        <label for="totalAmount">Общая сумма:</label>
                        <input type="number" id="totalAmount" min="0" step="0.01" placeholder="Введите сумму">
                    </div>
                    <div>
                        <label for="budgetCurrency">Валюта:</label>
                        <select id="budgetCurrency">
                            <option value="RUB">₽ Рубль</option>
                            <option value="USD">$ Доллар</option>
                            <option value="EUR">€ Евро</option>
                            <option value="OMR">OMR Риалы</option>
                        </select>
                    </div>
                </div>
                <div class="input-group">
                    <div>
                        <label for="startDate">Дата начала:</label>
                        <input type="date" id="startDate">
                    </div>
                    <div>
                        <label for="endDate">Дата окончания:</label>
                        <input type="date" id="endDate">
                    </div>
                </div>
                <button onclick="setBudget('main')">Установить бюджет</button>
            </div>

            <div class="summary" id="mainSummary"></div>

            <div class="progress-bar">
                <div class="progress-fill" id="mainProgress" style="width: 0%"></div>
            </div>

            <h2>Добавить трату</h2>
            <div class="input-group">
                <div>
                    <label for="expenseAmount">Сумма:</label>
                    <input type="number" id="expenseAmount" min="0" step="0.01" placeholder="0.00">
                </div>
                <div>
                    <label for="expenseCategory">Категория:</label>
                    <select id="expenseCategory"></select>
                </div>
                <div>
                    <label for="expenseCurrency">Валюта:</label>
                    <select id="expenseCurrency">
                        <option value="RUB">₽ Рубль</option>
                        <option value="USD">$ Доллар</option>
                        <option value="EUR">€ Евро</option>
                        <option value="OMR">OMR Риалы</option>
                    </select>
                </div>
            </div>
            <label for="expenseDescription">Описание:</label>
            <input type="text" id="expenseDescription" placeholder="Описание траты">
            <button onclick="addExpense('main')">Добавить трату</button>

            <button class="collapsible" onclick="toggleSection('expenseList')">
                Список трат
            </button>
            <div class="collapsible-content" id="expenseList">
                <div class="input-group">
                    <div>
                        <label for="filterDate">Фильтр по дате:</label>
                        <input type="date" id="filterDate" onchange="filterExpenses('main')">
                    </div>
                    <div>
                        <label for="filterCategory">Фильтр по категории:</label>
                        <select id="filterCategory" onchange="filterExpenses('main')">
                            <option value="">Все категории</option>
                        </select>
                    </div>
                    <div>
                        <button onclick="clearFilters('main')">Сбросить фильтры</button>
                    </div>
                </div>
                <div id="mainExpenseList"></div>
            </div>

            <div class="export-import">
                <button class="secondary" onclick="exportData('main')">📤 Экспорт данных</button>
                <label class="secondary" style="cursor: pointer; display: inline-block;">
                    📥 Импорт данных
                    <input type="file" id="importFile" accept=".json" onchange="importData('main')" style="display: none;">
                </label>
                <button class="danger" onclick="clearAllData('main')">🗑️ Очистить всё</button>
            </div>
        </div>

        <!-- Vacation Tab -->
        <div id="vacationTab" class="tab-content hidden">
            <button class="collapsible" onclick="toggleSection('budgetSetupVacation')">
                Настройка бюджета отпуска
            </button>
            <div class="collapsible-content" id="budgetSetupVacation">
                <div class="input-group">
                    <div>
                        <label for="totalAmountVacation">Общая сумма:</label>
                        <input type="number" id="totalAmountVacation" min="0" step="0.01" placeholder="Введите сумму">
                    </div>
                    <div>
                        <label for="budgetCurrencyVacation">Валюта:</label>
                        <select id="budgetCurrencyVacation">
                            <option value="RUB">₽ Рубль</option>
                            <option value="USD">$ Доллар</option>
                            <option value="EUR">€ Евро</option>
                            <option value="OMR">OMR Риалы</option>
                        </select>
                    </div>
                </div>
                <div class="input-group">
                    <div>
                        <label for="startDateVacation">Дата начала:</label>
                        <input type="date" id="startDateVacation">
                    </div>
                    <div>
                        <label for="endDateVacation">Дата окончания:</label>
                        <input type="date" id="endDateVacation">
                    </div>
                </div>
                <button onclick="setBudget('vacation')">Установить бюджет</button>
            </div>

            <div class="summary" id="vacationSummary"></div>

            <div class="progress-bar">
                <div class="progress-fill" id="vacationProgress" style="width: 0%"></div>
            </div>

            <h2>Добавить трату</h2>
            <div class="input-group">
                <div>
                    <label for="expenseAmountVacation">Сумма:</label>
                    <input type="number" id="expenseAmountVacation" min="0" step="0.01" placeholder="0.00">
                </div>
                <div>
                    <label for="expenseCategoryVacation">Категория:</label>
                    <select id="expenseCategoryVacation"></select>
                </div>
                <div>
                    <label for="expenseCurrencyVacation">Валюта:</label>
                    <select id="expenseCurrencyVacation">
                        <option value="RUB">₽ Рубль</option>
                        <option value="USD">$ Доллар</option>
                        <option value="EUR">€ Евро</option>
                        <option value="OMR">OMR Риалы</option>
                    </select>
                </div>
            </div>
            <label for="expenseDescriptionVacation">Описание:</label>
            <input type="text" id="expenseDescriptionVacation" placeholder="Описание траты">
            <button onclick="addExpense('vacation')">Добавить трату</button>

            <button class="collapsible" onclick="toggleSection('expenseListVacation')">
                Список трат
            </button>
            <div class="collapsible-content" id="expenseListVacation">
                <div class="input-group">
                    <div>
                        <label for="filterDateVacation">Фильтр по дате:</label>
                        <input type="date" id="filterDateVacation" onchange="filterExpenses('vacation')">
                    </div>
                    <div>
                        <label for="filterCategoryVacation">Фильтр по категории:</label>
                        <select id="filterCategoryVacation" onchange="filterExpenses('vacation')">
                            <option value="">Все категории</option>
                        </select>
                    </div>
                    <div>
                        <button onclick="clearFilters('vacation')">Сбросить фильтры</button>
                    </div>
                </div>
                <div id="vacationExpenseList"></div>
            </div>

            <div class="export-import">
                <button class="secondary" onclick="exportData('vacation')">📤 Экспорт данных</button>
                <label class="secondary" style="cursor: pointer; display: inline-block;">
                    📥 Импорт данных
                    <input type="file" id="importFileVacation" accept=".json" onchange="importData('vacation')" style="display: none;">
                </label>
                <button class="danger" onclick="clearAllData('vacation')">🗑️ Очистить всё</button>
            </div>
        </div>

        <!-- Analytics Tab -->
        <div id="analyticsTab" class="tab-content hidden">
            <h2>📊 Аналитика трат</h2>
            <div class="input-group">
                <div>
                    <label for="analyticsMode">Режим:</label>
                    <select id="analyticsMode" onchange="updateAnalytics()">
                        <option value="main">Основной</option>
                        <option value="vacation">Отпуск</option>
                    </select>
                </div>
                <div>
                    <label for="analyticsPeriod">Период:</label>
                    <select id="analyticsPeriod" onchange="updateAnalytics()">
                        <option value="7">Последние 7 дней</option>
                        <option value="30">Последние 30 дней</option>
                        <option value="all">Весь период</option>
                    </select>
                </div>
            </div>
            
            <div class="charts-container">
                <div class="chart-card">
                    <h3>Расходы по категориям</h3>
                    <div class="chart-container">
                        <canvas id="categoryChart"></canvas>
                    </div>
                </div>
                <div class="chart-card">
                    <h3>Динамика трат по дням</h3>
                    <div class="chart-container">
                        <canvas id="timeChart"></canvas>
                    </div>
                </div>
            </div>
            
            <div id="analyticsContent">
                <div id="categoryAnalytics" class="summary"></div>
            </div>
        </div>
    </div>

    <!-- Edit Expense Modal -->
    <div id="editModal" class="modal">
        <div class="modal-content">
            <div class="modal-header">
                <h2>Редактировать трату</h2>
                <span class="close" onclick="closeEditModal()">&times;</span>
            </div>
            <div class="input-group">
                <div>
                    <label for="editAmount">Сумма:</label>
                    <input type="number" id="editAmount" min="0" step="0.01">
                </div>
                <div>
                    <label for="editCurrency">Валюта:</label>
                    <select id="editCurrency">
                        <option value="RUB">₽ Рубль</option>
                        <option value="USD">$ Доллар</option>
                        <option value="EUR">€ Евро</option>
                        <option value="OMR">OMR Риалы</option>
                    </select>
                </div>
            </div>
            <label for="editCategory">Категория:</label>
            <select id="editCategory"></select>
            <label for="editDescription">Описание:</label>
            <input type="text" id="editDescription">
            <label for="editDate">Дата:</label>
            <input type="date" id="editDate">
            <div style="margin-top: 20px;">
                <button onclick="saveExpenseEdit()">Сохранить</button>
                <button class="secondary" onclick="closeEditModal()">Отмена</button>
            </div>
        </div>
    </div>

    <script>
        // Service Worker Registration
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                const swScript = `
                    const CACHE_NAME = 'budget-calculator-v1';
                    const urlsToCache = [
                        '/',
                        'https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js'
                    ];

                    self.addEventListener('install', (event) => {
                        event.waitUntil(
                            caches.open(CACHE_NAME)
                                .then((cache) => {
                                    return cache.addAll(urlsToCache);
                                })
                        );
                        self.skipWaiting();
                    });

                    self.addEventListener('fetch', (event) => {
                        event.respondWith(
                            caches.match(event.request)
                                .then((response) => {
                                    if (response) {
                                        return response;
                                    }
                                    return fetch(event.request).catch(() => {
                                        if (event.request.mode === 'navigate') {
                                            return caches.match('/');
                                        }
                                    });
                                })
                        );
                    });

                    self.addEventListener('activate', (event) => {
                        event.waitUntil(
                            caches.keys().then((cacheNames) => {
                                return Promise.all(
                                    cacheNames.map((cacheName) => {
                                        if (cacheName !== CACHE_NAME) {
                                            return caches.delete(cacheName);
                                        }
                                    })
                                );
                            })
                        );
                        self.clients.claim();
                    });
                `;

                const blob = new Blob([swScript], { type: 'application/javascript' });
                const swUrl = URL.createObjectURL(blob);

                navigator.serviceWorker.register(swUrl)
                    .then((registration) => {
                        console.log('SW registered: ', registration);
                    })
                    .catch((registrationError) => {
                        console.log('SW registration failed: ', registrationError);
                    });
            });
        }

        // PWA Install Logic
        let deferredPrompt;
        const installBanner = document.getElementById('installBanner');
        const installButton = document.getElementById('installButton');
        const dismissButton = document.getElementById('dismissButton');

        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            
            if (!localStorage.getItem('pwa-dismissed')) {
                installBanner.style.display = 'block';
            }
        });

        installButton.addEventListener('click', async () => {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                const { outcome } = await deferredPrompt.userChoice;
                console.log(`User response to the install prompt: ${outcome}`);
                deferredPrompt = null;
                installBanner.style.display = 'none';
            }
        });

        dismissButton.addEventListener('click', () => {
            installBanner.style.display = 'none';
            localStorage.setItem('pwa-dismissed', 'true');
        });

        // Connection Status
        function updateConnectionStatus() {
            const statusEl = document.getElementById('connectionStatus');
            if (navigator.onLine) {
                statusEl.className = 'connection-status online';
                statusEl.textContent = '🌐 Подключение активно';
            } else {
                statusEl.className = 'connection-status offline';
                statusEl.textContent = '📱 Работа офлайн';
            }
        }

        window.addEventListener('online', updateConnectionStatus);
        window.addEventListener('offline', updateConnectionStatus);
        updateConnectionStatus();

        // PWA Functions
        function showInstallPrompt() {
            if (deferredPrompt) {
                installBanner.style.display = 'block';
            } else {
                alert('Приложение уже установлено или установка недоступна');
            }
        }

        function clearAppCache() {
            if ('caches' in window) {
                caches.keys().then((cacheNames) => {
                    cacheNames.forEach((cacheName) => {
                        caches.delete(cacheName);
                    });
                    alert('Кеш очищен. Перезагрузите страницу.');
                });
            }
        }

        // Original BudgetCalculator class with offline enhancements
        class BudgetCalculator {
            constructor() {
                this.currentTab = 'main';
                this.editingExpense = null;
                this.editingMode = null;
                this.baseCurrency = 'RUB';
                this.exchangeRates = {
                    'RUB': 1,
                    'USD': 0.011,
                    'EUR': 0.010,
                    'OMR': 0.004,
                };
                this.currencySymbols = {
                    'RUB': '₽',
                    'USD': '$',
                    'EUR': '€',
                    'OMR': 'OMR',
                };
                this.defaultCategories = {
                    main: [
                        { name: 'Еда', icon: '🍕' },
                        { name: 'Транспорт', icon: '🚗' },
                        { name: 'Развлечения', icon: '🎬' },
                        { name: 'Покупки', icon: '🛍️' },
                        { name: 'Здоровье', icon: '🏥' },
                        { name: 'Образование', icon: '📚' },
                        { name: 'Коммунальные', icon: '💡' },
                        { name: 'Другое', icon: '📦' }
                    ],
                    vacation: [
                        { name: 'Проживание', icon: '🏨' },
                        { name: 'Питание', icon: '🍽️' },
                        { name: 'Транспорт', icon: '✈️' },
                        { name: 'Развлечения', icon: '🎭' },
                        { name: 'Экскурсии', icon: '🗺️' },
                        { name: 'Сувениры', icon: '🎁' },
                        { name: 'Медицина', icon: '💊' },
                        { name: 'Другое', icon: '📦' }
                    ]
                };
                this.customCategories = JSON.parse(localStorage.getItem('customCategories')) || { main: [], vacation: [] };
                this.categoryChart = null;
                this.timeChart = null;
                this.isOnline = navigator.onLine;
                this.init();
                this.startAutoBackup();
                this.updateExchangeRates();
            }

            init() {
                this.loadData();
                this.loadSettings();
                this.updateAllDisplays();
                this.setDefaultDates();
                this.populateCategories();
            }

            startAutoBackup() {
                setInterval(() => {
                    this.autoBackup();
                }, 60000);
            }

            autoBackup() {
                const backupData = {
                    data: this.data,
                    customCategories: this.customCategories,
                    baseCurrency: this.baseCurrency,
                    timestamp: new Date().toISOString()
                };
                localStorage.setItem('autoBackup', JSON.stringify(backupData));
                
                const statusEl = document.getElementById('backupStatus');
                statusEl.textContent = `✅ Последнее сохранение: ${new Date().toLocaleTimeString()}`;
            }

            async updateExchangeRates() {
                if (!navigator.onLine) {
                    const stored = localStorage.getItem('exchangeRates');
                    if (stored) {
                        this.exchangeRates = JSON.parse(stored);
                        document.getElementById('exchangeRateStatus').textContent = 
                            '📱 Используются сохранённые курсы (офлайн)';
                    }
                    return;
                }

                try {
                    document.getElementById('exchangeRateStatus').textContent = '🔄 Обновление курсов валют...';
                    
                    const response = await fetch('https://api.exchangerate-api.com/v4/latest/USD');
                    const data = await response.json();
                    
                    if (data.rates) {
                        this.exchangeRates = {
                            'USD': 1,
                            'RUB': data.rates.RUB || 90,
                            'EUR': data.rates.EUR || 0.85,
                            'OMR': data.rates.OMR || 0.385,
                        };
                        
                        localStorage.setItem('exchangeRates', JSON.stringify(this.exchangeRates));
                        localStorage.setItem('exchangeRatesUpdated', new Date().toISOString());
                        
                        document.getElementById('exchangeRateStatus').textContent = 
                            `✅ Курсы обновлены: ${new Date().toLocaleTimeString()}`;
                        
                        this.updateAllDisplays();
                    }
                } catch (error) {
                    const stored = localStorage.getItem('exchangeRates');
                    if (stored) {
                        this.exchangeRates = JSON.parse(stored);
                        document.getElementById('exchangeRateStatus').textContent = 
                            '⚠️ Используются сохранённые курсы';
                    } else {
                        document.getElementById('exchangeRateStatus').textContent = 
                            '❌ Ошибка загрузки курсов. Используются базовые значения.';
                    }
                }
            }

            loadSettings() {
                this.baseCurrency = localStorage.getItem('baseCurrency') || 'RUB';
                document.getElementById('baseCurrency').value = this.baseCurrency;
                
                const stored = localStorage.getItem('exchangeRates');
                if (stored) {
                    this.exchangeRates = JSON.parse(stored);
                }
            }

            setDefaultDates() {
                const today = new Date().toISOString().split('T')[0];
                document.getElementById('startDate').value = today;
                document.getElementById('startDateVacation').value = today;
                
                const endDate = new Date();
                endDate.setDate(endDate.getDate() + 30);
                document.getElementById('endDate').value = endDate.toISOString().split('T')[0];
                document.getElementById('endDateVacation').value = endDate.toISOString().split('T')[0];
            }

            loadData() {
                this.data = {
                    main: JSON.parse(localStorage.getItem('budgetData_main')) || this.getDefaultData(),
                    vacation: JSON.parse(localStorage.getItem('budgetData_vacation')) || this.getDefaultData()
                };
            }

            getDefaultData() {
                return {
                    budget: {
                        total: 0,
                        currency: 'RUB',
                        startDate: null,
                        endDate: null,
                        dailyLimit: 0
                    },
                    expenses: []
                };
            }

            saveData(mode) {
                localStorage.setItem(`budgetData_${mode}`, JSON.stringify(this.data[mode]));
                localStorage.setItem('customCategories', JSON.stringify(this.customCategories));
                this.autoBackup();
            }

            generateId() {
                return Date.now().toString(36) + Math.random().toString(36).substr(2);
            }

            getDaysBetween(start, end) {
                const startDate = new Date(start);
                const endDate = new Date(end);
                const diffTime = Math.abs(endDate - startDate);
                return Math.ceil(diffTime / (1000 * 60 * 60 * 24)) + 1;
            }

            getAllCategories(mode) {
                return [...this.defaultCategories[mode], ...this.customCategories[mode]];
            }

            populateCategories() {
                ['main', 'vacation'].forEach(mode => {
                    const categories = this.getAllCategories(mode);
                    const suffix = mode === 'vacation' ? 'Vacation' : '';
                    
                    const categorySelect = document.getElementById(`expenseCategory${suffix}`);
                    const filterSelect = document.getElementById(`filterCategory${suffix}`);
                    
                    categorySelect.innerHTML = '';
                    filterSelect.innerHTML = '<option value="">Все категории</option>';
                    
                    categories.forEach(cat => {
                        const option = new Option(`${cat.icon} ${cat.name}`, cat.name);
                        categorySelect.appendChild(option.cloneNode(true));
                        filterSelect.appendChild(option);
                    });
                });

                const editCategorySelect = document.getElementById('editCategory');
                editCategorySelect.innerHTML = '';
                const allCategories = [...this.getAllCategories('main'), ...this.getAllCategories('vacation')];
                const uniqueCategories = allCategories.filter((cat, index, self) => 
                    self.findIndex(c => c.name === cat.name) === index
                );
                
                uniqueCategories.forEach(cat => {
                    editCategorySelect.appendChild(new Option(`${cat.icon} ${cat.name}`, cat.name));
                });
            }

            updateCategoriesList() {
                const listEl = document.getElementById('categoriesList');
                let html = '';
                
                ['main', 'vacation'].forEach(mode => {
                    html += `<h3>${mode === 'main' ? 'Основные категории' : 'Категории отпуска'}</h3>`;
                    
                    this.customCategories[mode].forEach(cat => {
                        html += `
                            <div class="category-item">
                                <span>${cat.icon} ${cat.name}</span>
                                <button class="danger small" onclick="calculator.deleteCustomCategory('${mode}', '${cat.name}')">
                                    Удалить
                                </button>
                            </div>
                        `;
                    });
                });
                
                listEl.innerHTML = html;
            }

            addCustomCategory() {
                const name = document.getElementById('newCategoryName').value.trim();
                const icon = document.getElementById('newCategoryIcon').value.trim() || '📦';
                
                if (!name) {
                    this.showAlert('Введите название категории', 'danger');
                    return;
                }
                
                ['main', 'vacation'].forEach(mode => {
                    const exists = this.getAllCategories(mode).find(cat => cat.name === name);
                    if (!exists) {
                        this.customCategories[mode].push({ name, icon });
                    }
                });
                
                this.saveData('main');
                this.populateCategories();
                this.updateCategoriesList();
                this.showAlert('Категория добавлена!', 'success');
                
                document.getElementById('newCategoryName').value = '';
                document.getElementById('newCategoryIcon').value = '';
            }

            deleteCustomCategory(mode, categoryName) {
                if (confirm('Удалить категорию?')) {
                    this.customCategories[mode] = this.customCategories[mode].filter(cat => cat.name !== categoryName);
                    this.saveData('main');
                    this.populateCategories();
                    this.updateCategoriesList();
                    this.showAlert('Категория удалена', 'success');
                }
            }

            setBaseCurrency() {
                this.baseCurrency = document.getElementById('baseCurrency').value;
                localStorage.setItem('baseCurrency', this.baseCurrency);
                this.updateAllDisplays();
                this.showAlert('Основная валюта изменена', 'success');
            }

            convertAmount(amount, fromCurrency, toCurrency) {
                if (fromCurrency === toCurrency) return amount;
                
                const inUSD = amount / this.exchangeRates[fromCurrency];
                return inUSD * this.exchangeRates[toCurrency];
            }

            formatCurrency(amount, currency) {
                return `${this.formatNumber(amount)} ${this.currencySymbols[currency]}`;
            }

            formatNumber(number) {
                return new Intl.NumberFormat('ru-RU', {
                    minimumFractionDigits: 2,
                    maximumFractionDigits: 2
                }).format(number);
            }

            convertCurrency() {
                const amount = parseFloat(document.getElementById('convertAmount').value);
                const from = document.getElementById('fromCurrency').value;
                const to = document.getElementById('toCurrency').value;
                
                if (!amount || amount <= 0) {
                    this.showAlert('Введите корректную сумму', 'danger');
                    return;
                }
                
                const converted = this.convertAmount(amount, from, to);
                document.getElementById('conversionResult').innerHTML = `
                    <strong>${this.formatCurrency(amount, from)} = ${this.formatCurrency(converted, to)}</strong>
                `;
            }

            setBudget(mode) {
                const totalAmount = parseFloat(document.getElementById(`totalAmount${mode === 'vacation' ? 'Vacation' : ''}`).value);
                const currency = document.getElementById(`budgetCurrency${mode === 'vacation' ? 'Vacation' : ''}`).value;
                const startDate = document.getElementById(`startDate${mode === 'vacation' ? 'Vacation' : ''}`).value;
                const endDate = document.getElementById(`endDate${mode === 'vacation' ? 'Vacation' : ''}`).value;

                if (!totalAmount || totalAmount <= 0) {
                    this.showAlert('Введите корректную сумму бюджета', 'danger');
                    return;
                }

                if (!startDate || !endDate) {
                    this.showAlert('Выберите даты начала и окончания', 'danger');
                    return;
                }

                if (new Date(startDate) >= new Date(endDate)) {
                    this.showAlert('Дата окончания должна быть позже даты начала', 'danger');
                    return;
                }

                const days = this.getDaysBetween(startDate, endDate);
                const dailyLimit = totalAmount / days;

                this.data[mode].budget = {
                    total: totalAmount,
                    currency: currency,
                    startDate: startDate,
                    endDate: endDate,
                    dailyLimit: dailyLimit
                };

                this.saveData(mode);
                this.updateDisplay(mode);
                this.showAlert('Бюджет успешно установлен!', 'success');

                document.getElementById(`totalAmount${mode === 'vacation' ? 'Vacation' : ''}`).value = '';
            }

            addExpense(mode) {
                const amount = parseFloat(document.getElementById(`expenseAmount${mode === 'vacation' ? 'Vacation' : ''}`).value);
                const category = document.getElementById(`expenseCategory${mode === 'vacation' ? 'Vacation' : ''}`).value;
                const currency = document.getElementById(`expenseCurrency${mode === 'vacation' ? 'Vacation' : ''}`).value;
                const description = document.getElementById(`expenseDescription${mode === 'vacation' ? 'Vacation' : ''}`).value;

                if (!amount || amount <= 0) {
                    this.showAlert('Введите корректную сумму траты', 'danger');
                    return;
                }

                const expense = {
                    id: this.generateId(),
                    amount: amount,
                    currency: currency,
                    category: category,
                    description: description || 'Без описания',
                    date: new Date().toISOString().split('T')[0],
                    timestamp: new Date().toISOString()
                };

                this.data[mode].expenses.push(expense);
                this.saveData(mode);
                this.updateDisplay(mode);
                this.showAlert('Трата добавлена!', 'success');

                document.getElementById(`expenseAmount${mode === 'vacation' ? 'Vacation' : ''}`).value = '';
                document.getElementById(`expenseDescription${mode === 'vacation' ? 'Vacation' : ''}`).value = '';
            }

            editExpense(mode, expenseId) {
                const expense = this.data[mode].expenses.find(e => e.id === expenseId);
                if (!expense) return;

                this.editingExpense = expenseId;
                this.editingMode = mode;

                document.getElementById('editAmount').value = expense.amount;
                document.getElementById('editCurrency').value = expense.currency;
                document.getElementById('editCategory').value = expense.category;
                document.getElementById('editDescription').value = expense.description;
                document.getElementById('editDate').value = expense.date;

                document.getElementById('editModal').style.display = 'block';
            }

            closeEditModal() {
                document.getElementById('editModal').style.display = 'none';
                this.editingExpense = null;
                this.editingMode = null;
            }

            saveExpenseEdit() {
                if (!this.editingExpense || !this.editingMode) return;

                const expense = this.data[this.editingMode].expenses.find(e => e.id === this.editingExpense);
                if (!expense) return;

                expense.amount = parseFloat(document.getElementById('editAmount').value);
                expense.currency = document.getElementById('editCurrency').value;
                expense.category = document.getElementById('editCategory').value;
                expense.description = document.getElementById('editDescription').value;
                expense.date = document.getElementById('editDate').value;

                this.saveData(this.editingMode);
                this.updateDisplay(this.editingMode);
                this.closeEditModal();
                this.showAlert('Трата обновлена!', 'success');
            }

            deleteExpense(mode, expenseId) {
                if (confirm('Вы уверены, что хотите удалить эту трату?')) {
                    this.data[mode].expenses = this.data[mode].expenses.filter(expense => expense.id !== expenseId);
                    this.saveData(mode);
                    this.updateDisplay(mode);
                    this.showAlert('Трата удалена', 'success');
                }
            }

            updateDisplay(mode) {
                this.updateSummary(mode);
                this.updateExpenseList(mode);
                this.updateProgress(mode);
            }

            updateAllDisplays() {
                this.updateDisplay('main');
                this.updateDisplay('vacation');
                this.updateCategoriesList();
                if (this.currentTab === 'analytics') {
                    this.updateAnalytics();
                }
            }

            updateSummary(mode) {
                const data = this.data[mode];
                const budget = data.budget;
                const expenses = data.expenses;

                const totalSpent = expenses.reduce((sum, expense) => {
                    const convertedAmount = this.convertAmount(expense.amount, expense.currency, budget.currency);
                    return sum + convertedAmount;
                }, 0);

                const remaining = budget.total - totalSpent;
                
                const today = new Date().toISOString().split('T')[0];
                const todayExpenses = expenses.filter(expense => expense.date === today);
                const spentToday = todayExpenses.reduce((sum, expense) => {
                    const convertedAmount = this.convertAmount(expense.amount, expense.currency, budget.currency);
                    return sum + convertedAmount;
                }, 0);
                const remainingToday = budget.dailyLimit - spentToday;

                const currencySymbol = this.currencySymbols[budget.currency] || budget.currency;

                const summaryHtml = `
                    <div class="summary-card">
                        <h3>Общий бюджет</h3>
                        <div class="amount formatted-number">${this.formatNumber(budget.total)} ${currencySymbol}</div>
                    </div>
                    <div class="summary-card">
                        <h3>Потрачено</h3>
                        <div class="amount formatted-number" style="color: ${totalSpent > budget.total ? '#f44336' : '#ff9800'}">${this.formatNumber(totalSpent)} ${currencySymbol}</div>
                    </div>
                    <div class="summary-card">
                        <h3>Остаток</h3>
                        <div class="amount formatted-number" style="color: ${remaining < 0 ? '#f44336' : '#4caf50'}">${this.formatNumber(remaining)} ${currencySymbol}</div>
                    </div>
                    <div class="summary-card">
                        <h3>Лимит на день</h3>
                        <div class="amount formatted-number">${this.formatNumber(budget.dailyLimit)} ${currencySymbol}</div>
                    </div>
                    <div class="summary-card">
                        <h3>Потрачено сегодня</h3>
                        <div class="amount formatted-number" style="color: ${spentToday > budget.dailyLimit ? '#f44336' : '#ff9800'}">${this.formatNumber(spentToday)} ${currencySymbol}</div>
                    </div>
                    <div class="summary-card">
                        <h3>Остаток на сегодня</h3>
                        <div class="amount formatted-number" style="color: ${remainingToday < 0 ? '#f44336' : '#4caf50'}">${this.formatNumber(remainingToday)} ${currencySymbol}</div>
                    </div>
                `;

                document.getElementById(`${mode}Summary`).innerHTML = summaryHtml;
            }

            updateProgress(mode) {
                const data = this.data[mode];
                const totalSpent = data.expenses.reduce((sum, expense) => {
                    const convertedAmount = this.convertAmount(expense.amount, expense.currency, data.budget.currency);
                    return sum + convertedAmount;
                }, 0);
                const percentage = Math.min((totalSpent / data.budget.total) * 100, 100);
                
                const progressBar = document.getElementById(`${mode}Progress`);
                progressBar.style.width = `${percentage}%`;
                
                if (percentage > 90) {
                    progressBar.style.background = 'linear-gradient(90deg, #f44336, #d32f2f)';
                } else if (percentage > 70) {
                    progressBar.style.background = 'linear-gradient(90deg, #ff9800, #f57c00)';
                } else {
                    progressBar.style.background = 'linear-gradient(90deg, #4caf50, #8bc34a)';
                }
            }

            updateExpenseList(mode) {
                const expenses = this.getFilteredExpenses(mode);
                const listContainer = document.getElementById(`${mode}ExpenseList`);
                
                if (expenses.length === 0) {
                    listContainer.innerHTML = '<p style="text-align: center; color: #666;">Нет трат для отображения</p>';
                    return;
                }

                const expensesHtml = expenses.map(expense => `
                    <div class="expense-item">
                        <div class="expense-info">
                            <div>
                                <span class="expense-category">${expense.category}</span>
                                <span class="expense-currency">${expense.currency}</span>
                            </div>
                            <div class="expense-amount formatted-number">${this.formatNumber(expense.amount)} ${this.currencySymbols[expense.currency]}</div>
                            <div class="expense-description">${expense.description}</div>
                            <div class="expense-date">${this.formatDate(expense.date)}</div>
                        </div>
                        <div class="expense-actions">
                            <button class="warning small" onclick="calculator.editExpense('${mode}', '${expense.id}')">
                                Изменить
                            </button>
                            <button class="danger small" onclick="calculator.deleteExpense('${mode}', '${expense.id}')">
                                Удалить
                            </button>
                        </div>
                    </div>
                `).join('');

                listContainer.innerHTML = expensesHtml;
            }

            getFilteredExpenses(mode) {
                let expenses = [...this.data[mode].expenses];
                
                const dateFilter = document.getElementById(`filterDate${mode === 'vacation' ? 'Vacation' : ''}`).value;
                const categoryFilter = document.getElementById(`filterCategory${mode === 'vacation' ? 'Vacation' : ''}`).value;

                if (dateFilter) {
                    expenses = expenses.filter(expense => expense.date === dateFilter);
                }

                if (categoryFilter) {
                    expenses = expenses.filter(expense => expense.category === categoryFilter);
                }

                return expenses.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
            }

            filterExpenses(mode) {
                this.updateExpenseList(mode);
            }

            clearFilters(mode) {
                document.getElementById(`filterDate${mode === 'vacation' ? 'Vacation' : ''}`).value = '';
                document.getElementById(`filterCategory${mode === 'vacation' ? 'Vacation' : ''}`).value = '';
                this.updateExpenseList(mode);
            }

            formatDate(dateString) {
                const options = { 
                    year: 'numeric', 
                    month: 'long', 
                    day: 'numeric',
                    weekday: 'short'
                };
                return new Date(dateString).toLocaleDateString('ru-RU', options);
            }

            exportData(mode) {
                const data = {
                    budget: this.data[mode].budget,
                    expenses: this.data[mode].expenses,
                    customCategories: this.customCategories[mode],
                    exportDate: new Date().toISOString(),
                    mode: mode
                };

                const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `budget-${mode}-${new Date().toISOString().split('T')[0]}.json`;
                a.click();
                URL.revokeObjectURL(url);

                this.showAlert('Данные экспортированы!', 'success');
            }

            exportToDoc() {
                const modes = ['main', 'vacation'];
                let docContent = `
                    <html>
                    <head>
                        <meta charset="UTF-8">
                        <title>Отчет по бюджету</title>
                        <style>
                            body { font-family: Arial, sans-serif; margin: 20px; }
                            h1, h2 { color: #333; }
                            table { width: 100%; border-collapse: collapse; margin: 20px 0; }
                            th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
                            th { background-color: #f2f2f2; }
                            .summary { background-color: #e8f5e8; padding: 15px; margin: 20px 0; }
                            .formatted-number { font-family: monospace; }
                        </style>
                    </head>
                    <body>
                        <h1>Отчет по бюджету - ${new Date().toLocaleDateString()}</h1>
                `;

                modes.forEach(mode => {
                    const data = this.data[mode];
                    const budget = data.budget;
                    const expenses = data.expenses;

                    const totalSpent = expenses.reduce((sum, expense) => {
                        const convertedAmount = this.convertAmount(expense.amount, expense.currency, budget.currency);
                        return sum + convertedAmount;
                    }, 0);

                    const remaining = budget.total - totalSpent;
                    const currencySymbol = this.currencySymbols[budget.currency] || budget.currency;

                    docContent += `
                        <h2>${mode === 'main' ? 'Основной бюджет' : 'Бюджет отпуска'}</h2>
                        <div class="summary">
                            <p><strong>Общий бюджет:</strong> <span class="formatted-number">${this.formatNumber(budget.total)}</span> ${currencySymbol}</p>
                            <p><strong>Потрачено:</strong> <span class="formatted-number">${this.formatNumber(totalSpent)}</span> ${currencySymbol}</p>
                            <p><strong>Остаток:</strong> <span class="formatted-number">${this.formatNumber(remaining)}</span> ${currencySymbol}</p>
                            <p><strong>Период:</strong> ${budget.startDate} - ${budget.endDate}</p>
                        </div>

                        <h3>Список трат</h3>
                        <table>
                            <tr>
                                <th>Дата</th>
                                <th>Категория</th>
                                <th>Описание</th>
                                <th>Сумма</th>
                                <th>Валюта</th>
                            </tr>
                    `;

                    expenses.forEach(expense => {
                        docContent += `
                            <tr>
                                <td>${this.formatDate(expense.date)}</td>
                                <td>${expense.category}</td>
                                <td>${expense.description}</td>
                                <td class="formatted-number">${this.formatNumber(expense.amount)}</td>
                                <td>${expense.currency}</td>
                            </tr>
                        `;
                    });

                    docContent += '</table>';
                });

                docContent += '</body></html>';

                const blob = new Blob([docContent], { type: 'application/msword' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `budget-report-${new Date().toISOString().split('T')[0]}.doc`;
                a.click();
                URL.revokeObjectURL(url);

                this.showAlert('Отчет экспортирован в DOC!', 'success');
            }

            createBackup() {
                const backupData = {
                    data: this.data,
                    customCategories: this.customCategories,
                    baseCurrency: this.baseCurrency,
                    exchangeRates: this.exchangeRates,
                    version: '2.1',
                    timestamp: new Date().toISOString()
                };

                const blob = new Blob([JSON.stringify(backupData, null, 2)], { type: 'application/json' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `budget-backup-${new Date().toISOString().split('T')[0]}.json`;
                a.click();
                URL.revokeObjectURL(url);

                this.showAlert('Резервная копия создана!', 'success');
            }

            restoreBackup() {
                const input = document.createElement('input');
                input.type = 'file';
                input.accept = '.json';
                
                input.onchange = (e) => {
                    const file = e.target.files[0];
                    if (!file) return;

                    const reader = new FileReader();
                    reader.onload = (e) => {
                        try {
                            const backupData = JSON.parse(e.target.result);
                            
                            if (backupData.data && backupData.customCategories) {
                                this.data = backupData.data;
                                this.customCategories = backupData.customCategories;
                                this.baseCurrency = backupData.baseCurrency || 'RUB';
                                
                                if (backupData.exchangeRates) {
                                    this.exchangeRates = backupData.exchangeRates;
                                }
                                
                                this.saveData('main');
                                this.saveData('vacation');
                                localStorage.setItem('baseCurrency', this.baseCurrency);
                                localStorage.setItem('exchangeRates', JSON.stringify(this.exchangeRates));
                                
                                this.loadSettings();
                                this.updateAllDisplays();
                                this.populateCategories();
                                
                                this.showAlert('Данные успешно восстановлены!', 'success');
                            } else {
                                this.showAlert('Неверный формат файла резервной копии', 'danger');
                            }
                        } catch (error) {
                            this.showAlert('Ошибка при восстановлении данных', 'danger');
                        }
                    };
                    reader.readAsText(file);
                };
                
                input.click();
            }

            importData(mode) {
                const fileInput = document.getElementById(`importFile${mode === 'vacation' ? 'Vacation' : ''}`);
                const file = fileInput.files[0];

                if (!file) return;

                const reader = new FileReader();
                reader.onload = (e) => {
                    try {
                        const data = JSON.parse(e.target.result);
                        
                        if (data.budget && data.expenses) {
                            this.data[mode] = {
                                budget: data.budget,
                                expenses: data.expenses
                            };

                            if (data.customCategories) {
                                this.customCategories[mode] = data.customCategories;
                            }

                            this.saveData(mode);
                            this.updateDisplay(mode);
                            this.populateCategories();
                            this.showAlert('Данные успешно импортированы!', 'success');
                        } else {
                            this.showAlert('Неверный формат файла', 'danger');
                        }
                    } catch (error) {
                        this.showAlert('Ошибка при импорте данных', 'danger');
                    }
                };
                reader.readAsText(file);
                fileInput.value = '';
            }

            clearAllData(mode) {
                if (confirm('Вы уверены, что хотите удалить все данные? Это действие нельзя отменить.')) {
                    this.data[mode] = this.getDefaultData();
                    this.saveData(mode);
                    this.updateDisplay(mode);
                    this.showAlert('Все данные очищены', 'success');
                }
            }

            updateAnalytics() {
                const mode = document.getElementById('analyticsMode').value;
                const period = document.getElementById('analyticsPeriod').value;
                
                const expenses = this.getExpensesForPeriod(mode, period);
                this.displayCategoryAnalytics(expenses);
                this.createCharts(expenses, mode);
            }

            getExpensesForPeriod(mode, period) {
                const expenses = this.data[mode].expenses;
                
                if (period === 'all') return expenses;
                
                const days = parseInt(period);
                const cutoffDate = new Date();
                cutoffDate.setDate(cutoffDate.getDate() - days);
                
                return expenses.filter(expense => new Date(expense.date) >= cutoffDate);
            }

            displayCategoryAnalytics(expenses) {
                const categoryTotals = {};
                const totalAmount = expenses.reduce((sum, expense) => {
                    const baseAmount = this.convertAmount(expense.amount, expense.currency, this.baseCurrency);
                    categoryTotals[expense.category] = (categoryTotals[expense.category] || 0) + baseAmount;
                    return sum + baseAmount;
                }, 0);

                const categoryCards = Object.entries(categoryTotals)
                    .sort((a, b) => b[1] - a[1])
                    .map(([category, amount]) => {
                        const percentage = totalAmount > 0 ? (amount / totalAmount * 100).toFixed(1) : 0;
                        return `
                            <div class="summary-card">
                                <h3>${category}</h3>
                                <div class="amount formatted-number">${this.formatCurrency(amount, this.baseCurrency)}</div>
                                <div style="font-size: 12px; color: #666;">${percentage}%</div>
                            </div>
                        `;
                    }).join('');

                document.getElementById('categoryAnalytics').innerHTML = categoryCards || '<p>Нет данных для анализа</p>';
            }

            createCharts(expenses, mode) {
                this.createCategoryChart(expenses);
                this.createTimeChart(expenses, mode);
            }

            createCategoryChart(expenses) {
                const categoryTotals = {};
                expenses.forEach(expense => {
                    const baseAmount = this.convertAmount(expense.amount, expense.currency, this.baseCurrency);
                    categoryTotals[expense.category] = (categoryTotals[expense.category] || 0) + baseAmount;
                });

                const labels = Object.keys(categoryTotals);
                const data = Object.values(categoryTotals);
                const colors = [
                    '#FF6384', '#36A2EB', '#FFCE56', '#4BC0C0',
                    '#9966FF', '#FF9F40', '#FF6384', '#C9CBCF'
                ];

                const ctx = document.getElementById('categoryChart').getContext('2d');
                
                if (this.categoryChart) {
                    this.categoryChart.destroy();
                }

                this.categoryChart = new Chart(ctx, {
                    type: 'pie',
                    data: {
                        labels: labels,
                        datasets: [{
                            data: data,
                            backgroundColor: colors.slice(0, labels.length),
                            borderWidth: 2
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            legend: {
                                position: 'bottom'
                            },
                            tooltip: {
                                callbacks: {
                                    label: (context) => {
                                        const label = context.label || '';
                                        const value = this.formatCurrency(context.parsed, this.baseCurrency);
                                        const total = context.dataset.data.reduce((sum, val) => sum + val, 0);
                                        const percentage = ((context.parsed / total) * 100).toFixed(1);
                                        return `${label}: ${value} (${percentage}%)`;
                                    }
                                }
                            }
                        }
                    }
                });
            }

            createTimeChart(expenses, mode) {
                const dailyTotals = {};
                expenses.forEach(expense => {
                    const date = expense.date;
                    const baseAmount = this.convertAmount(expense.amount, expense.currency, this.baseCurrency);
                    dailyTotals[date] = (dailyTotals[date] || 0) + baseAmount;
                });

                const sortedDates = Object.keys(dailyTotals).sort();
                const data = sortedDates.map(date => dailyTotals[date]);

                const ctx = document.getElementById('timeChart').getContext('2d');
                
                if (this.timeChart) {
                    this.timeChart.destroy();
                }

                this.timeChart = new Chart(ctx, {
                    type: 'line',
                    data: {
                        labels: sortedDates.map(date => new Date(date).toLocaleDateString('ru-RU', { month: 'short', day: 'numeric' })),
                        datasets: [{
                            label: 'Траты по дням',
                            data: data,
                            borderColor: '#4BC0C0',
                            backgroundColor: 'rgba(75, 192, 192, 0.2)',
                            tension: 0.4,
                            fill: true
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            legend: {
                                display: false
                            },
                            tooltip: {
                                callbacks: {
                                    label: (context) => {
                                        return this.formatCurrency(context.parsed.y, this.baseCurrency);
                                    }
                                }
                            }
                        },
                        scales: {
                            y: {
                                beginAtZero: true,
                                ticks: {
                                    callback: (value) => this.formatNumber(value)
                                }
                            }
                        }
                    }
                });
            }

            showAlert(message, type) {
                const alertDiv = document.createElement('div');
                alertDiv.className = `alert ${type}`;
                alertDiv.textContent = message;
                
                document.body.appendChild(alertDiv);
                
                setTimeout(() => {
                    if (alertDiv.parentNode) {
                        alertDiv.parentNode.removeChild(alertDiv);
                    }
                }, 3000);
            }
        }

        // Global functions for HTML events
        const calculator = new BudgetCalculator();

        function switchTab(tab) {
            document.querySelectorAll('.tab-content').forEach(content => content.classList.add('hidden'));
            document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
            
            document.getElementById(`${tab}Tab`).classList.remove('hidden');
            const tabButtons = document.querySelectorAll('.tab');
            const tabIndex = { main: 0, vacation: 1, analytics: 2, settings: 3 }[tab];
            tabButtons[tabIndex].classList.add('active');
            
            calculator.currentTab = tab;
            
            if (tab === 'analytics') {
                calculator.updateAnalytics();
            } else if (tab === 'settings') {
                calculator.updateCategoriesList();
            }
        }

        function toggleSection(sectionId) {
            const section = document.getElementById(sectionId);
            const button = document.querySelector(`[onclick="toggleSection('${sectionId}')"]`);
            
            if (section.classList.contains('active')) {
                section.classList.remove('active');
                button.classList.remove('active');
            } else {
                section.classList.add('active');
                button.classList.add('active');
            }
        }

        function setBudget(mode) {
            calculator.setBudget(mode);
        }

        function addExpense(mode) {
            calculator.addExpense(mode);
        }

        function filterExpenses(mode) {
            calculator.filterExpenses(mode);
        }

        function clearFilters(mode) {
            calculator.clearFilters(mode);
        }

        function exportData(mode) {
            calculator.exportData(mode);
        }

        function importData(mode) {
            calculator.importData(mode);
        }

        function clearAllData(mode) {
            calculator.clearAllData(mode);
        }

        function updateAnalytics() {
            calculator.updateAnalytics();
        }

        function closeEditModal() {
            calculator.closeEditModal();
        }

        function saveExpenseEdit() {
            calculator.saveExpenseEdit();
        }

        function addCustomCategory() {
            calculator.addCustomCategory();
        }

        function setBaseCurrency() {
            calculator.setBaseCurrency();
        }

        function convertCurrency() {
            calculator.convertCurrency();
        }

        function createBackup() {
            calculator.createBackup();
        }

        function restoreBackup() {
            calculator.restoreBackup();
        }

        function exportToDoc() {
            calculator.exportToDoc();
        }

        function updateExchangeRates() {
            calculator.updateExchangeRates();
        }

        // Close modal when clicking outside
        window.onclick = function(event) {
            const modal = document.getElementById('editModal');
            if (event.target == modal) {
                closeEditModal();
            }
        }
    </script>
</body>

</html>
