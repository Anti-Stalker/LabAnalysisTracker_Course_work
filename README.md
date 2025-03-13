## Описание проекта
Название: Программное средство для отслеживания результатов обработки анализов частной лаборатории. 
Цель: Разработать кроссплатформенное мобильное приложение на C# (с использованием .NET MAUI), строго соответствующее принципам ООП. Приложение позволяет генерировать и сканировать QR-коды для отслеживания статуса анализов, моделировать случайные результаты, учитывать очередь заявок, время работы лаборатории (9:00–18:00 с обедом 13:00–14:00) и экспортировать результаты в Word или PDF. 
Область применения: Частные медицинские лаборатории, где пациенты и сотрудники могут управлять и отслеживать анализы через мобильные устройства. 
Технологии: C#, .NET MAUI, библиотека QRCoder для QR-кодов, DocX или iTextSharp для экспорта файлов.

## Автор
Имя: Коваленко Александр Александрович 
Группа: 353504

## Диаграмма классов

' Определение интерфейсов

interface IQRHandler {
  + GenerateQRCode(data: string): string
  + ScanQRCode(): string
}

interface IFileExporter {
  + ExportToFile(sample: AbstractTestSample, format: string): void
}

interface ITimeManager {
  + GetCurrentTime(): DateTime
  + CalculateCompletionProbability(expectedDate: DateTime): double
  + IsWorkingHour(): bool
  + SkipTime(days: int): void
}

' Абстрактный класс для анализов
abstract class AbstractTestSample {
  - id: string
  - receivedDate: DateTime
  - expectedCompletionDate: DateTime
  - status: string
  - result: string
  - qrCodeData: string
  + AbstractTestSample(id: string, receivedDate: DateTime, daysToComplete: int)
  + {abstract} GetSampleType(): string
  + GenerateQRCode(handler: IQRHandler): void
  + ExportResult(exporter: IFileExporter, format: string): void
  + GetId(): string
  + GetStatus(): string
  + SetStatus(status: string): void
  + GetResult(): string
  + SetResult(result: string): void
}

' Классы-наследники
class BloodTestSample {
  + BloodTestSample(id: string, receivedDate: DateTime, daysToComplete: int)
  + GetSampleType(): string
}

class UrineTestSample {
  + UrineTestSample(id: string, receivedDate: DateTime, daysToComplete: int)
  + GetSampleType(): string
}

' Класс ResultGenerator
class ResultGenerator {
  - rand: Random
  + GenerateResult(): string
}

' Класс TimeManager
class TimeManager {
  - currentTime: DateTime
  + GetCurrentTime(): DateTime
  + CalculateCompletionProbability(expectedDate: DateTime): double
  + IsWorkingHour(): bool
  + SkipTime(days: int): void
}

' Класс QueueManager
class QueueManager {
  - queueCount: int
  - queue: List<AbstractTestSample>
  + AddToQueue(sample: AbstractTestSample): void
  + GetQueueCount(): int
  + RemoveFromQueue(sample: AbstractTestSample): void
}

' Класс Laboratory
class Laboratory {
  - samples: List<AbstractTestSample>
  - queueManager: QueueManager
  + AddSample(sample: AbstractTestSample): void
  + GetSampleById(id: string): AbstractTestSample
  + UpdateAllSamples(timeManager: ITimeManager, resultGenerator: ResultGenerator): void
}

' Класс QRHandler для мобильного приложения
class QRHandler {
  + GenerateQRCode(data: string): string
  + ScanQRCode(): string
}

' Класс FileExporter
class FileExporter {
  + ExportToFile(sample: AbstractTestSample, format: string): void
}

' Определение связей
AbstractTestSample <|-- BloodTestSample
AbstractTestSample <|-- UrineTestSample
Laboratory o--> "many" AbstractTestSample : contains
Laboratory --> QueueManager : manages
AbstractTestSample --> ResultGenerator : uses
AbstractTestSample --> IQRHandler : uses
AbstractTestSample --> IFileExporter : uses
AbstractTestSample --> ITimeManager : uses
IQRHandler <|.. QRHandler
IFileExporter <|.. FileExporter
ITimeManager <|.. TimeManager

## Список функций
1. Добавление анализа: 
Позволяет сотруднику лаборатории добавить новый анализ с указанием ID, даты приёма и ожидаемой даты завершения. 
2. Генерация QR-кода: 
Для каждого анализа создаётся QR-код, содержащий ID и текущий статус. Пациент может отсканировать код для проверки статуса. 
3. Сканирование QR-кода: 
Мобильное приложение позволяет сканировать QR-коды с помощью камеры устройства для получения статуса анализа. 
4. Мониторинг статуса анализов: 
Система автоматически обновляет статус анализов ("InProgress", "Completed", "Delayed") на основе времени и вероятности готовности. 
5. Генерация случайных результатов: 
После завершения анализа генерируется случайный результат: "Normal" (80%), "MinorDeviation" (15%), "Critical" (5%). 
6. Учёт очереди: 
Система отслеживает количество заявок в очереди, чтобы учитывать нагрузку лаборатории. 
7. Учёт времени работы лаборатории: 
Рабочие часы: 9:00–18:00, с перерывом на обед 13:00–14:00. Ночные часы и выходные исключаются из обработки. 
8. Пропуск времени: 
Для тестирования можно "перемотать" время вперёд, чтобы симулировать процесс обработки анализов. 
9. Экспорт результатов: 
Результаты анализа можно экспортировать в форматах Word или PDF для печати или хранения.

## Модели данных
Основная модель данных — абстрактный класс AbstractTestSample: 
- id: Уникальный идентификатор анализа (string). 
- receivedDate: Дата и время приёма анализа (DateTime). 
- expectedCompletionDate: Ожидаемая дата завершения анализа (DateTime). 
- status: Текущий статус ("InProgress", "Completed", "Delayed"). 
- result: Результат анализа, если он готов ("Normal", "MinorDeviation", "Critical"). 
- qrCodeData: Строка с данными QR-кода (например, "TestID:001|Status:InProgress"). 

Дополнительные модели: 
- QueueManager: Хранит queueCount (int) и queue (List<AbstractTestSample>) для управления очередью. 
- TimeManager: Реализует ITimeManager для управления рабочим временем и вероятностью готовности. 
- QRHandler: Реализует IQRHandler для создания и сканирования QR-кодов. 
- FileExporter: Реализует IFileExporter для экспорта результатов. 
