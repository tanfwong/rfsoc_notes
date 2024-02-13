# Math Data Types & Library

Vitis HLS supports most of the simple as well as composite C/C++ data
types. Details of how different data types are synthesized are
available in {cite}`ug1399`. For DSP applications, we are most
interested in data types for arithmetics, such as the standard C/C++
integer and floating point data types as well as fixed-point data
types supported by Vitis HLS. In this section, we focus our
discussions on these math data types as well as the math operators and
functions that operate on them.

## Integer Types
* Vitis HLS supports synthesis of all standard C/C++ integer
  types. The bit-widths of these types are listed in the table below:
  ```{list-table} Vitis HLS-supported standard C/C++ integer type bit-widths
  :name: int_types
  :header-rows: 1

  * - Integer type
    - bit-width

  * - `(unsigned) char`, `(unsigned) int8_t`
    - 8

  * - `(unsigned) short`, `(unsigned) int16_t`
    - 16

  * - `(unsigned) int`, `(unsigned) int32_t`
    - 32

  * - `(unsigned) long`, `(unsigned) long long`, `(unsigned) int64_t`
    - 64
  ```
  ```{tip}
  The header file `<stdint.h>` must be included in order to use the
  exact bit-width integer types `(unsigned) int*_t`.
  ```
* Vitis HLS also supports arbitrary precision (AP) signed and unsigned integer types
  `ap_int<W>` and `ap_uint<W>` in C++, where `W` is the bit-width that can
  range from 1 to 1024. To use the AP integer types, the header file
  `<ap_int.h>` must be included. In addition, the maximum bit-width
  can also be extended to 4096 by defining the macro `AP_INT_MAX_W`
  before including the header file as below:
  ```c++
  #define AP_INT_MAX_W 4096
  #include <ap_int.h>
  ```
