# Moscow Estate Predictions
Предсказание цен на квартиры в Москве
## 📑 Данные
В датасете 3155 строк собранных с Аvito в 2022 году. Столбцы представляют собой:
 ```
 0   price_thousand        стоимость квартиры
 1   district              район Москвы
 2   living_area           жилая площадь
 3   kitchen_area          площадь кухни
 4   full_area             общая площадь
 5   n_rooms               число комнат
 6   plan                  вид планировки
 7   toilet                вид санузла
 8   lodge                 вид балкона
 9   n_floors              число этажей в доме
 10  floor                 этаж квартиры
 11  ceiling_h             высота потолка
 12  house                 вид дома
 13  design                дизайн квартиры
 14  year                  год постройки дома
 15  time_walk_to_station  время до метро пешком
 16  time_bus_to_station   время до метро на автобусе
 17  station               близжайшая станция метро
 18  n_stations_in_30_min  кол-во станций в 30 минутах ходьбы от квартиры
 19  view                  вид из окна
 ```
## 🤔 Exploratory data analysis
### 🧹 Очистка
Для эффективного анализа данных и дальнейшей работы с ними, нужно перевести их все в числовую форму. При этом, я обнаружил ряд нарушенных форматов (неразрывные пробелы, запятые как разделители тысяч и миллионов и т.п.), очевидно аномальных значений (2023 год постройки дома, 0 этажей, высота потолков в сантиметрах и т.п.) и дубликатов.
### ✍ Заполнение
Далее для заполнения пропусков
### 🏷️ Кодирование качественных переменных
## 📐 Выбор моделей
Будем выбирать из двух возможных моделей: линейной регрессии и CatBoost regression.
Обе модели показали дурные результаты, поэтому далее было решено построить систему регрессий.
Лучшим результатом стала
## 🧮 Итоги
