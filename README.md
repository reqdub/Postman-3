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
- <details>
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

4) http://162.55.220.72:5005/test_pet_info
req.
POST
age: int
weight: int
name: str
auth_token


Resp.
{'name': name,
 'age': age,
 'daily_food':weight * 0.012,
 'daily_sleep': weight * 2.5}


Тесты:
1) Статус код 200
2) Проверка структуры json в ответе.
3) В ответе указаны коэффициенты умножения weight, напишите тесты по проверке правильности результата перемножения на коэффициент.

===================

5) http://162.55.220.72:5005/get_test_user
req.
POST
age: int
salary: int
name: str
auth_token

Resp.
{'name': name,
 'age':age,
 'salary': salary,
 'family':{'children':[['Alex', 24],['Kate', 12]],
 'u_salary_1.5_year': salary * 4}
  }

Тесты:
1) Статус код 200
2) Проверка структуры json в ответе.
3) Проверить что занчение поля name = значению переменной name из окружения
4) Проверить что занчение поля age в ответе соответсвует отправленному в запросе значению поля age

===================

6) http://162.55.220.72:5005/currency
req.
POST
auth_token

Resp. Передаётся список массив объектов.
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

Тесты:
1) Можете взять любой объект из присланного списка, используйте js random.
В объекте возьмите Cur_ID и передать через окружение в следующий запрос.

 ===================

7) http://162.55.220.72:5005/curr_byn
req.
POST
auth_token
curr_code: int

Resp.
{
    "Cur_Abbreviation": str
    "Cur_ID": int,
    "Cur_Name": str,
    "Cur_OfficialRate": float,
    "Cur_Scale": int,
    "Date": str
}

Тесты:
1) Статус код 200
2) Проверка структуры json в ответе.


===============
***
1) получить список валют
2) итерировать список валют
3) в каждой итерации отправлять запрос на сервер для получения курса каждой валюты
4) если возвращается 500 код, переходим к следующей итреации
5) если получаем 200 код, проверяем response json на наличие поля "Cur_OfficialRate"
6) если поле есть, пишем в консоль инфу про фалюту в виде response
{
    "Cur_Abbreviation": str
    "Cur_ID": int,
    "Cur_Name": str,
    "Cur_OfficialRate": float,
    "Cur_Scale": int,
    "Date": str
}
7) переходим к следующей итерации
