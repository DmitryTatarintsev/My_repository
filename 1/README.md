# <a href='https://github.com/DmitryTatarintsev/Other-Projects/blob/main/5/5.ipynb'>Обработка фотографий покупателя</a>
## Описание проекта
Нужно определелить возраст по фотографии.
## Стэк
- **Python**
- **Keras**

## Вывод
**Провели исследование выборки:**

- Проанализировали дефекты изображений. Определили способы борьбы с дефектами и стратегию обучения.

**Построили и обучили свёрточную нейронную сеть на датасете с фотографиями людей:**

- Разбили код на блоки для обучения на сервере с gpu.
- Исправили дефеты выборки. Применили наклон по горизонтали и масштабировали изображения. Применили аугментации, создали разные генераторы для обучающей и валидационной выборок что бы повысть качество результатов.
- Что бы модель обучалась быстрее применили алгоритм Adam. Он подбирает различные параметры для разных нейронов, что также ускоряет обучение модели. Основной настраиваемый гиперпараметр в алгоритме Adam — скорость обучения (learning rate). Это шаг градиентного спуска, с которого алгоритм стартует.
- Полносвязные сети не могут работать с большими изображениями: если нейронов мало, сеть не найдёт зависимостей, а если много — переобучится. Эту проблему решает свёртка. Для решения задачи и повышения качества результатов использовали архитектуру свёрточных нейросетей ResNet. Главная особенность ResNet заключается в использовании Shortcut Connections — дополнительные связи внутри сети, которые позволяют избежать проблемы затухающего градиента.

**Добились значения MAE на тестовой выборке меньше 8. Полученная модель позволяет решать поставленную бизнесом задачу - определять возраст покупателей.**

## Статус проекта
Проект завершен.