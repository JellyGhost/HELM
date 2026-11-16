# Telemetry and Connection Monitoring

## Purpose

This document defines the telemetry collected from the connected PC and criteria used to evaluate the connection between the Pi and the Windows companion

It specifies metrics, sampling intervals, connection-health measurements, and connection states

Communication methods, transport protocols, and message structures can instead be found in 'communications-protocol.md'

## PC Telemetry

### Monitored Metrics

HELM monitors the following PC performance metrics:
| Metric | Measurement |
| --- | --- |
| CPU Usage | Percentage |
| GPU Usage | Percentage |
| RAM Usage | Percentage and MB |
| CPU Temperature | Temperature |
| GPU Temperature | Temperature |
| Storage | Remaining space on C: and D: drives |

The C: and D: drives are the default monitored drives.

### Sampling Interval

PC telemetry is sampled once every 15 seconds by default

## Connection Monitoring

Connection health is monitored independently of the PC telemetry sampling interval

### Monitoring Parameters

| Parameter | Default |
| --- | --- |
| Connection Check Interval | 2 seconds |
| Response Timeout | 1.5 seconds |
| Disconnection Threshold | 3 consecutive missed checks |
| Packet-Loss Measurement Window | 60 seconds |

### Connection Metrics

HELM tracks three primary connection metrics:
- **Latency:** Response time, measured in milliseconds
- **Packet Loss:** Percentage of unanswered connection checks
- **Connection State:** Current status of communication with the PC companion.

Packet loss is calculated over a rolling 60-second window:

`Packet Loss (%) = (Unanswered Checks / Total Checks Sent) × 100`

The rolling window prevents short-lived changes in connectivity from causing sudden fluctuations in the reported packet-loss percentage.

### Connection States

HELM uses three connection states:

#### Connected

- Zero consecutive missed checks
- The most recent connection check received a valid response
- The PC Companion is communicating normally

#### Unstable

- One or two missed checks
- Communication is intermittent or a response has exceeded the defined timeout
- The most recently recieved PC information remains available for display

#### Disconnected

- Three or more consecutive missed checks
- No valid response has been recieved within the defined timeout
- Previously recieved PC information must no longe be treated as current

### Stale Telemetry

When the connection becomes unstable, HELM continues displaying the most recently recieved telemetry

However, the displayed information should be distinguishable from current information so that outdated measurements are not mistaken for live data

When the connection becomes disconnected, HELM should indicate that the PC companion is unavailable. Previously received telemetry should no longer be presented as current.

## Default Configuration

| Setting | Default Value |
| --- | --- |
| Telemetry Sampling Interval | 15 seconds |
| Connection Check Interval | 2 seconds |
| Response Timeout | 1.5 seconds |
| Disconnection Threshold | 3 missed checks |
| Packet-Loss Measurement Window | 60 seconds |