<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Сайт объявлений с картой 2ГИС</title>
    <meta name="description" content="Сайт для размещения объявлений о грузах" />
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
    <style>
        html, body {
            margin: 0;
            font-family: 'Inter', sans-serif;
            background-color: #0a192f;
            color: #333;
            overflow: hidden;
            height: 100%;
        }

        #container {
            display: flex;
            height: 100%;
        }

        #sidebar {
            width: 300px;
            background: rgba(255, 255, 255, 0.9);
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            overflow-y: auto;
        }

        #map {
            flex: 1;
            height: 100%;
            border-radius: 0 8px 8px 0;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }

        input, textarea {
            width: calc(100% - 30px);
            padding: 10px;
            margin: 5px 0;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        button {
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        button:hover {
            background-color: #0033cc;
        }

        h2 {
            margin-top: 0;
        }

        ul {
            list-style: none;
            padding: 0;
        }

        li {
            background: #e9ecef;
            margin: 5px 0;
            padding: 10px;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <div id="container">
        <div id="sidebar">
            <h2>Fast Help</h2>
            <input type="text" id="title" placeholder="Название груза" required />
            <textarea id="description" rows="4" placeholder="Адрес, Имя, Номер" required></textarea>
            <button id="getLocationButton">Добавить объявление</button>
            <input type="text" id="location" placeholder="Местоположение IP адрес" required />
            <button id="addAdButton">Дайте разрешение на Местоположение</button>
            <h2>Регистрация Водителя</h2>
            <input type="text" id="driverName" placeholder="Имя водителя" required />
            <input type="text" id="driverPhone" placeholder="Номер телефона" required />
            <button id="registerDriverButton">Зарегистрировать водителя</button>          
            <h2>Объявления</h2>
            <ul id="cargo-list"></ul>
            <h2>Водители</h2>
            <ul id="driversList"></ul>
        </div>

        <div id="map"></div>
    </div>

    <script src="https://mapgl.2gis.com/api/js/v1"></script>
    <script>
        const map = new mapgl.Map('map', {
            center: [77.4400000, 42.1659900],
            zoom: 12,
            key: 'a96cbd91-bcc5-4b23-9a64-37fe49cbe979', // Замените на ваш API ключ 2ГИС
        });

        const drivers = []; // Массив для хранения зарегистрированных водителей

        function displayDrivers() {
            document.getElementById('driversList').innerHTML = ''; // Очищаем список водителей

            drivers.forEach(driver => {
                const driverListItem = document.createElement('li');
                driverListItem.textContent = `${driver.name} - ${driver.phone}`; // Имя и номер телефона водителя
                document.getElementById('driversList').appendChild(driverListItem);

                // Добавление маркера на карту
                const driverMarker = new mapgl.Marker(map, {
                    coordinates: driver.coordinates,
                    icon: 'https://img.icons8.com/ios/50/car.png',
                    scale: 1
                });

                // Обработчик клика для маркера водителя
                driverMarker.on('click', () => {
                    alert(`Водитель: ${driver.name}, телефон: ${driver.phone}`);
                });
            });
        }

        document.getElementById('getLocationButton').onclick = function() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(function(position) {
                    const lat = position.coords.latitude;
                    const lng = position.coords.longitude;

                    // Устанавливаем координаты в поле местоположения
                    document.getElementById('location').value = `${lat}, ${lng}`;
                    
                    // Центрируем карту на текущем местоположении
                    map.setCenter([lng, lat]);

                    // Добавление маркера на карту
                    new mapgl.Marker(map, {
                        coordinates: [lng, lat],
                        icon: 'https://img.icons8.com/ios/50/warehouse.png',
                        scale: 1
                    });
                }, function() {
                    alert("Ошибка при определении местоположения.");
                });
            } else {
                alert("Ваш браузер не поддерживает геолокацию.");
            }
        };

        document.getElementById('addAdButton').onclick = function() {
            const title = document.getElementById('title').value;
            const description = document.getElementById('description').value;
            const location = document.getElementById('location').value;

            if (!title || !description || !location) {
                alert("Пожалуйста, заполните все поля.");
                return;
            }

            // Добавление нового объявления в список
            const listItem = document.createElement('li');
            listItem.textContent = `${title}: ${description} - ${location}`;
            document.getElementById('cargo-list').appendChild(listItem);

            // Геокодирование местоположения
            fetch(`https://geocode-maps.yandex.ru/1.x/?apikey=ВАШ_API_ГЕОКОДИРОВАНИЯ_КЛЮЧ&geocode=${encodeURIComponent(location)}&format=json`)
                .then(response => response.json())
                .then(data => {
                    const coordinates = data.response.GeoObjectCollection.featureMember[0].GeoObject.Point.pos.split(' ');
                    const lat = parseFloat(coordinates[1]);
                    const lng = parseFloat(coordinates[0]);

                    // Добавление маркера на карту
                    new mapgl.Marker(map, {
                        coordinates: [lng, lat],
                        icon: 'https://img.icons8.com/ios/50/warehouse.png',
                        scale: 1
                    });

                    // Центрирование карты на маркере
                    map.setCenter([lng, lat]);
                })
                .catch(error => {
                    console.error('Ошибка получения координат:', error);
                    alert('Не удалось найти местоположение.');
                });

            // Очистка формы
            document.getElementById('title').value = '';
            document.getElementById('description').value = '';
            document.getElementById('location').value = '';
        };

        // Регистрация водителя
        document.getElementById('registerDriverButton').onclick = function() {
            const driverName = document.getElementById('driverName').value;
            const driverPhone = document.getElementById('driverPhone').value;

            if (!driverName || !driverPhone) {
                alert("Пожалуйста, заполните все поля для регистрации водителя.");
                return;
            }

            // Временные координаты для маркера водителя (например, фиксированные для примера)
            const driverCoordinates = [77.4400000 + Math.random() * 0.01, 42.1659900 + Math.random() * 0.01];

            // Добавляем нового водителя в массив
            drivers.push({
                name: driverName,
                phone: driverPhone,
                coordinates: driverCoordinates
            });

            // Очищаем поля ввода для регистрации
            document.getElementById('driverName').value = '';
            document.getElementById('driverPhone').value = '';

            // Обновляем список водителей
            displayDrivers(); // Обновляем отображение водителей на странице
        };

        // Инициализация отображения водителей (пока пустой)
        displayDrivers();
    </script>
</body>
</html>
