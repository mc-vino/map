<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Интерактивная карта ПВЗ с радиусами</title>
    <script src="https://api-maps.yandex.ru/2.1/?apikey=e9ae29b2-121b-4f83-89ce-dfe6e3df7e50&lang=ru_RU" type="text/javascript"></script>
    <style>
        body, html {
            width: 100%; height: 100%; margin: 0; padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
        }
        #controls {
            padding: 15px;
            background-color: #ffffff;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 10;
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
        }
        .control-group {
            padding: 10px;
            border: 1px solid #eee;
            border-radius: 8px;
            background-color: #fcfcfc;
        }
        .group-title {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
            color: #333;
        }
        input, select, button {
            padding: 6px 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            outline: none;
        }
        button {
            background-color: #ffdb4d; /* Яндекс-желтый */
            border: none;
            cursor: pointer;
            font-weight: 500;
            transition: background 0.2s;
        }
        button:hover {
            background-color: #ffd200;
        }
        #map {
            width: 100%;
            flex-grow: 1;
        }
    </style>
</head>
<body>

    <div id="controls">
        <div class="control-group">
            <span class="group-title">Управление радиусами (метров)</span>
            <div style="margin-bottom: 5px;">
                Общий: <input type="number" id="radiusAll" value="1000" style="width: 80px;">
                <button onclick="changeRadius('all')">ОК</button>
            </div>
            <div>
                Гр. A (синий): <input type="number" id="radiusGroupA" value="1000" style="width: 60px;"> 
                Гр. B (красный): <input type="number" id="radiusGroupB" value="1000" style="width: 60px;">
                <button onclick="changeRadius('groupA')">Сет A</button>
                <button onclick="changeRadius('groupB')">Сет B</button>
            </div>
        </div>

        <div class="control-group">
            <span class="group-title">Добавить новый ПВЗ по адресу</span>
            <input type="text" id="newAddress" placeholder="Напр: Москва, ул. Арбат, 1" style="width: 250px;">
            <select id="newGroup">
                <option value="groupA">Группа A</option>
                <option value="groupB">Группа B</option>
            </select>
            <input type="number" id="newRadius" value="1000" style="width: 80px;" placeholder="Радиус">
            <button onclick="addNewPoint()">Добавить</button>
        </div>
    </div>

    <div id="map"></div>

    <script type="text/javascript">
        let myMap;
        let pvzData = []; 

        ymaps.ready(init);

        function init() {
            myMap = new ymaps.Map("map", {
                center: [55.755864, 37.617698],
                zoom: 11,
                controls: ['zoomControl', 'searchControl', 'fullscreenControl']
            });

            // Тестовые точки
            addPvzByCoordinates([55.755864, 37.617698], 'groupA', 'Центральный ПВЗ', 1000);
        }

        function addPvzByCoordinates(coords, group, name, radius) {
            let color = (group === 'groupA') ? '#0000FF' : '#FF0000';
            
            // Метка
            let placemark = new ymaps.Placemark(coords, {
                balloonContent: `<b>${name}</b><br>Группа: ${group}<br>Радиус: ${radius}м`
            }, {
                preset: (group === 'groupA') ? 'islands#blueCircleDotIcon' : 'islands#redCircleDotIcon'
            });

            // Визуальный радиус
            let circle = new ymaps.Circle([coords, radius], {}, {
                fillColor: color + "33", // Добавляем прозрачность
                strokeColor: color,
                strokeOpacity: 0.6,
                strokeWidth: 2
            });

            myMap.geoObjects.add(placemark);
            myMap.geoObjects.add(circle);

            pvzData.push({
                placemark: placemark,
                circle: circle,
                group: group,
                radius: radius
            });
        }

        function addNewPoint() {
            const address = document.getElementById('newAddress').value;
            const group = document.getElementById('newGroup').value;
            const radius = Number(document.getElementById('newRadius').value);

            if (!address) return alert("Введите адрес");

            ymaps.geocode(address).then(function (res) {
                let obj = res.geoObjects.get(0);
                if (obj) {
                    let coords = obj.geometry.getCoordinates();
                    addPvzByCoordinates(coords, group, address, radius);
                    myMap.setCenter(coords, 14);
                    document.getElementById('newAddress').value = '';
                } else {
                    alert("Адрес не найден");
                }
            });
        }

        function changeRadius(target) {
            let inputId = (target === 'all') ? 'radiusAll' : (target === 'groupA' ? 'radiusGroupA' : 'radiusGroupB');
            let newRadius = Number(document.getElementById(inputId).value);

            if (isNaN(newRadius) || newRadius <= 0) return alert("Некорректный радиус");

            pvzData.forEach(item => {
                if (target === 'all' || item.group === target) {
                    item.circle.geometry.setRadius(newRadius);
                    item.radius = newRadius;
                }
            });
        }
    </script>
</body>
</html>
