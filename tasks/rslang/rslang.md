Получение подробного описания слова (например значение слова, транскрипция, часть речи, синонимы, антонимы, примеры использования в предложении) https://dictionary.skyeng.ru/doc/api/external. API полностью бесплатна и не требует регистрации.

### Пример получени подробной информации о слове

Сам запрос:

https://dictionary.skyeng.ru/api/public/v1/words/search?search=cat (`?search=${ вставить нужное слово }`)

Пример в коде:

```javascript
const getWordDetalization = async (word) => {
  const rawResponse = await fetch(
    `https://dictionary.skyeng.ru/api/public/v1/words/search?search=${word}`
  );

  const content = await rawResponse.json();

  console.log(content);
};

getWordDetalization('cat');
```

Возвращаемый JSON:

```json
[
  {
    "id": 174834,
    "text": "CAT",
    "meanings": [
      {
        "id": 248983,
        "partOfSpeechCode": "abb",
        "translation": {
          "text": "программно-техническое средство ",
          "note": null
        },
        "previewUrl": "//d2zkmv5t5kao9.cloudfront.net/images/7f116a91c1912c5cc24b2f44d0b945f3.png?w=96",
        "imageUrl": "//d2zkmv5t5kao9.cloudfront.net/images/7f116a91c1912c5cc24b2f44d0b945f3.png?w=640",
        "transcription": "kæt",
        "soundUrl": "//d2fmfepycn0xw0.cloudfront.net?gender=male&accent=british&text=CAT"
      }
    ]
  },
  {
    "id": 1560,
    "text": "cat",
    "meanings": [
      {
        "id": 65977,
        "partOfSpeechCode": "n",
        "translation": {
          "text": "кот",
          "note": "кошка"
        },
        "previewUrl": "//d2zkmv5t5kao9.cloudfront.net/images/55bd5010ef32706be7b7e371673c1b1c.jpeg?w=96",
        "imageUrl": "//d2zkmv5t5kao9.cloudfront.net/images/55bd5010ef32706be7b7e371673c1b1c.jpeg?w=640",
        "transcription": "kæt",
        "soundUrl": "//d2fmfepycn0xw0.cloudfront.net?gender=male&accent=british&text=cat"
      },
      ...
    ]
  }
]
```

Внизу документации по ссылке (https://dictionary.skyeng.ru/doc/api/external) есть расшифровки сокращений.

Например во вкладке Models есть расшифровка части речи:

partOfSpeechCode (string):

- n - noun,
- v - verb,
- j - adjective,
  
  ....
- phi - idiom.

#### Замечание: первое значение в meanings не всегда может быть тем переводом, что вам нужен. Будьте внимательны.

## Примеры получения исходных данных

Для этого создан REST API по адресу: https://afternoon-falls-25894.herokuapp.com/

Для тестирования API можно пользоваться Swagger докой по адресу: https://afternoon-falls-25894.herokuapp.com/doc/#/

Описание эндпоинтов(не полное, смотри Swagger Doc):

### Words

GET для получения списка слов:
`https://afternoon-falls-25894.herokuapp.com/words?page=2&group=0` - получить слова со 2-й страницы группы 0  
Строка запроса должна содержать в себе номер группы и номер страницы. Всего 6 групп(от 0 до 5) и в каждой группе по 30 страниц(от 0 до 29). В каждой странице по 20 слов. Группы разбиты по сложности от самой простой(0) до самой сложной(5).  

##### Word+Assets

GET запрос для получения одного слова по ID возвращает слово в виде JSON объекта в котором поля `image`, `audio`, `audioMeaning` и `audioExample` содержат изображения и аудиофайлы закодированные в `base64`.  
Пример получения и преобразования `base64` данных в соответствующие теги HTML:  
```html
<html>
<body></body>
<script>
  fetch('http://localhost:4000/words/5e9f5ee35eb9e72bc21af4a0')
    .then(r => r.json())
    .then(data => {
      const image = new Image();
      image.src = `data:image/jpg;base64,${data.image}`;
      document.body.appendChild(image);
      const audio = new Audio();
      audio.src = `data:audio/mpeg;base64,${data.audio}`;
      audio.controls = 'controls';
      document.body.appendChild(audio);
    })
</script>
</html>
```

### Users

Система поддерживает разграничение данных по пользователям, в рамках данной задачи вам понадобится создать форму для регистрации пользователя. Для этого надо использовать POST эндпоинт по адресу `/users`. В запросе надо передать JSON объект, который содержит name, e-mail и password пользователя. Пароль должен содержать не менее 8 символов, как минимум одну прописную букву, одну заглавную букву, одну цифру и один спецсимвол из `+-_@$!%*?&#.,;:[]{}`. Пример запроса:  
```javascript
const createUser = async user => {
     const rawResponse = await fetch('https://afternoon-falls-25894.herokuapp.com/users', {
       method: 'POST',
       headers: {
         'Accept': 'application/json',
         'Content-Type': 'application/json'
       },
       body: JSON.stringify(user)
     });
     const content = await rawResponse.json();
   
     console.log(content);
   };

createUser({ "name": "TestUser", "email": "hello@user.com", "password": "Gfhjkm_123" });
-------------------------------------------------------------------
Console: {
  id: "5ec993df4ca9d600178740ae", 
  email: "hello@user.com"
}
```

### Sign In

Чтобы пользоваться эндпоинтами требующими авторизации необходимо залогиниться в систему и получить JWT токен. Для этого существует POST эндоинт по адресу `/signin`. Токены имеют ограниченный срок жизни, в текущей реализации это 4 часа с момента получения. Пример запроса:  
```javascript
const loginUser = async user => {
  const rawResponse = await fetch('https://afternoon-falls-25894.herokuapp.com/signin', {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(user)
  });
  const content = await rawResponse.json();

  console.log(content);
};

loginUser({ "email": "hello@user.com", "password": "Gfhjkm_123" });
-------------------------------------------------------------------
Console: 
{
  "message":"Authenticated",
  "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVlYzk5M2RmNGNhOWQ2MDAxNzg3NDBhZSIsImlhdCI6MTU5MDI2OTE1OCwiZXhwIjoxNTkwMjgzNTU4fQ.XHKmdY_jk1R7PUbgCZfqH8TxH6XQ0USwPBSKNHMdF6I",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVlYzg5NDI4NzhjMWU4NGI0M2U4NzFhYyIsInRva2VuSWQiOiI3MTBiODZjZS1hMTJjLTQ3YzMtYjkzYy1kNDUzZmJiYmI0OGIiLCJpYXQiOjE1OTM3MDE5NTcsImV4cCI6MTU5MzcxNjM1N30.7wgIfUG_3fUcpb-yoZVm4pPzgdvkQvulOWiL3x7y85E",
  "userId":"5ec993df4ca9d600178740ae",
  "name": "TestUser"
}
```

### Refresh JWT token

JWT токен это JSON объект состоящий из трех частей(header, payload and signature), который закодирован `Base64`. В этом можно убедиться вставив полученный токен в соответствующее поле на сайте [jwt.io](https://jwt.io/#debugger-io). Если посмотреть на декодированный результат, то в `payload` можно увидеть поле `exp` это представление даты в миллисекундах до которой будет валидным токен. В браузерном API есть [функции которые позволяют кодировать/декодировать `Base64`](https://developer.mozilla.org/en-US/docs/Glossary/Base64).  
В API существует GET эндпоинт `/users/{id}/tokens` который позволяет получить свежий токен без повторной авторизации. Запросы к этому эндпоинту должны быть подписаны **refreshToken** полученном в ответе сервера после аутентификации пользователя. Успешный запрос к этому эндпоинту вернет новые токен и рефреш-токен.
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVlYzg5NDI4NzhjMWU4NGI0M2U4NzFhYyIsImlhdCI6MTU5MzcwMzQxNiwiZXhwIjoxNTkzNzE3ODE2fQ.6IKvBYz49az9ioasHZQB63NIXujXkY5K1pHMxYJ_-FE",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVlYzg5NDI4NzhjMWU4NGI0M2U4NzFhYyIsInRva2VuSWQiOiJjMDg3MDk2My1hNjhmLTRlMzUtYWYyNS03Mjg2ZDk0NmVhMmQiLCJpYXQiOjE1OTM3MDM0MTYsImV4cCI6MTU5MzcxNzgxNn0.fBHuICfTYePElVcmYyl7ZW2Qnzw0iHyFdNr7-KiRpG4"
}
```

### Users/Words

Полученный при успешном логине токен надо использовать при каждом запросе к эндпоинтам требующим авторизации (в Swagger такие эндпоинты имеют иконку навесного замка. При работе со Swagger полученный токен надо вставить в соответствующее поле формы, которая появляется при нажатии на кнопку `Authorize` вверху страницы справа). Примеры запросов:    
```javascript
const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjVlYzk5M2RmNGNhOWQ2MDAxNzg3NDBhZSIsImlhdCI6MTU5MDI2OTE1OCwiZXhwIjoxNTkwMjgzNTU4fQ.XHKmdY_jk1R7PUbgCZfqH8TxH6XQ0USwPBSKNHMdF6I';
const createUserWord = async ({ userId, wordId, word }) => {
  const rawResponse = await fetch(`https://afternoon-falls-25894.herokuapp.com/users/${userId}/words/${wordId}`, {
    method: 'POST',
    withCredentials: true,
    headers: {
      'Authorization': `Bearer ${token}`,
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(word)
  });
  const content = await rawResponse.json();

  console.log(content);
};

createUserWord({
  userId: "5ec993df4ca9d600178740ae",
  wordId: "5e9f5ee35eb9e72bc21af716",
  word: { "difficulty": "weak", "optional": {testFieldString: 'test', testFieldBoolean: true} }
});
-------------------------------------------------------------------
Console: {
  "id":"5ec9a92acbbd77001736b167",
  "difficulty":"weak",
  "optional":{
    "testFieldString":"test",
    "testFieldBoolean":true
  },
  "wordId":"5e9f5ee35eb9e72bc21af716"
}

const getUserWord = async ({ userId, wordId }) => {
  const rawResponse = await fetch(`https://afternoon-falls-25894.herokuapp.com/users/${userId}/words/${wordId}`, {
    method: 'GET',
    withCredentials: true,
    headers: {
      'Authorization': `Bearer ${token}`,
      'Accept': 'application/json',
    }
  });
  const content = await rawResponse.json();

  console.log(content);
};

getUserWord({
  userId: "5ec993df4ca9d600178740ae",
  wordId: "5e9f5ee35eb9e72bc21af716"
});
-------------------------------------------------------------------
Console: {
  "id":"5ec9a92acbbd77001736b167",
  "difficulty":"weak",
  "optional":{
    "testFieldString":"test",
    "testFieldBoolean":true
  },
  "wordId":"5e9f5ee35eb9e72bc21af716"
}
```

### Users/AggregatedWords

Данные эндпоинты позволяют получить список слов(или одно слово) объединенный с существующими данными в userWords. Например,  в БД существуют userWords следующего вида:
```json
[
  {
    "id": "5ee9d0709045590017eb504b",
    "difficulty": "weak",
    "optional": {
      "testFieldString": "test",
      "testFieldBoolean": true
    },
    "wordId": "5e9f5ee35eb9e72bc21af716"
  },
  {
    "id": "5eebcde0d3f9e856e68e74e7",
    "difficulty": "easy",
    "optional": {
      "field": "value1"
    },
    "wordId": "5e9f5ee35eb9e72bc21af4a1"
  },
  {
    "id": "5eee2d5d3b0620240caa89e1",
    "difficulty": "strong",
    "wordId": "5e9f5ee35eb9e72bc21af4a0"
  }
]
```
Запрос к `/users/{id}/aggregatedWords` со следующими параметрами:  
```
userId: 5ec8942878c1e84b43e871ac
group: 0
wordsPerPage: 3
filter: {
	"$or": [
	{"userWord.difficulty":"strong"},
	{"userWord":null}
	]
}
```
<details> 
  <summary>должен вернуть такой ответ:</summary>

  <p></p>

```json
[
  {
    "paginatedResults": [
      {
        "_id": "5e9f5ee35eb9e72bc21af4a0",
        "group": 0,
        "page": 0,
        "word": "alcohol",
        "image": "files/01_0002.jpg",
        "audio": "files/01_0002.mp3",
        "audioMeaning": "files/01_0002_meaning.mp3",
        "audioExample": "files/01_0002_example.mp3",
        "textMeaning": "<i>Alcohol</i> is a type of drink that can make people drunk.",
        "textExample": "A person should not drive a car after he or she has been drinking <b>alcohol</b>.",
        "transcription": "[ǽlkəhɔ̀ːl]",
        "textExampleTranslate": "Человек не должен водить машину после того, как он выпил алкоголь",
        "textMeaningTranslate": "Алкоголь - это тип напитка, который может сделать людей пьяными",
        "wordTranslate": "алкоголь",
        "wordsPerExampleSentence": 15,
        "userWord": {
          "difficulty": "strong"
        }
      },
      {
        "_id": "5e9f5ee35eb9e72bc21af4a2",
        "group": 0,
        "page": 0,
        "word": "boat",
        "image": "files/01_0005.jpg",
        "audio": "files/01_0005.mp3",
        "audioMeaning": "files/01_0005_meaning.mp3",
        "audioExample": "files/01_0005_example.mp3",
        "textMeaning": "A <i>boat</i> is a vehicle that moves across water.",
        "textExample": "There is a small <b>boat</b> on the lake.",
        "transcription": "[bout]",
        "textExampleTranslate": "На озере есть маленькая лодка",
        "textMeaningTranslate": "Лодка - это транспортное средство, которое движется по воде",
        "wordTranslate": "лодка",
        "wordsPerExampleSentence": 8
      },
      {
        "_id": "5e9f5ee35eb9e72bc21af4a3",
        "group": 0,
        "page": 0,
        "word": "arrive",
        "image": "files/01_0003.jpg",
        "audio": "files/01_0003.mp3",
        "audioMeaning": "files/01_0003_meaning.mp3",
        "audioExample": "files/01_0003_example.mp3",
        "textMeaning": "To <i>arrive</i> is to get somewhere.",
        "textExample": "They <b>arrived</b> at school at 7 a.m.",
        "transcription": "[əráiv]",
        "textExampleTranslate": "Они прибыли в школу в 7 часов утра",
        "textMeaningTranslate": "Приехать значит попасть куда-то",
        "wordTranslate": "прибыть",
        "wordsPerExampleSentence": 7
      }
    ],
    "totalCount": [
      {
        "count": 599
      }
    ]
  }
]
```
</details> 
 - поле `paginatedResults` это результат агрегации `Words & UserWords` с учетом заданных параметров запроса и фильтра.    
 - поле `totalCount` показывает сколько всего существует слов в данной группе соответствующих данному фильтру.  

Пример запроса к `/users/{id}/aggregatedWords` с более сложным фильтром:  
```
userId: 5ec8942878c1e84b43e871ac
group: 0
wordsPerPage: 3
filter: {
  "$or": [
    {
      "$and": [
        {
          "$or": [
            {
              "userWord.difficulty": "strong"
            },
            {
              "userWord.difficulty": "easy"
            }
          ]
        }
      ]
    },
    {
      "userWord": null
    }
  ]
}
```
<details> 
  <summary>должен вернуть такой ответ:</summary>

  <p></p>

```json
	
Response body
Download
[
  {
    "paginatedResults": [
      {
        "_id": "5e9f5ee35eb9e72bc21af4a0",
        "group": 0,
        "page": 0,
        "word": "alcohol",
        "image": "files/01_0002.jpg",
        "audio": "files/01_0002.mp3",
        "audioMeaning": "files/01_0002_meaning.mp3",
        "audioExample": "files/01_0002_example.mp3",
        "textMeaning": "<i>Alcohol</i> is a type of drink that can make people drunk.",
        "textExample": "A person should not drive a car after he or she has been drinking <b>alcohol</b>.",
        "transcription": "[ǽlkəhɔ̀ːl]",
        "textExampleTranslate": "Человек не должен водить машину после того, как он выпил алкоголь",
        "textMeaningTranslate": "Алкоголь - это тип напитка, который может сделать людей пьяными",
        "wordTranslate": "алкоголь",
        "wordsPerExampleSentence": 15,
        "userWord": {
          "difficulty": "strong"
        }
      },
      {
        "_id": "5e9f5ee35eb9e72bc21af4a2",
        "group": 0,
        "page": 0,
        "word": "boat",
        "image": "files/01_0005.jpg",
        "audio": "files/01_0005.mp3",
        "audioMeaning": "files/01_0005_meaning.mp3",
        "audioExample": "files/01_0005_example.mp3",
        "textMeaning": "A <i>boat</i> is a vehicle that moves across water.",
        "textExample": "There is a small <b>boat</b> on the lake.",
        "transcription": "[bout]",
        "textExampleTranslate": "На озере есть маленькая лодка",
        "textMeaningTranslate": "Лодка - это транспортное средство, которое движется по воде",
        "wordTranslate": "лодка",
        "wordsPerExampleSentence": 8
      },
      {
        "_id": "5e9f5ee35eb9e72bc21af4a1",
        "group": 0,
        "page": 0,
        "word": "agree",
        "image": "files/01_0001.jpg",
        "audio": "files/01_0001.mp3",
        "audioMeaning": "files/01_0001_meaning.mp3",
        "audioExample": "files/01_0001_example.mp3",
        "textMeaning": "To <i>agree</i> is to have the same opinion or belief as another person.",
        "textExample": "The students <b>agree</b> they have too much homework.",
        "transcription": "[əgríː]",
        "textExampleTranslate": "Студенты согласны, что у них слишком много домашней работы",
        "textMeaningTranslate": "Согласиться - значит иметь то же мнение или убеждение, что и другой человек",
        "wordTranslate": "согласна",
        "wordsPerExampleSentence": 8,
        "userWord": {
          "difficulty": "easy",
          "optional": {
            "field": "value1"
          }
        }
      }
    ],
    "totalCount": [
      {
        "count": 600
      }
    ]
  }
]
```
</details>  

Параметр filter должен быть валидным JSON объектом преобразованным в строку. Все ключи в этом объекте должны быть в двойных кавычках. Кроме этого этот объект должен соответствовать требованиям к `MongoDB Query`. Вот ряд ссылок на документацию и примеры использования `MonogoDB Query`:  
 - [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)  
 - [Query and Projection Operators](https://docs.mongodb.com/manual/reference/operator/query/#query-and-projection-operators)  
 - [An Introduction to MongoDB Query for Beginners](https://blog.exploratory.io/an-introduction-to-mongodb-query-for-beginners-bd463319aa4c)  
 - [MongoDB - Query Document](https://www.tutorialspoint.com/mongodb/mongodb_query_document.htm)  

Для преобразования строки filter в валидный query-параметр можно использовать следующую функцию: [encodeURIComponent()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent)  

Примеры фильтров:  
 - Получить все слова у которых difficulte="hard" **И** optional.key="value" `{"$and":[{"userWord.difficulty":"hard", "userWord.optional.key":"value"}]}`
 - Получить все слова у которых difficulty="easy" **ИЛИ** или нет соответствующего userWord `{"$or":[{"userWord.difficulty":"easy"},{"userWord":null}]}`
 - Получить все слова у которых (difficulty="easy" **И** optional.repeat=true) **ИЛИ**  или нет соответствующего userWord `{"$or":[{"$and":[{"userWord.difficulty":"easy", "userWord.optional.repeat":true}]},{"userWord":null}]}`

Эндпоинт `/users/{id}/aggregatedWords/{wordId}` позволяет получить агрегированый объект конкретного слова.  

### Прочие эндпоинты

Также существуют эндпоинты для сохранения статистики и настроек пользователя. `\users\{id}\statistics` и `\users\{id}\settings` соответственно. Работа с ними основывается на тех же принципах, что описаны и показаны в примерах выше.  
Объект `optional` у UserWord, Statistics, Settings имеет ограничение по размеру - не более 30 полей и общая длина объекта после `JSON.stringify()` не должна превышать 1500 символов. Структуру этих объектов вы разрабатываете сами исходя из требований и вашей реализации задачи.   

REST сервис возвращает только JSON, без изображений и звуковых файлов. Для доступа к ним сделайте fork репозитория https://github.com/irinainina/rslang-data. В папке files находятся изображения и аудиофайлы.

Например, для изображения files/01_0009.jpg ссылка будет следующей:
`https://raw.githubusercontent.com/irinainina/rslang-data/master/files/01_0009.jpg`  
Для аудио files/01_0009_example.mp3 ссылка будет следующей: `https://raw.githubusercontent.com/irinainina/rslang-data/master/files/01_0009_example.mp3`  
Обратите внимание, что вместо irinainina надо указать свой github username.

Ещё один вариант получения исходных данных - файлы, которые находятся в папке data форкнутого репозитория.

Также, если вам чего-то не хватает в текущей реализации, то вы можете форкнуть backend репозиторий, изменить его под свою реализацию, задеплоить и использовать.  
[Git repository link](https://github.com/rolling-scopes/LearnWords) 

## Cross-check
- инструкция по проведению cross-check: https://docs.rs.school/#/cross-check-flow
- форма для проверки задания https://rslang-cross-check.netlify.com/

## Документ для вопросов
- документ для вопросов, связанных с выполнением задания: https://docs.google.com/spreadsheets/d/13rqjNCjiTsQw95gfoubgDjUmdotgbIk3J_WxAiHbYBE/edit#gid=0
