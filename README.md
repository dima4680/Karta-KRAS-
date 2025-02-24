<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Карта загруженности ПНП на КРАС ЖД</title>
    <style>
        /* Стили для улучшения качества отображения изображения */
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f4f4f4; /* Цвет фона */
        }

        .image-container {
            max-width: 100%; /* Адаптивность */
            text-align: center;
        }

        img {
            max-width: 100%; /* Изображение будет адаптироваться под контейнер */
            height: auto; /* Сохранение пропорций */
            image-rendering: crisp-edges; /* Для сохранения четкости при масштабировании */
        }

        h1 {
            font-family: Arial, sans-serif;
            margin-bottom: 20px;
        }
    </style>
</head>
<body>
    <div class="image-container">
        <h1>Карта загруженности ПНП на КРАС ЖД</h1>
        <img src="https://i.ibb.co/d0jg4VgF/krasnoyarskaya-page-0001.jpg" alt="Карта загруженности ПНП на КРАС ЖД">
    </div>
</body>
    <!-- Секция авторизации -->
    <div class="auth-section">
        <input type="text" id="username" placeholder="Логин">
        <input type="password" id="password" placeholder="Пароль">
        <button id="loginBtn">Войти</button>
        <button id="logoutBtn" class="hidden">Выйти</button>
    </div>

    <!-- Заголовок -->
    <h1 style="position: fixed; top: 10px; left: 300px; z-index: 1000; background: rgba(255, 255, 255, 0.9); padding: 10px; border-radius: 5px;">
        Карта загруженности ПНП на КРАС ЖД
    </h1>

    <!-- Кнопки управления -->
    <div style="position: fixed; top: 80px; left: 10px; z-index: 1000; background: rgba(255, 255, 255, 0.9); padding: 10px; border-radius: 5px;">
        <input type="file" id="excelFile" accept=".xlsx, .xls" class="hidden">
        <button id="saveDataBtn" class="hidden">Сохранить данные</button>
        <button id="shareDataBtn" class="hidden">Поделиться данными</button>
    </div>

    <!-- Элементы управления зумом -->
    <div class="zoom-controls">
        <button id="zoomInBtn">+</button>
        <button id="zoomOutBtn">-</button>
    </div>

    <!-- Контейнер для карты -->
    <div id="map-container">
        <img id="map-image" src="https://i.ibb.co/d0jg4VgF/krasnoyarskaya-page-0001.jpg" alt="Карта">
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script>
        // Размеры оригинального изображения карты
        const IMAGE_WIDTH = 9934; // Ширина вашего изображения
        const IMAGE_HEIGHT = 7016; // Высота вашего изображения

        // Контейнер и изображение карты
        const container = document.getElementById('map-container');
        const mapImage = document.getElementById('map-image');

        // Масштабирование
        let scale = 1; // Начальный масштаб
        const ZOOM_FACTOR = 0.2; // Шаг масштабирования

        // Функция для обновления масштаба
        function updateScale(newScale) {
            scale = Math.max(0.5, Math.min(3, newScale)); // Ограничиваем масштаб от 0.5x до 3x
            mapImage.style.transform = `scale(${scale})`;
        }

        // Обработчики кнопок зума
        document.getElementById('zoomInBtn').addEventListener('click', () => {
            updateScale(scale + ZOOM_FACTOR);
        });

        document.getElementById('zoomOutBtn').addEventListener('click', () => {
            updateScale(scale - ZOOM_FACTOR);
        });

        // Функции для работы с координатами
        function getRelativeCoordinates(x, y) {
            const rect = mapImage.getBoundingClientRect();
            return {
                x: (x / IMAGE_WIDTH) * rect.width * scale,
                y: (y / IMAGE_HEIGHT) * rect.height * scale
            };
        }

        // Функции для маркеров
        function getMarkerColor(free, total) {
            const percentage = (free / total) * 100;
            if (percentage === 0) return '#ff4444';
            if (percentage === 100) return '#00c851';
            return '#ffbb33';
        }

        function getMarkerSize(total) {
            const minSize = 40;
            const maxSize = 70;
            const minWagons = 10;
            const maxWagons = 100;
            return Math.min(maxSize, Math.max(minSize, 
                ((total - minWagons) / (maxWagons - minWagons)) * (maxSize - minSize) + minSize
            ));
        }

        // Отображение данных
        function displayData(data) {
            // Удаляем старые маркеры
            document.querySelectorAll('.custom-marker').forEach(m => m.remove());

            data.forEach(row => {
                const x = parseFloat(row['Координата X']); // Преобразуем в число
                const y = parseFloat(row['Координата Y']); // Преобразуем в число
                const station = row['Станция'];
                const free = parseInt(row['Свободные вагоны']);
                const total = parseInt(row['Всего вагонов']);

                if (!isNaN(x) && !isNaN(y) && !isNaN(free) && !isNaN(total)) {
                    const pos = getRelativeCoordinates(x, y);
                    const size = getMarkerSize(total);

                    const marker = document.createElement('div');
                    marker.className = 'custom-marker';
                    marker.style.left = `${pos.x}px`;
                    marker.style.top = `${pos.y}px`;
                    marker.innerHTML = `
                        <div class="marker-circle" 
                            style="
                                width: ${size}px;
                                height: ${size}px;
                                background: ${getMarkerColor(free, total)};
                                font-size: ${size * 0.4}px;
                            ">
                            ${free}/${total}
                        </div>
                    `;

                    // Добавляем обработчик клика
                    marker.querySelector('.marker-circle').addEventListener('click', () => {
                        alert(`
                            ${station}\n
                            Свободно: ${free} из ${total} вагонов\n
                            В отстое: ${total - free} вагонов\n
                            Контакты для связи: 8(391)259-54-70\n
                            Обновлено: 12.02.25
                        `);
                    });

                    container.appendChild(marker);
                }
            });
        }

        // Сжатие данных
        function compressData(data) {
            return LZString.compressToEncodedURIComponent(JSON.stringify(data));
        }

        // Декомпрессия данных
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
        document.getElementById('excelFile').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
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

        // Инициализация
        window.addEventListener('resize', updateImageSize);
        updateImageSize();

        // Проверка авторизации при загрузке страницы
        checkAuth();
    </script>
</body>
</html>
