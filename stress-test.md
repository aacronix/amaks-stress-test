# Нагрузочное тестирование :hammer:

* Тестирование проводилось с использованием инструмента [Locust](https://locust.io)

## Locust
* Opensource инструмент
* Написан на языке Python
* Позволяет проводить нагрузочное тестирование с минимальными затратами
* Эмулирует заходы на сайт
* Имеет интерфейс для отображения хода тестирования
* Также этим инструментом проверялась MYSQL-репликация (создавались элменеты инфоблоков)


  ## Тестирование
  Тестирование проводилось обычными заходами на [сайт](https://yoshkar-ola.amaks-hotels.ru), а так же созданием элементов инфоблоков (заявка на бронирование конференцзала, отзыв). Это позволило нам проверить какую нагрузку может выдержать сервер и работает ли репликация.
  ### Код

```python
from locust import HttpLocust, TaskSet, task

class FlowException(Exception):
   pass

class Amaks(TaskSet):
    @task(2)
    def on_start(self):
	self.client.get("/")
	self.client.get("/special/")
	self.client.get("/rooms/")
	self.client.get("/booking/")
	self.client.get("/rest/")
	self.client.get("/news/")
	self.client.get("/ivisa/")
    @task(1)
    def check_review_add(self):
	new_review = {
	    "review_city": "LocustCity",
	    "review_email": "locust@email.here",
	    "review_name": "Locust Test",
	    "review_text": "Some text about locusts"
	}

	review_response = self.client.post("/bitrix/templates/common/ajax/review-add.php?lang=ru&section=yoshkar-ola-ru&event=FORM_REVIEW_YOSHKAR-OLA_RU", json=new_review)

	if review_response.status_code != 200:
	    raise FlowException('review not created')
	review_id = review_response.json().get("success")

    @task(1)
    def check_conference_claim_add(self):
	new_hall_claim = {
	    "budget": "1.000.000 dollars",
	    "daterange": "24.02.2019 - 18.03.2019",
	    "description": "Locust was here",
	    "email": "locust@email.here",
	    "hallname": "Locust GLory Hall",
	    "organization": "LocustComp",
	    "participants": "1000000000000000000",
	    "person": "Locust",
	    "phone": "Locust SMS",
	    "policy": "true",
	    "setup" : "Locust style"
	}

	hall_claim_response = self.client.post("https://yoshkar-ola.webtltest.ru/bitrix/templates/common/ajax/hallclaim-add.php?lang=ru&section=yoshkar-ola-ru&event=HALLCLAIM_YOSHKAR-OLA_RU", json=new_hall_claim)

	if hall_claim_response.status_code != 200:
	    raise FlowException("hall claim not created")
	hall_claim_id = hall_claim_response.json().get("success")

class WebsiteAmaks(HttpLocust):
    task_set = Amaks
    min_wait = 1000
    max_wait = 5000
```

## Результаты
### 100 пользователей, по 10 в секунду (~ 1 минута)

* Таблица с результатами
![alt](/resources/stress-test/10.png)
* Графики нагрузки
![alt](/resources/stress-test/20.png)

### 500 пользователей, по 100 в секунду (~ 5 минут)

* Таблица с результатами
![alt](/resources/stress-test/30.png)
* Графики нагрузки
![alt](/resources/stress-test/40.png)

### 1000 пользователей, по 50 в секунду (~ 10 минут)

* Таблица с результатами
![alt](/resources/stress-test/50.png)
* Графики нагрузки
![alt](/resources/stress-test/60.png)
* Нагрузка на 1 ноду
![alt](/resources/stress-test/70.png)
* Нагрузка на 2 ноду
![alt](/resources/stress-test/80.png)


# Проверка двусторонней синхронизации

```bash
cat create-structure.sh
mkdir test1
mkdir test2
cd test1
truncate -s 10M foo0
mkdir test1_1
mkdir test1_2
mkdir test1_3
cd test1_1
mkdir test1_1_1
cd ../
cd test1_2
mkdir test1_2_1
cd test1_2_1
mkdir test1_2_1_1
cd ../
cd ../
cd test1_3
mkdir test_1_3_1
cd test_1_3_1
mkdir test_1_3_1_1
cd test_1_3_1_1
mkdir test_1_3_1_1_1
cd ../
cd ../
cd ../
cd ../
truncate -s 100M foo1
cd test2
mkdir test2_1
cd test2_1
mkdir test2_1_1
cd test2_1_1
mkdir test2_1_1_1
mkdir test2_1_1_2
mkdir test2_1_1_3
mkdir test2_1_1_4
mkdir test2_1_1_5
mkdir test2_1_1_6
mkdir test2_1_1_7
mkdir test2_1_1_8
mkdir test2_1_1_9
mkdir test2_1_1_10
mkdir test2_1_1_11
mkdir test2_1_1_12
mkdir test2_1_1_13
mkdir test2_1_1_14
mkdir test2_1_1_15
mkdir test2_1_1_16
mkdir test2_1_1_17
mkdir test2_1_1_18
mkdir test2_1_1_19
mkdir test2_1_1_20
cd test2_1_1_20
truncate -s 1M foo1
truncate -s 2M foo2
truncate -s 3M foo3
truncate -s 4M foo4
truncate -s 5M foo5
truncate -s 6M foo6
truncate -s 7M foo7
truncate -s 8M foo8
truncate -s 9M foo9
truncate -s 8M foo8
truncate -s 7M foo7
truncate -s 6M foo6
truncate -s 5M foo5
truncate -s 4M foo4
truncate -s 3M foo3
truncate -s 2M foo2
truncate -s 1M foo1
```
