---
title: Выбор структуры состояния
---

<Intro>

Грамотная структура состояния может стать ключевым фактором для компонента, который приятно модифицировать и отлаживать, в отличие от компонента, который постоянно вызывает проблемы. Вот несколько полезных советов, которые помогут вам организовать состояние с умом.

</Intro>

<YouWillLearn>

* Когда следует использовать одну переменную состояния, а когда несколько
* Чего следует избегать при организации состояния
* Как можно устранить типичные проблемы, связанные с организацией состояния

</YouWillLearn>

## Принципы организации состояния {/*principles-for-structuring-state*/}

Когда вы создаете компонент, который содержит некоторое состояние, вам придется принять решения о том, сколько переменных состояния использовать и какой должна быть структура их данных. Хотя даже с неполной структурой состояния можно написать корректные программы, есть несколько принципов, которые помогут вам принимать более обдуманные решения:

1. **Группируйте связанные состояния.** Если вы всегда обновляете две или более переменные состояния одновременно, рассмотрите возможность объединения их в одну переменную.
2. **Избегайте противоречий в состоянии.**  Когда состояние структурировано таким образом, что несколько составляющих состояния могут противоречить и "не соглашаться" друг с другом, это открывает простор для ошибок. Старайтесь избегать таких ситуаций.
3. **Избегайте избыточного состояния.** Если вы можете вычислить какую-то информацию из пропсов компонента или уже существующих переменных состояния во время рендера, вам не следует помещать эту информацию в состояние компонента.
4. **Избегайте дублирования состояния.** Когда одни и те же данные дублируются между несколькими переменными состояния или внутри вложенных объектов, сложно поддерживать их согласованность. Сокращайте дублирование, когда это возможно.
5. **Избегайте глубокой вложенности состояния.** Глубоко иерархическое состояние не очень удобно обновлять. По возможности, структурируйте состояние в плоском виде.

Целью данных принципов является *упрощение процесса обновления состояния без возникновения ошибок*. Удаление избыточных и дублирующихся данных из состояния помогает гарантировать согласованность всех его компонентов. Это аналогично тому, как инженер баз данных стремится к ["нормализации" структуры базы данных](https://learn.microsoft.com/ru-ru/office/troubleshoot/access/database-normalization-description), чтобы минимизировать возможность ошибок. Перефразируя Альберта Эйнштейна, **"Сделайте своё состояние таким простым, как только возможно, но не проще."**

Теперь давайте посмотрим, как эти принципы применяются на практике.

## Группировка связанного состояния {/*group-related-state*/}

Иногда возникает неопределенность относительно использования одной или нескольких переменных состояния.

Стоит ли вам сделать так?

```js
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

Или так?

```js
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Технически вы можете использовать любой из этих подходов. Однако, **если две переменные состояния всегда изменяются вместе, возможно, имеет смысл объединить их в одну переменную состояния.** Таким образом, объединение их в одну переменную состояния позволит избежать проблем с их синхронизацией, как в этом примере, где перемещение курсора обновляет обе координаты красной точки:

<Sandpack>

```js
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  )
}
```

```css
body { margin: 0; padding: 0; height: 250px; }
```

</Sandpack>

Еще одна ситуация, когда имеет смысл объединять данные в объект или массив, возникает, когда вы не знаете, сколько состояний понадобится. Например, это может быть полезно в случае формы, где пользователь может добавлять настраиваемые поля.

<Pitfall>

Если ваше состояние представлено объектом, помните, что [что вы не можете обновить только одно поле в нем](/learn/updating-objects-in-state), не скопировав явно остальные поля. Например, вы не можете просто использовать `setPosition({ x: 100 })` в приведенном примере, поскольку оно не будет содержать свойство `y`! IВместо этого, если вы хотите установить только значение `x`, вам нужно использовать `setPosition({ ...position, x: 100 })`, либо разделить состояние на две переменные и использовать `setX(100)`.

</Pitfall>

## Избегайте противоречий в состоянии {/*avoid-contradictions-in-state*/}

Вот форма обратной связи отеля с переменными состояния `isSending` и `isSent`:

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Спасибо за обратную связь!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>Как прошло ваше пребывание в отеле "The Prancing Pony"?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Отправить
      </button>
      {isSending && <p>Отправка...</p>}
    </form>
  );
}

// Имитируем отправку сообщения.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

</Sandpack>

Хотя этот код работает, он открывает возможность для "невозможных" состояний. Например, если вы забудете вызвать `setIsSent` и `setIsSending` одновременно, может возникнуть ситуация, когда и `isSending`, и `isSent` одновременно будут равны `true`. Чем сложнее ваш компонент, тем сложнее понять, что произошло.

**Поскольку `isSending` и `isSent` не должны быть одновременно равны `true`, лучше заменить их одной переменной состояния `status`, которая может принимать одно из *трех* допустимых состояний:** `'typing'` (начальное), `'sending'` и `'sent'`:

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>Как прошло ваше пребывание в отеле "The Prancing Pony"?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Отправить
      </button>
      {isSending && <p>Отправка...</p>}
    </form>
  );
}

// Имитируем отправку сообщения.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

</Sandpack>

Вы можете объявить константы для удобства чтения:

```js
const isSending = status === 'sending';
const isSent = status === 'sent';
```

Они не являются переменными состояния, поэтому вам не нужно беспокоиться о том, что они не синхронизируются друг с другом.

## Избегайте избыточного состояния {/*avoid-redundant-state*/}

Если вы можете вычислить некоторую информацию на основе пропсов компонента или уже имеющихся переменных состояния во время процесса рендера, **не следует** сохранять эту информацию в состоянии компонента.

Например, рассмотрим эту форму. Она работает, но можете ли вы найти избыточное состояние в ней?

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Давайте вас зарегистрируем</h2>
      <label>
        Имя:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Фамилия:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Ваш билет будет оформлен на: <b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

Эта форма имеет три переменные состояния: `firstName`, `lastName`, и `fullName`. Однако переменная `fullName` является избыточной. **Во время рендера всегда можно вычислить `fullName` на основе `firstName` и `lastName` поэтому её можно удалить из состояния.**

Вот как можно это сделать:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>Давайте вас зарегистрируем</h2>
      <label>
        Имя:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Фамилия:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Ваш билет будет оформлен на: <b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

В данном случае `fullName` *не* является переменной состояния. Вместо этого она вычисляется во время рендера:

```js
const fullName = firstName + ' ' + lastName;
```

В результате обработчикам изменений не нужно выполнять никаких дополнительных действий для обновления `fullName`. Когда вы вызываете `setFirstName` или `setLastName`, происходит повторный рендер, и следующее значение `fullName` будет вычислено на основе новых данных.

<DeepDive>

#### Не дублируйте пропсы в состоянии {/*don-t-mirror-props-in-state*/}

Одним из распространенных примеров избыточного состояния является следующий код:

```js
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
```

Здесь переменная состояния `color` инициализируется значением пропса `messageColor`. Проблема в том, что **если родительский компонент позднее передаст другое значение `messageColor` (например, `'red'` вместо `'blue'`), *переменная состояния* `color` не будет обновлена!** Состояние инициализируется только во время первого рендера.

Вот почему "дублирование" какого-либо пропса в переменной состояния может привести к путанице. Вместо этого используйте проп `messageColor` непосредственно в своем коде. Если вы хотите дать ему более короткое имя, используйте константу:

```js
function Message({ messageColor }) {
  const color = messageColor;
```

Таким образом, значение не будет расходиться с переданным пропсом от родительского компонента.

"Дублирование" пропсов в состоянии имеет смысл только тогда, когда вы *хотите* игнорировать все обновления для определенного пропса. По соглашению, начните имя пропса с `initial` или `default`, чтобы ясно указать, что его новые значения игнорируются:

```js
function Message({ initialColor }) {
  // Переменная состояния `color` хранит *первоначальное* значение `initialColor`.
  // Дальнейшие изменения пропса `initialColor` игнорируются.
  const [color, setColor] = useState(initialColor);
```

</DeepDive>

## Избегайте дублирования состояния {/*avoid-duplication-in-state*/}

Этот компонент списка меню позволяет выбрать один туристический перекус из нескольких вариантов:

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'крендели', id: 0 },
  { title: 'хрустящая морская капуста', id: 1 },
  { title: 'злаковый батончик', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  return (
    <>
      <h2>Какой перекус взять в путешествие?</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.title}
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Выбрать</button>
          </li>
        ))}
      </ul>
      <p>Вы выбрали {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

В настоящее время выбранный элемент хранится в виде объекта в переменной состояния `selectedItem`. Однако это не совсем оптимально: **содержимое `selectedItem` является одним и тем же объектом, что и один из элементов в списке `items`.** Это означает, что информация об элементе дублируется в двух местах.

Почему это является проблемой? Давайте сделаем каждый элемент редактируемым:

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'крендели', id: 0 },
  { title: 'хрустящая морская капуста', id: 1 },
  { title: 'злаковый батончик', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>Какой перекус взять в путешествие?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Выбрать</button>
          </li>
        ))}
      </ul>
      <p>Вы выбрали {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

Обратите внимание, что если вы сначала нажмете кнопку "Выбрать" для элемента, а затем отредактируете его, **поле ввода обновится, но надпись внизу не отразит изменения.** Это происходит из-за дублирования состояния, a также вы забыли обновить `selectedItem`.

Хотя вы можете обновить `selectedItem`, более простым решением будет избавиться от дублирования. В этом примере, вместо объекта  `selectedItem` (который создает дублирование с объектами внутри `items`), вы сохраняете `selectedId` в состоянии, а *затем* получаете `selectedItem` выполнив поиск в массиве `items` по этому ID:

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'крендели', id: 0 },
  { title: 'хрустящая морская капуста', id: 1 },
  { title: 'злаковый батончик', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);

  const selectedItem = items.find(item =>
    item.id === selectedId
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>Какой перекус взять в путешествие?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedId(item.id);
            }}>Выбрать</button>
          </li>
        ))}
      </ul>
      <p>Вы выбрали {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

(В качестве альтернативы, вы можете использовать индекс выбранного элемента в состоянии.)

Ранее состояние дублировалось вот так:

* `items = [{ id: 0, title: 'крендели'}, ...]`
* `selectedItem = {id: 0, title: 'крендели'}`

Однако после изменения оно выглядит так:

* `items = [{ id: 0, title: 'крендели'}, ...]`
* `selectedId = 0`

Дублирование исчезло, и осталось только необходимое состояние!

Теперь, если вы редактируете *выбранный* элемент, сообщение ниже будет немедленно обновлено. Это происходит потому, что вызов `setItems` вызывает повторный рендер, и `items.find(...)` будет находить элемент с обновленным заголовком. Вам не нужно было хранить *выбранный элемент* в состоянии, потому что только  *выбранный ID* является важным. Остальное может быть вычислено во время рендера.

## Избегайте глубокой вложенности состояния {/*avoid-deeply-nested-state*/}

Представьте себе план путешествия, состоящий из планет, континентов и стран. Возможно, вам захотелось бы структурировать его состояние с помощью вложенных объектов и массивов, как в этом примере:

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ place }) {
  const childPlaces = place.childPlaces;
  return (
    <li>
      {place.title}
      {childPlaces.length > 0 && (
        <ol>
          {childPlaces.map(place => (
            <PlaceTree key={place.id} place={place} />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const planets = plan.childPlaces;
  return (
    <>
      <h2>Места, которые нужно посетить</h2>
      <ol>
        {planets.map(place => (
          <PlaceTree key={place.id} place={place} />
        ))}
      </ol>
    </>
  );
}
```

```js places.js active
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Earth',
    childPlaces: [{
      id: 2,
      title: 'Africa',
      childPlaces: [{
        id: 3,
        title: 'Botswana',
        childPlaces: []
      }, {
        id: 4,
        title: 'Egypt',
        childPlaces: []
      }, {
        id: 5,
        title: 'Kenya',
        childPlaces: []
      }, {
        id: 6,
        title: 'Madagascar',
        childPlaces: []
      }, {
        id: 7,
        title: 'Morocco',
        childPlaces: []
      }, {
        id: 8,
        title: 'Nigeria',
        childPlaces: []
      }, {
        id: 9,
        title: 'South Africa',
        childPlaces: []
      }]
    }, {
      id: 10,
      title: 'Americas',
      childPlaces: [{
        id: 11,
        title: 'Argentina',
        childPlaces: []
      }, {
        id: 12,
        title: 'Brazil',
        childPlaces: []
      }, {
        id: 13,
        title: 'Barbados',
        childPlaces: []
      }, {
        id: 14,
        title: 'Canada',
        childPlaces: []
      }, {
        id: 15,
        title: 'Jamaica',
        childPlaces: []
      }, {
        id: 16,
        title: 'Mexico',
        childPlaces: []
      }, {
        id: 17,
        title: 'Trinidad and Tobago',
        childPlaces: []
      }, {
        id: 18,
        title: 'Venezuela',
        childPlaces: []
      }]
    }, {
      id: 19,
      title: 'Asia',
      childPlaces: [{
        id: 20,
        title: 'China',
        childPlaces: []
      }, {
        id: 21,
        title: 'Hong Kong',
        childPlaces: []
      }, {
        id: 22,
        title: 'India',
        childPlaces: []
      }, {
        id: 23,
        title: 'Singapore',
        childPlaces: []
      }, {
        id: 24,
        title: 'South Korea',
        childPlaces: []
      }, {
        id: 25,
        title: 'Thailand',
        childPlaces: []
      }, {
        id: 26,
        title: 'Vietnam',
        childPlaces: []
      }]
    }, {
      id: 27,
      title: 'Europe',
      childPlaces: [{
        id: 28,
        title: 'Croatia',
        childPlaces: [],
      }, {
        id: 29,
        title: 'France',
        childPlaces: [],
      }, {
        id: 30,
        title: 'Germany',
        childPlaces: [],
      }, {
        id: 31,
        title: 'Italy',
        childPlaces: [],
      }, {
        id: 32,
        title: 'Portugal',
        childPlaces: [],
      }, {
        id: 33,
        title: 'Spain',
        childPlaces: [],
      }, {
        id: 34,
        title: 'Turkey',
        childPlaces: [],
      }]
    }, {
      id: 35,
      title: 'Oceania',
      childPlaces: [{
        id: 36,
        title: 'Australia',
        childPlaces: [],
      }, {
        id: 37,
        title: 'Bora Bora (French Polynesia)',
        childPlaces: [],
      }, {
        id: 38,
        title: 'Easter Island (Chile)',
        childPlaces: [],
      }, {
        id: 39,
        title: 'Fiji',
        childPlaces: [],
      }, {
        id: 40,
        title: 'Hawaii (the USA)',
        childPlaces: [],
      }, {
        id: 41,
        title: 'New Zealand',
        childPlaces: [],
      }, {
        id: 42,
        title: 'Vanuatu',
        childPlaces: [],
      }]
    }]
  }, {
    id: 43,
    title: 'Moon',
    childPlaces: [{
      id: 44,
      title: 'Rheita',
      childPlaces: []
    }, {
      id: 45,
      title: 'Piccolomini',
      childPlaces: []
    }, {
      id: 46,
      title: 'Tycho',
      childPlaces: []
    }]
  }, {
    id: 47,
    title: 'Mars',
    childPlaces: [{
      id: 48,
      title: 'Corn Town',
      childPlaces: []
    }, {
      id: 49,
      title: 'Green Hill',
      childPlaces: []
    }]
  }]
};
```

</Sandpack>

Теперь предположим, что вы хотите добавить кнопку для удаления посещенного вами места. Как бы вы это сделали? [Обновление вложенного состояния](/learn/updating-objects-in-state#updating-a-nested-object) включает создание копий объектов, начиная от измененной части и до самого верхнего уровня. Удаление глубоко вложенного места потребует копирования всей его цепочки родительских мест. Такой код может быть очень громоздким.

**Если состояние слишком глубоко вложено и обновляется с трудом, рассмотрите возможность "развернуть" его.** Вот один из способов переструктурировать эти данные. Вместо древовидной структуры, где каждое `place` имеет массив *дочерних мест*, вы можете сделать так, чтобы каждое место содержало массив *своих дочерних ID*. Затем сохраните отображение, которое связывает каждый ID места с соответствующим местом.

Измененная структура данных может напомнить вам структуру таблицы в базе данных:

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ id, placesById }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      {childIds.length > 0 && (
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              placesById={placesById}
            />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Места, которые нужно посетить</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            placesById={plan}
          />
        ))}
      </ol>
    </>
  );
}
```

```js places.js active
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 43, 47],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 27, 35]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  },
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  },
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Americas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  },
  14: {
    id: 14,
    title: 'Canada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Mexico',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad and Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25, 26],
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'Hong Kong',
    childIds: []
  },
  22: {
    id: 22,
    title: 'India',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Singapore',
    childIds: []
  },
  24: {
    id: 24,
    title: 'South Korea',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Thailand',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Vietnam',
    childIds: []
  },
  27: {
    id: 27,
    title: 'Europe',
    childIds: [28, 29, 30, 31, 32, 33, 34],
  },
  28: {
    id: 28,
    title: 'Croatia',
    childIds: []
  },
  29: {
    id: 29,
    title: 'France',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Germany',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Italy',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Portugal',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Spain',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Turkey',
    childIds: []
  },
  35: {
    id: 35,
    title: 'Oceania',
    childIds: [36, 37, 38, 39, 40, 41, 42],
  },
  36: {
    id: 36,
    title: 'Australia',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Bora Bora (French Polynesia)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Easter Island (Chile)',
    childIds: []
  },
  39: {
    id: 39,
    title: 'Fiji',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Hawaii (the USA)',
    childIds: []
  },
  41: {
    id: 41,
    title: 'New Zealand',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Vanuatu',
    childIds: []
  },
  43: {
    id: 43,
    title: 'Moon',
    childIds: [44, 45, 46]
  },
  44: {
    id: 44,
    title: 'Rheita',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Piccolomini',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Tycho',
    childIds: []
  },
  47: {
    id: 47,
    title: 'Mars',
    childIds: [48, 49]
  },
  48: {
    id: 48,
    title: 'Corn Town',
    childIds: []
  },
  49: {
    id: 49,
    title: 'Green Hill',
    childIds: []
  }
};
```

</Sandpack>

**Теперь, когда состояние является "плоским" (также известным как "нормализованным"), обновление вложенных элементов становится проще.**

Чтобы удалить место, вам достаточно обновить два уровня состояния:

- Обновленная версия *родительского* места должна исключать удаленный ID из своего массива  `childIds`.
- Обновленная версия корневого объекта "таблицы" должна включать обновленную версию родительского места.

Вот пример того, как вы можете это сделать:

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Создайте новую версию родительского места
    // которая не включает данный ID дочернего элемента.
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId)
    };
    // Обновите корневой объект состояния...
    setPlan({
      ...plan,
      // ...чтобы он содержал обновленного родителя.
      [parentId]: nextParent
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Места, которые нужно посетить</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Завершить
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

```js places.js
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 43, 47],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 27, 35]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  },
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  },
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Americas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  },
  14: {
    id: 14,
    title: 'Canada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Mexico',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad and Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25, 26],
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'Hong Kong',
    childIds: []
  },
  22: {
    id: 22,
    title: 'India',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Singapore',
    childIds: []
  },
  24: {
    id: 24,
    title: 'South Korea',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Thailand',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Vietnam',
    childIds: []
  },
  27: {
    id: 27,
    title: 'Europe',
    childIds: [28, 29, 30, 31, 32, 33, 34],
  },
  28: {
    id: 28,
    title: 'Croatia',
    childIds: []
  },
  29: {
    id: 29,
    title: 'France',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Germany',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Italy',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Portugal',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Spain',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Turkey',
    childIds: []
  },
  35: {
    id: 35,
    title: 'Oceania',
    childIds: [36, 37, 38, 39, 40, 41,, 42],
  },
  36: {
    id: 36,
    title: 'Australia',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Bora Bora (French Polynesia)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Easter Island (Chile)',
    childIds: []
  },
  39: {
    id: 39,
    title: 'Fiji',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Hawaii (the USA)',
    childIds: []
  },
  41: {
    id: 41,
    title: 'New Zealand',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Vanuatu',
    childIds: []
  },
  43: {
    id: 43,
    title: 'Moon',
    childIds: [44, 45, 46]
  },
  44: {
    id: 44,
    title: 'Rheita',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Piccolomini',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Tycho',
    childIds: []
  },
  47: {
    id: 47,
    title: 'Mars',
    childIds: [48, 49]
  },
  48: {
    id: 48,
    title: 'Corn Town',
    childIds: []
  },
  49: {
    id: 49,
    title: 'Green Hill',
    childIds: []
  }
};
```

```css
button { margin: 10px; }
```

</Sandpack>

Вы можете вкладывать состояние насколько угодно глубоко, но сделать его "плоским" может решить множество проблем. Это упрощает обновление состояния и помогает избежать дублирования информации в разных частях вложенного объекта.

<DeepDive>

#### Улучшение использования памяти {/*improving-memory-usage*/}

Идеально было бы также исключить удаленные элементы (и их потомков!) из объекта "table", чтобы улучшить использование памяти. В этой версии это сделано. Кроме того, [используется Immer](/learn/updating-objects-in-state#write-concise-update-logic-with-immer), чтобы сделать логику обновления более лаконичной.

<Sandpack>

```js
import { useImmer } from 'use-immer';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, updatePlan] = useImmer(initialTravelPlan);

  function handleComplete(parentId, childId) {
    updatePlan(draft => {
      // Удаляем дочерние ID из родительских мест.
      const parent = draft[parentId];
      parent.childIds = parent.childIds
        .filter(id => id !== childId);

      // Удаляем место и всё его поддерево.
      deleteAllChildren(childId);
      function deleteAllChildren(id) {
        const place = draft[id];
        place.childIds.forEach(deleteAllChildren);
        delete draft[id];
      }
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Места, которые нужно посетить</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Завершить
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

```js places.js
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 43, 47],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 27, 35]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  },
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egypt',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenya',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagascar',
    childIds: []
  },
  7: {
    id: 7,
    title: 'Morocco',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigeria',
    childIds: []
  },
  9: {
    id: 9,
    title: 'South Africa',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Americas',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  },
  14: {
    id: 14,
    title: 'Canada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamaica',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Mexico',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad and Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venezuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Asia',
    childIds: [20, 21, 22, 23, 24, 25, 26],
  },
  20: {
    id: 20,
    title: 'China',
    childIds: []
  },
  21: {
    id: 21,
    title: 'Hong Kong',
    childIds: []
  },
  22: {
    id: 22,
    title: 'India',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Singapore',
    childIds: []
  },
  24: {
    id: 24,
    title: 'South Korea',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Thailand',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Vietnam',
    childIds: []
  },
  27: {
    id: 27,
    title: 'Europe',
    childIds: [28, 29, 30, 31, 32, 33, 34],
  },
  28: {
    id: 28,
    title: 'Croatia',
    childIds: []
  },
  29: {
    id: 29,
    title: 'France',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Germany',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Italy',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Portugal',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Spain',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Turkey',
    childIds: []
  },
  35: {
    id: 35,
    title: 'Oceania',
    childIds: [36, 37, 38, 39, 40, 41,, 42],
  },
  36: {
    id: 36,
    title: 'Australia',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Bora Bora (French Polynesia)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Easter Island (Chile)',
    childIds: []
  },
  39: {
    id: 39,
    title: 'Fiji',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Hawaii (the USA)',
    childIds: []
  },
  41: {
    id: 41,
    title: 'New Zealand',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Vanuatu',
    childIds: []
  },
  43: {
    id: 43,
    title: 'Moon',
    childIds: [44, 45, 46]
  },
  44: {
    id: 44,
    title: 'Rheita',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Piccolomini',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Tycho',
    childIds: []
  },
  47: {
    id: 47,
    title: 'Mars',
    childIds: [48, 49]
  },
  48: {
    id: 48,
    title: 'Corn Town',
    childIds: []
  },
  49: {
    id: 49,
    title: 'Green Hill',
    childIds: []
  }
};
```

```css
button { margin: 10px; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

</DeepDive>

Иногда вы можете уменьшить вложенность состояния, переместив часть вложенного состояния в дочерние компоненты. Это хорошо работает для временного состояния UI, которое не требует сохранения, например, состояние наведения на элемент.

<Recap>

* Если две переменные состояния всегда обновляются вместе, рассмотрите возможность объединить их в одну.
* Тщательно выбирайте переменные состояния, чтобы избежать создания "невозможных" состояний.
* Структурируйте состояние таким образом, чтобы уменьшить вероятность ошибок при его обновлении.
* Избегайте избыточных и дублирующих состояний, чтобы не нужно было их синхронизировать.
* Не помещайте пропсы *в состояние*, если только вы специально не хотите предотвратить обновления.
* Для UI-паттернов, таких как выбор, храните в состоянии идентификатор или индекс, а не сам объект.
* Если обновление глубоко вложенного состояния усложняется, попробуйте упростить его структуру.
</Recap>

<Challenges>

#### Исправьте компонент, который не обновляется {/*fix-a-component-thats-not-updating*/}

Компонент `Clock` принимает два пропса: `color` и `time`. Когда вы выбираете другой цвет в выпадающем списке, компонент `Clock` получает новое значение `color` от родительского компонента. Однако, по какой-то причине отображаемый цвет не обновляется. Почему? Исправьте проблему.

<Sandpack>

```js Clock.js active
import { useState } from 'react';

export default function Clock(props) {
  const [color, setColor] = useState(props.color);
  return (
    <h1 style={{ color: color }}>
      {props.time}
    </h1>
  );
}
```

```js App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Выберите цвет:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

<Solution>

Проблема заключается в том, что этот компонент имеет состояние `color`, которое инициализируется начальным значением пропcа `color`. Однако, когда значение пропcа `color` изменяется, это не отражается на состоянии компонента, и они перестают синхронизироваться. Чтобы исправить эту проблему, удалите переменную состояния `color` полностью и используйте проп `color` напрямую.

<Sandpack>

```js Clock.js active
import { useState } from 'react';

export default function Clock(props) {
  return (
    <h1 style={{ color: props.color }}>
      {props.time}
    </h1>
  );
}
```

```js App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Выберите цвет:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

Или, используя синтаксис деструктуризации:

<Sandpack>

```js Clock.js active
import { useState } from 'react';

export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

```js App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Выберите цвет:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

</Solution>

#### Fix a broken packing list {/*fix-a-broken-packing-list*/}

This packing list has a footer that shows how many items are packed, and how many items there are overall. It seems to work at first, but it is buggy. For example, if you mark an item as packed and then delete it, the counter will not be updated correctly. Fix the counter so that it's always correct.

<Hint>

Is any state in this example redundant?

</Hint>

<Sandpack>

```js App.js
import { useState } from 'react';
import AddItem from './AddItem.js';
import PackingList from './PackingList.js';

let nextId = 3;
const initialItems = [
  { id: 0, title: 'Warm socks', packed: true },
  { id: 1, title: 'Travel journal', packed: false },
  { id: 2, title: 'Watercolors', packed: false },
];

export default function TravelPlan() {
  const [items, setItems] = useState(initialItems);
  const [total, setTotal] = useState(3);
  const [packed, setPacked] = useState(1);

  function handleAddItem(title) {
    setTotal(total + 1);
    setItems([
      ...items,
      {
        id: nextId++,
        title: title,
        packed: false
      }
    ]);
  }

  function handleChangeItem(nextItem) {
    if (nextItem.packed) {
      setPacked(packed + 1);
    } else {
      setPacked(packed - 1);
    }
    setItems(items.map(item => {
      if (item.id === nextItem.id) {
        return nextItem;
      } else {
        return item;
      }
    }));
  }

  function handleDeleteItem(itemId) {
    setTotal(total - 1);
    setItems(
      items.filter(item => item.id !== itemId)
    );
  }

  return (
    <>
      <AddItem
        onAddItem={handleAddItem}
      />
      <PackingList
        items={items}
        onChangeItem={handleChangeItem}
        onDeleteItem={handleDeleteItem}
      />
      <hr />
      <b>{packed} out of {total} packed!</b>
    </>
  );
}
```

```js AddItem.js hidden
import { useState } from 'react';

export default function AddItem({ onAddItem }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add item"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddItem(title);
      }}>Add</button>
    </>
  )
}
```

```js PackingList.js hidden
import { useState } from 'react';

export default function PackingList({
  items,
  onChangeItem,
  onDeleteItem
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <label>
            <input
              type="checkbox"
              checked={item.packed}
              onChange={e => {
                onChangeItem({
                  ...item,
                  packed: e.target.checked
                });
              }}
            />
            {' '}
            {item.title}
          </label>
          <button onClick={() => onDeleteItem(item.id)}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution>

Although you could carefully change each event handler to update the `total` and `packed` counters correctly, the root problem is that these state variables exist at all. They are redundant because you can always calculate the number of items (packed or total) from the `items` array itself. Remove the redundant state to fix the bug:

<Sandpack>

```js App.js
import { useState } from 'react';
import AddItem from './AddItem.js';
import PackingList from './PackingList.js';

let nextId = 3;
const initialItems = [
  { id: 0, title: 'Warm socks', packed: true },
  { id: 1, title: 'Travel journal', packed: false },
  { id: 2, title: 'Watercolors', packed: false },
];

export default function TravelPlan() {
  const [items, setItems] = useState(initialItems);

  const total = items.length;
  const packed = items
    .filter(item => item.packed)
    .length;

  function handleAddItem(title) {
    setItems([
      ...items,
      {
        id: nextId++,
        title: title,
        packed: false
      }
    ]);
  }

  function handleChangeItem(nextItem) {
    setItems(items.map(item => {
      if (item.id === nextItem.id) {
        return nextItem;
      } else {
        return item;
      }
    }));
  }

  function handleDeleteItem(itemId) {
    setItems(
      items.filter(item => item.id !== itemId)
    );
  }

  return (
    <>
      <AddItem
        onAddItem={handleAddItem}
      />
      <PackingList
        items={items}
        onChangeItem={handleChangeItem}
        onDeleteItem={handleDeleteItem}
      />
      <hr />
      <b>{packed} out of {total} packed!</b>
    </>
  );
}
```

```js AddItem.js hidden
import { useState } from 'react';

export default function AddItem({ onAddItem }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add item"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddItem(title);
      }}>Add</button>
    </>
  )
}
```

```js PackingList.js hidden
import { useState } from 'react';

export default function PackingList({
  items,
  onChangeItem,
  onDeleteItem
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <label>
            <input
              type="checkbox"
              checked={item.packed}
              onChange={e => {
                onChangeItem({
                  ...item,
                  packed: e.target.checked
                });
              }}
            />
            {' '}
            {item.title}
          </label>
          <button onClick={() => onDeleteItem(item.id)}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

Notice how the event handlers are only concerned with calling `setItems` after this change. The item counts are now calculated during the next render from `items`, so they are always up-to-date.

</Solution>

#### Fix the disappearing selection {/*fix-the-disappearing-selection*/}

There is a list of `letters` in state. When you hover or focus a particular letter, it gets highlighted. The currently highlighted letter is stored in the `highlightedLetter` state variable. You can "star" and "unstar" individual letters, which updates the `letters` array in state.

This code works, but there is a minor UI glitch. When you press "Star" or "Unstar", the highlighting disappears for a moment. However, it reappears as soon as you move your pointer or switch to another letter with keyboard. Why is this happening? Fix it so that the highlighting doesn't disappear after the button click.

<Sandpack>

```js App.js
import { useState } from 'react';
import { initialLetters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [letters, setLetters] = useState(initialLetters);
  const [highlightedLetter, setHighlightedLetter] = useState(null);

  function handleHover(letter) {
    setHighlightedLetter(letter);
  }

  function handleStar(starred) {
    setLetters(letters.map(letter => {
      if (letter.id === starred.id) {
        return {
          ...letter,
          isStarred: !letter.isStarred
        };
      } else {
        return letter;
      }
    }));
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isHighlighted={
              letter === highlightedLetter
            }
            onHover={handleHover}
            onToggleStar={handleStar}
          />
        ))}
      </ul>
    </>
  );
}
```

```js Letter.js
export default function Letter({
  letter,
  isHighlighted,
  onHover,
  onToggleStar,
}) {
  return (
    <li
      className={
        isHighlighted ? 'highlighted' : ''
      }
      onFocus={() => {
        onHover(letter);
      }}
      onPointerMove={() => {
        onHover(letter);
      }}
    >
      <button onClick={() => {
        onToggleStar(letter);
      }}>
        {letter.isStarred ? 'Unstar' : 'Star'}
      </button>
      {letter.subject}
    </li>
  )
}
```

```js data.js
export const initialLetters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
button { margin: 5px; }
li { border-radius: 5px; }
.highlighted { background: #d2eaff; }
```

</Sandpack>

<Solution>

The problem is that you're holding the letter object in `highlightedLetter`. But you're also holding the same information in the `letters` array. So your state has duplication! When you update the `letters` array after the button click, you create a new letter object which is different from `highlightedLetter`. This is why `highlightedLetter === letter` check becomes `false`, and the highlight disappears. It reappears the next time you call `setHighlightedLetter` when the pointer moves.

To fix the issue, remove the duplication from state. Instead of storing *the letter itself* in two places, store the `highlightedId` instead. Then you can check `isHighlighted` for each letter with `letter.id === highlightedId`, which will work even if the `letter` object has changed since the last render.

<Sandpack>

```js App.js
import { useState } from 'react';
import { initialLetters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [letters, setLetters] = useState(initialLetters);
  const [highlightedId, setHighlightedId ] = useState(null);

  function handleHover(letterId) {
    setHighlightedId(letterId);
  }

  function handleStar(starredId) {
    setLetters(letters.map(letter => {
      if (letter.id === starredId) {
        return {
          ...letter,
          isStarred: !letter.isStarred
        };
      } else {
        return letter;
      }
    }));
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isHighlighted={
              letter.id === highlightedId
            }
            onHover={handleHover}
            onToggleStar={handleStar}
          />
        ))}
      </ul>
    </>
  );
}
```

```js Letter.js
export default function Letter({
  letter,
  isHighlighted,
  onHover,
  onToggleStar,
}) {
  return (
    <li
      className={
        isHighlighted ? 'highlighted' : ''
      }
      onFocus={() => {
        onHover(letter.id);
      }}
      onPointerMove={() => {
        onHover(letter.id);
      }}
    >
      <button onClick={() => {
        onToggleStar(letter.id);
      }}>
        {letter.isStarred ? 'Unstar' : 'Star'}
      </button>
      {letter.subject}
    </li>
  )
}
```

```js data.js
export const initialLetters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
button { margin: 5px; }
li { border-radius: 5px; }
.highlighted { background: #d2eaff; }
```

</Sandpack>

</Solution>

#### Implement multiple selection {/*implement-multiple-selection*/}

In this example, each `Letter` has an `isSelected` prop and an `onToggle` handler that marks it as selected. This works, but the state is stored as a `selectedId` (either `null` or an ID), so only one letter can get selected at any given time.

Change the state structure to support multiple selection. (How would you structure it? Think about this before writing the code.) Each checkbox should become independent from the others. Clicking a selected letter should uncheck it. Finally, the footer should show the correct number of the selected items.

<Hint>

Instead of a single selected ID, you might want to hold an array or a [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) of selected IDs in state.

</Hint>

<Sandpack>

```js App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedId, setSelectedId] = useState(null);

  // TODO: allow multiple selection
  const selectedCount = 1;

  function handleToggle(toggledId) {
    // TODO: allow multiple selection
    setSelectedId(toggledId);
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              // TODO: allow multiple selection
              letter.id === selectedId
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            You selected {selectedCount} letters
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js data.js
export const letters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

<Solution>

Instead of a single `selectedId`, keep a `selectedIds` *array* in state. For example, if you select the first and the last letter, it would contain `[0, 2]`. When nothing is selected, it would be an empty `[]` array:

<Sandpack>

```js App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedIds, setSelectedIds] = useState([]);

  const selectedCount = selectedIds.length;

  function handleToggle(toggledId) {
    // Was it previously selected?
    if (selectedIds.includes(toggledId)) {
      // Then remove this ID from the array.
      setSelectedIds(selectedIds.filter(id =>
        id !== toggledId
      ));
    } else {
      // Otherwise, add this ID to the array.
      setSelectedIds([
        ...selectedIds,
        toggledId
      ]);
    }
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              selectedIds.includes(letter.id)
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            You selected {selectedCount} letters
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js data.js
export const letters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

One minor downside of using an array is that for each item, you're calling `selectedIds.includes(letter.id)` to check whether it's selected. If the array is very large, this can become a performance problem because array search with [`includes()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes) takes linear time, and you're doing this search for each individual item.

To fix this, you can hold a [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) in state instead, which provides a fast [`has()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/has) operation:

<Sandpack>

```js App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedIds, setSelectedIds] = useState(
    new Set()
  );

  const selectedCount = selectedIds.size;

  function handleToggle(toggledId) {
    // Create a copy (to avoid mutation).
    const nextIds = new Set(selectedIds);
    if (nextIds.has(toggledId)) {
      nextIds.delete(toggledId);
    } else {
      nextIds.add(toggledId);
    }
    setSelectedIds(nextIds);
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              selectedIds.has(letter.id)
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            You selected {selectedCount} letters
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js data.js
export const letters = [{
  id: 0,
  subject: 'Ready for adventure?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Time to check in!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival Begins in Just SEVEN Days!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

Now each item does a `selectedIds.has(letter.id)` check, which is very fast.

Keep in mind that you [should not mutate objects in state](/learn/updating-objects-in-state), and that includes Sets, too. This is why the `handleToggle` function creates a *copy* of the Set first, and then updates that copy.

</Solution>

</Challenges>
