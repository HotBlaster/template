# template

Starter skeletons for new C source (`.c`) and header (`.h`) files.

Every translation unit in the project follows the same fixed section layout. The templates encode that layout so new files begin with the correct structure — banners, guards, Doxygen header — and the developer only needs to fill in the relevant sections.

## Contents

| File | Purpose |
|---|---|
| `template.h` | Public header skeleton — Doxygen file block, include guard (`#ifndef`/`#define`/`#endif`), `extern "C"` wrapper, and **7** section banners: *Include Files → Public Define → Public Macros → Public Types → Public External Variables Declaration → Public Inline Functions Definitions → Public Functions Declaration*. Closes with the `End Of File` banner. |
| `template.c` | Translation unit skeleton — Doxygen file block followed by **10** section banners: *Include Files → Private Define → Private Macros → Private Types → Private (Static) Function Declaration → External Variables Definition → Private (Static) Variables Definition → Private (Static) Inline Function Definitions → Public Functions Definition → Private (Static) Function Definition*. Closes with the `End Of File` banner. |

## How to use

1. Copy both files into the target module directory.
2. Rename them to `<module>.c` / `<module>.h`.
3. Update the include guard macro (`TEMPLATE_H_` → `<PARENT_DIR>_<FILE>_H_`). The parent directory is the folder the file lives in — e.g. `system/bsp/bsp_adc.h` → `BSP_BSP_ADC_H_`.
4. Update the `@file` field to match the new filename and write a one-line `@brief`.
5. In the `.c` file, add `#include "<module>.h"` as the very first include.
6. Fill the relevant sections. **Leave unused banners in place** — empty sections are intentional; they reserve the layout so readers always know where to look.

---

## C Coding Rules

These rules reflect the conventions used across the project. Follow them whenever you add or edit a C source or header file.

### 1  File layout

#### Doxygen file header

Every file starts with:

```c
/**
 * @file    <filename>
 * @brief   <one-line description>
 *
 */
```

#### Section banners

Sections are separated by fixed 100-column banners at column 1:

```c
/***************************************************************************************************
 *                  <Section Title>                                                                *
 ***************************************************************************************************/
```

Do not reformat, shorten, or remove them.

#### End-of-file sentinel

Every file ends with:

```c
/***************************************************************************************************
 *  End Of File                                                                                    *
 ***************************************************************************************************/
```

Never place code after this banner.

#### `.h` section order

| # | Section |
|---|---|
| 1 | Include Files |
| 2 | Public Define |
| 3 | Public Macros |
| 4 | Public Types |
| 5 | Public External Variables Declaration |
| 6 | Public Inline Functions Definitions |
| 7 | Public Functions Declaration |

#### `.c` section order

| # | Section |
|---|---|
| 1 | Include Files |
| 2 | Private Define |
| 3 | Private Macros |
| 4 | Private Types |
| 5 | Private (Static) Function Declaration |
| 6 | External Variables Definition |
| 7 | Private (Static) Variables Definition |
| 8 | Private (Static) Inline Function Definitions |
| 9 | Public Functions Definition |
| 10 | Private (Static) Function Definition |

---

### 2  Header-specific rules

- **Include guard** — `<PARENT_DIR>_<FILE>_H_` (uppercase, underscores, trailing underscore).
  `system/bsp/bsp_adc.h` → `BSP_BSP_ADC_H_`, `src/app/app.h` → `APP_H_`, `lib/common/common.h` → `COMMON_H_`.

- **C++ guard** — wraps all declarations between the include section and the closing `#endif`:

  ```c
  #ifdef __cplusplus
  extern "C"
  {
  #endif /* __cplusplus */

  /* ... declarations ... */

  #ifdef __cplusplus
  }
  #endif /* __cplusplus */
  ```

- **Doxygen on declarations** — public APIs carry `@brief` / `@param` / `@return` comments in the **header**, not the `.c` definition:

  ```c
  /**
   * @brief  Return the last captured raw 12-bit ADC value for a channel.
   * @param  id     Channel identifier (BSP_ADC_ID_JOY0_X … BSP_ADC_ID_JOY3_Y).
   * @param  value  Output pointer — receives the raw 12-bit result (0–4095).
   * @return RES_OK, RES_ERROR_UNINITIALIZED, or RES_ERROR_INVALID_PARAMETER.
   */
  res_t Bsp_Adc_GetValue(bsp_adc_id_t id, uint16_t *value);
  ```

- **Inline field docs** — struct / enum members use `/*!< ... */` on the same line:

  ```c
  typedef struct
  {
      ADC_TypeDef    *adc;        /*!< ADC instance (ADC1 or ADC2)         */
      GPIO_TypeDef   *port;       /*!< GPIO port for the analog pin         */
      uint32_t        pin;        /*!< LL_GPIO_PIN_x                        */
  } bsp_adc_cfg_t;
  ```

---

### 3  Include conventions

| Priority | Kind | Syntax | Example |
|---|---|---|---|
| 1 | Own header | double quotes | `#include "bsp_adc.h"` |
| 2 | Same-module / sibling headers | double quotes | `#include "bsp_hwconfig.h"` |
| 3 | Project headers | angle brackets with folder prefix | `#include <common/common.h>`, `#include <bsp/bsp_uart.h>`, `#include <drv/stm32g4xx_ll_gpio.h>`, `#include <cli/cli.h>`, `#include <prj_cfg.h>` |
| 4 | Standard library | angle brackets | `#include <string.h>`, `#include <stdint.h>`, `#include <stdio.h>` |

The **own-header-first** rule is mandatory — it proves the header is self-contained. CMake adds `cfg/`, `src/`, `system/`, and `lib/` as include roots, so paths inside angle brackets omit those prefixes.

---

### 4  Naming conventions

| Symbol kind | Style | Examples |
|---|---|---|
| Public function | `PascalCase` with module prefix | `Bsp_Adc_Init`, `App_Fast`, `AppCli_GetResString` |
| Private (`static`) function | Same `PascalCase` with module prefix | `Bsp_Adc_InitInstance`, `AppCli_Do_Init`, `Bsp_Uart_InitHandle` |
| Type (`typedef`) | `snake_case` with `_t` suffix | `bsp_adc_id_t`, `app_state_t`, `res_t`, `bsp_flash_op_t` |
| Macro / `#define` | `UPPER_SNAKE_CASE` with module prefix | `BSP_ADC_POLL_TIMEOUT`, `APP_TASK_1MS_TIMEOUT`, `BSP_FLASH_KEY1` |
| Enum constant | `UPPER_SNAKE_CASE` with module prefix | `BSP_ADC_ID_JOY0_X`, `APP_STATE_RUNNING`, `BSP_FLASH_OP_WRITE` |
| Static module variable | `camelCase`, **no** module prefix | `bspAdc`, `app`, `bspDo`, `bspFlash` |
| Global (externally visible) variable | `camelCase`; when the value carries a measurement unit, append `_unit` | `systick_ms` |

---

### 5  Enumerations

- Enum values carry explicit unsigned integer literals.
- The first value is always explicitly assigned.
- **Countable / indexing enums** (used for array sizing or range checks) end with a
  `_NUM` sentinel:

```c
typedef enum
{
    BSP_ADC_ID_JOY0_X  = 0U,
    BSP_ADC_ID_JOY0_Y  = 1U,
    /* ... */
    BSP_ADC_ID_NUM     = 8U
} bsp_adc_id_t;
```

- **State-machine enums** and **bitmask enums** do **not** require a `_NUM` sentinel.
  However, a `_NUM` sentinel **is permitted** when the enum is also used as an
  array dimension (e.g. for lookup tables indexed by state):

```c
typedef enum
{
    NVMMGR_STATE_NOTINIT  = 0U,
    NVMMGR_STATE_IDLE     = 1U,
    NVMMGR_STATE_READING  = 2U,
    NVMMGR_STATE_WRITING  = 3U,
    NVMMGR_STATE_ERROR    = 4U
} nvmmgr_state_t;

typedef enum
{
    NVMMGR_PRTSTATEBM_DATAREADY        = 0x01U,
    NVMMGR_PRTSTATEBM_UNCOMMITTEDDATA  = 0x02U,
    NVMMGR_PRTSTATEBM_WRITEERROR       = 0x04U
} nvmmgr_prtstatusbm_t;
```

---

### 6  Types and constants

- `#define` values are **parenthesised**: `#define LED_USER_PIN (LL_GPIO_PIN_8)`.
- Integer literals carry **explicit suffixes**: `0U`, `1UL`, `50000UL`, `100U`. Never rely on implicit signedness.
- Booleans use `true` / `false` (from `<stdbool.h>`) or the project alias `bool_t` from `common.h`.
- Project-wide type aliases from `common.h`: `bool_t`, `char_t`, `float32_t`, `float64_t`, `func_ptr_t`, `res_t`.

---

### 7  Module state pattern

Each module concentrates all mutable state in **one** static struct variable:

```c
static app_t     app;
static bsp_adc_t bspAdc;
static bsp_do_t  bspDo;
```

Rules:
- The state struct contains at least an `init` flag (`bool_t init;`) when the module has an `Init()` function.
- Configuration tables are `static const <cfg_t> name[N]` arrays, indexed by the module's `_id_t` enum:
  ```c
  static const bsp_adc_cfg_t bspAdcCfg[BSP_ADC_ID_NUM] = { ... };
  ```
- `Init()` zeroes state with `memset(&<state>, 0U, sizeof(<state>_t));` before populating it.
- Designated initializers (`.field = value`) are used for complex static configuration:
  ```c
  static bsp_adc_inst_t bspAdcInst[BSP_ADC_INST_NUM] =
  {
      [BSP_ADC_INST_ADC1] =
      {
          .adc        = ADC1,
          .dma        = DMA1,
          .dmaChannel = LL_DMA_CHANNEL_1,
          /* ... */
      },
  };
  ```

---

### 8  Function contracts

#### Return type

- Functions that can fail return `res_t` (defined in `common.h`).
  - Positive values = success/status: `RES_OK` (0), `RES_ALREADY_INITIALIZED`, `RES_BUFFER_FULL`, `RES_OPERATION_IN_PROGRESS`, …
  - Negative values = errors: `RES_ERROR`, `RES_ERROR_INVALID_PARAMETER`, `RES_ERROR_UNINITIALIZED`, `RES_ERROR_TIMEOUT`, `RES_ERROR_BUSY`, …
- Return `void` only for operations that truly cannot fail.

#### Single return point

Accumulate the result in a local `res_t res;` and `return res;` at the end:

```c
res_t Bsp_Do_Set(bsp_do_id_t id, bool_t value)
{
    res_t res = RES_ERROR;
    if (false == bspDo.init)
    {
        res = RES_ERROR_UNINITIALIZED;
    }
    else if (id >= BSP_DO_ID_NUM)
    {
        res = RES_ERROR_INVALID_PARAMETER;
    }
    else
    {
        /* ... work ... */
        res = RES_OK;
    }
    return res;
}
```

> **Exception**: early returns are permitted in private helper functions for short-circuiting calibration loops or error cascades (see `Bsp_Adc_InitInstance`).

#### Guard order

1. Check `init` state → `RES_ERROR_UNINITIALIZED`.
2. Validate parameters (range, `NULL` pointer, alignment) → `RES_ERROR_INVALID_PARAMETER`.
3. Check busy / ready state → `RES_ERROR_BUSY` or `RES_ERROR_NOT_READY`.
4. Perform the work.

#### Discarding return values

Ignore an intentionally-discarded return with a `(void)` cast:

```c
(void)LL_ADC_Init(adc, &adcInit);
(void)snprintf(line, sizeof(line), "ADC[%u,%u]: %u", ...);
```

#### Unused parameters

Suppress warnings with the `UNUSED()` macro from `common.h`:

```c
static void AppCli_Do_Init(const cli_arg_t arg[CLI_CFG_ARG_NUM_MAX])
{
    UNUSED(arg);
    /* ... */
}
```

---

### 9  Comparisons and control flow

#### Yoda conditions

Place the constant on the **left** side of equality comparisons:

```c
if (true == bspAdc.init)
if (RES_OK == res)
if (NULL == value)
if (0UL == timeout)
```

#### Allman braces

Opening `{` goes on its own line for functions, `if`, `else`, `for`, `while`, `switch`, and `case`:

```c
if (id >= BSP_DO_ID_NUM)
{
    res = RES_ERROR_INVALID_PARAMETER;
}
else
{
    /* ... */
}
```

#### Always brace

Single-statement bodies are still braced:

```c
for (uint8_t i = 0U; i < BSP_ADC_ID_NUM; i++)
{
    Bdi_Rcc_EnablePort(bspAdcCfg[i].port);
}
```

#### `switch` rules

- Brace every `case` body.
- End every `case` with `break;`.
- Always provide a `default:` arm (even if it only `break;`s or groups with the invalid-state case).
- Related invalid cases may share the same arm:

```c
switch (app.state)
{
    case APP_STATE_INIT:
    {
        /* ... */
        break;
    }
    case APP_STATE_RUNNING:
    {
        /* ... */
        break;
    }
    case APP_STATE_NOTINIT:
    default:
        break;
}
```

---

### 10  Formatting

- **Indentation**: 4 spaces. No tabs.
- **One statement per line**.
- **Column-aligned `#define` values** when grouping related constants:
  ```c
  #define ADC_JOY0_X_ADC              (ADC2)
  #define ADC_JOY0_X_PORT             (GPIOA)
  #define ADC_JOY0_X_PIN              (LL_GPIO_PIN_0)
  #define ADC_JOY0_X_CHANNEL          (LL_ADC_CHANNEL_1)
  ```
- **Column-aligned struct member declarations** for readability:
  ```c
  typedef struct
  {
      ADC_TypeDef    *adc;
      GPIO_TypeDef   *port;
      uint32_t        pin;
      uint32_t        channel;
  } bsp_adc_cfg_t;
  ```
- **Column-aligned table arrays** — configuration tables are formatted as visual tables with comments:
  ```c
  static const bsp_adc_cfg_t bspAdcCfg[BSP_ADC_ID_NUM] =
  {
      /* id               adc              port              pin                    channel            */
      /* BSP_ADC_ID_JOY0_X */ { ADC_JOY0_X_ADC, ADC_JOY0_X_PORT, ADC_JOY0_X_PIN, ADC_JOY0_X_CHANNEL },
  };
  ```
- **No spaces inside parentheses** for function definitions or calls:
  ```c
  void App_Init(void)
  App_Init();
  ```
- Section banners stay at column 1 and fixed width.

---

### 11  Inline helpers

Small utility functions live in the `Private (Static) Inline Function Definitions` section as `static inline`:

```c
static inline void Bsp_Do_SetBit(uint32_t bit)
{
    bspDo.pinStatus[bit >> 5] |= (1UL << (bit & 0x1FU));
}

static inline bool_t Bsp_Flash_IsHwBusy(void)
{
    return (0UL != READ_BIT(FLASH->SR, FLASH_SR_BSY));
}
```

---

### 12  Comments

- **Doxygen** goes on public declarations (header side) — not on the `.c` definition.
- Inside `.c` files, prefer short explanatory comments only where the *why* is non-obvious (timing constraints, hardware quirks, datasheet references). Do not narrate what the code does.
- Use `/** ... */` for Doxygen documentation blocks; use `/* ... */` for inline explanations.
- End-of-line comments after struct fields use `/*!< ... */`.

---

### 13  String and CLI conventions

- CLI / serial line endings are DOS-style: `\r\n`.
- Multi-line string constants use the backslash-newline continuation:
  ```c
  static const char_t appCliWelcomeMsg[] =\
  "*****************************************\r\n"\
  "* Project console V1.0.0 " __DATE__ " " __TIME__ "\r\n"\
  "*****************************************\r\n";
  ```
- CLI command function tables use an auto-sized pattern — no need to manually update a count:
  ```c
  static const cli_command_t appCliCommands[] = { /* ... */ };
  static const uint16_t appCliCommandsNum = sizeof(appCliCommands) / sizeof(cli_command_t);
  ```

---

### 14  Compiler and language standard

- **C23** (`CMAKE_C_STANDARD 23`) with GNU extensions enabled.
- Compiler warnings: `-Wall -Wextra -Wconversion -Wswitch-enum -Wswitch-default`.
- Optimisation: `-Og -g3` (debug-friendly optimisation in release builds).
- `STATIC_ASSERT(cond, msg)` is available for compile-time checks (uses `_Static_assert` on C11+).
- **Fixed-underlying-type enums** (`typedef enum : uint8_t { ... }`) are permitted
  when the enum is module-private and constraining storage size is beneficial
  (e.g. lookup-table indices, serialization tags). Public enums in headers should
  use the plain `typedef enum { ... }` form for maximum portability.

---

### 15  Portability and safety

- `volatile` is used for hardware-facing variables and DMA buffers that are written asynchronously:
  ```c
  volatile uint32_t systick_ms = 0U;
  static volatile uint16_t bspAdcScanBufAdc1[BSP_ADC_ID_NUM];
  ```
- Critical sections use `__disable_irq()` / `__enable_irq()` for short atomic regions (e.g. UART queue operations).
- Aggregate bit masks for hardware registers use `|` combinations of named constants:
  ```c
  #define BSP_FLASH_SR_ERR_MASK  (FLASH_SR_OPERR | FLASH_SR_PROGERR | FLASH_SR_WRPERR | ...)
  ```
