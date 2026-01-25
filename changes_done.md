# TinyTapeout Adaptation Changes

This document summarizes all modifications made to adapt the SERV RISC-V SoC for TinyTapeout IHP-SG13G2.

## Summary of Changes

| File | Issue | Fix Applied | Reason |
|------|-------|-------------|--------|
| **tt_um_ECM24_serv_soc_top.sv** | Extra ports not in TinyTapeout interface | Removed `wb_clk`, `wb_rst`, `q`, `spi_miso`, `spi_mosi`, `spi_clk`, `spi_cs1`, `spi_cs2` from module declaration | TinyTapeout only supports standard interface: `ui_in[7:0]`, `uo_out[7:0]`, `uio_in[7:0]`, `uio_out[7:0]`, `uio_oe[7:0]`, `ena`, `clk`, `rst_n` |
| **tt_um_ECM24_serv_soc_top.sv** | Unmapped pins | Added pin mapping section to connect internal signals to TinyTapeout I/O | Map SPI and GPIO to standard pins according to `info.yaml` |
| **tt_um_ECM24_serv_soc_top.sv** | Unassigned outputs | Assigned all unused outputs to `1'b0` | TinyTapeout requires all outputs to be assigned |
| **tt_um_ECM24_serv_soc_top.sv** | RAM32 power pins | Added conditional `USE_POWER_PINS` wrapper around VPWR/VGND connections | RAM32 has conditional power pins; must match module definition |
| **tt_um_ECM24_serv_soc_top.sv** | Missing copyright | Added TinyTapeout copyright header | Standard TinyTapeout requirement |
| **tt_um_ECM24_serv_soc_top.sv** | Missing nettype directive | Added `` `default_nettype none`` | Best practice for TinyTapeout designs |
| **spi_sram.sv** | Implicit signal declarations | Declared all wire signals: `shift_enable`, `write_full_word`, `write_first_half`, `write_second_half`, `write_byte_1-4`, `shift_32_bits`, `shift_16_bits`, `shift_8_bits` | With `` `default_nettype none``, all signals must be explicitly declared |
| **spi_sram.sv** | Missing nettype directives | Added `` `default_nettype none`` at top and `` `default_nettype wire`` at bottom | Prevents implicit wire declarations and ensures strict typing |
| **spi_sram.sv** | Blocking assignment in sequential block | Changed `spi_mosi = ` to `spi_mosi <= ` in `always_ff @(negedge clk)` | Non-blocking assignments (`<=`) required for sequential logic |
| **serv_ram32_if.sv** | `reg` used with continuous assignment | Changed `output reg [31:0] o_rf_rdata` to `output wire [31:0] o_rf_rdata` | Signals assigned with `assign` must be `wire`, not `reg` |
| **serv_ram32_if.sv** | Missing nettype directives | Added `` `default_nettype none`` at top and `` `default_nettype wire`` at bottom | Consistency and strict signal typing |

## Pin Mapping

The following table shows how internal signals are mapped to TinyTapeout pins:

| Internal Signal | Direction | TinyTapeout Pin | Description |
|----------------|-----------|-----------------|-------------|
| `spi_miso` | Input | `ui_in[0]` | SPI Master In Slave Out |
| `gpio_out` | Output | `uo_out[0]` | GPIO output signal |
| `spi_mosi` | Output | `uo_out[1]` | SPI Master Out Slave In |
| `spi_clk` | Output | `uo_out[2]` | SPI clock |
| `spi_cs1_n` | Output | `uo_out[3]` | SPI Chip Select 1 (active low) |
| `spi_cs2_n` | Output | `uo_out[4]` | SPI Chip Select 2 (active low) |
| `wb_clk` | Internal | Connected to `clk` | Wishbone clock from TinyTapeout clock |
| `wb_rst` | Internal | Connected to `~rst_n` | Wishbone reset (active high) from TinyTapeout reset_n (active low) |

## Code Quality Improvements

| Category | Improvement | Impact |
|----------|-------------|--------|
| **Signal Declaration** | All signals explicitly declared | Eliminates implicit declarations, catches typos at compile time |
| **Assignment Types** | Proper use of blocking/non-blocking assignments | Prevents simulation/synthesis mismatches |
| **Type Consistency** | Correct use of `wire` vs `reg` | Matches SystemVerilog semantics, prevents warnings |
| **Conditional Compilation** | Power pins conditionally included | Works with both Verilator (with power pins) and Yosys (without) |
| **Standards Compliance** | TinyTapeout interface compliance | Ensures compatibility with TinyTapeout framework |

## Files Modified

### Primary Changes
- `src/tt_um_ECM24_serv_soc_top.sv` - Top-level module adapted for TinyTapeout
- `src/spi_sram.sv` - Fixed signal declarations and assignment types
- `src/serv_ram32_if.sv` - Fixed output port type

### No Changes Required
- All `serv/*.v` files - SERV CPU core files (unchanged)
- `src/subservient_gpio.v` - GPIO peripheral (unchanged)
- `src/RAM32.v` - Register file RAM macro (unchanged)

## Verification Status

| Check | Status | Notes |
|-------|--------|-------|
| Verilator Lint | ✅ Pass | Power pin warnings resolved with conditional compilation |
| Yosys Synthesis | ✅ Pass | All signals properly declared and typed |
| Port Compliance | ✅ Pass | Only standard TinyTapeout ports used |
| Signal Assignment | ✅ Pass | All outputs assigned, proper blocking/non-blocking usage |

## Known Issues

| Issue | Status | Notes |
|-------|--------|-------|
| `sg13g2_buf_4` ABC mapping error | 🔍 Under Investigation | Potential LibreLane/PDK library configuration issue, not code-related |
| LibreLane version | 🔍 Testing | Currently using `3.0.0.dev44`, may need adjustment |

## Build Configuration

Current GitHub Actions workflow configuration:
```yaml
- name: Build GDS
  uses: TinyTapeout/tt-gds-action@ttihp26a
  with:
    pdk: ihp-sg13g2
    librelane-version: 3.0.0.dev44
```

## References

- TinyTapeout Documentation: https://tinytapeout.com
- IHP SG13G2 PDK: https://github.com/IHP-GmbH/IHP-Open-PDK
- SERV CPU Core: https://github.com/olofk/serv
- SystemVerilog Best Practices: Use of `` `default_nettype none`` and explicit signal declarations

---

**Last Updated:** January 25, 2026  
**Authors:** FH Joanneum Team  
**TinyTapeout Submission:** TTIHP26a