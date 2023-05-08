---
title: useCallback
---

<Intro>

Хук `useCallback` позволяет кэшировать объявление функции между ре-рендерами.

```js
const cachedFn = useCallback(fn, dependencies)
```

</Intro>

<InlineToc />

---

## Справочник {/*reference*/}

### `useCallback(fn, dependencies)` {/*usecallback*/}

Вызовите `useCallback` на верхнем уровне вашего компонента, чтобы закэшировать объявление функции между ре-рендерами:

```js {4,9}
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```

[Больше примеров внизу.](#usage)

#### Параметры {/*parameters*/}

* `fn`: Функция, которую вы хотите мемоизировать (закэшировать). Она может принимать любые аргументы и возвращать любые значения. React возвратит (не вызовет!) вашу функцию при первоначальном рендере. При последующем рендеринге, React будет возвращать вам эту же функцию, если `dependencies` не изменились с момента последнего рендера. Если они изменились, то он отдаст новую функцию и закэширует её. React не будет вызывать вашу функцию. Функция возвращается вам, чтобы вы могли решить, когда ее вызывать и вызывать ли вообще.

* `dependencies`: Список всех реактивных значений, к которым есть обращения внутри кода функции `fn`. Реактивные значения включают в себя пропсы, состояние и все переменные и функции, которые декларируются в теле вашего компонента напрямую. Если ваш линтер [сконфигурирован под React](/learn/editor-setup#linting), он будет проверять, что каждое реактивное значение было указано как зависимость. Список зависимостей должен иметь постоянное число элементов и записывается в строку, например, `[dep1, dep2, dep3]`. React будет сравнивать каждую зависимость с ее предыдущим значением, используя алгоритм сравнения [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is).

#### Возвращает {/*returns*/}

При первоначальном рендере `useCallback` возвращает функцию `fn`, которую вы передали.

При последующих рендерах, `useCallback` вернет или уже сохраненную функцию `fn` из последнего рендера (если зависимости не изменились) или вернет функцию `fn`, которую вы передали в этом рендере.

#### Предостережения {/*caveats*/}

* `useCallback` -- это хук, поэтому вы можете вызвать его только **на верхнем уровне вашего компонента** или в ваших собственных хуках. Этот хук нельзя вызывать внутри условий или циклов. Если вам это нужно, то создайте новый компонент и переместите состояние внутрь него.
* React **не будет удалять кэшированную функцию без причины.** Например, в режиме разработки, React удаляет кэш, когда вы изменяете файл вашего компонента. И в режиме разработки и в продакшене React будет удалять кэш, если ваш компонент изначально рендерится с задержкой. В будущем в React возможно появятся новые фичи, которые будут использовать весь потенциал удаления кэша. Например, если будет добавлена встроенная поддержка виртуальных списков, то будет логично удалять кэш для тех элементов, которые находятся вне виртуального вьюпорта таблицы. Если вы рассчитываете на `useCallback` в части оптимизации производительности, то это должно вам подходить. Если нет, то [переменная состояния](/reference/react/useState#im-trying-to-set-state-to-a-function-but-it-gets-called-instead) или [реф](/reference/react/useRef#avoiding-recreating-the-ref-contents) могут быть более подходящими.

---

## Применение {/*usage*/}

### Пропуск повторного рендеринга компонентов {/*skipping-re-rendering-of-components*/}

Если вы оптимизируете производительность рендеринга, то иногда вам нужно будет кэшировать функции, которые вы передаете дочерним компонентам. Давайте посмотрим, как можно это сделать и проанализируем, когда это может быть полезно.

Чтобы закэшировать функцию между ре-рендерингом вашего компонента, оберните его объявление в хук `useCallback`:

```js [[3, 4, "handleSubmit"], [2, 9, "[productId, referrer]"]]
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
```

В `useCallback` нужно передать два параметра:

1. Функцию, которую вы хотите закэшировать.
2. <CodeStep step={2}>Список зависимостей</CodeStep> -- каждое значение внутри компонента, которое используется внутри вашего функции.

При изначальном рендеринге, <CodeStep step={3}>функция, которую вернет</CodeStep> `useCallback` будет функцией, которую вы передали.

При последующих рендерах, React будет сравнивать <CodeStep step={2}>зависимости</CodeStep> с зависимостями, которые были переданы в прошлом рендере. Если никакие из них не изменились (сравнивается с помощью [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), то `useCallback` вернет ту же функцию, что и раньше. В ином случае `useCallback` вернет функцию, которую вы передали в *этом* рендере.

Другими словами, `useCallback` будет кэшировать функцию, пока её зависимости не изменятся.

**Давайте посмотрим на пример, чем это может быть полезно.**

Допустим мы передаем функцию `handleSubmit` из компонента `ProductPage` в `ShippingForm`:

```js {5}
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
```

Вы можете заметить, что при переключении пропса `theme` приложение подтормаживает. Но если вы удалите `<ShippingForm />` из вашего JSX, то все работает быстро. Это подсказывает нам, что нужно попробовать оптимизировать компонент `ShippingForm`.

**По умолчанию, когда компонент ре-рендерится, React рекурсивно ре-рендерит все его дочерние компоненты.** Поэтому, когда `ProductPage` ре-рендерится с другой `theme`, компонент `ShippingForm` *тоже* ре-рендерится. Это нормально для компонентов, которые не требуют тяжелых вычислений. Но если вы заметили, что ре-рендеринг долгий, вы можете сделать так, чтобы `ShippingForm` пропускал ре-рендеринг, если его пропсы не изменились, обернув его в [`memo`:](/reference/react/memo)

```js {3,5}
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

**После такого изменения `ShippingForm` будет пропускать ре-рендеринг, если его пропсы *не изменились* с последнего рендера.** Это тот момент, когда кэширование становится важным! Допустим, мы объявили `handleSubmit` без `useCallback`:

```js {2,3,8,12-13}
function ProductPage({ productId, referrer, theme }) {
  // Every time the theme changes, this will be a different function...
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }
  
  return (
    <div className={theme}>
      {/* ... so ShippingForm's props will never be the same, and it will re-render every time */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

**В JavaScript, выражение `function () {}` или `() => {}` всегда создают _новую_ функцию,** также как объявление объекта через литерал `{}` всегда создает новый объект. Обычно это не является проблемой, но это значит, что пропсы `ShippingForm` никогда не будут одинаковыми и ваша оптимизация через [`memo`](/reference/react/memo) не будет работать. Здесь нам придет на помощь `useCallback`:

```js {2,3,8,12-13}
function ProductPage({ productId, referrer, theme }) {
  // Говорим React кэшировать функцию между ре-рендерами...
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ...если зависимости не меняются...

  return (
    <div className={theme}>
      {/* ...ShippingForm будет получать одни и те же пропсы и может пропускать ре-рендеринг */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

**Оборачивая `handleSubmit` в `useCallback`, вы гарантируете, что это *та же* функция между ре-рендерами** (пока не изменятся зависимости). *Не нужно* оборачивать функцию в `useCallback` без причины. В этом примере, причиной является то, что вы передаете ее внутрь компонента, обернутого в [`memo`,](/reference/react/memo) и это позволяет пропускать ре-рендеринг. Иные причины, по которым вам может понадобиться `useCallback` описаны далее на этой странице.

<Note>

**Вы должны полагаться на `useCallback` только как на оптимизацию производительности.** Если ваш код не работает без него, сначала найдите причину и исправьте её. После этого можно добавлять `useCallback` обратно.

</Note>

<DeepDive>

#### Как связаны useCallback и useMemo? {/*how-is-usecallback-related-to-usememo*/}

Вы будете часто видеть [`useMemo`](/reference/react/useMemo) рядом с `useCallback`. Они оба полезны, когда вы оптимизируете дочерний компонент. Они позволяют [мемоизировать](https://en.wikipedia.org/wiki/Memoization) (или, другими словами, закэшировать) что-либо, что вы передаете вниз по дереву:

```js {6-8,10-15,19}
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  const requirements = useMemo(() => { // Вызывает вашу функцию и кэширует результат
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback((orderDetails) => { // Кэширует саму функцию
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

Разница в том, *что* они позволяют кэшировать:

* **[`useMemo`](/reference/react/useMemo) кэширует *результат* вызова вашей функции.** В этом примере оно кэширует результат вызова `computeRequirements(product)`, что он не меняется, пока не изменится `product`. Это позволяет вам передавать объект `requirements` ниже по дереву без ненужного ре-рендеринга `ShippingForm`. Когда нужно, React вызовет функцию, которую вы передали в рендере, чтобы посчитать результат.
* **`useCallback` кэширует *саму функцию.*** В отличие от `useMemo`, он не вызывает функцию, которую вы передали. Вместо этого он кэширует функцию, чтобы `handleSubmit` *сам* не менялся, если `productId` или `referrer` не изменились. Это позволяет передавать функцию `handleSubmit` ниже по дереву без ненужного ре-рендеринга `ShippingForm`. Ваш код не будет выполняться, пока пользователь не отправит форму.

Если вы уже знакомы с [`useMemo`,](/reference/react/useMemo) вам может быть проще понять `useCallback` как:

```js
// Упрощенная реализация (внутри React)
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```

[Читать больше про разницу между `useMemo` и `useCallback`.](/reference/react/useMemo#memoizing-a-function)

</DeepDive>

<DeepDive>

#### Нужно ли везде добавлять useCallback? {/*should-you-add-usecallback-everywhere*/}

Если ваше приложение похоже на этот сайт и большинство взаимодействий простые, (например, замена страницы или целой секции), обычно мемоизация не нужна. С другой стороны, если ваше приложение больше похоже на редактор для рисования и большинство взаимодействий гранулярные (как перемещение форм), то мемоизация может вам помочь. 

Кэширование функции через `useCallback` приносит пользу в следующих случаях:

- Вы передаете ее как пропс в компонент, обернутый в [`memo`.](/reference/react/memo) Вы хотите пропустить ре-рендеринг, если значение не изменилось. Мемоизация позволяет вашему компоненту ре-рендерится только в том случае, если зависимости поменялись.
- Функция, которую вы передаете, после используется как зависимость в другом хуке. Например, другая функция, обернутая в `useCallback`, зависит от нее или вы используете эту функцию в [`useEffect.`](/reference/react/useEffect)

В иных случаях, оборачивание функции в `useCallback` не принесет пользы. Это также и не принесет вреда, поэтому некоторые команды не думают о таких индивидуальных случаях и мемоизируют всё. Минус -- код становится менее читаемым. К тому же, не вся мемоизация эффективна: одно значение, которое будет "всегда новым" может сломать мемоизацию всего компонента.

Обратите внимание, что `useCallback` не препятствует *созданию* функции. Вы всегда создаёте функцию (и это нормально!), но React это игнорирует и отдает закэшированную функцию, если зависимости не изменились.

**Вы можете сделать так, чтобы мемоизация не требовалась, если будете следовать следующим принципам:**

1. Если компонент визуально является обёрткой для других компонентов, пускай он [принимает JSX как дочерний компонент.](/learn/passing-props-to-a-component#passing-jsx-as-children) Тогда, если компонент-обёртка обновит своё состояние, React будет знать, что его дочерние компоненты ре-рендерить не нужно.
1. Используйте локальное состояние и не [поднимайте его](/learn/sharing-state-between-components) выше, чем это требуется. Не сохраняйте быстро изменяющиеся состояния формы или факт ховера какого-либо элемента на самом верху дерева или в глобальном состоянии.
1. Сохраняйте [логику рендера чистой.](/learn/keeping-components-pure) Если ре-рендер компонента вызывает проблему или создаёт какой-то заметный визуальный артефакт, то в вашем компоненте баг! Поправьте баг вместо того, чтобы добавлять мемоизацию.
1. Избегайте [ненужных эффектов, которые обновляют состояние.](/learn/you-might-not-need-an-effect) Большинство проблем с производительностью в React-приложениях связано с цепочками обновлений, которые возникают из эффектов и заставляют ваши компоненты ре-рендериться.
1. Попробуйте [удалить ненужные зависимости из ваших эффектов.](/learn/removing-effect-dependencies) Например, вместо мемоизации обычно проще переместить объект или функцию внутрь эффекта или вынести из компонента.

Если какое-то взаимодействие кажется тормозящим, [используйте профайлер из инструментов разработчика React](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) чтобы оценить, какие компоненты получают пользу от мемоизации и добавьте её там, где она нужна. Эти принципы делают ваши компоненты более простыми для отладки и понимания, поэтому им хорошо всегда следовать. В долгосрочной перспективе, мы изучаем возможность [автоматической мемоизации](https://www.youtube.com/watch?v=lGEMwh32soc), чтобы решить эту проблему раз и навсегда.

</DeepDive>

<Recipes titleText="Разница между useCallback и прямым объявлением функции" titleId="examples-rerendering">

#### Пропуск ре-рендера с помощью `useCallback` и `memo` {/*skipping-re-rendering-with-usecallback-and-memo*/}

В этом примере компонент `ShippingForm` **искусственно замедлен**, чтобы вы увидели, что происходит, когда рендеринг React-компонента действительно медленный. Попробуйте увеличить счётчик и переключить тему.

Увеличение счётчика кажется медленным, так как это заставляет замедленный `ShippingForm` ре-рендериться. Это ожидаемо, потому что счётчик поменялся и нужно отобразить новый выбор пользователя на экране.

Теперь попробуйте переключить тему. **Благодаря `useCallback` вместе с [`memo`](/reference/react/memo), это быстро, несмотря на искусственное замедление!** `ShippingForm` пропустил ре-рендер, потому что функция `handleSubmit` не изменилась. `handleSubmit` не изменилась, потому что ни `productId` ни `referrer` (зависимости `useCallback`) не изменились с момента последнего рендера.

<Sandpack>

```js App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js ProductPage.js active
import { useCallback } from 'react';
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // Представим, что этот код отправляет запрос...
  console.log('POST /' + url);
  console.log(data);
}
```

```js ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('[ARTIFICIALLY SLOW] Rendering <ShippingForm />');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Ничего не делаем 500 мс, чтобы сэмулировать очень медленный код
  }

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <p><b>Note: <code>ShippingForm</code> is artificially slowed down!</b></p>
      <label>
        Number of items:
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        Street:
        <input name="street" />
      </label>
      <label>
        City:
        <input name="city" />
      </label>
      <label>
        Postal code:
        <input name="zipCode" />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

<Solution />

#### Ре-рендерим компонент всегда {/*always-re-rendering-a-component*/}

В этом примере реализация `ShippingForm` тоже **искусственно замедлена**, чтобы вы увидели, что происходит, когда рендеринг React-компонента действительно медленный. Попробуйте увеличить счётчик и переключить тему.

В отличие от предыдущего примера переключение темы здесь также медленное! Это потому что **в этой версии нет вызова `useCallback`,** поэтому `handleSubmit` всегда является новой функцией и замедленный компонент `ShippingForm` не может пропустить ре-рендеринг.

<Sandpack>

```js App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js ProductPage.js active
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // Представим, что этот код отправляет запрос...
  console.log('POST /' + url);
  console.log(data);
}
```

```js ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('[ARTIFICIALLY SLOW] Rendering <ShippingForm />');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Ничего не делаем 500 мс, чтобы сэмулировать очень медленный код
  }

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <p><b>Note: <code>ShippingForm</code> is artificially slowed down!</b></p>
      <label>
        Number of items:
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        Street:
        <input name="street" />
      </label>
      <label>
        City:
        <input name="city" />
      </label>
      <label>
        Postal code:
        <input name="zipCode" />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>


А вот тот же код **без искусственного замедления.** Чувствуется отсутствие `useCallback` или нет?

<Sandpack>

```js App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js ProductPage.js active
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // Представим, что этот код отправляет запрос...
  console.log('POST /' + url);
  console.log(data);
}
```

```js ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('Rendering <ShippingForm />');

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Number of items:
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        Street:
        <input name="street" />
      </label>
      <label>
        City:
        <input name="city" />
      </label>
      <label>
        Postal code:
        <input name="zipCode" />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>


Очень часто код работает хорошо и без мемоизации. Если взаимодействие быстрое, то мемоизация вам не нужна.

Обратите внимание, что вам нужно запустить React в продакшен-режиме, отключить [инструменты разработчика React](/learn/react-developer-tools) и использовать устройства, которые похожи на устройства ваших пользователей, чтобы получить реалистичное представление о том, что действительно замедляет ваше приложение.

<Solution />

</Recipes>

---

### Обновление состояния из мемоизированного коллбэка {/*updating-state-from-a-memoized-callback*/}

Иногда нужно обновить состояние в зависимости от предыдущего, из мемоизированного коллбэка.

Эта функция `handleAddTodo` определяет `todos` как зависимости, потому что она вычисляет следующие todos из них:

```js {6,7}
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
  // ...
```

You'll usually want memoized functions to have as few dependencies as possible. When you read some state only to calculate the next state, you can remove that dependency by passing an [updater function](/reference/react/useState#updating-state-based-on-the-previous-state) instead:

```js {6,7}
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // ✅ No need for the todos dependency
  // ...
```

Here, instead of making `todos` a dependency and reading it inside, you pass an instruction about *how* to update the state (`todos => [...todos, newTodo]`) to React. [Read more about updater functions.](/reference/react/useState#updating-state-based-on-the-previous-state)

---

### Preventing an Effect from firing too often {/*preventing-an-effect-from-firing-too-often*/}

Sometimes, you might want to call a function from inside an [Effect:](/learn/synchronizing-with-effects)

```js {4-9,12}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    // ...
```

This creates a problem. [Every reactive value must be declared as a dependency of your Effect.](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) However, if you declare `createOptions` as a dependency, it will cause your Effect to constantly reconnect to the chat room:


```js {6}
  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🔴 Problem: This dependency changes on every render
  // ...
```

To solve this, you can wrap the function you need to call from an Effect into `useCallback`:

```js {4-9,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ Only changes when roomId changes

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // ✅ Only changes when createOptions changes
  // ...
```

This ensures that the `createOptions` function is the same between re-renders if the `roomId` is the same. **However, it's even better to remove the need for a function dependency.** Move your function *inside* the Effect:

```js {5-10,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() { // ✅ No need for useCallback or function dependencies!
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Only changes when roomId changes
  // ...
```

Now your code is simpler and doesn't need `useCallback`. [Learn more about removing Effect dependencies.](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)

---

### Optimizing a custom Hook {/*optimizing-a-custom-hook*/}

If you're writing a [custom Hook,](/learn/reusing-logic-with-custom-hooks) it's recommended to wrap any functions that it returns into `useCallback`:

```js {4-6,8-10}
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```

This ensures that the consumers of your Hook can optimize their own code when needed.

---

## Troubleshooting {/*troubleshooting*/}

### Every time my component renders, `useCallback` returns a different function {/*every-time-my-component-renders-usecallback-returns-a-different-function*/}

Make sure you've specified the dependency array as a second argument!

If you forget the dependency array, `useCallback` will return a new function every time:

```js {7}
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 🔴 Returns a new function every time: no dependency array
  // ...
```

This is the corrected version passing the dependency array as a second argument:

```js {7}
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ✅ Does not return a new function unnecessarily
  // ...
```

If this doesn't help, then the problem is that at least one of your dependencies is different from the previous render. You can debug this problem by manually logging your dependencies to the console:

```js {5}
  const handleSubmit = useCallback((orderDetails) => {
    // ..
  }, [productId, referrer]);

  console.log([productId, referrer]);
```

You can then right-click on the arrays from different re-renders in the console and select "Store as a global variable" for both of them. Assuming the first one got saved as `temp1` and the second one got saved as `temp2`, you can then use the browser console to check whether each dependency in both arrays is the same:

```js
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... and so on for every dependency ...
```

When you find which dependency is breaking memoization, either find a way to remove it, or [memoize it as well.](/reference/react/useMemo#memoizing-a-dependency-of-another-hook)

---

### I need to call `useCallback` for each list item in a loop, but it's not allowed {/*i-need-to-call-usememo-for-each-list-item-in-a-loop-but-its-not-allowed*/}

Suppose the `Chart` component is wrapped in [`memo`](/reference/react/memo). You want to skip re-rendering every `Chart` in the list when the `ReportList` component re-renders. However, you can't call `useCallback` in a loop:

```js {5-14}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 You can't call useCallback in a loop like this:
        const handleClick = useCallback(() => {
          sendReport(item)
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}
```

Instead, extract a component for an individual item, and put `useCallback` there:

```js {5,12-21}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // ✅ Call useCallback at the top level:
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}
```

Alternatively, you could remove `useCallback` in the last snippet and instead wrap `Report` itself in [`memo`.](/reference/react/memo) If the `item` prop does not change, `Report` will skip re-rendering, so `Chart` will skip re-rendering too:

```js {5,6-8,15}
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  function handleClick() {
    sendReport(item);
  }

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
});
```
