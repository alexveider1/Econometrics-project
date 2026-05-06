# Влияние высшего образования на здоровье

<div align="center">

[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![uv](https://img.shields.io/badge/managed%20by-uv-DE5FE9?style=flat)](https://github.com/astral-sh/uv)
[![DoubleML](https://img.shields.io/badge/DoubleML-0.11-4CAF50?style=flat)](https://docs.doubleml.org/)

**НИУ ВШЭ · Факультет экономических наук · Эконометрика‑2 (углубленный курс)**

</div>

---

## О проекте

Данный проект воспроизводит и расширяет исследование Е. Коссовой и М. Косоруковой [«Оценивание влияния высшего образования на здоровье: сравнение многомерной рекурсивной пробит-модели и мэтчинга»](/data/article.pdf). В проекте было применено три независимых подхода к идентификации причинно-следственного эффекта.

**Ключевой вопрос:** снижает ли наличие диплома о высшем образовании вероятность хронических заболеваний, и если да — насколько и для кого?

Анализ проводится раздельно для **мужчин** (1 926 наблюдений) и **женщин** (2 672 наблюдения) по **16 переменным заболеваний** и показателю **самооценки здоровья индивида**.

---

## Методы

Все три метода применяются к одним и тем же данным, что позволяет верифицировать устойчивость результатов. Однако в Probit модели в данные были добавлены инструментальные переменные, взятые на основе региональных показателей.

### 1 Двухшаговый метод контрольной функции на основе Probit модели

Control Function Approach для устранения **эндогенности** переменной образования: более образованные люди могут вести здоровый образ жизни не только благодаря знаниям, но и вследствие ненаблюдаемых характеристик (воспитание, среда, интеллект).

**Инструментальные переменные** (региональный уровень):

| Инструмент | Экзогенный источник вариации |
|---|---|
| Потребление алкоголя на душу населения | Региональный алкогольный рынок |
| Продажи сигарет на душу взрослого населения | Региональный табачный рынок |
| Число зарегистрированных браков на 1000 чел. | Демографическая среда |
| Рейтинговый балл приверженности ЗОЖ | Культура здоровья в регионе |

Для каждой из 5 эндогенных переменных (`diploma`, `mar_st`, `alcohol`, `smoking`, `phys_active`) на первом шаге строится пробит-модель и вычисляются обобщённые остатки; на втором шаге они включаются в основное уравнение здоровья. Экзогенность проверяется тестом Вальда. Всего оценено **34 модели** (17 исходов × 2 пола).

### 2 Мэтчинг

Метод подбора «двойников» для устранения смещения из-за самоотбора. Реализованы две метрики близости:

- Propensity Score Matching
- Mahalanobis distance Matching

Оценка эффекта лечения на сбалансированных подвыборках — **регрессия Ферта** (Firth logit), которая даёт устойчивые оценки при разреженных бинарных исходах.

### 3 Double ML

- Три базовых алгоритма: **LDA**, **Random Forest** и **XGBoost** (гиперпараметры оптимизируются через `OptunaSearchCV`)  
- Оцениваются и **ATE**, и **ATET** при уровне значимости $\alpha = 0.1$  


---

## Данные

**Источник:** РМЭЗ НИУ ВШЭ, волна 33 (файл `r33iall_84.sav`)  
**Итоговая выборка:** 4 598 наблюдений · 31 переменная · без пропусков

### Переменные

| Категория | Переменные |
|---|---|
| **Лечение** | `diploma` — наличие диплома о высшем образовании |
| **Исходы здоровья** | `heart`, `lungs`, `liver`, `kidneys`, `stomach`, `spine`, `diabetes`, `hypertension`, `joints`, `ENT_organs`, `neurology`, `eyes`, `allergy`, `veins`, `skin`, `oncology`, `is_health_good` |
| **Контрольные** | `age`, `income`, `n_child`, `type_area`, `invalid`, `mar_st`, `visit_doctor`, `work`, `alcohol`, `smoking`, `phys_active` |

### Распределение выборки

```
Всего: 4 598    Мужчины: 1 926 (41.9%)    Женщины: 2 672 (58.1%)
Доля с дипломом: 38.8%
Наиболее распространённые заболевания: гипертония (21.4%), суставы (14.9%), позвоночник (14.3%)
```

---

## Структура проекта

```
.
├── assets/                     # Графики и таблицы
│   ├── dml_forest_plot_ate.png     # Forest plot (ATE)
│   ├── dml_forest_plot_atet.png    # Forest plot (ATET)
│   ├── dml_volcano_plot.png        # Volcano plot
│   ├── dml_propensity_score.png    # Overlap propensity scores
│   ├── cor_map.png                 # Корреляционная матрица заболеваний
│   ├── disease_count.png           # Распространённость заболеваний
│   ├── disease_age_distr.png       # Заболевания по возрасту
│   ├── disease_income_distr.png    # Заболевания по доходу
│   ├── disease_spread_by_sex.png   # Заболевания по полу
│   ├── disease_spread_by_diploma.png # Заболевания по наличию высшего образования
│   ├── eda_hist_cont.png # Распределение небинарных переменных
│   ├── eda_hist_bin.png # Распределение бинарных переменных
│   ├── men.tex                     # LaTeX-таблица результатов (мужчины)
│   └── women.tex                   # LaTeX-таблица результатов (женщины)
│
├── data/                       # Данные
│   ├── r33iall_84.sav              # Исходные данные РМЭЗ НИУ ВШЭ (полная выборка)
│   ├── final_dataset.csv           # Обработанный датасет
│   ├── article.pdf                 # Статья, на которую опирались
│   ├── project_requirements.pdf    # Требования к проекту
│   └── R33_ind_codebook.pdf        # Документация опроса
│
├── data_preparation/           # ETL и EDA
│   ├── data_processing.ipynb       # Очистка и формирование датасета
│   └── data_exploration.ipynb      # EDA
│
├── double_ml/                  
│   └── double_ml.ipynb             # DoubleMLIRM: LDA / RF / XGB, ATE + ATET
│
├── matching/                  
│   ├── matching.ipynb              # PSM + Mahalanobis + Firth logit
|   └── matching_description.txt    # Описание мэтчинга
│
├── probit/                     
│   ├── probit.ipynb                # Control Function Approach, 34 модели
|   ├── probit_dataset.csv          # Специальный датасет для пробита
|   └── probit_text.txt             # Описание пробита
│
├── text.tex                    # Текст в LaTeX
├── text.pdf                    # Текст в pdf
├── presentation.pptx           # Презентация проекта
├── pyproject.toml              # Зависимости проекта
├── .python-version             # Зависимости проекта
└── uv.lock                     # Зависимости проекта
```

---

## Установка и запуск

### Установка зависимостей

```bash
git clone https://github.com/alexveider1/Econometrics-project.git
cd Econometrics-project

uv sync
```

```bash
uv run jupyter lab
```

**Рекомендуемый порядок запуска:**

```
data_preparation/data_processing.ipynb -> формирование final_dataset.csv
data_preparation/data_exploration.ipynb -> eda
probit/probit.ipynb -> двухшаговый пробит
matching/matching.ipynb -> PSM + Mahalanobis
double_ml/double_ml.ipynb -> Double ML
```

## Лицензия

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
