
# Pug Mixins -- How to use?

[![N|Solid](https://webcase.studio/wp-content/themes/webcase/build/static/images/logo3.svg)](https://webcase.studio/)

<!--[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://gitlab.com/web_case/wordpress_starter)-->

Чудо миксины для облегчения написание\понимания кода
  - tag    
  - exp
  - set
  - For
  - If/elif/else
  - ext
  - inc
  - trans


# Mixins -  What is this?
По-простому - это функции в Pag-e. Вот [примеры](https://pugjs.org/language/mixins.html).
По факту, мы все используем основной набор готовых миксинов и pug-переменных, о них и пойдет речь. 

Вообще по правильному, нужно разделять миксины и переменые. Если при вызове ты пишешь `+exp`, то ты вызвал миксин, а если `=exp` --То ты вызвал функцию. Не понятно, понимаю...  Давай пока будем все называть `миксинами`. 

Так как их писать труда особого не составляет, штука классная, только клиентские миксины так себе работают с jinja/twig (в них сложно вставить переменные). Поэтому используем готовые вещи, для всяких кастомных карточек и т.д пишем макросы или инклюды... об этом далее... 

P.S. Дока для twig - твиг на 99% такой же как jinja. такой же шаблонизатор, только для  Wordpress.Если тебе нужна jinja читай место twig - jinja) Все будет работать)

## Exp

Начнем с того, что миксин ` +exp('test') ` в компилированном варианте преобразуется в двойные фигурные скобочки `{{ test }}`.  Исходя из этого, нужно понимать для чего юзать. Вот например, есть переменные `title`, `link` и ты хочешь их использовать в своем шаблоне.

У нас есть ссылка в шаблоне: `a(href='') Ссылка `-- необходимо ее 'оживить'. 
Нужно писать так:
```sh
    a(href!=exp('link') ): +exp title
```

Рассмотрим подробнее. Если хочешь написать значение аттрибуту - то всегда пиши `=exp('some_var')`. Восклицательный знак перед этим выражением не разрешит замену на юникод. Вот пример исходника и компилированного кода с восклицательным знаком и без:
```sh
a(href=exp("test('request')"))   ----   <a href="{{ test(&quot;request&quot;) }}"></a>
a(href!=exp("test('request')"))  ----   <a href="{{ test('request') }}"></a>
```
Видишь разницу? Один знак, а сколько проблем решает. Кстати, для простых вызовов, если тебе надо только вывести `=exp('test')` можешь восклицательный знак не писать, разницы не будет. И еще важная вещь, если у тебя внутри переменной, надо написать еще одни кавычки, то всегда пиши сначала двойные, потом одинарные `+exp("function('admin_url', '/admin-ajax.php?action=feedback')")`. Запомни это. А то будет беда!

А что делать, если надо в `href` этой же ссылки вставить две переменые, спросишь ты? Или не спросишь... Но я отвечу! Просто используй конкатенацию строк. 

```sh
a(href=exp("test")+exp('other_var')+'/url/') *** /url/ - это просто строка(и восклицательный знак тут тоже не надо)
```
## tag
Миксин +tag() -- преобразуется в `{%  %}` . Во всех наших миксинах он используется как основа. Поэтому явно ты его можешь писать только для обьявления переменных в шаблоне. Для этого используется еще один миксин -- `+set` -- из его названия, думаю все понятно. Выглядит это так:  
``` sh
    +tag set new_var = 'some test variable'
    +tag set new_arr = [{'key': 'value', 'href': 'some_link.html'}, {'key': 'value1', 'href': 'other_link.php'}]
```
в твоем темплейте теперь есть две новые переменные

```sh
    p: +exp new_var \\pug 
    p {{ new_var }}  \\ jinja/twig 
    p some test variable \\ browser 
```
### For
Обыкновенный иттератор. У нас уже есть массив для иттерации из прошлого примера. Просто пиши: 
```sh
    +for('obj in new_arr')
        a(href=exp('obj.href'): +exp obj.key
```
Что получим в итоге? 
```sh
    a(href='some_link.html') value 
    a(href='other_link.php') value1 
```
###  If/elif/else

Это уже интереснее. 
Давай придумаем другой массив. 
   
     +tag set arr = [{'title': 'Hello', 'href': 'some_link.html'}, {'title': 'World'}, {'other': 'что-то пошло не так'}]

Как ты, надеюсь, заметил, есть обьекты у который есть `title` ,`href`, а у кото-то их нет.

Используем тот же `+for`, только добавим `if`, `else`, `elif`

```sh
   
  +for('obj in arr')
    +if('obj.href')
        a(href=exp('obj.href')): +exp obj.title
        +elif obj.title 
        p: +exp obj.title
        +else
        p: +exp obj.other
    \\\код в браузере 
    <a href="some_link.html">Hello</a>
    <p>World</p>
    <p>что-то пошло не так</p>
```
Наверное, ты подумал, что я что-то накосячила с табуляцией, но нет) Как раз-таки, как бы это не было страннно для понимания, надо писать так. Просто запомни и смирись. 

### ifte
`+ifre` - это инлайновый `+if()`. Самый простой пример:
```
+tag set test_var = true
p(class!=ifte('test_var', 'is-active', 'not-active'))
```
Сейчас если выполняется условие добавится класс `is-active`. Но также можно в положительный и отрицательный результат подавать переменные или оставлять пустую строку: 

```
p(class!=ifte('test_var', exp('active_class'), ''))

```

### Ext
Ext - extend, т.е наследование от какого-то шаблона. Тут необходимо упомянуть еще один миксин `+bl()`
Вот пример базового шаблона(base.pug):
```sh
    body.body
    +bl('header')
      +inc 'header.twig'
    +bl('main')
    +bl('footer')
      +inc 'footer.twig'
```
А Это главная страница, допустим:
```sh 
    +ext "base.twig" -- экстенд от файла  base.twig
    +ext "base.jinja" -- экстенд от файла  base.jinja
    +bl('main') -- перезапись только блока main
```
Обрати внимание, что пишем мы только   `.pug`, a экстендимся от `.twig` или `.jinja`. Так как это фишки шаблонизатора. а не препроцессора. Если нужен файл с какой-то папки, пиши `parts/header.twig`

### Inc
Inc - include, вставить другой файл. Тут будь внимателен с путями. 
Очень просто:
```
 +inc "parts/header.twig"
 +inc 'header.twig'
```
### Url
Url используется для написания ссылок 
То есть у тебя есть в сервере приложение `core`, а в его урлах есть путь `path('sitemap/', SiteMapItemListView.as_view(), name='sitemap')`. И вот ты хочешь, чтобы у тебя в хедере была ссылка на сайтмеп. Просто пиши: 
```
a(href!=url("'core:sitemap'"))
```
Стиль написания именно такой! Двойные ковычки, название приложения, двоеточие, name твоего path-a. Это стандарное, всем понятное, самое распостраненное написание. Лучше пиши так.
### Trans
Тоже два варинта написания:
```

p: +trans Hello, world  -- можешь вызывать миксин
img(alt!=trans('hello')) --- можешь функцию

```
Получишь `<p>{{ _(" Hello, world " ) }}</p>`

P.S. ЛЮБОЙ ИЗ МИКСИНОВ МОЖНО НАПИСАТЬ 3-4-МЯ РАЗНЫМИ СПОСОБАМИ, А ЕСЛИ ПОДУМАТЬ, ТО СПОСОБОВ ЕЩЕ БОЛЬШЕ. ДАВАЙ ДОГОВОРИМСЯ, БУДЕМ ПИСАТЬ КАК ТУТ, ДЛЯ ВСЕХ ОДИНАКОВО, НЕ БУДЕМ ИЗОБРЕТАТЬ ВЕЛОСИПЕД, А ПОТОМ ТВОЙ КОЛЛЕГА БУДЕТ ТРАТИТЬ ВРЕМЯ НА ДЕШИФРОВКУ ТВОИХ МИКСИНОВ :)
P.S.S не забывай про фильтры: 
  - 1.(jinja) https://docs.djangoproject.com/en/2.2/ref/templates/builtins/
  - 2.(twig) https://twig.symfony.com/doc/2.x/filters/index.html
  Они очень очень помогают, прям мега крутые помощники. С ними в темплейтах можно творить настоящую магию без js-са) Взять хотя бы самый распостраненный фильтр - `truncatechars` . Если 
  ```
  p: +exp('a_lot_of_text|truncatechars(50)')
  ```
 Получим ровно 50 символов текста и `...` в конце текста, согласись, это круто, особенно для каких-то карточек-превьюшек. 
 Кстати, лайфхак , как порезать текст с WYSIWYG(тектсу с визивига мы пишем фильтра `safe` и еще нужно ведь порезать? как не порезать сам HTML, чтобы теги закрылись и не обрезались?)
 ```
 p: +exp('description|truncatechars_html(200)|safe')
 ```
 Cначала режем html, потом safe)
 Об этом можно долго-долго доворить, это тема для отдельной доки) 
 
 
 
 
#### Возможно, что-то упустили... Не бойся пробовать! Пару раз напишешь - запомнишь) 

