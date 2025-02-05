# Case-study оптимизации

## Актуальная проблема
У нас в проекте были обнаружены следующие проблемы:

* А. ИМПОРТ ДАННЫХ - _загрзука в БД данных о рейсах происходит очень медленно(наивно и не эффективно)_
* Б. ОТОБРАЖЕНИЕ РАСПИСАНИЙ - _сами страницы расписаний тоже формируются не эффективно и при росте объемов на чинают сильно тормозить_

Я решил исправить обнаруженные проблемы оптимизировав загрузку данных и ускорив загрузку страниц расписаний.

Работать с обнаруженными проблемами я решил работать поэтапно.

---
## А. Импорт данных
### Формирование метрики
Для того, чтобы понимать, дают ли мои изменения положительный эффект на быстродействие программы я придумал использовать такую метрику:
* Время загрузки файлов
  * `small.json` (1K трипов) 
  * `medium.json` (10K трипов) 
  * `large.json` (100K трипов)
* Бюджет
  * загрузка файла `large.json` за `60сек`
    
### Гарантия корректности работы оптимизированной программы
Был написан тест `json_reloader_spec.rb`, который при использовании в фидбек-лупе позволяет не допустить изменения логики программы при оптимизации.

### Feedback-Loop
Для того, чтобы иметь возможность быстро проверять гипотезы я выстроил эффективный `feedback-loop`, который позволил мне получать обратную связь по эффективности сделанных изменений:
* `analyze` - работа с дешбордами, отчетами(их анализ)
* `optimize` - внесение изменений, оптимизация программы
* `benchmark` - замер времени выполнения 
* `feature test` - проверка тестом логики программы
* `commit/revert` - фиксация изменений или отказ от изменений

### Tools
Использовал следующие инструменты для анализа и обнаружения точек роста:
- [X] `мозг` и `zdennis/activerecord-import` для изменения порядка загрузки данных

### Iteration_1
* Сделал `benchmark` замер загрузки файла `small.json (1K трипов)` --- : **10.846510sec**
* Решение: переписать загрузку данных с помощью `zdennis/activerecord-import`
* Итого после изменения: 
  * файл `small.json (1K трипов)` --- : **3.012616sec** (было 10.846510sec)
  * файл `large.json (100K трипов)` удалось обработать за **35.892970sec** и уложиться в бюджет по первой проблеме(импорт данных).

### Performance regression protection
Для защиты от потери достигнутого прогресса при дальнейших изменениях программы были добавлены `perfomance-тесты` на обработку файлов small.json и large.json(по бюджету достигнутых значений).

---
## Б. Отображение расписаний
### Формирование метрики
Для того, чтобы понимать, дают ли мои изменения положительный эффект на быстродействие программы я придумал использовать такую метрику:
* Время рендеринга страницы `автобусы/Самара/Москва`

### Гарантия корректности работы оптимизированной программы
Добавил фича-тест `index_spec.rb`, чтобы  убедиться, что результат работы страницы `автобусы/Самара/Москва` для данных из файла `fixtures/example.json` не изменился, то есть не было внесено никаких функциональных изменений, только оптимизации.

### Feedback-Loop
Для того, чтобы иметь возможность быстро проверять гипотезы я выстроил эффективный `feedback-loop`, который позволил мне получать обратную связь по эффективности сделанных изменений:
* `analyze` - работа с дешбордами, отчетами(их анализ)
* `optimize` - внесение изменений, оптимизация программы
* `benchmark` - замер времени выполнения(рендеринга страницы)
* `feature test` - проверка тестом логики программы
* `commit/revert` - фиксация изменений или отказ от изменений

### Tools
Использовал следующие инструменты для анализа и обнаружения точек роста:
- [ ] `pghero`
- [ ] `new_relic`
- [ ] `rack-mini-profiler`
- [X] `bullet`
- [ ] `rails panel`
- [ ] `explain` запросов

### Iteration_1
* Начальный замер - рендеринг страницы `автобусы/Самара/Москва` с данными в объеме `fixtures/large.json`
===  71165ms (Views: 70026.8ms | ActiveRecord: 1003.5ms)
* Выскочил warning от bullet, предлагает добавить eager loading
* добавил `.includes(bus: :services)` при загрузке трипов
* Итого
=== 64572ms (Views: 64412.5ms | ActiveRecord: 30.4ms)
* ...не тот результат о котором я мечтал... продолжаю.
