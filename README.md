## Python_Advanced_files

____

## Ниже представлены решения задач по продвинутому уровню Python.

### 1. Задание Импорт файла csv. 

Возьмите данные по вызовам пожарных служб в Москве за 2015-2019 годы:
https://video.ittensive.com/python-advanced/data-5283-2019-10-04.utf.csv. 
Получите из них фрейм данных (таблицу значений). По этому фрейму вычислите среднее значение вызовов пожарных машин в месяц в одном округе Москвы, округлив до целых
Примечание: найдите среднее значение вызовов, без учета года

#### Решение:

```python
import pandas as pd
data = pd.read_csv("https://video.ittensive.com/python-advanced/data-5283-2019-10-04.utf.csv", delimiter=";")
print (data["Calls"].mean().round())
```
____

### 2. Задание: данные из нескольких источников

Получите данные по безработице в Москве:
https://video.ittensive.com/python-advanced/data-9753-2019-07-25.utf.csv. 
Объедините эти данные индексами (Месяц/Год) с данными из предыдущего задания (вызовы пожарных) для Центральный административный округ:
https://video.ittensive.com/python-advanced/data-5283-2019-10-04.utf.csv
Найдите значение поля UnemployedMen в том месяце, когда было меньше всего вызовов в Центральном административном округе.

#### Решение:

```python
import pandas as pd
data1 = pd.read_csv("https://video.ittensive.com/python-advanced/data-9753-2019-07-25.utf.csv", delimiter=";")
data1 = data1.set_index(["Year", "Period"])
data2 = pd.read_csv("https://video.ittensive.com/python-advanced/data-5283-2019-10-04.utf.csv", delimiter=";")
data2 = data2.set_index(["AdmArea", "Year", "Month"])
data2 = data2.loc["Центральный административный округ"]
data2.index.names = ["Year", "Period"]
data = pd.merge(data1, data2, left_index=True, right_index=True)
data = data.reset_index()
data = data.set_index("Calls")
data = data.sort_index()
print (data["UnemployedMen"][0:1])
```
____
### 3. Задание: выделение данных

Получите данные по безработице в Москве:
https://video.ittensive.com/python-advanced/data-9753-2019-07-25.utf.csv. 
Найдите, с какого года процент людей с ограниченными возможностями (UnemployedDisabled) среди всех безработных (UnemployedTotal) стал меньше 2%.

#### Решение:

```python
import pandas as pd
data = pd.read_csv("https://video.ittensive.com/python-advanced/data-9753-2019-07-25.utf.csv", delimiter=";")
data["Sum"] = data.apply(lambda x: 100*x[6]/x[7], axis=1)
data = data[data["Sum"] < 2]
data = data.set_index("Year")
data = data.sort_index()
print (data.index[0:1])
```
____

### 4. Задание: предсказание на 2020 год

Возьмите данные по безработице в городе Москва:
video.ittensive.com/python-advanced/data-9753-2019-07-25.utf.csv
> Сгруппируйте данные по годам, и, если в году меньше 6 значений, отбросьте эти годы.
Постройте модель линейной регрессии по годам среднего значения отношения UnemployedDisabled к UnemployedTotal (процента людей с ограниченными возможностями) за месяц и ответьте, какое ожидается значение процента безработных инвалидов в 2020 году при сохранении текущей политики города Москвы?
Ответ округлите до сотых. Например, 2,32

#### Решение:

```python
import pandas as pd
import numpy as np
from sklearn.linear_model import LinearRegression
data = pd.read_csv("D:\Обучение уроки\Продвинутый PYTHON\Часть 1 АНАЛИЗ ДАННЫХ\Безработные.csv", delimiter=";")

# после подгрузки нельзя просто разделить "UnemployedDisabled" на UnemployedTotal, т.к. получится отношение средних.  По условии нужно найти средние отношений, средние процента. Поэтому нужно для каждой строчки данных вычислить значение  некторого нового столбца "UDP" - UnemployedDisabledProcent: 100*data["UnemployedDisabled"]/data["UnemployedTotal"]
# Т.о. получим для каждой строки новую серию данных:

data["UDP"] = 100*data["UnemployedDisabled"]/data["UnemployedTotal"]

# Далее отфильтруем по году через метод .groupby и filter c ляюмда ф., где считаем число строк, которые вошли в эту группу  данных, больше 5(т.е. все записи, где их количество меньше 6, мы отбрасываем, и не будем строить статистку): 

data_group = data.groupby("Year").filter(lambda x: x["UDP"].count() > 5)

# т.к. фильтр возвращает не группы, а сами данные, поэтому нужно группы данных заново сгруппировать по году и взять для них  среднее значение: 

data_group = data_group.groupby("Year").mean()

# и после этого можем посмотреть на те данные, на которые хотим применять линейную регрессию:
# приводим индексы группы данных к массиву и изменяем их форму на двумерный массив:
# (np.array - делает одномерный массив из заданного индекса, a reshape делает двумерный массив из нашего списка  размером - размер списка(len) и 1). По факту получается строчка данных. А для "y" используем "UDP" для  каждой группы данных: 

x = np.array(data_group.index).reshape(len(data_group.index), 1)
y = np.array(data_group["UDP"]).reshape(len(data_group.index), 1)
model = LinearRegression()
model.fit(x, y)

# получаем двумерный массив, состоящих из одной ячейки: 

print (np.round(model.predict(np.array(2020).reshape(1, 1)), 2))
[[1.52]]
```
____

### 5. Задание: Получение данных по API

Изучите API Геокодера Яндекса
tech.yandex.ru/maps/geocoder/doc/desc/concepts/input_params-docpage/ и получите ключ API для него в кабинете разработчика.

Выполните запрос к API и узнайте долготу точки на карте (Point) для города Самара.
Внимание: активация ключа Геокодера Яндекса может занимать несколько часов (до суток).

#### Решение:

```python
Самое сложное получить ключ API. После получение нужно изучить формат API, для того чтобы отправить 
запрос. В квадратных скобках необязательные параметры, их достаточно много. И 2 обязательных параметра, которые нужно использовать(в данном запросе): 
- apikey (это ключ API, который можно получить в кабинете разработчика)
- goecode (чуть сложнее: нужно понять, что нужно передать в геокод - либо адрес, либо географические координаты искомого объекта). Т.к. нам требуется по г. Самаре узнать координаты, то нам нужно передать адрес, т.е. просто "Самара" - это и будет наш базовый запрос:

import requests
import json # нужен для разбора ответа, кот. получим в формате json
r = requests.get("https://geocode-maps.yandex.ru/1.x?geocode=Самара&apikey=0494f4b7-56d0-4cb4-a257-66585f726501&format=json&results=1")

# это базовый запрос. Но этого недостаточно. 
# Также есть параметр format - формат ответа(любо xml либо  json, выбираем json - скопируем в запрос)
# Также будет полезно результаты - result - укажем 1 результат, чтобы не путаться в возвращаемых #объектах

# отправляем сформарованный get запрос к API 
# И после этого говорим, что это объект json.loads <string>

geo = json.loads(r.content) # ответ, который мы получаем, разбираем в json
print(geo['response']['GeoObjectCollection']['featureMember'][0]['GeoObject']['Point']['pos'].split(" ")[0])

	50.100202


В ответе есть Point и координаты - 'pos' -50.100202 - искомая широта точки. Нужно ее найти, вычленить из geo. А для этого нужно последовательно сокращать до тех пор, пока не дойдем до самой точки.
Смотрим, что 'featureMember' - это массив(квадратные скобки) и берем первый объект массива - [0] и переходим к geo объекту (GeoObject) - далее переходим к 'Point' и 'pos', разделим по пробелу - .split(" ") и выведем первый объект с индексом [0] - и получаем искомую ширину координаты

```
____

### 6. Задание: получение котировок акций

Получите данные по котировкам акций со страницы:
mfd.ru/marketdata/?id=5&group=16&mode=3&sortHeader=name&sortOrder=1&selectedDate=01.11.2019
и найдите, по какому тикеру был максимальный рост числа сделок (в процентах) за 1 ноября 2019 года.

#### Решение: 

```python
import pandas as pd
import requests
from bs4 import BeautifulSoup
r=requests.get("https://mfd.ru/marketdata/?id=5&group=16&mode=3&sortHeader=name&sortOrder= 1&selectedDate=01.11.2019")
html = BeautifulSoup(r.content)

<table class="mfd-table" id="marketDataList">

# найдем тег <table> по корому будем искать данные. У нее id = "marketdatalist". 
# Можем старгерироваться по id, его id это marketDataList

table = html.find("table", {"id":'marketDataList'})    # дальше перейдем к разбору таблицы
rows = []
trs = table.find_all("tr") 

#  найдем все теги "tr" внутри таблицы и перебирая все теги и говорим, что  в строчку через генератор добавляем получение текста из ячеек  - td.get_text.  А сами td они находятся  как теги tr.find_all('td') на строчку(на тот html код, который был найден на строчку при разборе "tr")  внутри тега "table", т.е. внутри этого тега, мы ищем все теги "tr" и затем в каждом из этих html кодов  мы выбираем все ячейки и делаем из них небольшой список и перезаписываем в переменную "tr":
# tr = [td.get_text(strip=True) for td in tr.find_all('td')]

# Зачем это нужно? 

# Это нужно, потому что м.б. строки нулевой длины(строки без данных, напр. <th) и их нужно просто 
# отбросить  if len(tr) > 0:

for tr in trs:
    tr = [td.get_text(strip=True) for td in tr.find_all('td')]
    if len(tr) > 0:   # отбрасываем нулевые строки. Если длина больше 0, то соот-но добавляем в строку rows
        rows.append(tr)


# добавили выше аргумент strip = True в td.get_text, чтобы выбросить лишние пробелы и переводы строк и т.д.

# Дальше выгрузим в датафрейм весь этот список списков и сразу дадим названия столбцам(сериям данных), 
# посмотрев их сайте :

data = pd.DataFrame(rows, columns=["Тикер", "Дата", "Сделки", "C/рост", "С/%", "Закрытие", "Открытие", "min", "max", "avg", "шт", "руб", "Всего"])

# нам была нужна колонка Тикер, но для полноты данных все серии отобразим

# Выбросим из DataFrame отсутствующие сделки(N/A), чтобы они не влияли на сортировку процента по сделкам, 
# иначе они могут сбить ключи. Для этого сделаем фильтрацию:

data = data[data["Сделки"] != "N/A"]

# говорим, что все данные берем из всех данных, в том случае, если значение серии "Сделки" не равно N/A.

# Также преобразуем столбец процентов(в столбце есть + и -), и для сортировки по возрастанию и убыванию
# нужно привести его в числу: уберем из столбца дополнительные символы и если посмотреть внимательней
# " - " не является минусом, а дефисом(-). (для красоты используют его, и это для работы с данными неприменима
# И также удаляем знак процента(%) из данных, и приводим к типу float через метод pandas .astype:

data["С/%"] = data["С/%"].str.replace("−","-").str.replace("%", "").astype(float)

# выставляем индекс по сделкам по процентам и отсортируем:

data = data.set_index("С/%")
data = data.sort_index(ascending = False) # сортировка по возрастаниюб ascending = True - сортировка по 
# возрастанию, начиная с отрицательных значений,  а ascending = False - начиная с положительных

print(data["Тикер"].head(1)) # нужно вывести первое значение серии "Тикер" в уже отсортированных данных
```
____

#### 7. Задание: парсинг интернет-магазина

Используя парсинг данных с маркетплейса beru.ru, найдите, на сколько литров отличается общий объем холодильников Саратов 263 и Саратов 452?
Для парсинга можно использовать зеркало страницы beru.ru с результатами для холодильников Саратов по адресу:
video.ittensive.com/data/018-python-advanced/beru.ru/

### Решение: 

```python
Используя парсинг данных с маркетплейса beru.ru, найдите, на сколько литров отличается общий объем холодильников Саратов 263 и Саратов 452?
Для парсинга можно использовать зеркало страницы beru.ru с результатами для холодильников Саратов по адресу: video.ittensive.com/data/018-python-advanced/beru.ru/

import requests
from bs4 import BeautifulSoup

# для начала нужно отправная точка, это поиск по слову "Саратов" на сайте беру и выбор необходимых 
# холодильников. Получаем ссылки на эти страницы и получаем данные. 

# пишем имя нашего бота
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.97 YaBrowser/19.12.0.358 Yowser/2.5 Safari/537.36"}
r = requests.get("https://beru.ru/search?cvredirect=2&suggest_reqId=27865074762321487883702093457804&text=%D1%81%D0%B0%D1%80%D0%B0%D1%82%D0%BE%D0%B2", headers=headers)
html = BeautifulSoup(r.content)
print (r.content) 	# смотрим на html код

# в html коде нужно найти блоки с товарами ("Холодильник Саратов 263"), и далее во всех блоках найти ссылки
# ищем ссылку на товар. 

links = html.find_all("a", {"class": "grid-snippet__react-link"})
# 1. сначала указываем тег "а" 
# 2. указываем ссылку класса "class": "grid-snippet__react-link" на товар Саратов 263. 
# 3. В этих ссылках нужно найти определенный текст "grid-snippet__react-link"

link_263 = ''
link_452 = ''
for link in links:     # перебираем ссылки и ищем определенный текст 
    if str(link).find("Саратов 263") > -1:      # приводим ссылку к строке str и найти текст "Саратов 263" 
        # и если текст найден "> -1", то соответствующая ссылка будет равно атрибута href из html(т.е. нужная ссылка заключена в атрибуте href)
        
        link_263 = link["href"]  	# делам то же самое для Саратова 452
    if str(link).find("Саратов 452") > -1:
        link_452 = link["href"]
       
def find_volume (link): 
    # Мы будем получать один и тот же контент с двух страниц. Поэтому нужно написать получение 
    # контента с одной страницы и затем обернем это в функцию, для того, чтобы не дублировать код
    
    r = requests.get("https://beru.ru" + link)
    html = BeautifulSoup(r.content)
    volume = html.find_all("span", {"class": "_112Tad-7AP"}) # находим span с классом в коде для объема холодильника "_112Tad-7AP"
    
    # далее выделяем число 195 из текста "общий объем 195 литров":
    
    return int(''.join(i for i in volume[2].get_text() if i.isdigit())) # - сделаем генератор:
# 1. i for i in volume[2] - проверим все символы в строке "общий объем 195 литров" (volume[2] - третий из найденных текстов)
# 2. получаем текст  .get_text()
# 3. if i.isdigit() и символы яв-ся цифрами, то возвращаем эти цифры 
# 4. ''.join - объединяем полученный список через пустую строку "" и переводим к целому числу

if link_263 and link_452: # получаем контент с этих ссылок. Мы будем получать один и тот же контент 
    # с двух страниц. Поэтому нужно написать получение контента с одной страницы и затем обернем это в 
    # функцию, для того, чтобы не дублировать код
    volume_263 = find_volume(link_263) # для каждой ссылки находим объем
    volume_452 = find_volume(link_452)  
    diff = max(volume_263, volume_452) - min(volume_263, volume_452)  # выводим разницу между двумя значениями  через нахождение максимального и минимального значений у двух объемов
    print (diff)
```
____

#### 8. Задание: загрузка результатов в БД

Соберите данные о моделях холодильников Саратов с маркетплейса beru.ru: URL, название, цена, размеры, общий объем, объем холодильной камеры.
Создайте соответствующие таблицы в SQLite базе данных и загрузите полученные данные в таблицу beru_goods.
Для парсинга можно использовать зеркало страницы beru.ru с результатами для холодильников Саратов по адресу:
video.ittensive.com/data/018-python-advanced/beru.ru/

### Решение:

```python
import sqlite3
import requests
from bs4 import BeautifulSoup

Сначала нужно получить характеристики товара (длина х высота х габариты), для создания полей(таблицы). Нужно пройтись по всем товарам и выгрузить в таблицу.
Начнем с отработки url конкретного товара(парсить), потомучто мы получим ссылки в каталоге такого же вида. 
Для этого введем функцию find-data:

def find_number(text):   # функция для нахождения "общий объем"- приведение к целому числу некоторой строки, которую джойним через пустую строку "", если все символы яв-ся числами (т.е. извлекаем только цифры
    return int("0" + "".join(i for i in text if i.isdigit()))
def find_data (link):
    r = requests.get("https://beru.ru" + link)  

# - получаем контент с сайта беру.ру с указанной ссылкой, и нужно просто поставить домен с протоколом, чтобы получить 
    
    html = BeautifulSoup(r.content)
    title = html.find("h1", {"class": "_3TfWusA7bt"}).get_text() 

# находим название холодильника с классом, берем у него текст, потому что тегов h1 м.б. несколько
    # класс _3TfWusA7bt только один и поэтому лучше позиционироваться по этому классу. (Но можно было и без класса, найдя только первый тег h1)
    
    price = find_number(html.find("span", {"data-tid": "c3eaad93"}).get_text()) 

# найдем цену: найдем по тегу data-tid -c3eaad93 цену.(возьмем первый тег span c этим классом c3eaad93)
    
    tags = html.find_all("span", {"class": "_112Tad-7AP"}) # найдем все параметры ШхВхГ. Для этого спозиционируемся по span "_112Tad-7AP",  таких спанов 4, которые охватывают эти параметры
    
    # заведем переменные параметров:
    
    width = 0
    depth = 0
    height = 0
    volume = 0  	# объем холодильной камеры
    freezer = 0 	# объем морозильной камеры
    
    # Далее переберем теги,  вычленим эти параметры из тегов:
    for tag in tags:
        tag = tag.get_text()  # берем текст из тега
        if tag.find("ШхВхГ") > -1:  # если в теге есть строка "ШхВхГ", то эту строку нужно распарсить: разделяем по двоеточию(":")
                                   
            dims = tag.split(":")[1].split("х")  # и взять вторую часть из двоеточия с индексом [1], и разделить по разделителю "х"
           # получаем значение, которое потом приведем к числам: 

            width = float(dims[0])   # - первое значение из dims 
            depth = float(dims[1])   # - второе значение из dims
            height = float(dims[2].split(" ")[0])  # - третье значение из dims. Чтобы не вошли сантиметры в результат- 
            разобъем по пробелу и возьмем первый элемент [0]
        if tag.find("общий объем") > -1:  # если в теге есть "общий объем", то вычленяем из тега число объема
            volume = find_number(tag)  	 # находим через функцию find_number общий объем (tag- строка): 
            # def find_number(text): 	  # функция для нахождения "общий объем"- приведение к целому числу некоторой строки, которую джойним через пустую строку "", если все символы яв-ся числами (т.е. извлекаем только цифры     return int("0" + "".join(i for i in text if i.isdigit()))

        if tag.find("объем холодильной камеры") > -1:
            freezer = find_number(tag)  	# также находим через функцию find_number общий объем (tag- строка) 
    return [link, title, price, width, depth, height, volume, freezer] 	# вернем все то, что искали

''' Выше мы получили функцию хелпер (helper) -def find_data, которые находит нужные данные по каждому холодильнику '''

# Далее нужно взять со страницы (донора)


r=  requests.get("https://beru.ru/catalog/kholodilniki/79958/list?cvredirect=3&suggest_reqId=83526016473955609954771572320629&text=%D0%A1%D0%B0%D1%80%D0%B0%D1%82%D0%BE%D0%B2")
html = BeautifulSoup(r.content)  # нужно найти все ссылки на холодильники Саратов

links = html.find_all("a", {"class": "grid-snippet__react-link"}) # находим ссылки по тегу "а" и по классу "grid-snippet__react-link"
data = []
for link in links:
    if link["href"] and link.get_text().find("Саратов") > -1: # нужно получить атрибут href, если есть и текст ссылки содержит название "Саратов"

        data.append(find_data(link["href"]))  	# добавляем результаты работы функции "find_data" к той ссылке, которую нашли и произойдет перебор страницы беру.ру  для  получения нужных данных 
conn = sqlite3.connect("sqlite/data.db3")	# создаем подключение к БД
db = conn.cursor()

# выполняем запрос по созданию базы данных: 

db.execute("""CREATE TABLE beru_goods
            (id INTEGER PRIMARY KEY AUTOINCREMENT not null,
            url text,
            title text default '',
            price INTEGER default 0,
            width FLOAT default 0.0,
            depth FLOAT default 0.0,
            height FLOAT default 0.0,
            volume INTEGER default 0,
            freezer INTEGER default 0)""")
conn.commit()  # закоммитим результат

# После получения всех данных передаем их одним запросом на вставку данных в базу.
# Добавляем запрос в добавление в БД beru_goods и перечисляем те поля, куда добавляем url, title  и т.д.
(перечисляем поля)  и передаем в качестве значений список списков, кот. совпадают с количеством строк, 
# передаем в VALUES 8 знаков "?" и в качестве параметров передаем (data), который представляет собой список списков:

db.executemany("""INSERT INTO beru_goods (url, title, price, width, depth, height, volume, freezer)
           VALUES (?, ?, ?, ?, ?, ?, ?, ?)""", data)

# метод .executemany - если нужно добавить множество строк одновременно в БД   

conn.commit()
print (db.execute("SELECT * FROM beru_goods").fetchall()) 	# выведем все значения из созданной базы данных 
db.close() 	# закрываем БД
```
_____

#### 9. Загрузите данные по ЕГЭ за последние годы

https://video.ittensive.com/python-advanced/data-9722-2019-10-14.utf.csv 
выберите данные за 2018-2019 учебный год.
Выберите тип диаграммы для отображения результатов по административному округу Москвы, 
постройте выбранную диаграмму для количества школьников, написавших ЕГЭ на 220 баллов и выше.
Выберите тип диаграммы и постройте ее для районов Северо-Западного административного округа Москвы для
количества школьников, написавших ЕГЭ на 220 баллов и выше.

### Решение: 

```python
%matplotlib inline
import pandas as pd
import matplotlib.pyplot as plt
data = pd.read_csv("D:\Обучение уроки\Продвинутый PYTHON\Часть 3 ВИЗУАЛИЗАЦИЯ ДАННЫХ\Основы Matplotlib\ЕГЭ.csv", delimiter=";")

# Преобразуем данные по округу и району, чтобы устранить дубликаты и сделать подписи короче: удалим из названия районов слово "район"
# а из округа удалим все слова, кроме первого. 
# Дополнительно назначим район, как категории, чтобы группировка происходила быстрее:

data["District"] = data["District"].str.replace("район ","").astype("category")
data["AdmArea"] = data["AdmArea"].apply(lambda x:x.split(" ")[0]).astype("category") 
# из названия округа берем только первое слова[0]

# выставим индекс по году и отфильтруем по индексу, чтобы получить данные только за 2018-2019 гг.:
data = data.set_index("YEAR").loc["2018-2019"].reset_index()    # сброим индекс - reset.index()
# т.к. данных по районам и округам достаточно много и вместе они образуют совокупность, то лучше использовать круговую диаграмму:  две круговые диаграммы в одну строку:
fig = plt.figure(figsize=(12,12))

# добавим холст:
area = fig.add_subplot(1, 2, 1)

# сначала построим распределение отличников по ЕГЭ по округам
area.set_title("ЕГЭ в Москве", fontsize=20) # заголовок
data_adm = data.set_index("AdmArea")

# отсечем данные только отличников, сгруппируем по округам и выведем сумму на круговой диаграмме:
data_adm["PASSES_OVER_220"].groupby("AdmArea").sum().plot.pie(ax=area, label="")

# для второй диаграммы выведем данные в СЗ округе, для этого добавим область для вывода данных
area = fig.add_subplot(1, 2, 2)
area.set_title("ЕГЭ в СЗАО", fontsize=20)	 # выведем завголвок "ЕГЭ в СЗАО"

#  выведем данные по СЗАО, переустановим индекс на "District"
data_district = data_adm.loc["Северо-Западный"].reset_index().set_index("District")

# отсечем только отличников и построим круговую диаграмму по районам:
data_district = data_district["PASSES_OVER_220"].groupby("District").sum() 

# выведем подписи к районам, чтобы узнать ответ. Для этого нужно вычислить общее число отличников по району. 
total = sum(data_district)  

#  Рассчитав точное значение мы могли бы привести к целому типу чисел, но тогда у нас потеряется точность из-за  двойного приведения значений. Поэтому обязательно, сначала нужно округлить значения по району и только потом привести  целому числу, что и дает точный ответ - 188 отличников в 2018-2019 учебном году:
data_district.plot.pie(ax=area, label="", autopct=lambda x:int(round(total * x/100)))
plt.show()

![Exam results](https://user-images.githubusercontent.com/96381562/169074109-ed1808af-90bf-4862-9457-2c698c1bc350.png)
```
_____

