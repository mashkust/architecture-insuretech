## Task4

- osago-aggregator имеет свою БД: для correlationId, insurerRequestId, статусов, дедупликации, таймеров и частичных результатов.
- Между core-app и osago-aggregator — асинхронная интеграция через event-broker:
  - core-app публикует OSAGOQuoteRequested
  - osago-aggregator публикует OSAGOOfferReceived / RequestCompleted / RequestTimedOut
- Между web и core-app:
  - REST для создания заявки и получения snapshot
  - SSE для показа предложений сразу по мере поступления
- Паттерны отказоустойчивости показаны иконками:
  - у вызовов osago-aggregator -> страховые компании: Timeout, Retry, Rate Limiter, Circuit Breaker
  - у входа web -> core-app: Rate Limiter
  - рядом с SSE: Timeout на соединение/переподключение
