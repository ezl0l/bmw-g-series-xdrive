# Validation record

## Configuration and stationary observations

One 2021 BMW G22 M440i xDrive was tested over ENET. The relevant SGBD indices were DSC `0F2510`, LMV `0F2180`, EGS `0F25D0`, BDC `0F2680`. These indices identify the observed diagnostic variants, not a validated family of all hardware/software builds.

| Condition | DSC readback / result |
| --- | --- |
| Normal engine idle before the experiment | `62 DB E1 01` |
| Engine off, terminal 15 active, before writing | `62 DB E1 00` |
| One `2E DB E1 00` request | `6E DB E1` |
| Engine restarted in the same terminal-15 cycle | `62 DB E1 00`; informational AWD notice |
| Full terminal-15 off/on, then engine idle | `62 DB E1 01`; notice cleared |

LMV `DFEA` payload `02 00 00 00 01 00 00` occurred both during the experiment and after restoration. Zero front torque at idle also occurred in both states. Neither observation establishes mechanical clutch position or drive mode.

## Short closed-area comparison, 31 October 2026

| Measurement | Normal baseline (`DBE1=1`) | Diagnostic state (`DBE1=0`) |
| --- | ---: | ---: |
| Complete samples | 19 | 23 |
| Moving samples, at least 3 km/h | 12 | 9 |
| Moving samples, accelerator at least 5% | 8 | 5 |
| Peak observed speed | 29.1 km/h | 26.6 km/h |
| Peak LMV front-torque request during powered moving samples | 38.0 Nm | 0.0 Nm |
| Peak LMV estimated actual front torque in those samples | 39.5 Nm | -0.5 Nm |

The nearest distinct comparison was baseline 9.26 km/h / 15% accelerator / 38 Nm request, versus diagnostic-state 9.41 km/h / 18% accelerator / 0 Nm request, with the same reported gear enum. These were separate runs under nonidentical loads and paths, sampled approximately every 2.3 seconds. Most baseline samples were also zero. This is limited evidence of suppressed front-torque requests, not a mechanical torque measurement.

The diagnostic-state log ended after 52 seconds while stationary with RPM zero. That endpoint alone does not distinguish automatic stop/start from manual shutdown.

DSC ABS/ASC/stability availability fields `DC6C`, `DC6D`, `DC6E` remained `1`. Availability flags do not test actual intervention. After restoration, wait-aware `19 02 0D` reads returned no active/pending/confirmed DSC or LMV faults. No fault memory was cleared.

## Evidence boundaries

Confirmed: accepted DSC request, retained running-engine inactive flag in the same KL15 cycle, observed short-run LMV requests, and restoration through a full KL15 cycle.

Not established: physical front-wheel torque, complete clutch disengagement, ABS/DSC intervention, repeated/durable use, hot/cold or traction extremes, or compatibility with another vehicle/build. This is not a validated factory M xDrive or commercial 2WD implementation.

Raw captures are not distributed in this documentation-only repository. The table is a curated summary of the retained experiment records; publication did not run another vehicle test.
