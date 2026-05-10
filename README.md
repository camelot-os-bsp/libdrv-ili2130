# libdrv-ili2130

ILI2130 touch controller driver library for Camelot OS BSP.

This driver provides a small public API to probe, initialize, power-manage, and read the latest touch sample from an ILI2130 device.

## Overview

`libdrv-ili2130` is implemented as a static library and is intended to run inside the Camelot OS driver stack.

At runtime, it:

- Registers as a platform driver through Merlin.
- Uses I2C transfers to communicate with the ILI2130 controller.
- Maintains the latest touch sample internally and exposes it through a getter.

## Public API

Public declarations are provided in `include/ili2130.h`.

### Data structure

```c
struct touch_informations {
		uint16_t x;
		uint16_t y;
		uint8_t pressure;
		uint8_t touch_id;
};
```

### Explicit ILI2130 entry points

```c
int ilitech_2130_probe(void);
int ilitech_2130_init(void);
int ilitech_2130_sleep(void);
int ilitech_2130_wake(void);
int ilitech_2130_get_last_touch_info(struct touch_informations *touch_info);
```

### Portable alias entry points

```c
int touch_probe(void);
int touch_init(void);
int touch_sleep(void);
int touch_wake(void);
int touch_get_last_touch_info(struct touch_informations *touch_info);
```

These aliases map directly to the explicit ILI2130 functions and are provided for application portability.

## API semantics

- `ilitech_2130_probe()`:
	Registers the device in the platform layer and checks that the ILI2130 chip ID is present on I2C.

- `ilitech_2130_init()`:
	Resets and configures the controller for touch operation, then clears pending statuses.

- `ilitech_2130_sleep()`:
	Puts the touch controller in low-power sleep mode.

- `ilitech_2130_wake()`:
	Brings the touch controller back to active mode.

- `ilitech_2130_get_last_touch_info(...)`:
	Returns the most recently captured touch information into the caller-provided buffer.
	Returns an error if no touch sample is available yet or if the pointer is invalid.

## Typical usage flow

```c
#include <ili2130.h>

int rc;
struct touch_informations touch;

rc = ilitech_2130_probe();
if (rc != 0) {
		/* handle probe error */
}

rc = ilitech_2130_init();
if (rc != 0) {
		/* handle init error */
}

rc = ilitech_2130_get_last_touch_info(&touch);
if (rc == 0) {
		/* use touch.x, touch.y, touch.pressure, touch.touch_id */
}
```

## Dependencies and integration

This driver is built on top of two subprojects:

- `drvi2c`:
	Provides low-level I2C bus access used by this driver (read/write transactions to device registers).

- `merlin`:
	Provides platform-driver integration, including device registration and platform abstractions (GPIO and related system-facing integration).

In Meson, both are added as subprojects and exported as dependencies of `libdrv-ili2130`.

## Build system notes

- Project type: static C library.
- Header export: `include/ili2130.h`.
- Source implementation: `src/ili2130_driver.c`.
- Meson dependency object exported: `libdrv_ili2130_dep`.

## License

BSD-3-Clause
