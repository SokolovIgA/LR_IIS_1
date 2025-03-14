# Описание проекта

В данной работе описывается создание виртуальной среды и настройка git-репозитория для проекта. Также на этом этапе проводится разведочный анализ данных.
---
# Запуск проекта

**git clone https://github.com/SokolovIgA/LR_IIS_1**

**cd (директория с проектом)**

**python3 -m venv .venv_LR_proj**

**source .venv_LR_proj/bin/activate**

**pip install -r requirements.txt**

---

# Исследование данных

В результате исследования:

  Изменен тип данных с числового на категориальный для 9 столбцов (sex, cp, fbs, restecg, exang, slope, ca, thal, target)
  
  Изменен тип данных с int64 на int8/int16 для 4 столбцов (age, trestbps, chol, thalach)
  
  Изменен тип данных с float164 float16 для 1 столбца (oldpeak)
  
  Удален признак 'chol', 'fbs'  вследствие неинформативности.

При построении графиков были выделены следующие закономерности:

   Из графика распределения возраста пациентов видно, что большинство из них находятся в возрасте от 42 до 70 лет (см. график: .eda/histogram_age.png)
   
   Из графика распределение пола в зависимости от наличия заболевания - заболевания у мужчин встречаются чаще чем у женщин, но вероятность заболеть у женщин больше, чем у мужчин. (см. график: .eda/countplot_sex_target.png)
  
   
   Из графика систолического давления по налицию заболевания видно, что вероятность заболевания у людей, с систолическим давлением меьшн среднего. (см. график: .eda/boxplot_trestbps_target.png)
   
   Из графика зависимости уровня холестерина по налицию заболевания - уровень холестерина достаточно неинформативный признак для определения целевой переменной и линейная зависимость ненаблюдается, есть явня вероятность заболеваний, если холестирин сильно выше среднего,   что должно быть очевидно. (см. график: ./eda/violinplot_chol_target.png)
   
   Из графика заисимости максимальной частоты сердечного ритма от возраста - достаточно хорошо прослеживается обратная заисимость между возрастом и максимальным середчным ритмом, при этом можно отметить, что наблюдения с более высоким сердречным ритмом чаще поджвержены заболеваниям. (см. график: ./eda/graph5.png)
   
  Из графика матрицы корреляций видно, что:
  
  	Наиболее высокая положительная корреляция наблюдается между 'cp' (тип боли в груди) и 'target' (наличие заболевания), 'thalach' (максимальная частота сердечных сокращений) имеет высокую отрицательную корреляцию с 'target',
	 что говорит о том, что пациенты с более высокой частотой сердечных сокращений имеют меньший риск 	заболеваний.
	 
	Уровень холестерина ('chol') имеет положительную корреляцию с целевой переменной, что может указывать на то, что более высокий уровень холестерина ассоциируется с более высоким риском заболеваний.
	 'oldpeak' (депрессия ST) также имеет положительную корреляцию с 'target', что указывает на его важность как "признака риска заболеваний.
	 
	Другие признаки, такие как 'age' и 'trestbps' (систолическое давление), также имеют умеренные корреляции с 'target', что может указывать на их влияние на риск сердечно-сосудистых заболеваний.. (см. график: ./eda/correlation_heatmap.png) 
  

**Начальный объем датасета: 11.3 KB —> Итоговый объем датасета: 6.9 KB**

# Результаты исследования

В рамках исследования было разработано несколько моделей машинного обучения на методе **RandomForestClassifier** (**классификатор**). Все были обучены достаточно хорошо и имели высокий процент точности, но наилучшие показатели были достигнуты с моделью, использующей расширенный набор признаков (применение **PolynomialFeatures**) и оптимизированными параметрами, которые были найдены с помощью библиотеки **Optuna**, данный метод создает исследование, нацеленное на максимизацию F1-score. Optuna запускает 30 испытаний, каждый раз с новым набором гиперпараметров, после чего выбирает наилучшие параметры.
## Оценка качества наилучшей модели
Метрики, полученные при обучении модели с наилучшими параметрами, следующие:

- **Точность (Accuracy):** 0.8928571428571429
- **Точность (Precision):** 0.9411764705882353
- **Полнота (Recall):** 0.8888888888888888
- **F1-Score:** 0.9142857142857143
- **ROC-AUC:** 0.9388888888888889

### Оптимальные гиперпараметры модели, найденные методом **Optuna**:
- **n_estimators:** 183 
- **Глубина дерева (max_depth):** 49 
- **max_features:** 0.37997222517951745 

**Идентификатор итоговой модели:** 2877ebce28504b268555d41f58b7fb57

---
### Описание разработанного сервиса
Этот сервис предоставляет API для выполнения предсказаний с использованием модели машинного обучения. Модель была обучена ранее, сохранена через MLflow, и интегрирована в данный сервис.

**Основные папки и файлы:**
- `ml_service/`:
  - `main.py`: основной модуль приложения FastAPI.
  - `api_handler.py`: класс-обработчик API запросов, содержащий логику работы с моделью и предсказаниями.
  - `requirements.txt`: файл зависимостей, необходимых для работы сервиса.
  - `Dockerfile`: инструкции для сборки Docker-образа сервиса.
- `models/`:
  - `get_model.py`: скрипт для загрузки модели из MLflow.
  - `model.pkl`: сериализованная модель машинного обучения, используемая сервисом.
  
### Команды для создания Docker-образа
Сборка образа выполняется с помощью команды:
```
docker build . --tag disease_model:0
```

### Команда для запуска контейнера
Для запуска контейнера с пробросом порта и подключением модели выполните следующую команду:
```
docker run -p 8001:8000 -v $(pwd)/../models:/models disease_model:0
```
### Проверка работоспособности сервиса
1. Перейдите на страницу документации FastAPI по адресу:
   [http://localhost:8001/docs](http://localhost:8001/docs)

2. Выполните запрос к эндпоинту `/api/prediction`:
   **Пример тела запроса**:
```  
{"patient_id": 24,
"age":20,
"sex":1,
"cp":2,
"trestbps":125,
"restecg":2,
"thalach":156,
"exang":0,
"oldpeak":3.5,
"slope":0,
"ca":0,
"thal":1 
}
```
   **Тело ответа**:
```  
{
  "disease": "0",
  "patient_id": 24
}
```
### Описание мониторинга сервиса

**Описание разработанных сервисов**:
   - **Grafana**:
     - Сервис мониторинга и визуализации данных.
     - Включает в себя файлы для конфигурации, а также дашборды для отображения метрик.
     - Веб-интерфейс доступен по адресу `http://localhost:3000`.
     - Для доступа используется логин и пароль: `admin:admin`.

   - **Prometheus**:
     - Сервис для сбора и хранения метрик.
     - Конфигурационный файл для Prometheus расположен в директории `./services/prometheus`.
     - Веб-интерфейс доступен по адресу `http://localhost:9090`.
     - Prometheus собирает метрики сервиса предсказаний и отображает их через графики.

   - **Сервис отправки запросов (requests)**:
     - Скрипт, который генерирует случайные запросы к сервису предсказаний с промежутками времени от 0 до 5 секунд.
     - Позволяет протестировать стабильность и нагрузку на сервис предсказаний.
     
Для сборки и запуска проекта с использованием Docker Compose, используйте команду, находясь в папке `services`:

```
docker compose up
```

Пример мониторинга сервиса:

Гистограмма предсказаний:
![Гистограмма предсказаний](photo/gist.png)

Частота запросов к сервису:
![Частота запросов к сервису](photo/rate.png)

Запросы со статусом 4хх и 5хх:
![Запросы со статусом 4хх и 5хх](photo/err.png)

Дашборд с использованием Grafana:

Панель:
![Панель](photo/1.png)

Прикладные графики:
 - Частота запросов в минуту.
 - Кол-во предсказаний модели.
 - Статус запросов к сервису.
 
 Инфраструктурные графики:
 - Используемая оперативная память.
 - Время использования процессора. 
 
Графики качества модели:
 - Гистограмма предсказаний.
 
