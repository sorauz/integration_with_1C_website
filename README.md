# integration_with_1C_website
1c 8.3.20 program, through a GET request through the incoming HTTP service
Brief information about the product, price of the product, release the picture on the web!


//1c hhp servis  get 
Функция ProductsGET(Запрос)	


	КодПоследнегоЭлемента = Запрос.ПараметрыЗапроса.Получить("lastcode");
	
	Запрос = Новый Запрос;
	Запрос.Текст = 
	"ВЫБРАТЬ ПЕРВЫЕ 4
	|	СпрНоменклатура.Код КАК Код,
	|	СпрНоменклатура.Ссылка,
	|	СпрНоменклатура.Наименование,
	|	СпрНоменклатура.Артикул,
	|	СпрНоменклатура.НаименованиеПолное,
	|	СпрНоменклатура.ДополнительноеОписаниеНоменклатуры,
	|	СпрНоменклатура.ОсновноеИзображение,
	|	ЦеныНоменклатурыСрезПоследних.Цена,
	|	ЦеныНоменклатурыСрезПоследних.ХарактеристикаНоменклатуры
	|ИЗ
	|	Справочник.Номенклатура КАК СпрНоменклатура
	|		ЛЕВОЕ СОЕДИНЕНИЕ РегистрСведений.ЦеныНоменклатуры.СрезПоследних(, ТипЦен = &ТипЦен) КАК ЦеныНоменклатурыСрезПоследних
	|		ПО (ЦеныНоменклатурыСрезПоследних.Номенклатура = СпрНоменклатура.Ссылка)
	|ГДЕ
	|	СпрНоменклатура.Код > &Код
	|	И СпрНоменклатура.ЭтоГруппа = ЛОЖЬ
	|
	|УПОРЯДОЧИТЬ ПО
	|	Код";
	Запрос.УстановитьПараметр("ТипЦен", Справочники.ТипыЦенНоменклатуры.НайтиПоКоду("000000004"));
	Если ЗначениеЗаполнено(КодПоследнегоЭлемента) Тогда
		Запрос.УстановитьПараметр("Код", КодПоследнегоЭлемента);
	Иначе	
		Запрос.Текст = СтрЗаменить(Запрос.Текст, "СпрНоменклатура.Код > &Код", "ИСТИНА");	
	КонецЕсли; 
	РезультатЗапроса = Запрос.Выполнить();
	
	СтруктураОтвета = ПолучитьСтруктуруОтвета();
	
	Если РезультатЗапроса.Пустой() Тогда
		Возврат СформироватьОтвет(СтруктураОтвета, 204);
	КонецЕсли; 
	
	Код = 0; 
	Выборка = РезультатЗапроса.Выбрать();
	Пока Выборка.Следующий() Цикл
		ДанныеТовара = Новый Структура;
		ДанныеТовара.Вставить("guid", XMLСтрока(Выборка.Ссылка));
		ДанныеТовара.Вставить("title", СтрШаблон("%1 - %2", Выборка.Наименование, Выборка.ХарактеристикаНоменклатуры));
		ДанныеТовара.Вставить("price", Выборка.Цена);
		ДанныеТовара.Вставить("desk", Выборка.ДополнительноеОписаниеНоменклатуры);
		ДанныеТовара.Вставить("img", ПолучитьКартинкуДляHTML(Выборка.ОсновноеИзображение));
		СтруктураОтвета.products.Добавить(ДанныеТовара);
		Код = Выборка.Код;
	КонецЦикла; 
	СтруктураОтвета.lastcode = Код;	
	Возврат СформироватьОтвет(СтруктураОтвета, 200);
КонецФункции

Функция ПолучитьСтруктуруОтвета()
  Товары = Новый Структура;
  Товары.Вставить("lastcode", 0);
  Товары.Вставить("products", Новый Массив);
  Возврат Товары;
КонецФункции


Функция СформироватьОтвет(СтруктураОтвета, КодОтвета)
    Ответ = Новый HTTPСервисОтвет(КодОтвета);
    Ответ.УстановитьТелоИзСтроки(СформироватьJSON(СтруктураОтвета), КодировкаТекста.UTF8, ИспользованиеByteOrderMark.НеИспользовать);
    Возврат Ответ;
КонецФункции

Функция ПолучитьКартинкуДляHTML(ОсновноеИзображение)
    ДвоичныеДанные = БиблиотекаКартинок.Заглушка.ПолучитьДвоичныеДанные();
    Если НЕ ОсновноеИзображение.Хранилище.Получить() = Неопределено Тогда
         ДвоичныеДанные = ОсновноеИзображение.Хранилище.Получить().ПолучитьДвоичныеДанные();
    КонецЕсли; 
    СтрокаBase64 = ПолучитьBase64СтрокуИзДвоичныхДанных(ДвоичныеДанные);
    СтрокаBase64 = СтрЗаменить(СтрокаBase64, Символы.ПС, "");
    СтрокаBase64 = СтрЗаменить(СтрокаBase64, Символы.ВК, "");
    Возврат СтрШаблон("data:image/jpg;base64, %1", СтрокаBase64);
КонецФункции

