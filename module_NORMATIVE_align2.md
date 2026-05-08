#  Правила: Унификация логирования и эскалации ошибок


**NORMATIVE SPEC + EXAMPLES + INVENTORY + APPENDIX** для модулей pipeline.

- **[NORMATIVE]** правила обязательны, не интерпретируются “по вкусу”.
- **[EXAMPLES]** таблицы и маппинги иллюстрируют ожидаемую интеграцию и не отменяют [NORMATIVE].
- **[INVENTORY]** фиксирует текущее состояние вызовов `__DEGRADE__` в коде; не является нормативом и не гарантирует runtime-порядок событий.
- **[APPENDIX]** определяет обязательное приложение-шаблон для единообразного оформления примеров и таблиц.

## Быстрые ссылки (состав документа)

- APPENDIX (таблицы/шаблоны маппинга выходов): `Samples4Context\Contracts_mandaratory\DEGRADE_3.0\APPENDIX_MODULE_CODE_TEMPLATE.md`
- INVENTORY (вызовы `__DEGRADE__`): `Samples4Context\Contracts_mandaratory\DEGRADE_3.0\DEGRADE_CALLS_SUMMARY.md`
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




### Основной контракт

- ✅ Строго следовать правилам Внешнего нормативного контракта по реализации `object/function/proxy/property/kinds`
[Policy_implement_reg.md](file:///c:/55555/switch/Evensteam/Samples4Context/Contracts_mandaratory/1._Policy_implement_reg.md)
- ✅ Выпополнять требования Hidden state-контракта `CanvasPatchContext`
[Hidden_State_CanvasPatchContext_Contract.md](file:///c:/55555/switch/Evensteam/Samples4Context/Contracts_mandaratory/2._Hidden_State_CanvasPatchContext_Contract)
- ✅ Модульные targets/группы обязаны использовать ровно те поля и значения, `kind`  / `policy` и тд,  которые определены в реестре  `Samples4Context\Contracts_mandaratory\APPENDIX_MODULE_CODE_TEMPLATE.md`: 
- ❌ Никакого архитектурного рефакторинга
- ❌ Никаких новых абстракций
- ❌ Никаких изменений сигнатур функций


####  Code of conduct

- Любой `catch` MUST NOT быть пустым: он MUST записать причину через `__DEGRADE__` и затем выполнить атомарный `skip`/`rollback` (или fail-fast для обязательного шага по `policy`).
- ❌ Разрозненная наблюдаемость в модуле
- ❌ Любой несанкционированный Вывод в `console.*` 
- ❌ Вывод без маршрутизации `__DEGRADE__`
- ❌ Silent-swallow (События теряются "молчаливыми catch")
---

### Запрещено менять

| Что                                                                                                                                 | Почему                                                                                                                                                      |
| ----------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Сигнатуры функций                                                                                                                   | Контракты должны остаться неизменными                                                                                                                       |
| Новые экспорты/глобалы                                                                                                              | Не добавлять новую поверхность                                                                                                                              |
| Bootstrap/Pipeline объекты (window.**, window.rand.\*, window.CanvasPatchContext, **ensureMarkAsNative, **DEGRADE**, реестры/мосты) | Не являются Browser APIs Chromium, не имеют движкового веб-контракта. Только мягкий preflight-контроль наличия (`skip`+`.diag` при отсутствии), без API-контролей |


---

## [NORMATIVE] Минимальный стандарт структры модуля

- `preflight`
- `guardFlag`
- `apply`
- `export entrypoints by normative defineProperty(enumerable:false)`
- `hide-after-apply` для служебных own surfaces
- `releaseGuardFlag(true)` ( Guard (Core/client): `Samples4Context\Contracts_mandaratory\5._GuardFlagSEED.md`)
- при ошибке: `rollback descriptors` + `releaseGuardFlag(rollbackOk)`



---

## [NORMATIVE] Единый формат оформления функций 

В целом регулируется Hidden state-контрактом `CanvasPatchContext`
[Hidden_State_CanvasPatchContext_Contract.md](file:///c:/55555/switch/Evensteam/Samples4Context/Contracts_mandaratory/2._Hidden_State_CanvasPatchContext_Contract)

  В каждом модуле держим один и тот же нормативный стандарт.

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

## [INVENTORY] Фактическое состояние вызовов `__DEGRADE__` (не норматив)

Инвентаризация текущих мест/порядка подключения и вызовов `__DEGRADE__` ведётся в:

- `Samples4Context\Contracts_mandaratory\DEGRADE_CALLS_SUMMARY.md`

Это **не** норматив и **не** гарантирует runtime-порядок событий (он зависит от того, какие ветки/ошибки реально срабатывают).

Также [INVENTORY] сейчас не фиксирует соблюдение `Observed Exit Contract` (наличие `data.outcome`) и идемпотентный экспорт window-утилит (guard + `Object.defineProperty`): это проверяется отдельно при аудитах модулей.

---


