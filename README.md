## 1) POST http://162.55.220.72:5005/login
### Params:
- login : postlogin
- password : postpass
#### Получаю токен : /s34lfgbj/None/jjd909/58316kjkWpqc1036None238912evny
#### В окружении создаю переменную auth_token со значением токена, которая будет далее передаваться в запросы
#### Тесты : ---нет---
---
## 2) http://162.55.220.72:5005/user_info
### Params (raw_json):
        {
            "age" : 30,
            "salary" : 1000,
            "name" : "Aleksey",
            "auth_token" : "/s34lfgbj/None/jjd909/58316kjkWpqc1036None238912evny"
        }

### Response:
        {
            'start_qa_salary':salary,
            'qa_salary_after_6_months': salary * 2,
            'qa_salary_after_12_months': salary * 2.9,
            'person': {
                'u_name':[user_name, salary, age],
                'u_age':age,
                'u_salary_1.5_year': salary * 4
            }
        }

### Тесты:
#### Переменные:
- var requestData = JSON.parse(pm.request.body.raw);
- var responseData = pm.response.json();
<details>
  <summary>JSON Schema</summary>
                const schema = {
                  "type": "object",
                  "properties": {
                    "person": {
                      "type": "object",
                      "properties": {
                        "u_age": {
                          "type": "number"
                        },
                        "u_name": {
                          "type": "array",
                          "items": [
                            {
                              "type": "string"
                            },
                            {
                              "type": "number"
                            },
                            {
                              "type": "number"
                            }
                          ]
                        },
                        "u_salary_1_5_year": {
                          "type": "number"
                        }
                      }
                    },
                    "qa_salary_after_12_months": {
                      "type": "number"
                    },
                    "qa_salary_after_6_months": {
                      "type": "number"
                    },
                    "start_qa_salary": {
                      "type": "number"
                    }
                  }
                }
</details>

#### 1) Статус код 200
    pm.test("Status code is 200", function () {
        pm.response.to.have.status(200);
    });
#### 2) Проверка структуры json в ответе
        pm.test("Validate schema", () => {
                pm.response.to.have.jsonSchema(schema);
        });

#### 3) В ответе указаны коэффициенты умножения salary, напишите тесты по проверке правильности результата перемножения на коэффициент
        pm.test("Validate salarie coefficients", () => {
            pm.expect(responseData["start_qa_salary"]).equal(requestData["salary"]);
            pm.expect(responseData["qa_salary_after_6_months"]).equal(requestData["salary"] * 2);
            pm.expect(responseData["qa_salary_after_12_months"]).equal(requestData["salary"] * 2.9);
            pm.expect(responseData["person"]["u_salary_1_5_year"]).equal(requestData["salary"] * 4);
        });

#### 4) Достать значение из поля 'u_salary_1.5_year' и передать в поле salary запроса http://162.55.220.72:5005/get_test_user
        pm.environment.set("salary", responseData.person.u_salary_1_5_year);

---

## 3) POST http://162.55.220.72:5005/new_data
### Body form-data:
- age: 30
- salary: 1000
- name: Aleksey
- auth_token = {{auth_token}}

### Response:
        {
          'name':name,
          'age': int(age),
          'salary': [salary, str(salary*2), str(salary*3)]
        }

### Тесты:
#### Переменные:
- var RQ = pm.request.body.formdata;
- var RSP = pm.response.json();
<details>
        <summary>JSON Schema</summary>
          var schema = {
                        "type": "object",
                        "properties": {
                                "age": {
                                        "type": "number"
                                },
                                "name": {
                                        "type": "string"
                        },
                                "salary": {
                                        "type": "array"
                                }
                        }
                }
</details>

#### 1) Статус код 200
        pm.test("Status code is 200", function () {
            pm.response.to.have.status(200);
        });
#### 2) Проверка структуры json в ответе.
        pm.test("Validate schema", () => {
            pm.response.to.have.jsonSchema(schema);
        });
#### 3) В ответе указаны коэффициенты умножения salary, напишите тесты по проверке правильности результата перемножения на коэффициент.
        pm.test("Coeficcients check", () => {
            pm.expect(RSP.salary[0]).equal(+RQ.get("salary"));
            pm.expect(+RSP.salary[1]).equal(RQ.get("salary") * 2);
            pm.expect(+RSP.salary[2]).equal(RQ.get("salary") * 3);
        });
#### 4) проверить, что 2-й элемент массива salary больше 1-го и 0-го
        pm.test("Salaries compare", () => {
            pm.expect(+RSP.salary[2]).greaterThan(RSP.salary[0]);
            pm.expect(+RSP.salary[2]).greaterThan(+RSP.salary[1]);
        });
---

## 4) POST http://162.55.220.72:5005/test_pet_info
### Body (form-data):
- age: 2
- weight: 8
- name: Chucha
- auth_token: {{auth_token}}

### Response
        {
        'name': name,
         'age': age,
         'daily_food':weight * 0.012,
         'daily_sleep': weight * 2.5
         }

### Тесты:
#### Переменные:
- var RQ = pm.request.body.formdata;
- var RSP = pm.response.json();
<details>
        <summary>JSON Schema</summary>
        var schema = {
        "type": "object",
        "properties": {
                        "age": {
                                "type": "number"
                        },
                        "daily_food": {
                                "type": "number"
                        },
                        "daily_sleep": {
                                "type": "number"
                        },
                        "name": {
                                "type": "string"
                        }
                }
        }
</details>

#### 1) Статус код 200
        pm.test("Status code is 200", function () {
            pm.response.to.have.status(200);
        });
#### 2) Проверка структуры json в ответе.
        pm.test("Coeficcients check", function () {
            pm.expect(RSP.daily_food).equal(RQ.get("weight") * 0.012);
            pm.expect(RSP.daily_sleep).equal(RQ.get("weight") * 2.5);
        });
#### 3) В ответе указаны коэффициенты умножения weight, напишите тесты по проверке правильности результата перемножения на коэффициент.
        pm.test("Validate schema", () => {
            pm.response.to.have.jsonSchema(schema);
        });

---

## 5) POST http://162.55.220.72:5005/get_test_user
### Body (form-data):
- age: 30
- salary: 1000
- name: Aleksey
- auth_token: {{auth_token}}

### Response:
        {
         'name': name,
         'age':age,
         'salary': salary,
         'family':{'children':[['Alex', 24],['Kate', 12]],
         'u_salary_1.5_year': salary * 4}
        }

### Тесты:
#### Переменные:
- var RSP = pm.response.json();
- var RQ = pm.request.body.formdata;
<details>
        <summary>JSON Schema</summary>
        var schema = {
                "type": "object",
                "required": [
                        "age",
                        "family",
                        "name",
                        "salary"
                ],
                "properties": {
                        "age": {
                                "type": "string"
                        },
                        "family": {
                                "type": "object",
                                "required": [
                                        "children",
                                        "u_salary_1_5_year"
                                ],
                                "properties": {
                                        "children": {
                                                "type": "array"
                                        },
                                        "u_salary_1_5_year": {
                                                "type": "integer"
                                        }
                                }
                        },
                        "name": {
                                "type": "string"
                        },
                        "salary": {
                                "type": "integer"
                        }
                }
        }
</details>

#### 1) Статус код 200
        pm.test("Status code is 200", function () {
            pm.response.to.have.status(200);
        });
#### 2) Проверка структуры json в ответе.
        pm.test("Validate schema", () => {
            pm.response.to.have.jsonSchema(schema);
        });
#### 3) Проверить что занчение поля name = значению переменной name из окружения
        pm.test("Status code is 200", function () {
            pm.expect(RSP.name).equal(pm.environment.get("name"));
        });
#### 4) Проверить что занчение поля age в ответе соответсвует отправленному в запросе значению поля age
        pm.test("Status code is 200", function () {
            pm.expect(RSP.age).equal(RQ.get("age"));
        });

---

## 6) POST http://162.55.220.72:5005/currency
### Body(form-data):
- auth_token

### Response:
        [
        {"Cur_Abbreviation": str,
         "Cur_ID": int,
         "Cur_Name": str
        }
        …
        {"Cur_Abbreviation": str,
         "Cur_ID": int,
         "Cur_Name": str
        }
        ]

### Тесты:
#### Переменные:
- var random_currency = getRandomCurrency(pm.response.json().length);
#### 1) Можете взять любой объект из присланного списка, используйте js random.
        function getRandomCurrency(number) {
          return Math.floor(Math.random() * number);
        }
#### 2) В объекте возьмите Cur_ID и передать через окружение в следующий запрос.
        pm.environment.set("curr_id", pm.response.json()[random_currency]);

---

## 7) POST http://162.55.220.72:5005/curr_byn
### Body(form-data):
- auth_token: {{auth_token}}
- curr_code: {{curr_id}}

### Response:
        {
            "Cur_Abbreviation": str
            "Cur_ID": int,
            "Cur_Name": str,
            "Cur_OfficialRate": float,
            "Cur_Scale": int,
            "Date": str
        }

### Тесты:
#### Переменные:

#### 1) Статус код 200
#### 2) Проверка структуры json в ответе.

---

### 1) получить список валют
### 2) итерировать список валют
### 3) в каждой итерации отправлять запрос на сервер для получения курса каждой валюты
### 4) если возвращается 500 код, переходим к следующей итреации
### 5) если получаем 200 код, проверяем response json на наличие поля "Cur_OfficialRate"
### 6) если поле есть, пишем в консоль инфу про фалюту в виде response
        {
            "Cur_Abbreviation": str
            "Cur_ID": int,
            "Cur_Name": str,
            "Cur_OfficialRate": float,
            "Cur_Scale": int,
            "Date": str
        }
### 7) переходим к следующей итерации

        pm.test("Star", () => {
            var currencies_amount;
            var request = {
                url : 'http://54.157.99.22/currency',
                method : 'POST',
                header : '',
                body : {
                    mode : 'formdata',
                    formdata : {key: "auth_token", value: "123"}
                }
            }
        
            pm.sendRequest(request, function (err, res) {
                currencies_amount = res.json().length;
                console.log("Total currencies amount = " + currencies_amount);
                for (var i = 1; i < currencies_amount; i++) {
                    var new_request = {
                        url : 'http://54.157.99.22/curr_byn',
                        method : 'POST',
                        header : '',
                        body : {
                            mode : 'formdata',
                            formdata : [
                                {key: "auth_token", value: "123"},
                                {key: "curr_code", value: i}
                                ]
                        }
                    }
                    pm.sendRequest(new_request, function (err, res) {
                        if (res.code === 500 || res.code === undefined) {
                        }
                        if(res.code === 200){
                            if (pm.expect(res.json()).property("Cur_OfficialRate")){
                                console.log(res.json());
                            }
                        }
                    });
                }
            });
        });
