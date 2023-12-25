# ElemetsOfFormDynamicP
##### <a name="pageup"></a>

1С Elemets of Form with dynamic properties / Динамическая установка свойств элементам формы
Обработка позволяет программно задать значение любому свойству элемента формы.
(Ширина, Высота, Доступность, ТолькоПросмотр, Видимость, Шрифт и т.д.)

https://youtu.be/bS7Gx8dLoow

# Почему не типовое решение или БСП:

Типовое решение позволяет управлять элементом как ВидимостьюПоРолям или БСП которая дает возможность "УстановитьСвойствоЭлементаФормы"
для единичного элемента, а данная обработка добавляет немного удобства к БСП методу УстановитьСвойствоЭлементаФормы(ЭлементыФормы, ИмяЭлемента, ИмяСвойства, Значение)
позволяя задать значения всем элементам формы и отдельно еденичным, а также проверить вхождение в группу доступа. 
Как дополнение данную обработку можно применять в конфигурациях без БСП, метод БСП выполняется для одного элемента:
	
 	// ОбщегоНазначенияКлиентСервер.УстановитьСвойствоЭлементаФормы(ЭтаФорма.Элементы, "Наименование", "Доступность", Истина);
 	Процедура УстановитьСвойствоЭлементаФормы(ЭлементыФормы, ИмяЭлемента, ИмяСвойства, Значение) Экспорт	
		ЭлементФормы = ЭлементыФормы.Найти(ИмяЭлемента);
		Если ЭлементФормы <> Неопределено И ЭлементФормы[ИмяСвойства] <> Значение Тогда
			ЭлементФормы[ИмяСвойства] = Значение;
		КонецЕсли;	
	КонецПроцедуры

# Краткое описание вызова

	1. форма - СсылкаНаБлокируемуюформу,
	2. свойстваЭлементов - Массив ИЗ Структура (имя элемента, свойство и его значение которые будут установлены)
	3.1 свойстваЭлементовОбщее - Структура описывает одно свойство и знч которое будет задано ВСЕМ элементам формы!

Обработка.УстановитьСвойствоЭлементаФормы(форма, свойстваЭлементов, свойстваЭлементовОбщее, видыИТипы);

	3.2 свойстваЭлементовОбщее - например всем элементам устанавливаем "ТолькоПросмотр" Истина, а
	свойстваЭлементов - добавляем элемент "Наименование" и "Родитель" которым устанавливаем свойство "ТолькоПросмотр" в Ложь
	и таким образом мы запрещаем редактировать ВСЕ элементы кроме "Наименование" и "Родитель"!

 	4. видыИТипы - это Соответствие (не обязательный параметр), которое перечислят какие типы элементов формы обрабатывать.
  	Если не передавать тогда будет как видыИТипы = внешняяОбработкаОбъект.НаборВидовИТиповЭлементовФормы();
   	*Например: ВидПоляФормы.ПолеВвода, ВидКнопкиФормы.КнопкаКоманднойПанели, ВидПоляФормы.ПолеФлажка и т.д.

# (Пример) Задача

Пользователи которые не входят в группуДоступа "РасширенныйДоступСправочникНоменклатура"
должны получить доступ на редактирование исключительно к реквизиту Наименование и команда "Записать и закрыть"
, а те пользователи которые входят в группу "РасширенныйДоступСправочникНоменклатура" к ним ограничения применятся не должны!

# Внедрение

+ [ВАЖНО, требования к методу проверки группы доступа](#add0); Рекомендую использовать с проверкой вхождения в ГруппыДоступа,
см. Область ПроверитьВхождениеВПрофиль_СвойствЭлементовФормы, но реализация в текущей обработке 
требует права на чтение для справочников "ГруппыДоступа" и "ПрофилиГруппДоступа"


1. Открывает Модуль формы в основной конфигурации (если редактирование разрешено) или в расширения (если редактирование основной конфигурации запрещено)
2. Переходим в событие "ПриСозданииНаСервере"
3. Копируем (Ctrl + C -> Ctrl + V) код

		#Область НовыеЗнчДляСвойствЭлементовФормы

		#Область ПолучениеВнешнейОбработки_СвойствЭлементовФормы
		внешняяОбработкаИмя    = "ДинамическоеСвойствоЭлементаФормы";
		внешняяОбработкаСсылка = Справочники.ДополнительныеОтчетыИОбработки.НайтиПоРеквизиту("ИмяОбъекта", внешняяОбработкаИмя);
		внешняяОбработкаОбъект = ДополнительныеОтчетыИОбработки.ОбъектВнешнейОбработки(внешняяОбработкаСсылка);
		#КонецОбласти

		#Область ПроверитьВхождениеВПрофиль_СвойствЭлементовФормы
		мГруппыДоступа = Новый Массив;
		мГруппыДоступа.Добавить("РасширенныйДоступСправочникНоменклатура");
		доступРазрешен = внешняяОбработкаОбъект.ТекущийПользовательДобавленВГруппыДоступа(мГруппыДоступа);
		#КонецОбласти

		// Описываете каким элементам формы какое значение хотите установить +++
		свойстваЭлементовОбщее = Новый Структура("Свойство,Значение", "Доступность", Ложь);

		свойстваЭлементов = Новый Массив; 
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "ФормаЗаписатьИЗакрыть", "Доступность", Истина) );
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "Наименование", "Доступность", Истина) );
		// Описываете каким элементам формы какое значение хотите установить +++

		#Область Выполнение_СвойствЭлементовФормы
		Если НЕ доступРазрешен Тогда
			внешняяОбработкаОбъект.УстановитьСвойствоЭлементаФормы(ЭтаФорма, свойстваЭлементов, свойстваЭлементовОбщее);
   		КонецЕсли
		#КонецОбласти

		

		#КонецОбласти

# Если хотите вызвать без использования БСП

![image](https://github.com/KistanovSerhii/ElemetsOfFormDynamicP/assets/28355711/901217c0-3499-430a-be13-163d2a5f5937)
Можно добавить внешнюю обработку в ветку конфигурации к типу "Обработки" и использовать как:
внешняяОбработкаОбъект = Обработки.ДинамическоеСвойствоЭлементаФормы.Создать();

Тогда полный код будет таким:

		#Область НовыеЗнчДляСвойствЭлементовФормы
  
		#Область ПолучениеВнешнейОбработки_СвойствЭлементовФормы
		внешняяОбработкаОбъект = Обработки.ДинамическоеСвойствоЭлементаФормы.Создать();
		#КонецОбласти

		#Область ПроверитьВхождениеВПрофиль_СвойствЭлементовФормы
		мГруппыДоступа = Новый Массив;
		мГруппыДоступа.Добавить("РасширенныйДоступСправочникНоменклатура");
		доступРазрешен = внешняяОбработкаОбъект.ТекущийПользовательДобавленВГруппыДоступа(мГруппыДоступа);
		#КонецОбласти

		// Описываете каким элементам формы какое значение хотите установить +++
		свойстваЭлементовОбщее = Новый Структура("Свойство,Значение", "Доступность", Ложь);

		свойстваЭлементов = Новый Массив; 
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "ФормаЗаписатьИЗакрыть", "Доступность", Истина) );
		свойстваЭлементов.Добавить( Новый Структура("Имя,Свойство,Значение", "Наименование", "Доступность", Истина) );
		// Описываете каким элементам формы какое значение хотите установить +++

		#Область Выполнение_СвойствЭлементовФормы
		Если НЕ доступРазрешен Тогда
			внешняяОбработкаОбъект.УстановитьСвойствоЭлементаФормы(ЭтаФорма, свойстваЭлементов, свойстваЭлементовОбщее);
   		КонецЕсли
		#КонецОбласти

		#КонецОбласти

# ВАЖНО:

##### <a name="add0"></a> ВАЖНО [(начало)](#pageup); Если будешь использовать метод проверки вхождения предоставленный текущей обработкой Тогда
У весх пользователей должна быть роль чтения справочника "ГруппыДоступа" и "ПрофилиГруппДоступа"
Если в твоей конфигурации есть профиль который не содержит ни одной роли разрешающей читать 
справочника "ГруппыДоступа" и "ПрофилиГруппДоступа" Тогда добавь роль и группу доступа с этой ролью куда добавь всех пользователей
![image](https://github.com/KistanovSerhii/ElemetsOfFormDynamicP/assets/28355711/e32c5837-bcc8-44ee-918e-6b656ab49b44)

##### + При использовании через расширение - обязательно снимите флаг "Безопасный режим, имя профиля безопасности"
![image](https://github.com/KistanovSerhii/ElemetsOfFormDynamicP/assets/28355711/7a0d51e4-fb60-4885-857a-61993c5aa62b)
