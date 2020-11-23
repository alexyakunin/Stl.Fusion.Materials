---
marp: true
theme: default
class: 
  - invert
---
<style>
section.video {
  padding: 0px;
  margin: 0px;
}
section.video iframe {
  width: 100%;
  height: 100%;
}

div.col2 {
  margin-top: 35px;
  column-count: 2;
}
div.col2 p:first-child,
div.col2 h1:first-child,
div.col2 h2:first-child,
div.col2 h3:first-child,
div.col2 ul:first-child,
div.col2 ul li:first-child,
div.col2 ul li p:first-child {
  margin-top: 0 !important;
}
div.col2 .break {
  break-before: column;
  margin-top: 0;
}
</style>

![bg left:60%](./img/Racoon.gif)

## Is real-time UI </br>really hard to code</br> or do I suck?

---
<!-- _class: video -->
<iframe src="https://www.youtube.com/embed/xJyE2QDEASA" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---
![bg right](./img/DogOnFire.jpg)

С вами все в прорядке. Это с real-time UI все сложно.

## О чем вы узнаете:
- Какие проблемы нужно решить, чтоб сделать real-time UI?
- При чем здесь инвалидация кэша?
- Что именно делает React (и Blazor) настолько удобным?
- Почему real-time UI - это важно?

---
# Поток данных при отображении в UI

![bg brightness:0.2](./img/Flow1.jpg)

БД и внешние сервисы
 &nbsp; &rarr; Сервисы приложения 
 &nbsp; &nbsp; &rarr; API 
 &nbsp; &nbsp; &nbsp; &rarr; Клиент 
 &nbsp; &nbsp; &nbsp; &nbsp; &rarr; UI

---

![bg](./img/SupplyChain.jpg)

---
# UI, как композиция функций

```cs
// Client
string RenderAppUI() { 
  // Uses router, which ends up calling RenderUserName
} 

string RenderUserName(string userId) {
  var user = UserApiClient.GetUser(userId);
  return $"<div>{user.Name}</div>";
}

// API controller
UserModel GetUser(string userId) {
  var user = UserRepository.Get(string userId);
  return new UserModel(user.Id, user.Name, ...);
}

// UserRepository
User Get(string userId) { ... }
```

---
# Почему же на практике мы делаем все иначе?

1. Вычислять все заново на каждый рендер - дорого
2. Часть вызовов требуют RPC, а это еще и долго.

---
# Но постойте...

1. Вычислять все заново на каждый рендер - дорого
   <span style="color: #f44">А не для этого ли выдумали кеширование?</span>
2. Часть вызовов требуют RPC, а это еще и долго.
   <span style="color: #f44">А не поэтому ли их все временно хранят на клиенте?<span>

---
# А что это вообще за зверь - кеширование?

![bg right](./img/ShockedDog.jpg)

Это просто временное хранение + повторное использование результата, вычисленного ранее.

<span style="color: #f44">Но ведь тогда получается, что <b>мы кешируем вообще все!</b></span>

---
# Кеширование, как higher order function

```cs
Func<object[], object> Cached(Func<object[], object> computer)
  => args => {
    var key = CreateKey(args);
    if (TryGetCached(key, out var cached)) return cached;
    lock (GetLock(key)) { // Double-check locking
      if (TryGetCached(key, out var cached)) return cached;
      cached = computer(args);
      StoreCached(key, cached);
      return cached;
    }
  }

var getUser = (Func<string[], string>) (args => UserRepository.Get(args[0]));
var getUserCached = Cached(getUser);
```
---
# Un problema*

![bg right](./img/ShockedCat1.jpg)

**computer** в примере выше должен быть *pure function*, иначе вся наша замечательная теория превращается в тыкву!

<footer>
(*) Мелочи вроде не-async кода пока пропустим. Представим, что у нас не потоки, а goroutines.
</footer>

---
# Возможные решения*

<div class="col2">
<p>1. Сделать чистыми все функции</p>
<p><img src="./img/Hell.gif" width="100%"></p>

<div class="break">
<p>2. Отслеживание зависимостей и каскадная инвалидация</p>
<p><img src="./img/Chance.jpg" width="100%"></p>
</div>

<footer>
(*) Уверен, есть и другие решения, но для них не хватило места на слайде.
</footer>
