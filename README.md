# Задание #8 Распределённые транзакции
## Постановка задачи
### Ограничение пользования приложением

Требуется ораничить количество проверок текста в единицу времени: не более N запросов в минуту (значение N указывается в конфигурации). Предполагается, что пользование системой сверх обозначенного лимита будет доступно после приобретения платного абонемента. На достижение лимита влияют только запросы, в которых текст получил статус "Успешный" (rank > 0.5).
Также требуется вести статистику по количеству отвергнутых запросов.

## Реализация
### Backend

Добавляется компонент *TextProcessingLimiter*, предназначение которого - пропускать или не пропускать запрос на обработку.  *TextProcessingLimiter* реагирует на события *TextCreated(contextId)* и информирует о свём решении через публикацию события *ProcessingAccepted(contextId, status)*, в котором аргумент *status* будет иметь значение True, если компонент разрешил обработку текста, False – в противном случае. При каждом разрешении компонент уменьшает количество оставшихся запросов на обработку текста. Выданное разрешение аннулируется при возникновении события *TextRankCalculated* со значением *rank <= 0.5*, соответственно *TextProcessingLimiter* должен также реагировать на эти события.

Компонент *TextRankCalc* теперь должен начинать обработку только при появлении события *ProcessingAccepted* со значением *status==True*.

Компонент *TextStatistics* начинает реагировать на события *ProcessingAccepted* и ведёт подсчёт сообщений, у которых поле *status == False*.


### Замечания

В компоненте *TextProcessingLimiter* используется шаблон *«Компенсирующая транзакция»*, когда действие транзакции выдачи разрешения затем «компенсируется» в случае если текст оказался неуспешным.

### Frontend

На странице с результатом обработки текста, на которую перенаправляется пользователь после отправки формы, нужно добавить отображение сообщения о невозможности обработки из-за достижения лимита, когда такое случается.

На странице статистики нужно выводить количество отвергнутых запросов.
