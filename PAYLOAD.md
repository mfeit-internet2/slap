# The SLAP Payload


## General Information

This document describes protocol version `1`.

All multi-byte integer values are represented in network-standard (big-endian) order.

The protocol is UDP and the goal is to fit within the Internet-wide
MTU of 576 bytes.  Subtract 40 bytes for an IPv6 header and 8 for a
UDP header, leaving **528 bytes** for the payload.

Terminology governing required and optional behavior should be
interpreted per [RFC
2119](https://datatracker.ietf.org/doc/html/rfc2119).


## Definitions

### `TIMESTAMP`

A timestamp.

All values are in network-standard (big-endian) byte order.

 * `SECONDS` - Integer seconds since 
 * `NANOSECONDS` - 

### `SyncSource``

Up to four bytes of ASCII identifying the method.  Any unused bytes
must be `0x0` (`NUL`).

Known values:

 * GPS - Global Positioning System Receiver
 * NTP - Network Time Protocol
 * RTC - System Real-Time Clock (Not Synchronized)
 * PTP - Precision Time Protocol

## Flag Bits

TODO: Number these

`x` - First - REQ considers this the first packet of a multi-packet
measurement and all prior state at the RES end should be reset.
(Usually counters for packet loss.)

`x` - Final - This is the last packet REQ will be sending.  After
producing a response, RES should remove any state stored for 

`x` - Kickback - REQ should send a copy of the completed packet back
to RES with the RES->REQ received timestamp filled in as a way to
provide a simultaneous measurement in the other direction.




## Status code

| `200` | Required | OK |
| `400` | Required | Bad request |
| `401` | Optional | Unauthorized |
| `403` | Optional | Forbidden |
| `426` | Optional | Upgrade required (Server does not support specified protocol version) |
| `429` | Optional | Too many requests |
| `500` | Optional | Internal server error |
| `503` | Optional | Service unavailable |

For any response other than `200`, the responder must not alter any
data in the payload other than the status code field.


## Payload

| Start | End | Type | Description |
| :---: | :-: | :--: | ----------- |
| 0 | 1 | Bytes(2) | Magic number, `SL` or `0x53 0x4c` |
| 2 | 3 | uint16 | Protocol version |
| 4 | 5 | Bits(16) | Flags |
| 6 | 7 | uint16 | Response code |

| x | x | unit64 | REQ reference number |
| x | x | unit64 | Sequence number |

| x | x | uint64 | REQ->RES sequence number |
| x | x | TimeStamp | REQ->RES packet departure time |
| x | x | TimeStamp | REQ->RES packet arrival time |
| x | x | uint64 | RES packets received |

| x | x | uint64 | RES->REQ sequence number |
| x | x | TimeStamp | RES->REQ packet departure time |
| x | x | TimeStamp | RES->REQ packet arrival time |
| x | x | uint64 | REQ packets received (Note 1) |

| x | x | SyncSource | A synchronization source |
| x | x | SyncSource | B synchronization source |

| x | x | TODO | HMAC |

NOTES:

 * 1 - Provided only if the `Kickback` bit is `1`