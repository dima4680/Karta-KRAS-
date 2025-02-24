<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Карта загруженности ПНП на КРАС ЖД</title>
    <script src="https://cdn.jsdelivr.net/npm/lz-string@1.4.4/libs/lz-string.min.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
        }

        #svg-container {
            width: 100vw;
            height: 100vh;
            overflow: auto;
            background: #f0f0f0;
        }

        .marker {
            cursor: pointer;
            transition: transform 0.2s;
        }

        .marker:hover {
            transform: scale(1.1);
        }

        .auth-section {
            position: fixed;
            top: 10px;
            left: 10px;
            background: rgba(255, 255, 255, 0.9);
            padding: 10px;
            border-radius: 5px;
            z-index: 1000;
        }

        .hidden {
            display: none;
        }

        .tooltip {
            position: fixed;
            background: white;
            padding: 10px;
            border-radius: 5px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
            pointer-events: none;
        }

        .controls {
            position: fixed;
            top: 80px;
            left: 10px;
            background: rgba(255, 255, 255, 0.9);
            padding: 10px;
            border-radius: 5px;
            z-index: 1000;
        }
    </style>
</head>
<body>
    <!-- Секция авторизации -->
    <div class="auth-section">
        <input type="text" id="username" placeholder="Логин">
        <input type="password" id="password" placeholder="Пароль">
        <button id="loginBtn">Войти</button>
        <button id="logoutBtn" class="hidden">Выйти</button>
    </div>

    <!-- Кнопки управления -->
    <div class="controls">
        <input type="file" id="excelFile" accept=".xlsx, .xls" class="hidden">
        <button id="saveDataBtn" class="hidden">Сохранить данные</button>
        <button id="shareDataBtn" class="hidden">Поделиться данными</button>
    </div>

    <!-- Контейнер для SVG -->
    <div id="svg-container">
        <svg id="main-svg" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 9934 7016" preserveAspectRatio="xMidYMid meet">
            <image href="https://i.ibb.co/d0jg4VgF/krasnoyarskaya-page-0001.jpg" width="9934" height="7016" />
        </svg>
        <div id="tooltip" class="tooltip hidden"></div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script>
        const SVG_NS = "http://www.w3.org/2000/svg";
        const svg = document.getElementById('main-svg');
        const tooltip = document.getElementById('tooltip');

        // Функция для создания маркера
        function createMarker(x, y, free, total) {
            const percentage = (free / total) * 100;
            let color = '#ffbb33';
            if (percentage === 0) color = '#ff4444';
            if (percentage === 100) color = '#00c851';

            const size = Math.min(70, Math.max(40, total * 0.7));

            const g = document.createElementNS(SVG_NS, 'g');
            g.setAttribute('class', 'marker');
            g.setAttribute('transform', `translate(${x},${y})`);

            const circle = document.createElementNS(SVG_NS, 'circle');
            circle.setAttribute('r', size / 2);
            circle.setAttribute('fill', color);

            const text = document.createElementNS(SVG_NS, 'text');
            text.setAttribute('text-anchor', 'middle');
            text.setAttribute('dy', '0.3em');
            text.setAttribute('fill', 'black');
            text.textContent = `${free}/${total}`;

            g.appendChild(circle);
            g.appendChild(text);

            return g;
        }

        // Функция для отображения данных
        function displayData(data) {
            // Очистка старых маркеров
            const oldMarkers = svg.querySelectorAll('.marker');
            oldMarkers.forEach(m => m.remove());

            data.forEach(row => {
                const x = parseFloat(row['Координата X']);
                const y = parseFloat(row['Координата Y']);
                const free = parseInt(row['Свободные вагоны']);
                const total = parseInt(row['Всего вагонов']);

                if (!isNaN(x) && !isNaN(y)) {
                    const marker = createMarker(x, y, free, total);
                    marker.dataset.info = JSON.stringify({
                        station: row['Станция'],
                        free,
                        total,
                        contacts: '8(391)259-54-70',
                        updated: '12.02.25'
                    });

                    // Обработчик наведения на маркер
                    marker.addEventListener('mousemove', (e) => {
                        tooltip.classList.remove('hidden');
                        const info = JSON.parse(marker.dataset.info);
                        tooltip.innerHTML = `
                            <b>${info.station}</b><br>
                            Свободно: ${info.free}/${info.total}<br>
                            В отстое: ${info.total - info.free}<br>
                            Контакты: ${info.contacts}<br>
                            Обновлено: ${info.updated}
                        `;
                        tooltip.style.left = `${e.clientX + 15}px`;
                        tooltip.style.top = `${e.clientY + 15}px`;
                    });

                    // Обработчик ухода курсора с маркера
                    marker.addEventListener('mouseleave', () => {
                        tooltip.classList.add('hidden');
                    });

                    svg.appendChild(marker);
                }
            });
        }

        // Функция для сжатия данных
        function compressData(data) {
            return LZString.compressToEncodedURIComponent(JSON.stringify(data));
        }

        // Функция для декомпрессии данных
        function decompressData(compressed) {
            try {
                return JSON.parse(LZString.decompressFromEncodedURIComponent(compressed));
            } catch (e) {
                alert('Ошибка декомпрессии данных: ' + e.message);
                return null;
            }
        }

        // Проверка авторизации
        function checkAuth() {
            const isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
            document.getElementById('excelFile').classList.toggle('hidden', !isLoggedIn);
            document.getElementById('logoutBtn').classList.toggle('hidden', !isLoggedIn);
            document.getElementById('loginBtn').classList.toggle('hidden', isLoggedIn);
            document.getElementById('saveDataBtn').classList.toggle('hidden', !isLoggedIn);
            document.getElementById('shareDataBtn').classList.toggle('hidden', !isLoggedIn);
        }

        // Обработчик входа
        document.getElementById('loginBtn').addEventListener('click', () => {
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;
            if (username === 'admin' && password === 'admin') {
                localStorage.setItem('isLoggedIn', 'true');
                checkAuth();
            } else {
                alert('Неверный логин или пароль');
            }
        });

        // Обработчик выхода
        document.getElementById('logoutBtn').addEventListener('click', () => {
            localStorage.setItem('isLoggedIn', 'false');
            checkAuth();
        });

        // Обработчик загрузки файла
        document.getElementById('excelFile').addEventListener('change', function (e) {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function (e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, { type: 'array' });
                const jsonData = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]]);
                localStorage.setItem('mapData', JSON.stringify(jsonData));
                displayData(jsonData);
            };
            reader.readAsArrayBuffer(file);
        });

        // Обработчик сохранения данных
        document.getElementById('saveDataBtn').addEventListener('click', () => {
            alert(localStorage.getItem('mapData') ? 'Данные сохранены!' : 'Нет данных');
        });

        // Обработчик кнопки "Поделиться данными"
        document.getElementById('shareDataBtn').addEventListener('click', async () => {
            const rawData = localStorage.getItem('mapData');
            if (!rawData) return alert('Нет данных для分享');

            try {
                const jsonData = JSON.parse(rawData);
                const compressed = compressData(jsonData);
                const shareUrl = `${window.location.href.split('?')[0]}?d=${compressed}`;

                // Копирование в буфер обмена
                try {
                    await navigator.clipboard.writeText(shareUrl);
                    alert('Сокращенная ссылка скопирована:\n' + shareUrl);
                } catch (err) {
                    const textarea = document.createElement('textarea');
                    textarea.value = shareUrl;
                    document.body.appendChild(textarea);
                    textarea.select();
                    document.execCommand('copy');
                    document.body.removeChild(textarea);
                    alert('Ссылка скопирована вручную:\n' + shareUrl);
                }
            } catch (e) {
                alert('Ошибка генерации ссылки: ' + e.message);
            }
        });

        // Загрузка данных из URL
        const urlParams = new URLSearchParams(window.location.search);
        const compressedData = urlParams.get('d');
        if (compressedData) {
            const jsonData = decompressData(compressedData);
            if (jsonData) {
                displayData(jsonData);
                document.querySelectorAll('.auth-section, #excelFile, #saveDataBtn, #shareDataBtn')
                    .forEach(el => el.classList.add('hidden'));
            }
        } else if (localStorage.getItem('mapData')) {
            displayData(JSON.parse(localStorage.getItem('mapData')));
        }

        // Проверка авторизации при загрузке страницы
        checkAuth();
    </script>
</body>
</html>
