@frey: там же есть фейк-стабы для воркеров https://github.com/mperham/sidekiq/wiki/Testing

Не, ты можешь по хардкору код вызывать

``` ruby
MyWorker.new.perform()
```

И дальше смотреть, что изменилось/вернулось
+ в самом сайдкике есть функция выполнения воркеров синхронно. Ссылка выше вроде об этом