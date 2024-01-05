# Анализ результатов A/Б эксперимента в приложении по доставке товаров

## Анализ приложения для доставки

Команда разработки приложения разработала новую умную систему рекомендации товаров, которая позволяет пользователям эффективнее работать с приложением и лучше находить необходимые товары. Для того, чтобы проверить эффективность новой системы приложений был проведен AB-тест, в котором пользователей разделили на контрольную (0) и тестовую (1) группы. Пользователи в контрольной группе пользовались старой версией приложения, а пользователи в тестовой группе - новой

**Правда ли что новая система рекомендаций улучшила качество сервиса?**

Давайте узнаем!

## Предварительный анализ данных

Перед основной работой с данными их необходимо "предварительно" просмотреть, чтобы сформировать первое представление о том, с каким вообще данными мы имеем дело. При поверхностном анализе полученных результатов были сделаны следующие выводы:

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

Видно, что данные собраны нормально и ошибок в алгоритме сбора данных не наблюдается, так как соблюдаются (по смыслу) уникальные значения, отсутствуют NULL-значения и типы данных также на местах. Также можем сказать, что размеры выборки приблизительно по **500 человек**

## Интерпретация результатов

Попробуем поразмыслить в сторону метрик в рамках используемой бизнес-модели. Улучшением качества сервиса для пользователя может считаться нахождение и, соответственно, заказ необходимого для него товара, что в свою очередь будет приносить прибыль для продукта - это win-win. С одной стороны, чем больше покупок сделает пользователь, тем больше прибыли сможет получить продукт, но есть одно большое НО! Если наша рекомендательная система работает таким образом, что предлагает пользователю необходимые ему товары, но по более низкой стоимости, то даже несмотря на увеличение количества покупок пользователями, прибыль для продукта не будет увеличиваться и может как оставаться на том же уровне, что и была, так и снижаться.

Нужно чтобы данная система удовлетворяла потребностям как пользователя, так и продукта. Это произойдет, если пользователь делает больше покупок и при этом стоимость его покупок не падает. Получается, что чем больше покупок по той же стоимости (или даже выше), тем больше прибыли получает продукт. Все просто!

Исследуемыми метриками в рамках анализа нашей платформы могут выступать:

- Количество покупок, сделанные пользователем
- Средний чек пользователя
- Прибыль с пользователя 

Таким образом, нужно проверить, что у нас **при увеличении количества покупок, увеличивается или остается неизменным средний чек пользователя. Все это ведет к увеличению прибыли, что мы также проверим. Profit!**

<img src = "https://assets-global.website-files.com/61bcbae3ae2e8e0565a790d1/62144c21b255406deefa0cff_image2.png" height = 400 width = 550>

## Метрика 1: Количество покупок

Для начала посмотрим, как новая система рекомендаций влияет на количество заказов, которые делает пользователь. В качестве покупки будем считать неотмененные заказы, то есть заказы без статусa "cancel_order". Сравнивать будем средние значения количества покупок в контрольной и тестовой группах. 

- Среднее количество покупок в контрольной группе = 364.25
- Среднее количество покупок в тестовой группе = 352.34

Для дальнейшего анализа нам необходимо понять, какой статистический критерий для сравнения средних мы будем использовать. У нас есть варианты в виде параметрического и непараметрического тестов, выбор которых будет зависеть от соблюдения определенных условий, необходимых для использования параметрического критерия.

- Независимость данных
- Нормальное распределение данных
- Отсутствие выбросов
- Гомогенность дисперсий

Проверим, соответствуют ли наши данные перечисленным условиям. Можно сказать, что независимость данных у нас соблюдается, так как количество покупок одного пользователя не влияет на количество покупок другого пользователя. Поэтому перейдем сразу к проверке нормальности распределения, которое будем анализировать как визуально, так и более профессиональными статистическими критериями.

#### Метрика 1: Гистограммы распределения

Для начала посмотрим гистограммы распределения выборочных данных. Это позволит сделать нам первое предположение о распределении 

<image src = "/images/quantities_hist.png" alt = "Гистограммы распределения количества покупок на пользователя" height = 400 width = 1300>

#### Метрика 1: Описательные статистики

Также давайте посмотрим на описательные статистики 2-х наборов данных, так как это тоже может нам сказать о потенциальном нормальном распределении. У нормального распределения медианное значение должно стремиться к среднему значению, коэффициент эксцесса стремится к 3, а коэффициент ассиметрии - к 0

| Группа | Среднее | Медиана | Ассиметрия | Эксцесс |
|:--------|:---------:|:---------:|:------------:|:---------:|
|Контрольная| 2.96   |    3.0    |      1.659      |       3.674 |
|Тестовая|   4.74     |    5.0     |       0.442     |   0.194      |

Исходя из результатов полученных гистограмм и описательных статистик, можно сказать, что на первый взгляд распределение данных не является нормальным. Однако, необходимо провести проверку более профессиональными статистическими критериями. 

#### Метрика 1: Статистические критерии

- Шапиро-Уилка (для выборок < 1000)
- Харке-Бера
- Д'агостино (Normal Test)

Для проверки выберем какой-нибудь один. В данном случае будем использовать метод Шапиро-Уилка, так как он наиболее чувствителен из всех представленных и параметры наших данных удовлетворяют условиям использования данного критерия

`stat, p = stats.shapiro(control_purchases)`

`stat, p = stats.shapiro(test_purchases)`

- Результаты контрольной группы: stat = 0.84, p_value = 1.47e-22
- Результаты тестовой группы: stat = 0.96, p_value = 2.07e-09

Согласно полученным результатам, у нас есть все основания отклонить нулевую гипотезу о равенстве нормальности распределения анализируемых данных и сказать, что с большой вероятностьб данные распределены ненормально. Наши статистические критерии подтвердили, что выборочные данные в обеих группах не распределены нормально. Из этого следует, что у нас есть несколько вариантов:

- Использовать непараметрический тест Манна-Уитни
- Преобразовать данные логарифмированием или преобразованием Бокса-Кокса и провести t-тест для сравнения средних 2-х независимых выборок
- Использовать t-критерий Стъюдента без каких-либо модификаций, так как размер нашей выборки > 100 позволяет нам так делать (да и вообще для t-теста распределение выборочных данных не обязательно должно быть нормальным)

Попробуем использовать t-критерий Стъюдента. Однако, чтобы использовать Стъюдента необходимо понять, одинаковая ли изменчивость в 2-х выборках или нет. Для этого будем использовать тест Левена, так как он более робастный к ненормальным распределениям, чем условный тест Бартлетта

`stat, p = stats.levene(control_purchases, test_purchases)`

Результаты теста Левена: stat = 16.843, p = 4.384e-05

Тест Левена показал, что у нас достаточно оснований отклонить нулевую гипотезу о равенстве дисперсий в 2-х выборках, а это значит, что, если мы будем использовать t-тест, то нужно брать модификацию Уэлча для данного теста.

#### Метрика 1: Достоверность результатов 

Очень важная вещь! Обычно, перед проведением AB-тестов заранее определяется размер выборки (от него зависит, как долго будет идти тест). Он определяется для определенных параметров:

- Уровень значимости 1 - alpha: вероятность не найти различия там, где их нет
- Мощность эксперимента 1 - beta: вероятность найти различия там, где они есть
- MDE - минимальный истинный эффект, который можно будет обнаружить с определенной мощностью эксперимента для определенного уровня статистической значимости

Так как у нас на руках уже финальные результаты, мы знаем размер выборки, мощность (как правило, берется на уровне 80, но чем выше, тем лучше) и уровень значимости (alpha = 0.05). Соотвественно, имея эти вводные мы можем рассчитать минимальный истинный эффект, который можно будет обнаружить с помощью критерия Стъюдента при таких параметрах alpha, beta и размере выборки

#### Метрика 1: Расчет MDE

`power_analysis = TTestIndPower()`

`nobs1 = users_data.loc[users_data.group == 0].user_id.nunique()`

`nobs2 = users_data.loc[users_data.group == 1].user_id.nunique()`

`ratio = nobs2 / nobs1`

`mde = power_analysis.solve_power(nobs1=nobs1, alpha=0.05, power=0.8, ratio=ratio)`

Минимальный детектируемый эффект при учете данных параметров уровня значимости, мощности и размере выборки = 0.18

При используемых в нашем случае данных, минимальный истинный эффект, который мы можем обнаружить c вероятностью 80% с такими параметрами alpha, beta и размер выборки n, равняется = 0.18. После того, как мы нашли наши средние по группам можно посчитать lift - насколько у нас изменилась метрика в тестовой группе относительно контрольной группы , а затем посчитать размер эффекта. В данном случае стандартизированный размер эффекта будет лучше отображать отличие тестовой группы от контрольной, так как будет учитывать выборочные дисперсии и размер выборок. 

#### Метрика 1: Lift

`lift = (test_purchases.mean() / control_purchases.mean()) - 1`

Согласно расчета относительный прирост количества покупок в тестовой группе относительно контрольной группы, составил 60.03%

#### Метрика 1: Standartized Effect Size

`control_var = control_purchases.var()`

`test_var    = test_purchases.var()`

`pooled_var  = calc_pooled_var(control_var, test_var, nobs1, nobs2)`

`effect_size = (test_purchases.mean() - control_purchases.mean()) / pooled_var`

Стандартизированный размер эффекта, выявленный в результате эксперимента =  0.88

В результате эксперимента мы получили, что размер эффекта, полученный в результате внедрения новой рекомендательной системы, для такой метрики, как количество покупок на пользователя, составляет 0.88. В данном случае он показывает степень отклонения наблюдаемых значений метрики от тех, которые можно было бы ожидать при отсуствии эффекта. Согласно таблице интерпретации размеров эффекта Коэна, полученный размер эффекта между 2-мя выборочными данными является довольно-таки значительным и уж точно больше, чем определяемый нами минимально детектируемый эффект при используемых параметрах уровня значимости, мощности и размера выборки. Из этого следует, что мы можем доверять ррезультатам нашего A/Б-тестирования. Теперь можем провести сравнение 2-х средних значений с помощью T-критерия Уэлча, так как дисперсии в 2-х выборках не являются однородной

#### Метрика 1: Сравнение 2-х средних t-критерием Уэлча

`stat, p = stats.ttest_ind(control_purchases, test_purchases, equal_var = False)`

Результаты сравнения 2-х средних: stat = -14.005, p = 8.93e-46

Результаты сравнения 2-х средних значений критерием Уэлча показали, что между средними значениями существует статистически значимое различие и есть все основания, чтобы отклонить нулевую гипотезу о равенстве средних. Из этого следует, что в группе с новой системой рекомендации товаров среднее количество покупок, совершаемых пользователем выше, чем в группе со старым алгоритмом.

## Метрика 2: Средний чек пользователя

Теперь проверим, траты пользователей, так как количество покупок может вырасти, но покупаемые товары станут дешевле. Например, пользователь мог раньше покупать 1 товар по 300 рублей, а теперь покупает 2 товара по 150 рублей или 3 товара за 100 рублей - в таком случае количество покупок выросло, но средний чек пользователя стал меньше. Для нашей компании это может быть невыгодно, так как теперь наши курьеры больше загружены (расходы могут стать больше), но прибыли с этого больше не стало. Поэтому нужно выяснить, как изменились средний чек для наших пользователей

Для того, чтобы посмотреть средние траты пользователя необходимо сначала подготовить данные. Для этого нужно объединить данные по пользователям и данные по заказам и товарам, рассчитать нужный показатель для пользователя, а после разбить данные на группы (разбивка последним шагом, чтобы не делать расчет 2 раза). 

#### Метрика 2: Сравнение метрик отношения дельта-методом

В данном случае мы будем работать с метриками отношения (ratio-метриками) и для их сравнения мы не сможем использовать простой t-критерий Стъюдента, так как присутствуют зависимые данные. Данные о покупках разных пользователей с большой вероятностью не являются зависимыми, но данные о покупках одного пользователя (присутствуют в наших данных) вероятнее всего будут зависимы. Обычный критерий t-Стъюдента не сработает, потому что он не учитывает дисперсию зависимых данных.

Для проведения сравнения ratio-метрик можно использовать Bootstrap, но на больших данных он будет очень ресурсозатратен. Мы будем использовать дельта-метод для метрик отношения, в котором будет следующая формула оценки дисперсии зависимых данных

<image src = "/images/delta-method.png" alt = "Формула оценки дисперсии зависимых данных, используемая в дельта-методе оценки сравнения 2-х ratio-метрик" height = 400 width = 1000>

После оценки дисперсии по вышеуказанной формуле, проводим сравнение 2-х средних значений по формуле ниже

<image src = "/images/Student's_Criteria.svg" alt = "Формула расчета t-критерия для сравнения 2-х ratio-метрик с использованием формулы оценки дисперсии для сравнения зависимых данных" height = 400 width = 1000>

Результаты сравнения 2-х средних чеков в контрольной и тестовой группах: p_value =  0.25

Исходя из результатов сравнения 2-х средних чеков (ratio-метрик) при помощи определения дисперсии зависимых данных дельта-методом, можно сделать вывод, что статистически значимые различия между средними чеками в контрольной и тестовой группах отсутствуют и у нас недостаточно оснований отклонить нулевую гипотезу об их равенстве. Можно предположить, что средний чек пользователя в 2-х группах не отличается друг от друга. Тогда мы знаем, что количество покупок на пользователя в тестовой группе преобладает, и следовательно средняя прибыль с пользователя должна в тестовой группе также быть выше. Давайте проверим эту гипотезу!

<image src = "/images/dwight_meme.gif" alt = "Мем с Дуайтом Шрутом">

## Метрика 3: Прибыль с пользователя

Как в случае со средним значением количества покупок гипотезу о равенстве средних значений прибыли в контрольной и тестовой группах мы будем проверять одним из параметрических или непараметрических вариантов. Для того, чтобы определлиться с методом необходимо посмотреть, удовлетворяют ли данные в обеих группах условиям для проведения сравнения параметрическим критерием, а именно t-критерием Стъюдента

- Независимость данных
- Нормальность распределения
- Отсутствие выбросов
- Гомогенность дисперсий

Сперва проверим соответствуют ли распределения данных из контрольной и тестовых групп нормальному распрелелению. Проверять будем различными способами, как графическими, так и статистическими критериями.

#### Метрика 3: Гистограммы распределений

<image src = "/images/revenue_hist.png" alt = "Гистограммы частот для распределения прибыли, которую приносит пользователь в контрольной и тестовой группах" height = 400 width = 1300>

#### Метрика 3: Описательные статистики

Также давайте посмотрим на описательные статистики 2-х наборов данных, так как это тоже может нам сказать о потенциальном нормальном распределении. 

| Группа | Среднее | Медиана | Ассиметрия | Эксцесс |
|:--------|:---------:|:---------:|:------------:|:---------:|
|Контрольная| 1132.92   |    951.19    |      1.534     |       3.691 |
|Тестовая|   1750.25     |    1628.89     |       0.514     |   -0.171      |

#### Метрика 3: QQ-Plots

<image src = "/images/revenue_qq_plots.png" alt = "Квантиль-квантиль графики для распределения прибыли, которую приносит пользователь в контрольной и тестовой группах">

На основании результатов построенной выше визуализации, а также результатов описательных статистик, можно сделать предварительный вывод о том, что распределение данных о прибыли, которую приносит пользователь, не соответствует нормальному, как в контрольной, так и в тестовой группах. Проведем еще одну проверки статистическим критерием Шапиро-Уилка (выбрали его, так как он наиболее чувствителен среди остальных критериев)

#### Метрика 3: Статистические критерии

`shapiro = {
    'control_p' : stats.shapiro(control_revenue)[1],
    'test_p' : stats.shapiro(test_revenue)[1]
}`

Критерий Шапиро-Уилка продемонстрировал следующие результаты:
-  Контрольная группа: stat = 0.891, p_value = 1.232e-18
-  Тестовая группа: stat = 0.976, p_value = 2.221e-07

Результаты проверки данных с помощью критерия Шапиро-Уилка говорят о том, что у нас есть все основания отклонить нулевую гипотезу о нормальности распределения данных как в контрольной, так и в тестовой группах. Для дальнейшего анализа данных мы можем использовать как непараметрический критерий Манна-Уитни, так и все еще придерживаться стратегии использования t-критерия Стъюдента для сравнения средней прибыли с пользователя - размер выборки позволяет нам это сделать, да и распределение наших данных не такие уж ассиметричные. Однако, необходимо понять, что у нас с гомогенностью дисперсий в 2-х выборках и для этого мы будем использовать критерий Левена, так как он более робастный к отклонениям от нормальности, нежели критерий Бартлетта

`stats.levene(control_revenue, test_revenue)`

Критерий Левена продемонстрировал следующие результаты: stat = 8.406, p_value = 1.955e-05

Проверка на нормальность и гомоскедастичность наших выборочных данных показала, что данные распределены ненормально и дисперсии между выборками статистически значимо различаются. Следовательно для оценки среднего используем модификацию t-критерия Уэлча

`stat, p = stats.ttest_ind(control_revenue, test_revenue, equal_var = False)`

T-критерий Уэлча продемонстрировал следующие результаты: stat = -11.256, p_value = 9.627e-28

Можно сделать вывод, что среднее значение прибыли в контрольной и тестовой группах статистически значимо различаются, и в тестовой группе значение прибыли выше, чем в контрольной. Существует важный момент, связанный с достоверностью результатов сравнения: необходимо, как и в случае с предыдущими метриками, выяснить получившийся размер эффекта для сравниваемых показателей в друх группах и сравнить его с минимальным истинным эффектом, чтобы понять, можем ли мы вообще доверять результатам дальнейшего сравнения при используемых параметрах alpha, beta и размер выборки n.

#### Метрика 3: Lift

Для начала рассчитаем относительное изменение значения прибыли в тестовой группе относительно контрольной:

`lift = (lift = (test_revenue.mean() / control_revenue.mean()) - 1`

Относительное изменение прибыли после введения нового алгоритма составило: 54.49 %. Теперь рассчитаем стандартизированный размер эффекта, который более наглядно показывает изменения, так как учитывает изменчивость в данных. 

#### Метрика 3: Effect Size

`control_var = control_revenue.var()`
`test_var    = test_revenue.var()`
`pooled_var  = calc_pooled_var(control_var, test_var, nobs1, nobs2)`
`effect_size = (test_revenue.mean() - control_revenue.mean()) / pooled_var`

Стандартизированный размер эффекта, выявленный в результате эксперимента: 0.71

Согласно шкале интерпретации эффекта Коэна - это довольно-таки большой эффект. Сравнивая его с MDE (0.18) можем заключить, что вероятность обнаружения такого эфффекта при используемых параметрах довольно-таки высокая, а значит мы можем доверять результатам нашего AB-тестирования. Можно сделать вывод, что прибыль и правда выросла за счет того, что люди стали совершать больше покупок, а не за счет покупок более дорогих товаров.

<image src = "/images/mask_meme.jpeg" alt = "Мем с Маской">

## Выводы

На основе полученных результатов, можно сделать следующий вывод. Анализ показал, что в тестовой группе пользователи делают больше заказов, нежели в контрольной группе. При этом также было доказано, что данное отклонение количества покупок в тестовой группе от такой же метрики в контрольной группе, можно обнаружить с довольно-таки большой вероятностью с используемыми параметрами уровня значимости, мощности и размере выборки. 

При сравнении средних чеков в 2-х группах дельта-методом было выяснено, что различия средних чеков в контрольной и тестовой группах не являются статистически значимыми и нет достаточных оснований отклонить нулевую гипотезу об их равенстве. Далее, мы сделали предположение, что различий между средними чеками в 2-х группах правда может не быть, а так как количество покупок в тестовой группе увеличилось, что можно предположить, что вырастет и общая прибыль с пользователя. Сравнение средних значений в 2-х группах также подвердило эту гипотезу (effect_size < MDE, p_value <0.05). Следовательно, пользователи в тестовой группе не просто покупают больше товаров, а еще и делают это по той же стоимости, что и пользователи в контрольной группе. В результате, это приносит платформе больше прибыли, что так же было статистически подтверждено. Поэтому можем сказать, что новая рекомендательная система очень полезна как для пользователя (теперь он чаще находит необходимые ему товары), так и для продукта (приносит ему дополнительную прибыль). Можно провести еще одно тестирование, но с выборкой большего размера, и если результаты анализа также покажут, что различий между средними чеками в 2-х группах нет, то внедрять ее в production
















