## Вопросы на собеседовании по Angular  [![Angular-RU](https://img.shields.io/badge/Telegram_chat:-Angular_RU-216bc1.svg?style=flat)](https://t.me/angular_ru)

Вопросы подготовлены непосредственно для того, чтобы определить уровень разработчика, насколько глубоко, поверхностно или сносно он знает Angular. Вопросы для собеседования на знание JavaScript или Web-стека хорошо освещены в других местах, поэтому ниже будет добавлен список ресурсов по этой теме:

**Fundamentals**:

- [Coding Interview University](https://github.com/jwasham/coding-interview-university)
- [Awesome Interviews](https://github.com/alex/what-happens-when)
- [Angular Interview Questions](https://github.com/sudheerj/angular-interview-questions)

**Frontend**: 

- [Front-end Job Interview Questions](https://github.com/h5bp/Front-end-Developer-Interview-Questions)
- [The Best Frontend JavaScript Interview Questions](https://performancejs.com/post/hde6d32/The-Best-Frontend-JavaScript-Interview-Questions-(Written-by-a-Frontend-Engineer))
- [Frontend Guidelines Questionnaire](https://github.com/bradfrost/frontend-guidelines-questionnaire)
- [Подготовка к интервью на Front-end разработчика](https://proglib.io/p/frontend-interview/)

**Angular**:

- [Angular Interview Questions by Google Developer Expert](https://github.com/Yonet/Angular-Interview-Questions)

##### Основны TypeScript

<details>
<summary>Что такое декоратор?</summary>
  <div><br>
  Декоратор - способ добавления метаданных к объявлению класса. Это специальный вид объявления, который может быть присоединен к объявлению класса, методу, методу доступа, свойству или параметру. 
  <br>Декораторы используют форму @expression, где expression - функция, которая будет вызываться во время выполнения с информацией о декорированном объявлении.
  <br>Чтобы написать собственный декоратор, нам нужно сделать его factory и определить тип:
    <ul>
  <li>ClassDecorator</li>
  <li>PropertyDecorator</li>
  <li>MethodDecorator</li>
  <li>ParameterDecorator</li>
    </ul>
    
  <b>Декоратор класса</b>
  <div>
  Вызывается перед объявлением класса, применяется к конструктору класса и может использоваться для наблюдения, изменения или замены определения класса. Expression декоратора класса будет вызываться как функция во время выполнения, при этом конструктор декорированного класса является единственным аргументом. Если класс декоратора возвращает значение, он заменит объявление класса вернувшимся значением.

  ```typescript
    export function logClass(target: Function) {
        // Сохранение ссылки на оригинальный конструктор
        const original = target;
    
        // Функция генерирует экземпляры класса
        function construct(constructor, args) {
            const c: any = function () {
                return constructor.apply(this, args);
            }
            c.prototype = constructor.prototype;
            return new c();
        }
    
        // Определение поведения нового конструктора
        const f: any = function (...args) {
            console.log(`New: ${original['name']} is created`);
            //New: Employee создан
            return construct(original, args);
        }
    
        // Копирование прототипа, чтобы оператор intanceof работал
        f.prototype = original.prototype;
    
        // Возвращает новый конструктор, переписывающий оригинальный
        return f;
    }

    @logClass
    class Employee {
    
    }

    let emp = new Employee();
    console.log('emp instanceof Employee');
    //emp instanceof Employee
    console.log(emp instanceof Employee);
    //true
  ```
  </div>
  
  <br><b>Декоратор свойства</b>
  <div>
Объявляется непосредственно перед объявлением метода. Будет вызываться как функция во время выполнения со следующими двумя аргументами:
  <ul>
    <li>target - прототип текущего объекта, т.е. если Employee является объектом, Employee.prototype</li>
    <li>propertyKey - название свойства</li>
  </ul>

  ```typescript
    function logParameter(target: Object, propertyName: string) {
    
        // Значение свойства
        let _val = this[propertyName];
    
        // Геттер свойства
        const getter = () => {
            console.log(`Get: ${propertyName} => ${_val}`);
            return _val;
        };
    
        // Сеттер свойства
        const setter = newVal => {
            console.log(`Set: ${propertyName} => ${newVal}`);
            _val = newVal;
        };
    
        // Удаление свойства
        if (delete this[propertyName]) {
    
            // Создает новое свойство с геттером и сеттером
            Object.defineProperty(target, propertyName, {
                get: getter,
                set: setter,
                enumerable: true,
                configurable: true
            });
        }
    }
    
    class Employee {
    
        @logParameter
        name: string;
    }
    
    const emp = new Employee();
    emp.name = 'Mohan Ram';
    console.log(emp.name);
    // Set: name => Mohan Ram
    // Get: name => Mohan Ram
    // Mohan Ram
  ```
  </div>
  <br><b>Декоратор метода</b>
  <div>
Объявляется непосредственно перед объявлением метода. Будет вызываться как функция во время выполнения со следующими двумя аргументами:
  <ul>
    <li>target - прототип текущего объекта, т.е. если Employee является объектом, Employee.prototype</li>
    <li>propertyName - название свойства</li>
    <li>descriptor - дескриптор свойства метода т.е. - Object.getOwnPropertyDescriptor (Employee.prototype, propertyName)</li>
  </ul>
 
   ```typescript
    export function logMethod(
        target: Object,
        propertyName: string,
        propertyDesciptor: PropertyDescriptor): PropertyDescriptor {
        const method = propertyDesciptor.value;
    
        propertyDesciptor.value = function (...args: any[]) {
    
            // Конвертация списка аргументов greet в строку
            const params = args.map(a => JSON.stringify(a)).join();
    
            // Вызов greet() и получение вернувшегося значения
            const result = method.apply(this, args);
    
            // Конвертация результата в строку
            const r = JSON.stringify(result);
    
            // Отображение в консоли деталей вызова
            console.log(`Call: ${propertyName}(${params}) => ${r}`);
    
            // Возвращение результата вызова
            return result;
        }
        return propertyDesciptor;
    }
    
    class Employee {
    
        constructor(
            private firstName: string,
            private lastName: string
        ) {
        }
    
        @logMethod
        greet(message: string): string {
            return `${this.firstName} ${this.lastName} says: ${message}`;
        }
    
    }
    
    const emp = new Employee('Mohan Ram', 'Ratnakumar');
    emp.greet('hello');
    //Call: greet("hello") => "Mohan Ram Ratnakumar says: hello"
   ```
   </div>
   
   <br><b>Декоратор параметра</b>
  <div>
Объявляется непосредственно перед объявлением метода. Будет вызываться как функция во время выполнения со следующими двумя аргументами:
  <ul>
    <li>target - прототип текущего объекта, т.е. если Employee является объектом, Employee.prototype</li>
    <li>propertyKey - название свойства</li>
    <li>index - индекс параметра в массиве аргументов</li>
  </ul>
   
   ```typescript
    function logParameter(target: Object, propertyName: string, index: number) {
    
        // Генерация метаданных для соответствующего метода
        // для сохранения позиции декорированных параметров
        const metadataKey = `log_${propertyName}_parameters`;
        
        if (Array.isArray(target[metadataKey])) {
            target[metadataKey].push(index);
        }   
        else {
            target[metadataKey] = [index];
        }
    }
    
    class Employee {
        greet(@logParameter message: string): void {
            console.log(`hello ${message}`);
        }
    }
    const emp = new Employee();
    emp.greet('world');
 ```
 </div>
   
</details>




##### Основные концепции

<details>
<summary>Что такое Angular?</summary>
<div><br>
<img src="https://d2eip9sf3oo6c2.cloudfront.net/series/square_covers/000/000/033/thumb/egghead-angular-material-course-sq.png" align="left"><p><b>Angular</b>&nbsp;&mdash; это платформа для разработки мобильных и&nbsp;десктопных веб-приложений. Наши приложения теперь представляют из&nbsp;себя &laquo;толстый клиент&raquo;, где управление отображением и&nbsp;часть логики перенесены на&nbsp;сторону браузера. Так сервер уделяет больше времени доставке данных, плюс пропадает необходимость в&nbsp;постоянной перерисовке. С&nbsp;Angular мы&nbsp;описываем структуру приложения декларативно, а&nbsp;с&nbsp;TypeScript начинаем допускать меньше ошибок, благодаря статической типизации. В&nbsp;Angular присутствует огромное количество возможностей из&nbsp;коробки. Это может быть одновременно и&nbsp;хорошо и&nbsp;плохо, в&nbsp;зависимости от&nbsp;того, что вам необходимо.</p><hr>
  
<b>Какие плюсы можно выделить</b>:
<ul>
  <li>Поддержка Google, Microsoft</li>
  <li>Инструменты разработчика (CLI)</li>
  <li>Typescript из коробки</li>
  <li>Реактивное программирование с RxJS</li>
  <li>Единственный фреймворк с Dependency Injection из коробки</li>
  <li>Шаблоны, основанные на расширении HTML</li>
  <li>Кроссбраузерный Shadow DOM из коробки (либо его эмуляция) </li>
  <li>Кроссбраузерная поддержка HTTP, WebSockets, Service Workers</li>
  <li>Не нужно ничего дополнительно настраивать. Больше никаких оберток. jQuery плагины и D3 можно использовать на прямую</li>
  <li>Более современный фреймворк, чем AngularJS (на уровне React, Vue)</li>
  <li>Большое комьюнити</li>
</ul>

<b>Минусы</b>:

<ul>
  <li>Выше порог вхождения из-за Observable (RxJS) и Dependency Injeciton</li>
  <li>Чтобы все работало хорошо и быстро нужно тратить время на дополнительные оптимизации 
    (он не супер быстрый, по умолчанию, но быстрее AngularJS во много раз)</li>
  <li>Если вы планируете разрабатывать большое enterprise-приложение, то в этом случае, у вас нет архитектуры из коробки - нужно добавлять Mobx, Redux, MVVM, CQRS/CQS или другой state-менеджер, чтобы потом не сломать себе мозг</li>
  <li>Angular-Univesal имеет много подводных камней</li>
  <li>Динамическое создание компонентов оказывается нетривиальной задачей</li>
</ul>

</div>
</details>

<details>
<summary>В чем разница между AngularJS и Angular?</summary>
<div>
  
<br><b>AngularJS</b> является фреймворком, который может помочь вам в разработке Single Page Application. Он появился в 2009 году и с годами выяснилось, что имел много проблем. <b>Angular</b> (Angular 2+) же в свою очередь направлен на тоже самое, но дает больше преимуществ по сравнению с AngularJS 1.x, включая лучшую производительность, ленивую загрузку, более простой API, более легкую отладку.

<b>Что появилось в Angular</b>: <br>

<ul>
  <li>Angular ориентирован на мобильные платформы и имеет лучшую производительность</li>  
  <li>Angular имеет встроенные сервисы для поддержки интернационализации</li>
  <li>AngularJS проще настроить, чем Angular</li>
  <li>AngularJS использует контроллеры и $scope</li>
  <li>Angular имеет много способов определения локальных переменных</li>
  <li>В Angular новый синтаксис структурных директив (camelCase)</li>
  <li>Angular работает напрямую с свойства и собитиями DOM элементов</li>
  <li>Одностороннее связывание данных через [property]</li>
  <li>Двустороннее связывание данных через [(property)]</li>
  <li>Новый механизм DI, роутинга, запуска приложения</li>
</ul>

<b>Основные преимущества Angular</b>: <br>

<ul>
  <li>Обратная совместимость Angular 2, 4, 5, ..</li>
  <li>TypeScript с улучшенной проверкой типов</li>
  <li>Встроенный компилятор с режимами JIT и AOT (+ cокращение кода)</li>
  <li>Встроенные анимации</li>
</ul>

</div>
</details>

<details>
<summary>Какой должна быть структура каталогов компонентов любого Angular приложения и почему?</summary>
<div>
  in progress..
</div>
</details>


<details>
<summary>Что такое MVVM и в чем разница перед MVC?</summary>
<div>
  MVVM - шаблон проектирования архитектуры приложения. Состоит из 3 компонентов: Model, View, ViewModel.
  <br>Отличие от MVС заключаются в:
  <li>View реагирует на действия пользователя и передает их во View Model через Data Binding.</li>
  <li>View Model, в отличие от контроллера в MVC, имеет Binder, автоматизирущий связь между View и связанными свойствами в View Model.</li>
  <br>Сам Binding может быть односторонним или двухсторонним.
</div>
</details>


##### Angular Template синтаксис

<details>
<summary>Что такое интерполяция в Angular?</summary>
<div><br>
  
Разметка интерполяции с внедренными выражениями используется в Angular для присвоение данных текстовым нодам и значения аттрибутов. Например:
  
  ```html
    <a href="img/{{username}}.jpg">Hello {{username}}!</a>
  ```
  
<br>
</div>

</details>


<details>
<summary>Какие способы использования шаблонов в Angular вы знаете?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>В чем разница между структурной и атрибутной директивой, назовите встроенные директивы?</summary>
<div>
  <li>Структурные директивы влияют на DOM и могут добавлять/удалять элементы. Встроенные: ng-template, NgIf,  NgFor, NgSwitch,  etc.</li>
  <li>Атрибутные директивы меняют внешний вид или поведение элементов, компонентов или других директив. Встроенные: NgStyle, NgClass.</li>
</div>
</details>



<details>
<summary>Для чего нужны директивы ng-template, ng-container, ng-content?</summary>
<div>
  <h4>1. ng-template</h4>
  
  `<template>` — это механизм для отложенного рендера клиентского контента, который не отображается во время загрузки, но может быть инициализирован при помощи JavaScript. <br><br>
  Template можно представить себе как фрагмент контента, сохранённый для последующего использования в документе. Хотя парсер и обрабатывает содержимое элемента `template` во время загрузки страницы, он делает это только чтобы убедиться в валидности содержимого; само содержимое при этом не отображается. <br><br>
  
  `<ng-template>` - является имплементацией стандартного элемента template, данный элемент появился с четвертой версии Angular, это было сделано с точки зрения совместимости со встраиваемыми на страницу template элементами, которые могли попасть в шаблон ваших компонентов по тем или иным причинам. <br><br>

Пример:

```html
<div class="lessons-list" *ngIf="lessons else loading">
  ... 
</div>

<ng-template #loading>
    <div>Loading...</div>
</ng-template>
```

  <h4>2. ng-container</h4>
  
  `<ng-container>` - это логический контейнер, который может использоваться для группировки узлов, но не отображается в дереве DOM как узел (node).

  На самом деле структурные директивы (*ngIf, *ngFor, ..) являются синтаксическим сахаром для наших шаблонов. В реальности, данные шаблоны трансформируются в такие конструкции:
  
```html
<ng-template [ngIf]="lessons" [ngIfElse]="loading">
   <div class="lessons-list">
     ... 
   </div>
</div>

<ng-template #loading>
    <div>Loading...</div>
</ng-template>
```

Но что делать, если я хочу применить несколько структурных директив?
(спойлер: к сожалению, так нельзя сделать)

```html
<div class="lesson" *ngIf="lessons" *ngFor="let lesson of lessons">
  <div class="lesson-detail">
      {{lesson | json}}
  </div>
</div> 
```

```
Uncaught Error: Template parse errors:
Can't have multiple template bindings on one element. Use only one attribute 
named 'template' or prefixed with *
```

Но можно сделать так:

```html
<div *ngIf="lessons">
  <div class="lesson" *ngFor="let lesson of lessons">
    <div class="lesson-detail">
        {{lesson | json}}
    </div>
  </div> 
</div>
```

Однако, чтобы избежать необходимости создавать дополнительный div, мы можем вместо этого использовать директиву ng-container:

```html
<ng-container *ngIf="lessons">
    <div class="lesson" *ngFor="let lesson of lessons">
        <div class="lesson-detail">
            {{lesson | json}}
        </div>
    </div>
</ng-container>
```

Как мы видим, директива ng-container предоставляет нам элемент, в котором мы можем использовать структурную директиву, без необходимости создавать дополнительный элемент.

Еще пара примечательных примеров, если все же вы хотите использовать ng-template вместо ng-container, по определенным правилам вы не сможете использовать полную конструкцию структурных директив.

Вы можете писать либо так:

```html
<div class="mainwrap">
    <ng-container *ngIf="true">
        <h2>Title</h2>
        <div>Content</div>
    </ng-container>
</div>
```

Либо так:

```html
<div class="mainwrap">
    <ng-template [ngIf]="true">
        <h2>Title</h2>
        <div>Content</div>
    </ng-template>
</div>
```

На выходе, при рендеринге будет одно и тоже:

```html
<div class="mainwrap">
      <h2>Title</h2>
      <div>Content</div>
</div>
```

 <h4>3. ng-content</h4>
 
 `<ng-content>` - позволяет внедрять родительским компонентам html-код в дочерние компоненты.
 
Здесь на самом деле, немного сложнее уже чем с ng-template, ng-container. Так как ng-content решает задачу проецирования контента в ваши веб-компоненты. Веб-компоненты состоят из нескольких отдельных технологий. Вы можете думать о Веб-компонентах как о переиспользуемых виджетах пользовательского интерфейса, которые создаются с помощью открытых веб-технологий. Они являются частью браузера и поэтому не нуждаются во внешних библиотеках, таких как jQuery или Dojo. Существующий Веб-компонент может быть использован без написания кода, просто путем импорта выражения на HTML-страницу. Веб-компоненты используют новые или разрабатываемые стандартные возможности браузера.

Давайте представим ситуацию от обратного, нам нужно параметризировать наш компонент. Мы хотим сделать так, чтобы на вход в компонент мы могли передать какие-либо статичные данные. Это можно сделать несколькими способами. 

comment.component.ts:

```ts
@Component({
  selector: 'comment',
  template: `
    <h1>Комментарий: </h1>
    <p>{{data}}</p>
  `
})
export class CommentComponent {
  @Input() data: string = null;
}
```

app.component.html

```html
<div *ngFor="let message of comments">
  <comment [data]="message"></comment>
</div>
```

Но можно поступить и другим путем. <br>
comment.component.ts:

```ts
@Component({
  selector: 'comment',
  template: `
    <h1>Комментарий: </h1>
    <ng-content></ng-content>
  `
})
export class CommentComponent { 
}
```

app.component.html

```html
<div *ngFor="let message of comments">
  <comment>
    <p>{{message}}</p>
  </comment>
</div>
```

Конечно, эти примеры плохо демонстрируют подводные камни, свои плюсы и минусы. Но второй способ демонстрирует подход при работе, когда мы оперируем независимыми абстракциями и можем проецировать контент внутрь наших компонентов (подход веб-компонентов).

</div>
</details>

##### Angular development enviroments

<details>
<summary>Что такое директива и как создать собственную?</summary>
<div>
  Директивы бывают трех видов: компонент, структуные и атрибутные (см. выше). 

  <h4>Создание атрибутных директив:</h4>
  
    @Directive({
    selector: '[appHighlight]'
    })
    export class HighlightDirective {
     constructor() { }
     }

  <br>Декоратор определяет селектор атрибута [appHighlight], [] указывают что это селектор атрибута. Angular найдет каждый элемент в шаблоне с этим атрибутом и применит к ним логику директивы.

    export class HighlightDirective {
      constructor(el: ElementRef) {
         el.nativeElement.style.backgroundColor = 'yellow';
      }
    }
  
  <br>Необходимо указать в конструткторе ElementRef, чтобы через его свойство nativeElement иметь прямой доступ к DOM элементу, который должен быть изменен.
  <br>Теперь, используя @HostListener, можно добавить обработчики событий, взаимодействующие с декоратором.

    @HostListener('mouseenter') onMouseEnter() {
    this.highlight('yellow');
     }

    @HostListener('mouseleave') onMouseLeave() {
    this.highlight(null);
    }
    
    private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
    }


  <h4>Структурные директивы создаются так:</h4>

  Напишем UnlessDirective, которая будет противоположна NgIf. 
  <br>Необходимо использовать @Directive, и импортировать Input, TemplateRef, и ViewContainerRef. Они вам понадобятся при воздании любой структурной директивы. 

      import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';
    
      @Directive({ selector: '[appUnless]'})
      export class UnlessDirective {
      }

В конструкторе мы получаем доступ к viewcontainer и содержимое <ng-template>.

      constructor(
        private templateRef: TemplateRef<any>,
        private viewContainer: ViewContainerRef) { }

Наша директива предполагает работу с true/false. Для этого нужно свойство appUnless, добавленное через @Input. 

      @Input() set appUnless(condition: boolean) {
        if (!condition && !this.hasView) {
          this.viewContainer.createEmbeddedView(this.templateRef);
          this.hasView = true;
        } else if (condition && this.hasView) {
          this.viewContainer.clear();
          this.hasView = false;
        }
      }
</div>
</details>


<details>
<summary>Что такое директива, компонент, модуль, сервис, пайп в Angular и для чего они нужны?</summary>
<div>
  <li>Директива - см. выше.</li>
  <li>Компонент контролирует участок экрана, т.н. view.</li>
  <li>Сервис это класс с узкой, четко определенной целью. Это может быть значение, функция, запрос, etc. Главное в них то, что они повторно используются, отделяя чистую функциональность компонента. </li>
  <li>Пайп преобразует отображение значений в шаблоне, к примеру отображение дат в разных локалях или изменяют в отображении регистр строк.</li>
</div>
</details>


<details>
<summary>Расскажите об основных параметрах @NgModule, @Component, @Directive, @Injectable, @Pipe</summary>
<div>
 Декораторы динамически подключают дополнительное поведение к объекту. Они помечают класс и предоставляют конфигурационные метаданные.
 <h4>@NgModule может содержать следующие параметры:</h4>
 <li>providers - список инжектируемых объектов, которые добавляются в этот модуль</li>
 <li>declarations - компоненты, директивы и пайпы, принадлежащие этому модулю</li>
 <li>imports - модули, которые экспортируются декларируемыми и доступны в шаблоне этого модуля</li>
 <li>exports - компоненты, директивы и пайпы, которые объявлены декларируемыми, и могут быть импользованы в шаблоне любого компонента, которые принадлежит NgModule импортирующему их.</li>
 <li>entryComponent - компилируемые компоненты при определенни NgModule, для динамической загрузки в view.</li>
 <li>bootstrap - компоненты, которые загружаются при загрузке этого модуля, автоматически добавляются в entryComponent. </li>
 <li>schemas - набор схем, объявляющих разрешенные элементы в MgModules</li>
 <li>id - имя или путь, уникальный идентификатор этого NgModule в getModuleFactory. Если не заоплнять - не будет там зарегистрирован.</li>
 <li>jit - если true, то этот модуль будет пропущен компилятором AOT и всегда будет компилироваться JIT.</li>
 
 <h4>@Component может содержать следующие параметры:</h4>
 <li>changeDetection - стратегия обнаружения изменений, используемая для этого компонента</li>
 <li>viewProviders - инжектируемые объекты, которые видны DOM children этого компонента. </li>
 <li>moduleId - id модуля, к которому относится компонент.</li>
 <li>templateUrl - относительный путь или абсолютный URL к шаблону компонента.</li>
 <li>template - инлайн шаблон для этого компонента.</li>
 <li>styleUrls - один и более путь до файла, содержащего CSS, абсолютный или относительный.</li>
 <li>styles - инлайн CSS, используемые в этом компоненте.</li>
 <li>animations - один и более вызовов анимации trigger(), содержащих state() и transistion(). </li>
 <li>encapsulation - правила инкапсуляции для шаблона и CSS.</li>
 <li>interpolation - переопределение базовых знаков интерполяции.</li>
 <li>entryComponents - компоненты, которые должны быть скомпилированы вместе с этим компонентом. Для каждого упомянутого здесь компонента создается ComponentFactory и сохраняется в ComponentFactoryResolver.</li>
 <li>preserveWhitespaces - при значении true удаляются потенциально лишние пробелы из скомпилированного шаблона. </li>
 
 <h4>@Directive может содержать следующие параметры:</h4>
  <li>selector - CSS-селектор, который идентифицирует эту директиву в шаблоне и запускает создание этой директивы.</li>
  <li>inputs - ?????.</li>
  <li>outputs -?????? </li>
  <li>providers - настраивает инжектор этой директивы или компонента с помощью токена.</li>
  <li>exportAs - определяет имя, которое можно использовать в шаблоне для присвоения этой директивы переменной.</li>
  <li>queries - настраивает запросы, которые могут быть инжектированы в директиву.</li>
  <li>host - состоставляет свойства класса со сбайнженными элементами для свойств, атрибутов и ивентов. </li>
  <li>jit - если true, то этот модуль будет пропущен компилятором AOT и всегда будет компилироваться JIT.</li>
  
  <h4>@Injectable может содержать следующие параметры:</h4>
  <li>providedIn - определяет, где будет заинжектировано, либо, если объявлено "root" растространится на все приложение. </li>
  
  <h4>@Pipe может содержать следующие параметры:</h4>
  <li>name - имя пайпа, которое будет использовано в шаблоне.</li>
  <li>pure - если true, то пайп считается "чистым", и метод transform() вызовется только при изменении его входных агрументов. По умолчанию стоит true. </li>
 
</div>
</details>


<details>
<summary>Что такое динамические компоненты и как их можно использовать в Angular?</summary>
<div>
 in progress
</div>
</details>


<details>
<summary>Как применить анимацию к компонентам?</summary>
<div>
  in progress..
</div>
</details>


##### Angular render lifecycle and core environments

<details>
<summary>Объясните механизм загрузки (bootstrap) Angular-приложения в браузере?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Как происходит взаимодействие компонентов в Angular (опишите components view)?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Каков жизненный цикл у компонентов?</summary>
<div>
  <b>После</b> создания компонента или директивы через вызов конструктора, Angular вызывает методы жизненного цикла в следующей последовательности в строго определенные моменты:
  <li>ngOnChanges() - вызывается когда Angular при/переприсваивает привязанные данные к input properties. Метод получает объект SimpleChanges, со старыми и новыми значениями. Вызывается перед NgOnInit и каждый раз, когда изменяется одно или несколько связанных свойств.</li>
  <li>ngOnInit() - инициализует директиву/компонент после того, как Angular впервые отобразит связанные свойства и устанавливает входящие параметры.</li>
  <li>ngDoCheck() - при обнаружении изменений, которые Angular не может самостоятельно обнаружить, реаигрует на них. </li>
  <li>ngAfterContentInit() - вызывается после того, как Angular спроецирует внешний контент в отображение компонента или отображение с директивой. Вызывается единожды, после первого ngDoCheck().</li>
  <li>ngAfterContentChecked() - реагирует на проверку Angular-ом проецируемого содержимого. Вызывается после ngAfterContentInit() и каждый последующий ngDoCheck().</li>
  <li>ngAfterViewInit() - вызывается после инициализации отображения компонента и дочерних/директив. Вызывается единожды, после первого ngAfterContentChecked().</li>
  <li>ngAfterViewChecked() - реагирует на проверку отображения компонента и дочерних/директив. Вызывается после ngAfterViewInit() и каждый последующий ngAfterContentChecked().</li>
  <li>ngOnDestroy() - после уничтожния директивы/компонента выполняется очистка. Отписывает Observables и отключает обработчики событий, чтобы избежать утечек памяти.</li>
  
</div>
</details>


<details>
<summary>Что такое Shadow DOM и как с ним работать в Angular?</summary>
<div>
  in progress..
</div>
</details>


<details>
<summary>Что такое Data Binding и какие проблемы связанные с ним вы знаете?</summary>
<div>
  Angular поддерживает одностороннюю и двустороннюю Data Binding. Это механизм координации частей шаблона с частями компонента. 
  <br>Добавление специальной разметки сообщает Angular как соединять обе стороны. Следующая диаграмма показывает четыре формы привязки данных.
  <br>Односторонние:
  <li>От компонента к DOM с привязкой значения: {{hero.name}}</li>
  <li>От компонента к DOM с привязкой свойства и присвоением значения: [hero]="selectedHero"</li>
  <li>От DOM к компоненту с привязкой на ивент: (click)="selectHero(hero)"</li>
  
  <br>Двусторонняя в основном используется в template-driven forms, сочетает в себе параметр и ивент. Вот пример, использующий привязку с директивой ngModel.
  <li><input [(ngModel)]="hero.name"></li>
  <br>Здесь значение попадает в input из компонента, как при привязке значения, но при изменении юзером значения новое передается в компонент и переопределяется. 
 
</div>
</details>

<details>
<summary>Как вы используете одностороннюю и двухстороннюю привязку данных?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>В чем преимущества и недостатки Regular DOM (Angular) перед Virtual DOM (React)?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Что такое ngZone?</summary>
<div>
  Сервис внедрения зависимостей, который может работать вне Angular. Распространено использование этого сервиса для оптимизации производительности при запуске работы, состоящец из одной или нескольких асинхронных задач, которые не требуют обновления Angular-ом пользовательского интерфейса или обработки ошибок.
</div>
</details>

<details>
<summary>Как обновлять представление, если ваша модель данных обновляется вне 'зоны'?</summary>
<br>

1. Используя метод `ApplicationRef.prototype.tick`, который запустит `change detection` на всем дереве компонентов.

```typescript
import { Component, ApplicationRef, NgZone } from '@angular/core';

@Component({
    selector: 'app-root',
    template: `
        <h1>Hello, {{ name }}!</h1>
    `
})
export class AppComponent {

    public name: string = null;

    constructor(private app: ApplicationRef, private zone: NgZone) {
        this.zone.runOutsideAngular(() => {
            setTimeout(() => {
                this.name = window.prompt('What is your name?', 'Jake');
                this.app.tick();
            }, 5000);
        });
    }
    
}
```

2. Используя метод `NgZone.prototype.run`, который также запустит `change detection` на всем дереве.

```typescript
import { Component, NgZone } from '@angular/core';
import { SomeService } from './some.service'

@Component({
    selector: 'app-root',
    template: `
        <h1>Hello, {{ name }}!</h1>
    `,
    providers: [SomeService]
})
export class AppComponent {

   public name: string = null;

   constructor(private zone: NgZone, private service: SomeService) {
       this.zone.runOutsideAngular(() => {
           this.service.getName().then((name: string) => {
               this.zone.run(() => this.name = name);
           });
       });
   }
   
}
```

Метод `run` под капотом сам вызывает `tick`, а параметром принимает функцию, которую нужно выполнить перед `tick`. То есть:

```typescript
this.zone.run(() => this.name = name);

// идентично

this.name = name;
this.app.tick();
```

3. Используя метод `ChangeDetectorRef.prototype.detectChanges`, который запустит `change detection` на текущем компоненте и дочерних.

```typescript
import { Component, NgZone, ChangeDetectorRef } from '@angular/core';

@Component({
    selector: 'app-root',
    template: `
        <h1>Hello, {{ name }}!</h1>
    `
})
export class AppComponent {

   public name: string = null;

   constructor(private zone: NgZone, private ref: ChangeDetectorRef) {
       this.zone.runOutsideAngular(() => {
           this.name = window.prompt('What is your name?', 'Jake');
           this.ref.detectChanges();
       });
   }
   
}
```
</details>


<details>
<summary>Что такое EventEmmiter и как подписываться на события?</summary>
<div>
    Используется в директивах и компонентах для подписки на пользовательские ивенты синхронно или асинхронно, и регистрации обработчиков для этих ивентов.
</div>
</details>

<details>
<summary>Что такое Change Detection, как работает Change Detection Mechanism?</summary>

<h4>1. Change Detection</h4>
  
Change Detection - процесс синхронизации модели с представлением. В Angular поток информации однонаправленный, даже при использовании `ngModel` для реализации двустороннего связывания, которая является синтаксическим сахаром поверх однонаправленного потока.

<h4>2. Change Detection Mechanism </h4>

Change Detection Mechanism - продвигается только вперед и никогда не оглядывается назад, начиная с корневого (рут) компонента до последнего. В этом и есть смысл одностороннего потока данных. Архитектура Angular приложения очень проста — дерево компонентов. Каждый компонент указывает на дочерний, но дочерний не указывает на родительский. Односторонний поток устраняет необходимость `$digest` цикла. 

<br>
</details>

<details>
<summary>Какие существуют стратегии обнаружения изменений?</summary>
<br>

Всего есть две стратегии - `Default` и `OnPush`. Если все компоненты используют первую стратегию, то `Zone` проверяет все дерево независимо от того, где произошло изменение. Чтобы сообщить Angular, что мы будем соблюдать условия повышения производительности нужно использовать стратегию обнаружения изменений `OnPush`. Это сообщит Angular, что наш компонент зависит только от входных данных и любой объект, который передается ему должен считаться immutable. Это все построено на принципе автомата Мили, где текущее состояние зависит только от входных значений. 

<br>

</details>

<details>
<summary>Сколько Change Detector'ов может быть во всем приложении?</summary>
У каждого компонента есть свой Change Detector, все Change Detector'ы наследуются от AbstractChangeDetector.
  
<br>
</details>

<details>
<summary>Основное отличие constructor от ngOnInit?</summary>
<br>
  
Конструктор сам по себе является фичей самого класса, а не Angular. Основная разница в том, что Angular запустит `ngOnInit`, после того, как закончит настройку компонента, то есть, это сигнал, благодаря которому свойства `@Input()` и другие байндинги, и декорируемые свойства доступны в `ngOnInit`, но не определены внутри конструктора, по дизайну.

<br>
</details>

##### Angular data flow

<details>
<summary>Что такое Dependency Injection?</summary>
<div>
  Это важный паттерн шаблон проектирования приложений. В Angular внедрение зависимостей реализовано из-под капота.
  <br> Зависимости - это сервисы или объекты, которые нужны классу для выполнения своих функций. DI -позволяет запрашивать зависимости от внешних источников.
</div>
</details>

<details>
<summary>Что такое Singleton Service и с какой целью его используют в Angular?</summary>
<div>
  Это сервисы, объявленные в приложении и имеющие один экземляр на все приложение. 
  Его можно объявить двумя способами:
  <li>Объявить его @Injectable(root)</li>
  <li>Включить его в AppModule в providers, либо в единственный модуль импортируемый в AppModule.</li>
</div>
</details>

<details>
<summary>Как можно определить свой обработчик ErrorHandler, Logging, Cache в Angular?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Что такое Observable?</summary>
<div>
  Observable — это набюдатель, который подписывается "зрелище" и реагирует на все события до момента отписки. 
</div>
</details>

<details>
<summary>В чём разница между Observable и Promise?</summary>
<div>
  Promise обрабатывает одно значение по завершению асинхронной операции, вне зависимости от ее исхода, и не поддерживают отмену операции.
 <br>Observable же является потоком, и позволяет передавать как ноль, так и несколько событий, когда callback вызывается для каждого события.
</div>
</details>

<details>
<summary>В чём разница между Observable и BehaviorSubject/Subject?</summary>
<div>
  Subjects - специальные Observable. Представьте, что есть спикер с микрофоном, который выступает в комнате, полной людей. Это и есть Subjects, их сообщение передается сразу нескольким получателям. Обычные же Observables можно сравнить с разговором 1 на 1.
  <br><li>Subject - является multicast, то есть может передавать значение сразу нескольким подписчикам.</li>
  <br><li>BehaviorSubject - требует начальное значение и передает текущее значение новым подпискам.</li>
</div>
</details>

<details>
<summary>В чём разница между switchMap, mergeMap, concatMap?</summary>
<div>
  <li>switchMap - отменит подписку на Observable, возвращенный ее аргументом project, как только он снова вызовет ее в новом элементе.</li>
  <li>mergeMap - в отличие от switchMap позволяет реализовать одновременно несколько внутренних подписок. </li>
  <li>concatMap - послеждовательно обрабатывает каждое событие, в отличие от mergeMap.</li>
</div>
</details>



##### Angular with Backend integrations

<details>
<summary>Какими способами можно взаимодействовать API бекенда, что требуется для проксирования запросов?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Что такое HTTP Interceptors?</summary>
<br>

Interceptor (перехватчик) - просто причудливое слово для функции, которая получает запросы / ответы до того, как они будут обработаны / отправлены на сервер. Нужно использовать перехватчики, если имеет смысл предварительно обрабатывать многие типы запросов одним способом. Например нужно для всех запросов устанавливать хедер авторизации `Bearer`:

 token.interceptor.ts
 
```typescript
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs/Observable';

@Injectable()
export class TokenInterceptor implements HttpInterceptor {

    public intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        const token = localStorage.getItem('token') as string;

        if (token) {
            req = req.clone({
                setHeaders: {
                    'Authorization': `Bearer ${token}`
                }
            });
        }

        return next.handle(req);
    }
}
```

И регистрируем перехватчик как синглтон в провайдерах модуля:

app.module.ts

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { AppComponent } from './app.component';
import { TokenInterceptor } from './token.interceptor';

@NgModule({
    imports: [
        BrowserModule
    ],
    declarations: [
        AppComponent
    ],
    bootstrap: [AppComponent],
    providers: [{
        provide: HTTP_INTERCEPTORS,
        useClass: TokenInterceptor,
        multi: true // <--- может быть зарегистрирован массив перехватчиков
    }]
})
export class AppModule {}
```
<br>

</details>


<details>
<summary>Как использовать Json Web Tokens для аутентификации при разработке на Angular?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Как обрабатываются атаки XSS и CSRF в Angular?</summary>
<div>
  in progress..
</div>
</details>

##### Angular routing

<details>
<summary>Что такое роутинг и как его создать в Angular?</summary>
<div>
  Роутинг позволяет реализовать навигацию от одного view приложения к другому при работе пользователя с приложением.
  <br>Это реализовано через взаимодействие с адресной строкой, Angular Router интерпретирует ее как инструкцию по переходу между view. Возможна передача параметров вспомогательному компоненту для конкретизирования предоставляемого контента. Навигация может осуществлять по ссылкам на странице, кнопкам или другим элементам, как кнопки "вперед-назад" в браузере.
  <br>Для создания роутинга первым делом необходимо импортировать "RouterModule" и "Routes" в AppModule.
  <br>Затем необходимо реализовать конфигурацию по приложению, определить path и относящие к ним компоненты, и в метод RouterModule.forRoot() передать конфигурацию.
  <br>Наконец необходимо добавить routerLink в шаблон.
</div>
</details>


<details>
<summary>Каков жизненный цикл у Angular Router?</summary>
<div>
  in progress..
</div>
</details>


<details>
<summary>Что такое ленивая загрузка (Lazy-loading) и для чего она используется?</summary>
<div>
  in progress..
</div>
</details>


<details>
<summary>В чем разница между Routing и Navigation?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Как загрузить данные до того как активируется роут?</summary>
<div>
  in progress..
</div>
</details>


##### Angular Forms (also big ui enterprise)

<details>
<summary>Что такое FormGroup и FormControl и для чего они используются?</summary>
<div>
  <li>FormControl - отслеживает значение и статус валидации отдельного элемента формы.</li>
  <li>FormGroup - отслеживает состояние и статус валидациии группы FormControl </li>
  <br>Они используются для создания и работы с формами.
</div>
</details>

<details>
<summary>Что такое реактивные формы в Angular?</summary>
<div>
  Реактивные формы обеспечивают управляемый моделями подход к обработке входных данных форм, значения которых могут меняться со временем. Они строятся вокруг наблюдаемых потоков, где входные данные и значения форм предоставляются в виде потоков входных значений, к которым можно получить синхронный доступ. 
</div>
</details>


<details>
<summary>Как применять валидацию для простых и сложных форм?</summary>
<div>
  В реактивных формах валидация реализуется в компоненте. Есть два типа валидаторов: синхронные и асинхронные.
  <br>Можно использовать встроенные валидаторы, либо создать свои. Валидаторы доабвляются 
</div>
</details>


##### Build environments

<details>
<summary>В чем разница между Angular CLI и Webpack Development Environment?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Что такое JIT и AOT, в чем их отличия и каковы сферы применения?</summary>
<div>
  in progress..
</div>
</details>

##### Test development

<details>
<summary>Что такое Unit-тестирование, интеграционное, e2e-тестирование (End-to-End) и как оно применяется в Angular?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Что такое Karma, Jest, Jasmine (зачем их используют совместно при разработке на Angular)?</summary>
<div>
  in progress..
</div>
</details>

<details>
<summary>Как протестировать входные параметры и всплывающие события компонентов?</summary>
<div>
  in progress..
</div>
</details>
