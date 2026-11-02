# BMW G-series xDrive diagnostic mode

A documented, temporary xDrive diagnostic-disable sequence for one tested **BMW G22 M440i xDrive (2021)** ECU combination. The DSC accepts a one-shot request, retains the inactive function flag through an engine start in the same ignition cycle, and restores its normal state after a full terminal-15 cycle.

**This is a protocol reference, not a firmware flash, factory M xDrive conversion, or universal 2WD switch.** It contains no ECU binaries, access keys, vehicle identifiers, or executable vehicle-control tool.

[Русская версия](README.ru.md) · [Protocol and state handling](docs/PROTOCOL.md) · [Validation](docs/VALIDATION.md)

## The essential sequence

These are **UDS payloads**, not complete ENET packets or CAN frames. BMW logical diagnostic addresses are shown separately.

| Step | ECU | Request | Observed response / requirement |
| --- | --- | --- | --- |
| Identify the DSC | DSC `0x29` | `22 F1 50` | Tested SGBD index `0F2510` (`DSC_G20`) |
| Record the normal baseline, engine idling | DSC `0x29` | `22 DB E1` | `62 DB E1 01` |
| Stop engine; keep ignition / terminal 15 active | — | No write | P, parking brake, zero wheel speeds, zero accelerator, zero RPM |
| Send the request **once** | DSC `0x29` | `2E DB E1 00` | Exactly `6E DB E1` |
| Start engine without cycling terminal 15; remain stationary | DSC `0x29` | `22 DB E1` | `62 DB E1 00`; informational AWD notice observed |
| Restore | — | Full terminal-15 off/on cycle, then engine start | Verify `62 DB E1 01` with engine running; AWD notice clears |

A zero `DBE1` value **with the engine off is normal on the tested car**, including before the request. It does not establish that a mode was activated. The meaningful check is a fresh read after the engine starts within the same terminal-15 cycle.

## What was actually observed

- The DSC accepted `2E DB E1 00`, and running-engine `DBE1` stayed `0` within the same terminal-15 cycle.
- In a short closed-area comparison, the LMV reported zero requested front torque during the diagnostic-state log. A separate stock-AWD log included a request of `38 Nm`.
- After a full terminal-15 cycle, running-engine `DBE1` returned to `1` and the AWD information message disappeared.
- Wait-aware DTC reads after restoration returned no active/pending/confirmed DSC or LMV faults. No fault memory was cleared.

**ECU-requested and ECU-estimated torque are not direct wheel-torque measurements.** This does not prove physical clutch position in all conditions, stability-control behavior, long-term suitability, or compatibility with another ECU build. Full results and limitations are in [VALIDATION.md](docs/VALIDATION.md).

## Compatibility

| ECU | BMW logical address | Tested SGBD index |
| --- | ---: | --- |
| DSC | `0x29` | `0F2510` (`DSC_G20`) |
| LMV / transfer case | `0x19` | `0F2180` |
| EGS / transmission | `0x18` | `0F25D0` |
| BDC | `0x40` | `0F2680` |

These are identification results from one vehicle, **not a list of supported firmware versions**. An identical model name or SGBD index does not guarantee identical behavior. Other G-series variants and VIP-based paths are not validated here.

## Implementation rules

1. Read identities and normal running-engine baseline before offering a write.
2. Perform the write only while stationary in P with parking brake applied, engine off, terminal 15 active, zero accelerator, and fresh zero-speed readings from all wheels. Reject missing or stale data.
3. Check DSC/LMV fault status; block on active faults or unavailable/malformed responses. Handle UDS response-pending rather than treating a short timeout as an empty fault list.
4. Require explicit operator action. No startup write, moving-vehicle toggle, heartbeat, repeated write, coding, flashing, or automatic retry after a lost response.
5. Match the positive response exactly. A timeout means **unknown**, not success or permission to send again.
6. Restore through the tested ignition-cycle path and verify the running-engine flag. Do not substitute an untested `DBE1=1` write or ECU reset.

See [PROTOCOL.md](docs/PROTOCOL.md) for the state machine, raw readback examples, and failure handling. All work should be performed by a qualified operator in a controlled stationary setup; this reference does not validate driving safety.

## License

Documentation © 2026 [ezl0l](https://github.com/ezl0l), licensed under [CC BY 4.0](LICENSE.md). BMW and xDrive are trademarks of their respective owners; this project is independent.
