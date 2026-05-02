#  Правила: Унификация логирования и эскалации ошибок


**NORMATIVE SPEC + EXAMPLES + INVENTORY + APPENDIX** для модулей pipeline.

- **[NORMATIVE]** правила обязательны, не интерпретируются “по вкусу”.
- **[EXAMPLES]** таблицы и маппинги иллюстрируют ожидаемую интеграцию и не отменяют [NORMATIVE].
- **[INVENTORY]** фиксирует текущее состояние вызовов `__DEGRADE__` в коде; не является нормативом и не гарантирует runtime-порядок событий.
- **[APPENDIX]** определяет обязательное приложение-шаблон для единообразного оформления примеров и таблиц.

## Быстрые ссылки (состав документа)

- APPENDIX (таблицы/шаблоны маппинга выходов): `Samples4Context\Contracts_mandaratory\DEGRADE_3.0\APPENDIX_MODULE_CODE_TEMPLATE.md`
- INVENTORY (вызовы `__DEGRADE__`): `Samples4Context\Contracts_mandaratory\DEGRADE_CALLS_SUMMARY.md`
- Guard (Core/client): `Samples4Context\Contracts_mandaratory\5._GuardFlagSEED.md`

[NORMATIVE]: #normative
[EXAMPLES]:  #examples
[INVENTORY]: #inventory
[APPENDIX]:  #appendix


---
## [NORMATIVE] Термины и определения

  1. **Единый канал логирования .diag**: `CanvasPatchContext.__logger.__DEGRADE__?.diag?.(level, code, ctx, err)` (далее - `.diag`)

  2. **Skip патча**: дескриптор не меняется (или группа возвращает `group_skipped`), наружу остаётся исходное поведение (далее - `skip`)
  3. **Pass-through**: внутри hook вернуть “не трогаем” и вызвать исходную реализацию по месту через `Reflect.apply(...)`.
  4. **Rollback**: если уже применили часть группы, откатываем дескрипторы и оставляем исходное поведение (далее - `rollback`)

  5. **Fail-fast** + `.diag`  (обязательно)
    - Отсутствие обязательных точек (`Core`, bridge, обязательные descriptors, критичный контракт Promise/brand).
    - Нарушение целостности состояния после preflight/post-check.

  6. **Soft-fail** (допустимо только при всех 4 условиях)
    1. Безопасный fallback (возврат к исходному состоянию по месту).
    2. Нет частичного состояния (`rollback`/`skip` целиком).
    3. Есть наблюдаемость (единый `.diag`  канал).
    4. Проверено, что можно продолжать (инварианты выдержаны).

  7. **Возврат к исходноему состоянию**  реализуется как одно из:
        **Skip патча**/ **Pass-through** / **Rollback** 

---

## Цель

Унификация логирования и эскалации ошибок/диагностики через `.diag` без утечек в runtime логирования/диагностикм посредством явно описанной матрицы:

- `policy` (как задано в target/group)
- `throw/skip` (что реально происходит при ошибке)
- `rollback` (есть ли откат и когда)
- `soft-fail/fail-fast` (смысловая стратегия)

---

### Решение

### Ликвидировать Catch-блоки:

Любой `catch` MUST NOT быть пустым: он MUST записать причину через `__DEGRADE__` и затем выполнить атомарный `skip`/`rollback` (или fail-fast для обязательного шага пайплайна по `policy`).

- ❌ Silent-swallow (молчаливые catch)
- ❌ Разрозненную наблюдаемость в модуле
- ❌ События теряются "молчаливыми catch"
- ❌ Любой несанкционированный Вывод в `console.*` 
- ❌ Вывод без маршрутизации `__DEGRADE__`

---


## Ограничения

### Основной контракт

- ✅ Строго следовать правилам Внешнего нормативного контракта по реализации `object/function/proxy/property/kinds`
[Policy_implement_reg.md](file:///c:/55555/switch/Evensteam/Samples4Context/Contracts_mandaratory/1._Policy_implement_reg.md)
- ✅ Выпополнять требования Hidden state-контракта `CanvasPatchContext`
[Hidden_State_CanvasPatchContext_Contract.md](file:///c:/55555/switch/Evensteam/Samples4Context/Contracts_mandaratory/2._Hidden_State_CanvasPatchContext_Contract)
- ❌ Никакого архитектурного рефакторинга
- ❌ Никаких новых абстракций
- ❌ Никаких изменений сигнатур функций

### Запрещено менять

| Что                                                                                                                                 | Почему                                                                                                                                                      |
| ----------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Сигнатуры функций                                                                                                                   | Контракты должны остаться неизменными                                                                                                                       |
| Новые экспорты/глобалы                                                                                                              | Не добавлять новую поверхность                                                                                                                              |
| Bootstrap/Pipeline объекты (window.**, window.rand.\*, window.CanvasPatchContext, **ensureMarkAsNative, **DEGRADE**, реестры/мосты) | Не являются Browser APIs Chromium, не имеют движкового веб-контракта. Только мягкий preflight-контроль наличия (`skip`+`.diag` при отсутствии), без API-контролей |

### Разрешённый тип изменений

✅ Разрешены только “редукционные” правки, которые:

- не меняют поведение (кроме устранения утечек ошибок/унификации `.diag`),
- не меняют порядок и точки входа,
- не меняют сигнатуры,
- не добавляют новые абстракции.

✅ То есть: допустимы только точечные правки, которые строго приводят к:

- единственному пути **DEGRADE**.diag,
- единственному формату ctx,
- устранению служебных throw наружу,

❌ Удаление/сворачивание кода как "чистка" (даже без изменения поведения) = рефакторинг и в рамках этой задачи запрещено.

---

## [NORMATIVE] Минимальный стандарт структры модуля

- `preflight`
- `guardFlag`
- `apply`
- `export entrypoints by normative defineProperty(enumerable:false)`
- `hide-after-apply` для служебных own surfaces
- `releaseGuardFlag(true)` ( Guard (Core/client): `Samples4Context/GuardFlagSEED.md`)
- при ошибке: `rollback descriptors` + `releaseGuardFlag(rollbackOk)`



---

## [NORMATIVE] Единый формат оформления функций 

В каждом модуле держим один и тот же минимальный стандарт.

1. `entrypoint` и утилиты, которые должны жить на `window`, экспортируем только по нормативу:
   `hasOwnProperty` + `Object.defineProperty(..., enumerable:false)`.

Когда функция является модульным entrypoint/утилитой, доступной через `window`, она MUST быть экспортирована идемпотентно:

1) Существование через `hasOwnProperty('__function_example__')` (чтобы не перетирать уже определённую реализацию)
2) закрепление через `Object.defineProperty(window, '__function_example__', { value: fn, writable:true, configurable:true, enumerable:false })`

Нормативная форма (устойчива к shadowing `hasOwnProperty`):

```js
const W = (typeof window !== "undefined") ? window : null;
if (W && !Object.prototype.hasOwnProperty.call(W, "__function_example__")) {
  Object.defineProperty(W, "__function_example__", {
    value: fn,
    writable: true,
    configurable: true,
    enumerable: false
  });
}
```


---

## [NORMATIVE] Норматив поведения и контракт `__DEGRADE__` 

1. - **Назначение - единый вход**: единый вход для диагностических событий
- **Не менять**:  поведение патчей и диагностики
- **Требование**: любая "служебная проблема" должна проходить через него с контекстом
2. `status/guard` ключи обрабатываем штатно через `guardFlag/releaseGuardFlag`.
   После успешного `apply`guard *НЕ снимается*. Успешный `apply` = больше не запускаться в этом document.

3. На rollback восстанавливаем именно descriptor state, а не только value, чтобы не ломать `enumerable/configurable/writable/get/set`.

---





### Единый вызов 

**Единица фиксации проблемы** — сообщение через единый канал `__DEGRADE__` aka (`.diag`)

Сообщение должно содержать:

- Контекст (`level`, `type`, `module`, `diagTag`, `surface`, `key`, `stage`, `message`, `data`)
- Выполнение продолжается с исходным состоянием после `fallback`/`skip`/`rollback` (если это не обязательный `fail-fast`)


```javascript
window.CanvasPatchContext?.__logger?.__DEGRADE__?.diag?.(level, code, ctx, err);
```

Нормативный вызов для модулей: `CanvasPatchContext.__logger.__DEGRADE__.diag(...)`.


###  `__DEGRADE__.diag(level, code, ctx, err?)`
Это non-enumerable метод на функции `__DEGRADE__`, который:

1. Валидирует `level` (`info|warn|error|fatal`) и приводит его к безопасному значению.
2. Приводит `code` к строке.
3. Гарантирует, что `ctx` это plain-object (иначе заменяет на `{}`).
4. Не нормализует `ctx.type`: если это строка, пишет как есть; если нет — оставляет `undefined`. Для missing-data классификации использовать `pipeline missing data` / `browser structure missing data`.
5. Нормализует `ctx.data` и другие поля через существующую безопасную сериализацию (обрезание DOM/host объектов, циклов, больших строк).
6. Формирует `extraObj` в едином shape и вызывает базовый контракт:

- `code`: строка (идентификатор события)
- `err`: `Error|null|any` (ошибка или контекстная причина)
- `extra`: объект контекста (shape как у `.diag`, должен быть сериализуемым)

`extra` должен следовать shape `diag`: `level`, `type`, `module`, `diagTag`, `surface`, `key`, `stage`, `message`, `data`; отсутствующие поля допустимы как `undefined|null` и нормализуются в `set_log.js`.


**Смысл:** записать единый structured-event в буфер логгера.
Логгер хранит буфер приватно (не через публичный `window.`): внутренний буфер живет внутри `set_log.js` и доступен только через `__DEGRADE__.getBuffer()`, которая возвращает копию.

---

## [NORMATIVE] Шаблон локального адаптера (`__module_name_Diag*` как единственный шлюз)

Цель:  adapter обеспечивает единый маршрут `.diag -> callable -> noop` без `Illegal invocation`, без `console.*`, без локальной пере-классификации `stage/type`.

Нормативные свойства:

- Adapter берёт `.diag.bind(__D)` (корректный method-call).
- Adapter `never-throw` и `safe-noop` при отсутствии каналов.
- Adapter не нормализует `ctx.stage/ctx.type` и не мутирует `extra/ctx`.


## Физический шаблон:

```js
const __MODULE = "module_name";
const __SURFACE = "module_name";

const __loggerRoot = (window.CanvasPatchContext && window.CanvasPatchContext.__logger && typeof window.CanvasPatchContext.__logger === "object")
  ? window.CanvasPatchContext.__logger
  : null;
const __D = (__loggerRoot && typeof __loggerRoot.__DEGRADE__ === "function")
  ? __loggerRoot.__DEGRADE__
  : null;
const __diag = (__D && typeof __D.diag === "function") ? __D.diag.bind(__D) : null;
function __emit(level, code, ctx, err) {
  try {
    if (__diag) return __diag(level, code, ctx, err);
    if (typeof __D === "function") {
      const safeCtx = (ctx && typeof ctx === "object") ? ctx : {};
      const safeLevel = (level === undefined || level === null) ? "info" : level;
      const safeErr = (err === undefined || err === null) ? null : err;
      return __D(code, safeErr, Object.assign({}, safeCtx, { level: safeLevel }));
    }
  } catch (emitErr) {
    return undefined;
  }
}

function __moduleDiag(level, code, extra, err) {
  const x = (extra && typeof extra === "object") ? extra : {};
  const ctx = {
    module: __MODULE,
    diagTag: (typeof x.diagTag === "string" && x.diagTag) ? x.diagTag : __MODULE,
    surface: __SURFACE,
    key: (typeof x.key === "string" || x.key === null) ? x.key : null,
    stage: x.stage,    // no local normalization/re-classification
    message: x.message,
    data: Object.prototype.hasOwnProperty.call(x, "data") ? x.data : null,
    type: x.type       // no local normalization/re-classification
  };
  return __emit(level, code, ctx, err);
}
```

## [NORMATIVE] Constraints (restrictions)

- Придумывать новые форматы `code` (ломает фильтрацию и сравнимость прогонов).
- Менять `ctx.stage/ctx.surface/ctx.type` на произвольные строки.
- Логировать проблемы только в `console.*` или в сторонние буферы, минуя `__DEGRADE__`.
- Подменять поведение публичных API “для удобства” (возвращать не-движковые значения, гасить движковые `TypeError/Illegal invocation`).
- Повторно считывать метод/функцию после патча вместо возврата к исходному состоянию по месту.

---


## [NORMATIVE] Нормализация терминов `ctx` 

Все нормализации (`level`, сериализация `data`, защита от host-объектов/циклов, дедуп) идут в ОДНОМ месте (`set_log.js`), а модули не создают свою нормализацию и свои форматы.

**Смысл:** модули вызывают только `.diag`, а нормализация и единый shape делаются централизованно

```js
G.__DEGRADE__(normalizedCode, err, extraObj);
```
 Нежелательно: __DEGRADE__(code, err) без extra/ctx — теряется структурный контекст события; при отсутствии данных передавать extra/ctx как null/{} по контракту.




### Единый `ctx`-shape (обязательные поля)

`ctx` MUST иметь форму `{ module, diagTag, surface, key, stage, message, type, data }`, где:

- `stage` ∈ `{ preflight, apply, rollback, contract, hook, runtime, guard }`
- `data.outcome` ∈ `{ return, skip, rollback, throw }`
- для missing-data `type` MUST быть строго `pipeline missing data` либо `browser structure missing data`


- актуальный logger entrypoint резолвится через `CanvasPatchContext.__logger.__DEGRADE__`

- `module` (строка)
- `diagTag` (строка)
- `surface` (строка)
- `key` (строка|null)
- `stage` (строка)
- `message` (строка)
- `data` (object|null; в logger безопасно сериализуется; `null` сохраняется)
- `type` (строка; для missing-data строго: `pipeline missing data` / `browser structure missing data`; для прочих причин — стабильный классификатор причины)

---




Важно: `ctx.type` классифицирует причину события, но **не задаёт** `policy` и не отменяет `throw/skip`. Приоритеты описаны в [NORMATIVE] правилах ниже.

---
---
## [NORMATIVE] Приоритеты: `policy` 

### Rule P0 (главное правило приоритета)

1. `policy` из `target/group` обязателен: если `policy:'throw'` **или** `policy:'strict'`, ошибка считается `fail-fast` **для шага патча** (патч не должен продолжаться в неопределённом состоянии).
2. **Исключение (публичные API):** если `throw` приведёт к тому, что “наружу летит служебка” из публичного API, то наружу **не выбрасываем** служебную ошибку. Вместо этого:
   - фиксируем событие через `__DEGRADE__.diag(...)`;
   - обеспечиваем атомарность (`skip`/`rollback` группы);
   - наружу пробрасываем ответ движка.

`ctx.type` и `level` всегда пишутся, но они не имеют права “переписать” `policy`.

---


## [NORMATIVE] Допустимые значения `ctx.stage` 

Строго одно из:

`preflight | apply | rollback | contract | hook | runtime | guard`

❌ Запрещено вводить новые строковые стадии “по вкусу”.

❌ Запрещена повторная локальная валидация, модуль передаёт `ctx`, а стандартизацию делает централизованно логгер.

## [NORMATIVE] Допустимые значения `ctx.surface` 

- Для `nav_total_set.js`: `ctx.surface` **всегда** `navigator` (не плодить `nav`/`ua`/`navTotal` и т.п.).
- Для остальных модулей: `ctx.surface`  = имя модуля (`webgl`, `webgpu`, `hide_webdriver`, `canvas`, `screen`, `fonts`, `rng_set`, `audio`, `timezone`, `geolocation`, `rtcp`,`core_window`).

## [NORMATIVE] Допустимые значения  `ctx.diagTag`

Чтобы не плодить самодельные теги:

- Базовый `diagTag` = имя модуля (`nav_total_set`, `webgl`, `context`, ...).
- Для групповых операций разрешено использовать **только уже существующие** значения, которые модуль/ядро уже использует (например `groupTag`, `planItem.tag`, `nav_total_set:safeDefineAcc`).

---


## [NORMATIVE]  Классификация проблем `missing data`

Строго **2 группы**:

### 1. `pipeline missing data`

Всё, что "наше":

- profile
- bridge
- meta
- seed
- ожидаемые пайплайном поля
- внутренние несостыковки данных

### 2. `browser structure missing data`

Всё, что "из движка/структуры":
---js
- descriptor
- proto
- brand
- receiver
- Illegal invocation
- unexpected exception
- и т.п.
---



## [NORMATIVE] Observed Exit Contract  - Политика обработки исключений (обязателен для patch-функций и хелперов модуля)

### Наблюдаемость выходов

Любой контролируемый выход шага патча (`return` / `skip` / `rollback` / `throw`) MUST быть зафиксирован **до выхода** через `__DEGRADE__.diag(level, code, ctx, err)`.
- Если `.diag` отсутствует: `__DEGRADE__(code, err, extra)`.
- Если отсутствуют оба канала: safe-noop (без `console.*`).
---



## [NORMATIVE] `Function.prototype.toString` (native-shaped)

Здесь цель одна: **сохранить ожидаемое поведение `Function.prototype.toString`**, не ломая специфику движка.

Норма совместимости:
- Если `Function.prototype.toString` вызван с non-function receiver, движковый `TypeError` MUST проходить как есть (тот же тип/сообщение/объект), без “нормализации” нашим wrapper-ом.

❌ Модуль MUST NOT:
  * перехватывать/патчить `Function.prototype.toString`,
  - реализовывать локальные toString-механики/мосты/нормализаторы.
  - патчить `Function.prototype.toString`,
  - вводить альтернативные “nativizer”/глобальные перемычки.


* `native behavior` MUST означать: вызвать `orig` с корректными `receiver/args` **без** изменения движковых ошибок.
* `native-shaped` MUST означать только маску для `Function.prototype.toString`; это **не** “источник оригинала” и **не** “настоящий native”.



#### Источник `orig` (запрещены “самостоятельные поиски”)

* Модуль MUST NOT получать `orig` повторным чтением `obj[prop]` после патча.
* Hook/chain-модули MUST захватывать `orig` как `const orig = proto[method]` **до** замены.
* Standalone через `Core.applyTargets` MUST получать `orig` из `desc.value/get/set` на preflight и получать его **как параметр** в `invoke(orig, args)` (модуль не ищет оригинал сам).

#### Нормативный вызов оригинала (обязательный)

* `orig` MUST быть материализован как **ссылка** на `function/get/set` **до** `defineProperty`.
* `Proxy`/`Reflect`: traps обязаны соблюдать инварианты цели; нарушения (в т.ч. на `get`/prototype paths) считаются contract violation и не маскируются.
* `Object.defineProperty`: flags/shape (`configurable/enumerable/writable/get/set/value`) должны соответствовать descriptor-owner контракту, без частичных shape-drift переходов.

**Методы**

```js
return Reflect.apply(orig, thisArg, argsArray);
```

**Геттер**

```js
return Reflect.apply(origGet, thisArg, []);
```

**Сеттер**

```js
return Reflect.apply(origSet, thisArg, [value]);
```

**Promise-returning**

```js
const out = Reflect.apply(orig, thisArg, argsArray);
// дальше — только post-hooks/then, без подмены receiver/brand
return out;
```



---

## [NORMATIVE] WebIDL / движковый throw  Engine-throw pass-through (публичный API)

Если вызов публичного API нарушает движковый контракт (валидность receiver/аргументов/состояния), проверка делегируется движку:
-  вызывается *нативная* реализация (getter/метод) с **тем же** `receiver` и **теми же** `args`; если движок бросает исключение — оно **пробрасывается без каких-либо изменений** (тот же тип/сообщение/объект) 
- модуль делает только **репорт факта** через `__DEGRADE__.diag(...)`.

Любые текущие THW/контрольные throw в геттерах/методах публичного API заменяются на:
канонический паттерн:

```js
try {
  return Reflect.apply(nativeFn, receiver, args);
} catch (e) {
   __loggerRoot.__DEGRADE__?.diag?.(/* ... */, e);
  throw e; // без обёрток/замены/служебных префиксов
}
```

---
**




---

## [EXAMPLES] Ожидаемое маппинг `code`/`ctx` по модулям 

Ниже таблицы: в каждом типовом месте фиксируем `code` + `ctx.*` + стратегию.

### [EXAMPLES] Observed Exit Contract (шаблон выхода)

```js
// Optional helpers (must not change behavior; only emit + return/rethrow).
function __throw(code, ctx, err) { __emit("error", code, ctx, err); throw err; }

// return
return __exit("info", code, { module, diagTag, surface, key, stage:"apply", message:"ok", type:"ok", data:{ outcome:"return" } }, true);

// `skip`/`rollback`
return __exit("warn", code, { module, diagTag, surface, key, stage:"preflight", message:"skipped", type:"pipeline missing data", data:{ outcome:"skip", reason:"missing_dep" } }, false);

// throw (do not use for service errors that would leak into public API)
__throw(code, { module, diagTag, surface, key, stage:"apply", message:"apply threw", type:"browser structure missing data", data:{ outcome:"throw" } }, err);
```




---

## [INVENTORY] Фактическое состояние вызовов `__DEGRADE__` (не норматив)

Инвентаризация текущих мест/порядка подключения и вызовов `__DEGRADE__` ведётся в:

- `Samples4Context/DEGRADE_CALLS_SUMMARY.md`

Это **не** норматив и **не** гарантирует runtime-порядок событий (он зависит от того, какие ветки/ошибки реально срабатывают).

Также [INVENTORY] сейчас не фиксирует соблюдение `Observed Exit Contract` (наличие `data.outcome`) и идемпотентный экспорт window-утилит (guard + `Object.defineProperty`): это проверяется отдельно при аудитах модулей.

---

## [APPENDIX] Шаблон модульного оформления

Приложение: `Samples4Context/APPENDIX_MODULE_CODE_TEMPLATE.md`.

Назначение приложения — зафиксировать единообразный формат представления примеров и таблиц (как обязательное правило оформления) без переопределения приоритетов [NORMATIVE] разделов настоящего документа.
