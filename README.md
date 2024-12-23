# Проект. Улучшение baseline-модели

**Задача** - улучшить основную метрику проекта, которая влияет на точность предсказаний стоимости недвижимости и, как следствие, на количество успешных сделок на маркетплейсе.

**Цель** — сделать процесс воспроизводимым и улучшить ключевые модельные метрики, которые влияют на бизнес-показатели компании, в частности, на увеличение количества успешных сделок.

**Имя бакета:** s3-student-mle-20240921-750d983cc8

**Начало работы с проектом:**

1. git clone *https://github.com/RumKam/mle-project-sprint-2-v001.git*
2. pip install -r requirements.txt
3. тетрадка с исследованием cd model_improvement/project_template_sprint_2.ipynb

*https://github.com/RumKam/mle-project-sprint-2-v001/blob/main/model_improvement/project_template_sprint_2.ipynb*

**Этап 1: Разворачивание MLflow и регистрация модели**

1. Для запуска MLFlow необходимо перейти в папку с shell файлом *cd mlflow_server*
2. Далее запустить сервер командой *sh run_mlflow.sh*
3. EXPERIMENT_NAME = 'real_estate_project_#2'
4. EXPERIMENT_ID = 21

- RUN_NAME = 'base_model'
- Метрика на базовой модели: rmse = 2404247.7


**Этап 2: Проведение EDA**

RUN_NAME = 'feature_statistics'

В ходе исследования данных были проанализированы типы данных, количество пропусков, основные статистики численных признаков и численность в категориальных данных.

Основные выводы:
1. Признаки is_apartment и studio бесполезны. В первом признаке сильный дисбаланс, а в studio одно значение.
2. Цена жилья растет с увеличением площади и этажности.
3. Цена на жилье четко делится в цене по году постройки, старый фонд (до 1960 года) имеет более высокую цену.
4. Сам целевой признак имеет нормальное распределение.
5. Наиболее широко представ слелующий тип жилья:
    - без лифта;
    - тип 4;
    - 5, 9 и 17 этажные строения;
    - квартиры находятся до 5 этажа;
    - квартиры с 2 комнатами;
    - жилая площадь до 40 кв. метров;
    - высота потолков 2,6 м, в старом фонде 3 м.

**Этап 3: Генерация признаков и обучение модели**

- RUN_ID = 'feature_generation_model'
-  Полученная метрика с сгенерированными признаками: rmse = 2381652.5

Вручную сгенерированы признаки:
- 'build_year': категория года постройки. До 1960 года, с 1960 по 1990, с 1990 до 2000 и новые после 2000 года постройки.
- 'floor_builder': категория этажности здания - 5-этажное, 7, 9 и 17.
- 'floor_type': категория этажа, первый, последний и остальные.
- 'kmeans_cluster': разделение на 5 районов на основе геоданных.
- 'distance_to_center: расстояние от дома до центра города.

Автоматически сгенерированы признаки:
- при помощи PolynomialFeatures() и KBinsDiscretizer() для признаков 'ceiling_height', 'total_area' (если сгенерировать для всех числовых, дальнейшие этапы отбора и обучения занимают более 4 часов - это лимит беспрерывной работы вм).
- при помощи AutoFeatRegressor() и функций 'exp' и 'sqrt' для признаков "rooms" и "living_area".

**Этап 4: Отбор признаков и обучение новой версии модели**

- RUN_NAME = 'feature_selected_model'
- Полученная метрика на отобранном наборе признаков: rmse = 2391802.5

Отбор признаков при помощи SFS двумя способами: 15 признаков с пустого набора и 15 с полного. Далее отобранные признаки были объеденены. Отбор признаков производился на случайном наборе 1000 строк тренировочного датасета из-за ограничений по времени работы вм (запуск полного датасета занимает более 2 часов).

Итоговоый набор признаков:
['cat__building_type_int_2',
 'cat__cluster_region_2',
 'cat__build_type_by_year_medium_2000',
 'num__rb__rooms',
 'num__rb__floor',
 'num__rb__ceiling_height',
 'cat__build_type_by_floors_other_level',
 'cat__floor_type_other_level',
 'exp(-living_area + sqrt(rooms))',
 'num__rb__latitude',
 'num__rb__longitude',
 'num__pol__ceiling_height total_area',
 'exp(sqrt(living_area) - exp(rooms))',
 'cat__building_type_int_4',
 'cat__building_type_int_3',
 'num__rb__distance_to_center',
 'cat__cluster_region_4',
 'num__rb__build_year',
 'num__pol__total_area^2',
 'num__kbd__total_area',
 'exp(rooms)',
 'num__rb__floors_total',
 'num__pol__ceiling_height^2',
 'cat__cluster_region_1']

**Этап 5: Подбор гиперпараметров и обучение новой версии модели**

- RUN_NAME = 'random_gs_best_model'
- Подбор параметров случайным методом по метке. Итоговая метрика: rmse = 2680502.6


- RUN_NAME = 'optuna_model'
- Подбор параметров при помощи optuna. Лучшая метрика: rmse = 2424998.7

- RUN_NAME = 'best_model_optuna'
- Финальная модель с отобранными optuna гиперпараметрами: : rmse = 2447826.9

  В итоге, лучшая модель получена на полном наборе сгенерированных данных, с параметрами модели по умолчанию.



