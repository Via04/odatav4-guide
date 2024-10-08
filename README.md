# Odatav4. Quickstart.
Итак, начать делать сервис можно как обычно в транзакции SEGW, можно сделать сервис полностью вручную, а затем его опубликовать или можно использовать RESTful Application Programming (RAP) Model. Но последний подход не реализован в SAP в версии 750, поэтому здесь рассматриваться не будет. 
## Начало.
В транзакции SEGW жмем Create project и заполняем название/описание, в project type выбираем загадочное 4 в generation strategy выбираем 1 - Standard OData 4.0.
![segw_create.png](https://github.com/Via04/odatav4-guide/static/segw_create.png)

При выборе появится предупреждение с нотой.
![segw_warn.png](https://github.com/Via04/odatav4-guide/blob/master/static/segw_warn.png)

Суть в том, что нас предупреждают, что начиная с текущей версии SEGW является legacy способом создания сервисов и лучше либо полностью с нуля реализовывать сервис (потому что таким образом можно полностью через eclipse разрабатывать сервис) либо через RAP. В нашем случае можно смело это предупреждение игнорировать, но тем не менее ниже я расскажу, как создать сервис без SEGW.
Итак, мы получим в SEGW сервис, принципы работы с которым, в основном, такие же как и с oData v2. Главным исключением является создание Navigation Property. Теперь ассоциации исчезли и связь между типами устанавливается напрямую в Navigation Property, а тип связи контролируется двумя галочками Collection и Nullable (см. скриншот ниже).
![nav_prop.png](https://github.com/Via04/odatav4-guide/blob/master/static/nav_prop.png)


