# Анализ результатов A/Б эксперимента в приложении по доставке товаров

Анализ приложения для доставки
Команда разработки приложения разработала новую умную систему рекомендации товаров, которая позволяет пользователям эффективнее работать с приложением и лучше находить необходимые товары. Для того, чтобы проверить эффективность новой системы приложений был проведен AB-тест, в котором пользователей разделили на контрольную (0) и тестовую (1) группы. Пользователи в контрольной группе пользовались старой версией приложения, а пользователи в тестовой группе - новой

**Правда ли что новая система рекомендаций улучшила качество сервиса?**

Давайте узнаем!

## Предварительный анализ данных

Перед основной работой с данными их необходимо "предварительно" посмотреть, чтобы сформировать первое представление о том, с каким вообще данными мы имеем дело. При поверхностном просмотре были сделаны следующие выводы:

#### Данные о пользователях

- Количество строк: **4337**
- Количество столбцов: **6**

| Название столбца         |  Что значит | Тип данных | Уникальность в таблице| Количество в дата-фрейме| NULL-значения|
|:------------- |:---------------| :-------------:| :--:| :--:|:--: |
| user_id          | идентификатор пользователя          | Int64       | Нет| 1017 | Нет|
| order_id         | идентификатор заказа         | Int64       |Нет | 4123|Нет|
| action         | действие пользователя          | String       | Нет|2 |Нет|
| time         | время действия          | Datetime64        | Нет|4312 |Нет|
| date        | дата действия          | Datetime64        | Нет| 14|Нет|
| group       | группа          | Int64        | Нет| 2|Нет|

#### Данные о заказах

- Количество строк: **4123**
- Количество столбцов: **3**

| Название столбца         |  Что значит | Тип данных | Уникальность в таблице| Количество в дата-фрейме| NULL-значения|
|:------------- |:---------------| :-------------:| :--:| :--:|:--: |
| order_id         | идентификатор заказа         | Int64       |Да | 4123|Нет|
| сreation_time         | создание заказа          | Datetime64       | Нет|4098 |Нет|
| product_ids       | идентификаторы товаров в заказа          | Object (list)        | Нет| 3877|Нет|

#### Данные о товарах

- Количество строк: **87**
- Количество столбцов: **3**

| Название столбца         |  Что значит | Тип данных | Уникальность в таблице| Количество в дата-фрейме| NULL-значения|
|:------------- |:---------------| :-------------:| :--:| :--:|:--: |
| product_id         | идентификатор товара         | Int64       |Да | 87|Нет|
| name        | имя товара          | String       | Да|87 |Нет|
| price       | цена товара          | Float64        | Нет| 63|Нет|

Видно, что данные собраны нормально и, кажется, что не было ошибок в алгоритме сбора данных, так как соблюдаются (по смыслу) уникальные значения, отсутствуют NULL-значение и типы данных также на местах. Также можем сказать, что размеры выборки приблизительно по **500 человек**

## Интерпретация результатов

Попробуем поразмыслить в сторону метрик в рамках монетизации. Улучшением качества сервиса для пользователя может считаться нахождение неоходимого ему товара, что в свою очередь принесет прибыль нашему продукту - это win-win. С одной стороны, чем больше покупок сделает пользователь, тем больше прибыли сможет получить продукт, но есть одно большое НО! Если наша рекомендательная система работает таким образом, что предлагает пользователю необходимые ему товары, но в несколько раз дешевле, то даже не смотря на увеличение количества покупок пользователями, прибыль для продукта не будет увеличиваться (если вообще не уменьшиться).

Нужно чтобы было данная система удовлетворяла потребностям и пользователя и продукта. Это можно устроить - если пользователь делает больше покупок и при этом стоимость его покупок не падает. Получается, что чем больше покупок, тем больше прибыль продукту. Все просто!

Соответственно, метриками в рамках монетизации нашей платформы могут выступать:

- Количество покупок, сделанные пользователем 
- Средний чек пользователя
- Прибыль с пользователя 

Таким образом, нужно проверить, что у нас **при увеличении количества покупок, увеличивается или остается неизменным средний чек пользователя. Все это ведет к увеличению прибыли. Profit!**


<img src = "https://assets-global.website-files.com/61bcbae3ae2e8e0565a790d1/62144c21b255406deefa0cff_image2.png">

## 1. Количество покупок на пользователя

Для начала посмотрим, как новая система рекомендаций влияет на количество покупок на пользователя. В качестве покупки будет считать заказ без статусa "cancel_order". Сравнивать будем средние значения количества покупок в контрольной и тестовой группах. 

- **Среднее количество покупок в контрольной группе = 364.25**
- **Среднее количество покупок в тестовой группе = 352.34**

Для дальнейшего анализа нам необходимо понять, какой статистический критерий для сравнения мы будем использовать. У нас есть варианты в виде параметрического и непараметрического тестов, выбор которых будет зависеть от соблюдения определенных условий. 

- Нормальное распределение данных
- Отсутствие выбросов
- Гомогенность дисперсий

Проверим, соответствуют ли наши данные перечисленным условиям. Для начала проверим нормальность распределения сразу несколькими способами. 

#### 1.1 Гистограммы распределения

Для начала посмотрим гистограммы распределения выборочных данных, чтобы понять, как вообще распределены данные. Это позволит сделать нам первое предположение о распределении 

<image src = "/images/quantities_hist.png" alt = "Гистограммы распределения количества покупок на пользователя">

#### 1.2 Описательные статистики

Также давайте посмотрим на описательные статистики 2-х наборов данных, так как это тоже может нам сказать о потенциальном нормальном распределении. 

| Группа | Среднее | Медиана | Ассиметрия | Эксцесс |
|:--------|:---------:|:---------:|:------------:|:---------:|
|Контрольная| 2.96   |    3.0    |      1.659      |       3.674 |
|Тестовая|   4.74     |    5.0     |       0.442     |   0.194      |

Исходя из результатов полученных гистограмм и описательных статистик, можно сказать, что на первый взгляд распределение данных не является нормальным. Однако, необходимо провести проверку более профессиональными статистическими критериями. 

#### 1.3 Статистические критерии

- Шапиро-Уилка (для выборок < 1000)
- Харке-Бера
- Д'агостино (Normal Test)

Для проверки выберем какой-нибудь один. В данном случае будем использовать метод Шапиро-Уилка, так как он наиболее чувствителен из всех представленных

`shapiro = {
    'control_p' : stats.shapiro(control_purchases)[1],
    'test_p' : stats.shapiro(test_purchases)[1]
}`

- **Результаты контрольной группы: stat = 0.84, p_value = 1.47e-22**
- **Результаты тестовой группы: stat = 0.96, p_value = 2.07e-09**

Наши статистические критерии подтвердили, что выборочные данные в обеих группах не распределены нормально. Из этого следует, что у нас есть несколько вариантов:

- Использовать непараметрический тест Манна-Уитни
- Преобразовать данные логарифмированием или преобразованием Бокса-Кокса и провести T-Test
- Использовать T-Test в любом случае, так как размер нашей выборки > 100 и позволяет нам так делать (да и вообще для t-теста распределение выборочных данных не обязательно должно быть нормальным)

Попробуем использовать T-критерий Стъюдента. Однако, чтобы использовать Стъюдента необходимо понять, одинаковая ли изменчивость в 2-х выборках или нет. Для этого будем использовать тест Левена, так как он более робастный к ненормальным распределениям, чем условный тест Бартлетта

`stat, p = stats.levene(control_purchases, test_purchases)`

**Результаты теста Левена: stat = 16.843, p = 4.384e-05**


### Достоверность результатов 

Очень важная вещь! Обычно, перед проведением AB-тестов заранее определяется размер выборки (от него зависит, как долго будет идти тест). Он определяется для определенных параметров:

- Уровень значимости 1 - alpha: вероятность не найти различия там, где их нет
- Мощность эксперимента 1 - beta: вероятность найти различия там, где они есть
- MDE - минимальный истинный эффект, который можно будет обнаружить с определенной мощностью эксперимента для определенного уровня статистической значимости

Так как у нас на руках уже финальные результаты, мы знаем размер выборки, мощность теста (стандартный) и уровень значимости (стандартный). Соотвественно, имея эти вводные мы можем рассчитать минимальный истинный эффект, который можно будет обнаружить с помощью критерия Стъюдента при таких параметрах alpha, beta и размере выборки

### Расчет MDE

`power_analysis = TTestIndPower()`

`nobs1 = users_data.loc[users_data.group == 0].user_id.nunique()`

`nobs2 = users_data.loc[users_data.group == 1].user_id.nunique()`

`ratio = nobs2 / nobs1`

`mde = power_analysis.solve_power(nobs1=nobs1, alpha=0.05, power=0.8, ratio=ratio)`

**Минимальный детектируемый эффект при учете данных параметров уровня значимости, мощности и размере выборки = 0.18**

При используемых в нашем случае данных, минимальный истинный эффект, который мы можем обнаружить c вероятностью 80% с такими параметрами alpha, beta и размер выборки n, равняется = 0.18. Согласно приведенной ниже таблице - эффект довольно-таки слабый

<image src = "/images/cohens_effect_ranges.png" alt = "Шкала Коэна с размерами эффекта">




