# Реализация REST API части функционала автомойки

по данной диаграмме, построить БД https://drawsql.app/teams/alexxx/diagrams/servicev2

Описать API через Swagger:

- Реализовать CRUD для каждой сущности
- Реализовать фильтр (поля выбирайте на свой выбор, например услуга: цена, имя; )
- Реализовать сортировку (поля выбирайте на свой выбор)
- Реализовать пагинацию для всех сущностей

**Cars:**

При GET запросе, включить по связи модель (brand)

**Services:**

у услуги, цену и время храним в мин. единицах,
т.е когда создается услуга, цена и время указывается в рублях и в минутах, соо-о,
вам нужно эти данные преобразовать в копейки и секунды и сохранить в бд

цена и время, это VO, при GET запросе на услугу, в ответе должно быть примерно такое

```
GET /services/1
{
	id: 1,
	name: "Мойка окон",
	price: {
	 minValue: 100,
	 maxValue: 1,
	 format: "1 руб."
	},
	time: {
	 second: 600,
	 minute: 10,
	},
}
```

**User:**

При GET запросе, включить по связи роли,
но тут можно не делать логику связей мн. ко мн. а просто в таблице user, указать role_id = 1,2,3 (1 - админ, 2 - работник, 3 - клиент)

**CustomerCars:**

При GET запросе, включить по связи машину(бренд и марка) и пользователя (у пользователя берем только ФИО и email)

**Orders:**

При GET запросе, включить по связи услугу, машину клиента, работника, админа

При создании заказа, в заказ указываются услуги, после создания в ответе от сервера, мы должны увидеть всю информацию о заказе и увидеть общее время выполнения заказа и общую стоимость

время и цену берем из услуг

end_date у услуги, должно рассчитаться автоматически на основе времени услуг

start_date — это текущая дата

Должен быть отдельный ендпоинт для обновления статуса заказа

Пример

```
GET /order/1
{
	id: 1,
	status: 1,
	start_date: '2024-10-10 12:00:00',
	end_date: '2024-10-10 12:40:00',
	totalTime: 30 // минуты
	totalPrice: 500 // рублей
	administrator: {
		id: 10,
		fullName: "Иван Иванов Иванович"
	},
	employee: {
		id: 11,
		fullName: "Иван Иванов Иванович",
	},
	customerCar: {
		id: 2,
		year: 2020,
		number: "23424",
		customer: {
			id: 3,
			fullName: "Иван Иванов Иванович",
			email: "test@gm.com"
		},
		car: {
			model: "A3"
			brand: "Audi"
		}
	}
}
```

---

Добавить проверку ролей:

- Только роль администратор может все создавать
- Просматривать свои заказы (GET) может работник и клиент, остальные действия запрещены


## Общая логика

У пользователя 3 роли:

- администратор
- работник
- клиент

Заказ создает текущий пользователь - администратор, выбирает машину клиента, работника и услуги

Статусов у заказа 2

- в работе
- завершен

Если у клиента поле is_send_notify в true, значит после завершения заказа отправить письмо

Используем сервисный слой, т.е данные получили и куда-то их передаем. Контролер только получает request и возвращает response

Можно добавлять услуги, уже в созданный заказ, но, когда статус заказа не выполнен (т.е status = 1)

Если добавить услуги, которые уже есть в заказе, должна быть ошибка, например "данная услуга (какая именно указать) уже присутствует в заказе"
