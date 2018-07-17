---
revealOptions:
  transition: slide
---



# Click the button

## the hard way

---

## Enzyme

```jsx
const wrapper = mount(<Button />)
wrapper.find('button').simulate('click')

// Работать не будет
// wrapper.simulate('click') 
```

- Не честные клики, вызов `props.onClick`
- Завязка на внутреннюю разметку

---

## TestCafe

```jsx
test('Click the button', async t => {
  await t.click('[data-prop-tid=button]')
  // ...
})
```

- Удобная штука для e2e тестов
- Можно инжектить код на клиент
- Мокать и навешивать spy, слишком сложно

---

## DOM Events

```jsx
const button = document.querySelector('[data-prop-tid=button]')

const event = new MouseEvent('click')
button.querySelector('button').dispatchEvent(event)
// Или
button.querySelector('button').click()
```

- Всплытие и tid навешивается на label
- Завязка на внутреннюю разметку компонентов

---

## Chrome DevTools Protocol

Решение в лоб

---

```javascript
const ws = new WebSocket(
    `ws://localhost:9222/devtools/page/${PageId}`)
const send = data =>
	ws.send(JSON.stringify({ id: uuid(), ...data }))
```

```javascript
send({method: 'DOM.getDocument', params: {}})
send({
    method: 'DOM.querySelector',
    params: {nodeId: 1, selector: '[data-prop-tid=button]'}})
```

```javascript
send({method: 'DOM.getBoxModel', params: { nodeId: 23}})
send({
    method: 'Input.dispatchMouseEvent',
    params: { type: 'mousePressed', x: 40, y: 357,
               button: 'left', clickCount: 1}})
send({
    method: 'Input.dispatchMouseEvent',
    params: { type: 'mouseReleased', x: 40, y: 357,
             button: 'left', clickCount: 1}})
```



---

## Puppeteer 

- Хорошее и удобное API, но...
- Не работает в браузере из коробки

---

## attachDevTools

```javascript
import { Connection } from 'puppeteer/lib/Connection'
import Browser from 'puppeteer/lib/Browser'
```

```javascript
const response =
    await fetch('http://localhost:9222/json/version')
const data = await response.json()
```

```javascript
const connection = await Connection
	.createForWebSocket(data.webSocketDebuggerUrl)
const browser = await Browser
    .create(connection, { appMode: true })
const pages = await browser.pages()
```

```javascript
// Disables network tracking,
// prevents network events from being sent to the client
// NOTE Because we don't want crash browser under events flood
pages.forEach(p => p._client.send('Network.disable', {}))
```

---

```javascript
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

---

## PROFIT

- Удобное API
- Честные клики и не только
- Работает только в хроме

---

## Bonus

```javascript
const promiseHandler = {
  apply(target, _thisArg, args) {
    return target().then(func => func(...args))
  },
  get(target, prop) {
    return new Proxy(() => target()
      .then(obj => obj[prop]), promiseHandler)
  },
}

const devtools = new Proxy(() => attachDevTools(),
                           promiseHandler)
```

```typescript
render(<Button tid="button" />, container)
await devtools.click('[data-prop-tid=button]')
// ...
render(<DatePicker tid="datepicker" />, container)
await devtools.click('[data-prop-tid=datepicker]')
await devtools.type('[data-prop-tid=datepicker]', '23.10.2018')
await devtools.mouse.click(0, 0)
```

<https://github.com/skbkontur/react-ui-testing/tree/master/SeleniumTesting> 