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
Для эффективного анализа данных и дальнейшей работы с ними, нужно перевести их все в числовую форму. При этом, я обнаружил ряд **нарушенных форматов** (неразрывные пробелы, запятые как разделители тысяч и миллионов и т.п.), очевидно **аномальных значений** (2023 год постройки дома, 0 этажей, высота потолков в сантиметрах и т.п.) и **дубликатов**.

Также, заменим колонку _year_ на _age_, чтобы было проще выполнять линейную регрессию.
### ✍ Заполнение
Теперь перейдём к заполнению пропущённых (NaN) и нелогично-нулевых значений (например, в time_bus_to_station и time_walk_to_station). Для этого используем библиотеку **fancyimpute** и её методы **KNN, NuclearNormMinimization, IterativeSVD**, которые, однако, оказались хуже, чем заполнение руками.

При заполнении вручную, я подбирал строки со схожими значениями, а также заполнял time_bus_to_station на основе time_walk_to_station и наоборот.
### 🏷️ Кодирование качественных переменных
Применим несолько разных методов кодирования, проверяя каждый на простейшей модели линейной регрессии, и выберем наилучший на основе совокупности метрик RMSLE, RMSE и Adjusted R^2. Среди рассмотренных вариантов были:
- Кодирование всего через **OneHotEncoder**;
- Кодирование всего через **BinaryEncoder**;
- Кодирование переменных district и station через **LabelEncoder** и всех остальных - через **OneHotEncoder**;
- **Смешанное** с учётом дисбаланса классов.

Последний вариант показал наилучший результат. Он состоит из следующих этапов:
1. Ручное кодирование переменных, в которых есть малочисленные классы (например, вид дома).
2. Кодирование переменных district и station через **LabelEncoder**
3. Кодированеи оставшихся переменных через **OneHotEncoder**
### 👽 Обнаружение аномалий
После этого разделим выборки на тренировочную, тестовую и валидационную; проведём масштабирование на [0,1]; переименуем колонки по единому шаблону. Далее необходимо очистить данные от аномалий, потому как они снизят предсказательные способности модели. Аномалии действительно присутствуют в данных, это видно, например, по скрипичному графику цены:

![image](https://github.com/ForgotWhatItWas/MoscowEstatePredictions/assets/134389286/69f89cc8-bccb-425f-8339-3fdc3be67047)

Среди рассмотренных вариантов избавления от аномалий были:
- **DBSCAN**;
- **IsolationForest**;
- **Квартильное** исключение;
- **LocalOutlierFactor**;
- **SVM**;
- **EllipticEnvelope**.

Последний вариант показал наилучший результат (тестирование проводилось аналогично выбору лучшего способа кодирования):

![image](https://github.com/ForgotWhatItWas/MoscowEstatePredictions/assets/134389286/416e8e7b-a8ff-4653-a585-61658d5f66aa)
## 📐 Выбор моделей
Будем выбирать из двух возможных моделей: линейной регрессии и CatBoost regression.

Изначально, обе модели, несмотря на кропотливую работу по подготовке данных, показывали кране низкие результаты (так, Adjusted R^2 мог иметь отрицательные значения). Применяя grid search для catboost удалось повысить показатели модели до **Adjusted R^2 = 0.35** и **RMSE = 18.536**. Для линейной регрессии же были достигнуты результаты в **Adjusted R^2 = 0.38** и **RMSE = 18.076** после устранения мультиколлинеарности, а также гетероскедастичности по одной из переменных (автокоррелированность остатков первого порядка отсутствовала).

Наличие **мультиколлинеарности** очевидно из матрицы корреляций объясняющих признаков:

![image](https://github.com/ForgotWhatItWas/MoscowEstatePredictions/assets/134389286/0974c14a-5f4a-40e3-a136-577d6d433787)

Так, видно, что показатели, связанные с площадью и числом комнат сильно коррелированны. Также наблюдается естественная корреляция между числом станций и временем до метро пешком/на автобусе. Помимо этого, наблюдается связь между некоторыми качественными переменными. Устранять её будем путём регрессии с исключением переменных.

Наличие **гетероскедастичности** очевидно из теста Голдфелда-Квандта, показывающего, что у следующих переменных присутствует гетероскедастичность:
```
living_area p =  0.005
kitchen_area p =  5.006e-10
full_area p =  8.8e-06
n_rooms p =  0.01
n_floors p =  0.002
ceiling_h p =  3.7e-17
n_stations_in_30_min p =  0.01
age p =  0.004
house_panel p =  5.1e-05
```

Отсутствие **автокоррелированности** остатков первого порядка очевидно из результата теста Дарбина-Уотстона, равного 2.064.

Обе модели показали неважные результаты, поэтому далее было решено построить систему регрессий: для каждого района Москвы будет строиться своя модель. В случае с CatBoost - будут подбираться свои гиперпараметры через grid search, в случае линейной регрессии - кроме подбора параметров, будет устраняться гетероскедастичность и мультиколлинеарность.

Catboost показал более плохой результат, чем без системы моделей: **RMSE = 19.85, Adjusted R^2 = 0.25.** При этом, вторая модель наоборот - улучшила показатели.
## 🧮 Итоги и перспективы работы
Лучшей моделью стала система линейных регрессиий с устранением гетероскедастичности и мультиколлинеарности. Её показатели на выборках:
|  | Валидационная | Тестовая |
| ------------- | ------------- | ------------- |
| RMSLE  | 1 | 2  |
| RMSE =   | 1 | 2 |
| Adjusted R^2 =   | 1 | 2 |

Данный результат меня удовлетворяет, модель обладает достаточной степенью предсказательных способностей.

Одним из возможных способов дальнейшего улучшения модели может стать **подбор нелинейной функции регрессии**. Так, при проведении эксперементов (до формирования ситемы регрессий), логарифмическая и степенная трансформации показали хороший результат с **R-squared = 0.48**. Однако, модель отдавала чрезмерное предпочтение свободному члену в уравнении регрессии, что снижает её адекватность.

Также, можно **заменить проблемные переменные**, такие как district и station, на количественные (например, расстояние до центра города).

Помимо этого, можно добавить такие переменные, как **кол-во и вид центров притяжения** жителей (кафе, университеты, офисы и т.п.), которые, вероятно, помогут объяснить ряд аномальных значений.
