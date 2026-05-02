
### `core_window.js` (`module:'core_window'`, `surface:'core'`)

| Где                        | code                                     | ctx.key                       | ctx.stage                  | ctx.type                         | policy                   | throw/`skip`                                     | `rollback`                                           | soft-fail/fail-fast                         |
| -------------------------- | ---------------------------------------- | ----------------------------- | -------------------------- | -------------------------------- | ------------------------ | ---------------------------------------------- | -------------------------------------------------- | ------------------------------------------- |
| `safeDefine` define failed | `core_window:safeDefine:define_failed`   | `<prop>`                      | `guard`                    | `browser structure missing data` | n/a                      | throw                                          | нет                                                | fail-fast (инициализация core)              |
| toString bridge invariant  | `core_window:toString:*`                 | `Function.prototype.toString` | `contract`                 | `browser structure missing data` | n/a                      | throw                                          | частично (restore paths)                           | fail-fast (core инварианты)                 |
| applyTargets fail-path     | `<planItem.tag>:<fail_code>`             | `<planItem.key>`              | `preflight/apply/contract` | по причине                       | `planItem.policy`        | `throw` если policy=throw/strict иначе `skip`    | да (группа plans)                                  | policy-dependent                            |
| hooksPost failed           | `<planItem.tag>:hooksPost_failed`        | `<planItem.key>`              | `hook`                     | `pipeline missing data`          | `planItem.policy`        | зависит от policy, но наружу = original result | да (на уровне результата, дескриптор уже применён) | soft-fail (fallback к `out`)                |
| promise contract failed    | `<planItem.tag>:promise_contract_failed` | `<planItem.key>`              | `contract`                 | `pipeline missing data`          | `throw` (внутри wrapper) | throw                                          | нет                                                | fail-fast (нарушен контракт promise_method) |

### `context.js` (`module:'context'`, `surface:'canvas'`)

| Где                            | code                                   | ctx.key      | ctx.stage | ctx.type                | policy | throw/`skip` | `rollback` | soft-fail/fail-fast      |
| ------------------------------ | -------------------------------------- | ------------ | --------- | ----------------------- | ------ | ---------- | -------- | ------------------------ |
| ctx2d hook failed              | `context:getContext:ctx2d_hook_failed` | `hook.name`  | `hook`    | `pipeline missing data` | n/a    | skip       | n/a      | soft-fail (оставить ctx) |
| webgl hook failed              | `context:getContext:webgl_hook_failed` | `hook.name`  | `hook`    | `pipeline missing data` | n/a    | skip       | n/a      | soft-fail                |
| html hook failed               | `context:getContext:html_hook_failed`  | `hook.name`  | `hook`    | `pipeline missing data` | n/a    | skip       | n/a      | soft-fail                |
| chain failed                   | `context:getContext:chain_failed`      | `getContext` | `hook`    | по причине              | n/a    | skip       | n/a      | soft-fail                |
| silent-catch в 2D прокси-хуках | `context:ctx2d:<method>_hook_failed`   | `<method>`   | `hook`    | `pipeline missing data` | n/a    | skip       | n/a      | soft-fail                |

### `nav_total_set.js` (`module:'nav_total_set'`, `surface:'navigator'`)

| Где              | code                                | ctx.key   | ctx.stage   | ctx.type                         | policy      | throw/`skip` | `rollback`    | soft-fail/fail-fast |
| ---------------- | ----------------------------------- | --------- | ----------- | -------------------------------- | ----------- | ---------- | ----------- | ------------------- |
| Core missing     | `<groupTag>:core_missing`           | `<key>`   | `preflight` | `pipeline missing data`          | groupPolicy | throw/skip | n/a         | policy-dependent    |
| preflight_failed | `<groupTag>:preflight_failed`       | `<key>`   | `preflight` | `pipeline missing data`          | groupPolicy | throw/skip | n/a         | policy-dependent    |
| group_skipped    | `<groupTag>:group_skipped` (reason) | `<key>`   | `preflight` | `pipeline missing data`          | groupPolicy | throw/skip | n/a         | policy-dependent    |
| apply_failed     | `<groupTag>:apply_failed`           | `<key>`   | `apply`     | по причине                       | groupPolicy | throw/skip | да          | policy-dependent    |
| `rollback`_failed  | `<groupTag>:rollback_failed`        | `<p.key>` | `rollback`  | `browser structure missing data` | groupPolicy | throw/skip | best-effort | policy-dependent    |

### `screen.js` (`module:'screen'`, `surface:'screen'`)

| Где                           | code                                   | ctx.key         | ctx.stage                  | ctx.type                         | policy      | throw/skip | `rollback` | soft-fail/fail-fast           |
| ----------------------------- | -------------------------------------- | --------------- | -------------------------- | -------------------------------- | ----------- | ---------- | -------- | ----------------------------- |
| applyCoreTargetsGroup fail    | `<groupTag>:<event>`                   | `<p.key>`       | `preflight/apply/rollback` | по причине                       | groupPolicy | throw/skip | да       | policy-dependent              |
| redefine failed (orientation) | `screen:orientation_*_redefine_failed` | `orientation.*` | `apply`                    | `browser structure missing data` | n/a         | skip       | n/a      | soft-fail (оставить `native`) |

### `font_module.js` (`module:'fonts'`, `surface:'fonts'`)

| Где                     | code                                 | ctx.key         | ctx.stage   | ctx.type                | policy      | throw/skip | `rollback`    | soft-fail/fail-fast |
| ----------------------- | ------------------------------------ | --------------- | ----------- | ----------------------- | ----------- | ---------- | ----------- | ------------------- |
| target preflight failed | `<groupTag>:target_preflight_failed` | `<target.key>`  | `preflight` | `pipeline missing data` | groupPolicy | throw/skip | n/a         | policy-dependent    |
| group_skipped           | `<groupTag>:group_skipped`           | n/a             | `preflight` | `pipeline missing data` | skip        | skip       | n/a         | soft-fail           |
| apply_failed            | `<groupTag>:apply_failed`            | `<key>`         | `apply`     | по причине              | groupPolicy | throw/skip | best-effort | policy-dependent    |
| dispatch_failed         | `fonts:event:dispatch_failed`        | `dispatchEvent` | `runtime`   | `pipeline missing data` | n/a         | skip       | n/a         | soft-fail           |

### `canvas.js` (`module:'canvas'`, `surface:'canvas'`)

| Где                       | code                               | ctx.key         | ctx.stage | ctx.type                | policy | throw/skip | `rollback` | soft-fail/fail-fast           |
| ------------------------- | ---------------------------------- | --------------- | --------- | ----------------------- | ------ | ---------- | -------- | ----------------------------- |
| toBlob hook failed        | `canvas:toBlob:hook_failed`        | `toBlob`        | `hook`    | `pipeline missing data` | n/a    | skip       | n/a      | soft-fail (pass-through blob) |
| convertToBlob hook failed | `canvas:convertToBlob:hook_failed` | `convertToBlob` | `hook`    | `pipeline missing data` | n/a    | skip       | n/a      | soft-fail                     |

### `webgl.js` (`module:'webgl'`, `surface:'webgl'`)

| Где                       | code                                           | ctx.key        | ctx.stage | ctx.type                         | policy | throw/skip | `rollback` | soft-fail/fail-fast         |
| ------------------------- | ---------------------------------------------- | -------------- | --------- | -------------------------------- | ------ | ---------- | -------- | --------------------------- |
| native getExtension throw | `webgl:getExtension:debug_renderer_info_throw` | `getExtension` | `runtime` | `browser structure missing data` | n/a    | skip       | n/a      | soft-fail (dbg=null)        |
| whitelist miss            | `webgl:param_whitelist_miss`                   | `getParameter` | `guard`   | `pipeline missing data`          | n/a    | skip       | n/a      | soft-fail (pass-through)    |
| shaderSource hook error   | `webgl:shaderSourceHook:error`                 | `shaderSource` | `hook`    | `pipeline missing data`          | n/a    | skip       | n/a      | soft-fail (pass-through)    |
| hooks define failed       | `webgl:webglHooks:define_failed`               | `webglHooks`   | `apply`   | `browser structure missing data` | n/a    | skip       | n/a      | soft-fail (fallback object) |

### `webgpu.js` (`module:'webgpu'`, `surface:'webgpu'`)

| Где                     | code                 | ctx.key                            | ctx.stage                  | ctx.type                         | policy      | throw/skip             | `rollback` | soft-fail/fail-fast      |
| ----------------------- | -------------------- | ---------------------------------- | -------------------------- | -------------------------------- | ----------- | ---------------------- | -------- | ------------------------ |
| applyTargets group fail | `<groupTag>:<event>` | `<p.key>`                          | `preflight/apply/rollback` | по причине                       | groupPolicy | throw/skip             | да       | policy-dependent         |
| promise reject (native) | `webgpu:*:rejected`  | `requestAdapter/requestDevice/...` | `runtime`                  | `browser structure missing data` | n/a         | throw (native promise) | n/a      | n/a (не меняем контракт) |

### `audiocontext.js` (`module:'audiocontext'`, `surface:'audio'`)

| Где             | code                          | ctx.key | ctx.stage | ctx.type   | policy | throw/skip | `rollback` | soft-fail/fail-fast                      |
| --------------- | ----------------------------- | ------- | --------- | ---------- | ------ | ---------- | -------- | ---------------------------------------- |
| guard noteIssue | `audiocontext:<reason>:<key>` | `<key>` | `guard`   | по причине | n/a    | skip       | n/a      | soft-fail (не патчим/оставляем `native`) |

---

### `TimezoneOverride_source.js` (`module:'tz'`, `surface:'timezone'`)

| Где                   | code                 | ctx.key             | ctx.stage   | ctx.type                         | policy | throw/skip | `rollback`           | soft-fail/fail-fast        |
| --------------------- | -------------------- | ------------------- | ----------- | -------------------------------- | ------ | ---------- | ------------------ | -------------------------- |
| missing prerequisites | `tz:missing_<field>` | `<field>`           | `preflight` | `pipeline missing data`          | throw  | throw      | n/a                | fail-fast (нельзя патчить) |
| missing engine parts  | `tz:missing_<part>`  | `<part>`            | `preflight` | `browser structure missing data` | throw  | throw      | n/a                | fail-fast                  |
| offset mismatch       | `tz:offset_mismatch` | `getTimezoneOffset` | `contract`  | `pipeline missing data`          | throw  | throw      | n/a                | fail-fast                  |
| apply failed          | `tz:apply_failed`    | `patchTimeZone`     | `apply`     | по причине                       | throw  | throw      | да (restore stack) | fail-fast с `rollback`       |

### `GeoOverride_source.js` (`module:'geo'`, `surface:'geolocation'`)

| Где                     | code                  | ctx.key            | ctx.stage        | ctx.type                | policy     | throw/skip       | rollback | soft-fail/fail-fast |
| ----------------------- | --------------------- | ------------------ | ---------------- | ----------------------- | ---------- | ---------------- | -------- | ------------------- |
| applyTargetGroup errors | `geo:methods:<event>` | `<p.key>`          | `apply/rollback` | по причине              | throw/`skip` | policy-dependent | да       | policy-dependent    |
| patched marker          | `geo:patched`         | `patchGeolocation` | `apply`          | `pipeline missing data` | n/a        | `skip`             | n/a      | soft-fact (инфо)    |

---
