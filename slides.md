---
revealOptions:
  transition: slide
  controls: false
---



# CLICK THE BUTTON

## The hard way

Дмитрий Лазарев

Note: Меня зовут Дмитрий Лазарев. Сегодня я вам расскажу про то как иногда бывает трудно просто кликнуть по кнопке. Но перед этим пара слов про проект.

---

![](./logo-forms.png)

Note: Я работаю в команде Формы контру.экстерна. Как можно догадаться по названию мы делаем формы. Но как и все нормальные разработчики, нам лень заниматься такой рутинной работой, поэтому мы занимается кодогенерацией. Пишем код, который делает формы за нас. В основном это формы отчетности в контролирующие органы: ФНС, РосСтат, ПФР и тд.

---

![](./knpro.png)

Note: Вот так например выглядит часть формы отчетности по НДС.

---

![](./one-does-not-simply.jpg)

Note: И так, как я уже говорил, нельзя просто так взять и кликнуть по кнопке, особенно в тестах и тем более если хочется делать честные клики. Вы спросите почему? И чтобы ответить на этот вопрос мы обратимся к вот этому чуваку

---

![](./mike-cohn.jpg)

Note: Кто-нибудь знает кто это такой? Пауза. Его зовут Майк Кон, автор книги Гибкая методология разработки Scrum. В своей книге он описывает концепт тестовой пирамиды.

---

## The Test Pyramid

![](./test-pyramid.svg)

Note: Тестовая пирамида прекрасно иллюстрирует категории тестов, их особенности и соотношение этих тестов в рамках проекта. При движении вверх по пирамиде можно увидеть, что растет не только размер теста и покрываемое им кол-во кода, но также растет стоимость поддержки и время выполнения. Поэтому правильно выбрать подходящий для нашей задачи уровень.

---

## Требования к тестам

- Честные клики <!-- .element class="fragment" data-fragment-index="1" -->
- Тестирование взаимодействия <!-- .element class="fragment" data-fragment-index="2" -->
- Достаточно быстрые <!-- .element class="fragment" data-fragment-index="3" -->
- Относительно стабильные <!-- .element class="fragment" data-fragment-index="4" -->

Note: Чего же мы хотим от тестов? Ну, в первую очередь нам нужны честные клики, под честными кликами я подразумеваю любое пользовательское действие. Далее мы хотим тестировать взаимодействие компонента с приложением. Кроме этого было бы замечательно обеспечить оптимальную скорость выполнения тестов. Ну и конечно не стоит забывать о том, чтобы тест не падал от фазы луны или дня недели.

Но вернемся к нашей изначальной проблеме. Нам необходимо кликать по кнопке и тестировать некоторое взаимодействие, при этом хотелось не потерять сильно в скорости выполнения тестов, а также обеспечить приемлемый уровень стабильности. Очевидно, что юнит тесты нам не подходят, они не позволяют тестировать взаимодействие. e2e тесты дают нам честные клики и в целом мы можем протестировать взаимодействие, но такие тесты не подходят из-за своей нестабильности и скорости выполнения. Из-за чего нам придется остановить свой выбор на интеграционных тестах, котоыре в большей мере отвечают нашим требованиям. Теперь нам необходимо выбрать инструмент для тестирования, так как в формах мы используем React, то первое что может придти на ум, использовать популярную библиотеку для тестирования React компонентов Enzyme.

---

## Enzyme

![](./logo-react.svg) <!-- .element style="width: 55%; float: left;" -->

![](./logo-airbnb.svg)<!-- .element style="width: 40%; float: left;" -->

http://airbnb.io/enzyme/

Note: Enzyme — это библиотечка от разработчиков компании Airbnb. Enzyme предоставляет удобное API для тестирования React компонентов, позволяет легко протестировать как ведет себя компонент при изменении props/state, вызовы life-cycle методов, разметку.

---

## Enzyme

```typescript
const wrapper = mount(<AwesomeButton />)
wrapper.find('button').simulate('click')
```

```typescript
// Тоже самое что и
wrapper.find('button').prop('onClick')()
```

<!-- .element class="fragment" data-fragment-index="1" -->

```typescript
// Работать не будет
wrapper.simulate('click')
```

<!-- .element class="fragment" data-fragment-index="2" -->

- <!-- .element class="fragment" data-fragment-index="3" --> Нечестные клики, вызов `props.onClick`
- <!-- .element class="fragment" data-fragment-index="3" --> Завязка на внутреннюю разметку

Note: К примеру у нас есть крутая кнопка, со сложной разметкой внутри, некоторой логикой и где-то там внутри есть взаимодействие с APIшкой. И нам хочется протестировать поведение при клике. Чтобы это сделать, нам нужно найти элемент button и эмулировать событие click на данном элементе. Кажется, что всё хорошо, за исключением того, что в тестах появляется знание о внутреней разметке компонента. Но, на самом деле вызов simulate('click') ничто иное, как просто вызов функции onClick из props нашей кнопки, именно так работает подкапотом enzyme по словам авторов, в следующей версии они планируют удалить метод simulate, за его неявное поведение. В результате мы имеем, нечестные клики и завязку на внутрености компонента. К сожаление, оба этих пункта не дают нам использовать enzyme в наших тестах. Попробуем на этот раз взять TestCafe.

---

![](./logo-testcafe.png) <!-- .element style="background: #fff; padding: 20px" -->

http://devexpress.github.io/testcafe/

Note: TestCafe — это инструмент на nodejs, позволяющий автоматизировать e2e тестирование. Основными фишками testcafe, можно назвать: быстрый старт и отсутствие необходимости в WebDriver'е, большую стабильность тестов засчет умных тайм-аутов и поддержку TypeScript'а. Благодаря тому что  testcafe позиционирует себя как инструмент для e2e тестирования, мы можем ожидать, выполнения как минимум одного из пунктов, а именно честных кликов. Попробуем написать простой тест.

---

## TestCafe

```typescript
fixture `AwesomeButton`
	.page `http://localhost:3000/awesome-button`

test('Click the button', async t => {
    await t.click('[data-prop-tid=button]')
    // ...
})
```

Note: Выглядит достаточно лаконично. В начале объявляем название для группы тестов и говорим, что страничка, которую мы хотим тестировать находится по такому урлу. Здесь я специально не буду приводить пример кода серверной части, просто имейте в виду, что поэтому урлу отдается простой html и рисуется наша крутая кнопка. Дальше по коду описываются сами тесты. Можно заметить, что первым аргументом в функцию теста приходит некое t, здесь это тестовый контроллер, объект, предоставляющий удобное API для взаимодействия со страницей, обработку эвентов и выполнения проверок. Попробуем теперь усложнить пример и реализовать тест, проверяющий взаимодействия нашей кнопки с приложением. 

---

## TestCafe

```typescript
import { ClientFunction } from 'testcafe'

const getClickValue = ClientFunction(
    () => window.getInternalData())

fixture `AwesomeButton`
	.page `http://localhost:3000/awesome-button`

test('Click the button', async t => {    
    await t.click('[data-prop-tid=button]')
    
    const expectedValue = { /* ... */ }
    const receivedValue = await getClickValue()
    await t.expect(receivedValue).eql(expectedValue)
})
```

- <!-- .element class="fragment" data-fragment-index="1" --> Сложно тестировать взаимодействие

Note: Для этого воспользуемся такой штукой, как ClientFunctions. ClientFunctions позволяет объявить функцию и выполнить её на стороне клиента, а также вернуть любое сериализуемое значение обратно на сервер. Здесь у нас на клиенте вызывается некая глобальная функция getInternalData. Её необходимо объявить в скрипте, который отдается вместе с тестируемой страницей. Проще говоря, в тестах нам нужно выставлять кишки приложения наружу, чтобы протестировать взаимодействие. В реальности, при увеличении тестов, кол-во хелперов растет линейно, а это приводит к увеличению поддерживаемого кода. Кроме того, для каждого тестируемого компонента необходимо фомировать свою отдельную страницу. Всё это приводит к сложности при тестировании взаимодействия. И мы пока не готовы платить такую цену за возможность делать честные клики. Но прежде чем перейти к следующему подходу, я расскажу ещё немного о том как устроен testcafe.

---

![](./testcafe-diagram.png)

Note: Как я уже говорил, для использования TestCafe вам не нужен WebDriver, чтобы взаимодействовать с браузером. Вместо него TestCafe использует URL-Rewriting Proxy. Прокси инжектит свой скрипт, который позволяет эмулировать пользовательские действия и выполнять код на стороне клиента. Иначе говоря для эмуляции используется обычный DOM Events API, в чем можно легко убедится подсмотрев исходники. Это наталкивает на мысль о том, что мы сами можем попробовать использовать DOM Events чтобы кликнуть по кнопке.

---

## DOM Events

```typescript
const button = document
    .querySelector('[data-prop-tid=button]')
button.querySelector('button').click()

// Или
const event = new MouseEvent('click')
button.querySelector('button').dispatchEvent(event)
```

```typescript
const {x, y, width, height} = button.getBoundingClientRect()
const realButton = document
	.elementFromPoint(x + width / 2, y + height / 2)
```

<!-- .element class="fragment" data-fragment-index="2" -->

- <!-- .element class="fragment" data-fragment-index="1" --> Завязка на внутреннюю разметку

Note: В самом простом случае клик выглядит вот таким образом. Тут можно заметить, что мы как и в случае с Enzyme завязаны на внутреннюю разметку компонента, но в то же время можем делать более-менее честные клики. В TestCafe эта проблема каким-то образом решена, я не стал копать дальше и решил попробовать несколько другой подход. Но когда готовил презентацию я выяснил, что на самом деле можно получить реальную кнопку просто взяв элемент по определенным координатам с помощью elementFromPoint. И всё же я хотел бы рассказать о том решении на котором я в конечном итоге остановился. Другой подход начался с использования Chrome DevTools Protocol.

---

## Chrome DevTools Protocol

![](./logo-chrome.png)

https://chromedevtools.github.io/devtools-protocol/

Note: CDP — это низкоуровневое API для управления браузеров Chrome и ему подобных, построеное поверх вебсокетов. Позволяет делать множество интересных вещей с браузером, инспектировать и взаимодействовать с DOM, отлаживать скрипты, заниматься профилированием, работать с сетью, а так же эмулировать пользовательские действия. Сами devtools браузера используют этот протокол чтобы взаимодействовать со страницей. Попробуем воспользоваться.

---

## Chrome DevTools Protocol

```bash
chrome --remote-debugging-port=9222
open http://localhost:9222
```

<!-- .element class="fragment" data-fragment-index="1" -->

![](./chrome-devtools.png) <!-- .element class="fragment" data-fragment-index="2" width="80%" -->

Note: Чтобы иметь возможность удаленно подключится к браузеру, необходимо запустить хром с флагом remote-debugging-port. После этого, если перейти по ссылке можно увидеть веб интерфейс работы с ChromeDevTools. В нашем случае, мы хотим подключится программно, для этого необходимо получить websocket url страницы с которой хотим работать. Для этого в конец ссылки надо добавить json и в ответе мы получим массив страниц открытых на данный момент в браузере.

---

![](./chrome-devtools-json.png)

---

```typescript
const ws = new WebSocket(
    `ws://localhost:9222/devtools/page/${PageId}`)
const send = data =>
	ws.send(JSON.stringify({ id: uuid(), ...data }))
```

<!-- .element class="fragment" data-fragment-index="1" -->

```typescript
send({method: 'DOM.getDocument', params: {}})
send({
    method: 'DOM.querySelector',
    params: {nodeId: 1, selector: '[data-prop-tid=button]'}})
```

<!-- .element class="fragment" data-fragment-index="2" -->

```typescript
send({method: 'DOM.getBoxModel', params: { nodeId: 23 }})
send({
    method: 'Input.dispatchMouseEvent',
    params: { type: 'mousePressed', x: 40, y: 357,
               button: 'left', clickCount: 1 }})
send({
    method: 'Input.dispatchMouseEvent',
    params: { type: 'mouseReleased', x: 40, y: 357,
             button: 'left', clickCount: 1 }})
```

<!-- .element class="fragment" data-fragment-index="3" -->

Note: У нас выходит что-то такое. В первой строке мы подключаемся по вебсокетам для того чтобы взаимодействовать с определенной страницей (под страницей может выступать вкладка, devtools браузера, экстеншоны). Далее мы получаем nodeId документа, тут нам повезло и документ имеет nodeId: 1. После этого применяем селектор, который нам возвращает nodeId найденного элемента. После этого запрашиваем BoxModel для этого элемента и получаем размеры и координаты кнопки. Две последние команды нажимают/отпускают левую кнопку мыши по координатам внутри кнопки. Как видно, получается очень громоздко. И вместо того чтобы писать свою обертку воспользуемся уже готовой библиотекой.

---

![](./logo-puppeteer.png)

<!-- .element style="background: #fff; width: 40%; padding: 20px; display: inline-block" -->

Note: На самом деле существует достаточно много реализаций высокоуровнего API над CDP. Я выбрал Puppeteer из-за того что он предоставляет очень удобное API для работы с DOM.

---

## Puppeteer

- Хорошее и удобное API, но...
- Не работает в браузере из коробки

Note: Единственным недостатком в нашем случае является то что Puppeteer работает только в nodejs окружении. Нам же необходимо уметь работать с puppeteer из браузера. Поэтому напишем свою небольшую инициализацию.

---

```typescript
const response =
    await fetch('http://localhost:9222/json/version')
const data = await response.json()
// Ответ в json
```

```typescript
import { Connection } from 'puppeteer/lib/Connection'
import Browser from 'puppeteer/lib/Browser'

const connection = await Connection
	.createForWebSocket(data.webSocketDebuggerUrl)
const browser = await Browser
    .create(connection, { appMode: true })
const pages = await browser.pages()
```

```typescript
// Disables network tracking,
// prevents network events from being sent to the client
// NOTE Because we don't want crash browser under events flood
pages.forEach(p => p._client.send('Network.disable', {}))
```

Note: Тут мы получаем вебсокет урл для подключения. Подключаемся и запрашиваем доступные на данный момент странички. И в конце для каждой странички отключаем обработку сетевых событий. Тут хотелось бы пояснить, для чего это нужно. Как изначально я говорил, наша задача писать интеграционные тесты, а это значит, что код тестов выполняется на странице в браузере. Что в свою очередь ведет к тому что подключаться к браузеру необходимо со страницы самого браузера, чтобы управлять браузером находясь в браузере. Взрыв мозга. Но так как мы должны работать с определенной страницей, нам каким-то образом нужно выяснить какая страница относится к текущей вкладке в которой мы сейчас находимся.

Пояснить что за урл. Пояснить про инстанс (быть готовым). Разбить на слайды

---

```typescript
const page = await new Promise(resolve => {
  const expectedGuid = uuid()
  const findCurrentPage = index => async msg => {
    if (msg.type() != 'debug') return
    const [firstArg] = msg.args()
    const receivedGuid = await firstArg.jsonValue()
    if (receivedGuid == expectedGuid) {
      pages[index].removeAllListeners('console')
      resolve(pages[index])
    }
  }
  pages.forEach((p, index) =>
                p.on('console', findCurrentPage(index)))
  console.debug(expectedGuid)
})
```

Note:

---

```typescript
it('Should dispatch action with value', async () => {
  const { form } = render(
      <DatePicker path={'a/b'} tid={'datepicker'} />)
  const selector = '[data-prop-tid=datepicker]'
  await devtools.click(selector)
  await devtools.type(selector, '23102018')
  await devtools.mouse.click(0, 0)
  
  expect(form.store.dispatch)
    .to.have.been.calledWithExactly({
      type: UPDATE_MODEL,
      payload: { path: 'a/b', field: 'value',
        value: '23.10.2018' },
    })
})
```

Note: Пример теста. Код. Повторить пробему

---

Фотка, ФИО, рабочее мыло

![](./click-the-button.svg)