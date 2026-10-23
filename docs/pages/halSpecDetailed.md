# xDSL JSON HAL Detailed Reference

See the [overview](halSpec.md). RdkXdslManager consumes `xdsl_hal_schema.json`; schema locations below are JSON Pointers within that contract.

## Envelope and Transport
The schema declares draft-07 and requires:
| Field | Contract |
| --- | --- |
| `module` | String constant `xdslhal`. |
| `version` | String constant `0.0.1`. |
| `action` | One of the eleven case-sensitive strings below. |
| `reqId` | String matching `^[0-9]+$`, not a number. Preserve correlation in replies. |

The shared JSON HAL library supplies IPv4 TCP, configured port `4098`; the supplied server binds loopback. This is not HTTP or JSON-RPC 2.0. Serialized messages have no reliable explicit stream framing in the supplied transport. Do not equate socket receive boundaries with complete JSON documents.

## Action API Reference
| Action | Direction and payload contract |
| --- | --- |
| `getSchema` | Manager to vendor; schema retrieval described without parameters. No action-specific root condition prohibits extra fields. |
| `getSchemaResponse` | Vendor to manager. Required `SchemaInfo` object with required `FilePath` string matching `^(.+)/([^/]+)$`; no other properties inside that object. The path is not inline schema content. |
| `getParameters` | Manager to vendor. Required nonempty unique `params` array. Items require `name` and read-list membership. |
| `getParametersResponse` | Vendor to manager. Same read membership, with required `name`, `type`, and `value` per item. |
| `setParameters` | Manager to vendor. Required nonempty unique `params` array. Items require `name`, `type`, and `value` and write-list membership. Atomicity is unspecified. |
| `subscribeEvent` | Manager to vendor. Required nonempty unique `params` array. Items require `name`, `notificationType`, and event-list membership. |
| `publishEvent` | Vendor to manager. Required nonempty unique `params` array. Items require `name` and `value`; unlike GPON/Voice, the action does not additionally require `type`. Leaf constraints still apply. |
| `deleteObject` | Manager to vendor. Required nonempty unique `params` array. Item `name` must match `/definitions/dslLineObjectName`, pattern `^Device\.DSL\.Line\.\d+\.$`. Runtime support and restoration are not established by this shape. |
| `getActiveSubscriptions` | Enumerated only; no action-conditioned payload contract. |
| `getActiveSubscriptionsResponse` | Enumerated only; no action-conditioned payload contract. |
| `result` | Required `Result` object with required `Status`; no extra properties within `Result`. |

`Result.Status` permits `Success`, `Failed`, `Invalid Argument`, and `Not Supported`. A default does not make the required field optional. Check library status separately.

`uniqueItems` compares complete elements, not names alone. Root and leaf additional-property policies differ; do not impose a universal rule absent from the schema.

## Complete Parameter-Contract Lookup
The schema contains 422 definitions, including structural definitions and enums. Every action parameter is reached through these indexes:

| Index | Alternatives |
| --- | ---: |
| `/definitions/getParameterSupportedList/anyOf` | 348 |
| `/definitions/getParameterOptionalList/anyOf` | 45 |
| `/definitions/setParameterSupportedList/anyOf` | 18 |
| `/definitions/setParameterOptionalList/anyOf` | 6 |
| `/definitions/subscribeEventSupportedList/anyOf` | 2 |
| `/definitions/deleteObject/anyOf` | 1 |

These are reference counts, not unique capabilities. DSL, FAST, ATM, and PTM parameter families must remain action-qualified.

For each referenced `/definitions/{key}`:
- Read `description` for semantics, units, and access notes.
- Apply the exact `properties.name.pattern`, including instance indices and trailing dots for objects.
- Apply `properties.type.const` where present and resolve the actual JSON constraint under `properties.value`.
- Preserve bounds, lengths, enums, patterns, arrays, and nested references; do not turn every value into a string.
- Combine action-required fields with leaf `additionalProperties` and any leaf-required fields.
- Determine writability from write-list membership, not simply a read-side description.
- Treat `examples` as annotations, not alternative constraints or proof of validity.

The original schema is the field-level source of truth; this reference does not merge optional/supported lists or silently repair malformed constraints.

## Event Contracts
| Definition | Parameter path | Meaning |
| --- | --- | --- |
| `dslLineLinkStatus` | `Device.DSL.Line.{index}.LinkStatus` | DSL line link status. |
| `fastLineLinkStatus` | `Device.FAST.Line.{index}.LinkStatus` | FAST line link status. |

Both reference `linkStatusEnumList` for values and declare `notificationType`. Use `/definitions/notificationType` for permitted subscription values. Do not infer an entire line-training state machine, notification ordering, or delivery guarantee from these declarations.

## Minimal Envelope Example
This demonstrates shape, not verified server support:
```json
{"module":"xdslhal","version":"0.0.1","action":"getSchema","reqId":"1"}
```

## Verified Schema Defects
- References to `#/definitions/dslLineXTURVersion` and `#/definitions/dslLineXTURSerial` do not resolve.
- `atmLinkDestinationAddress.properties.value.examples` and `atmLinkVCSearchList.properties.value.examples` are strings, whereas the draft-07 annotation expects an array.
- DestinationAddress uses pattern `^(d+/d+)$`, which matches literal `d` characters rather than the numeric VPI/VCI examples. Do not silently replace it with a digit pattern.
- Active-subscription actions lack conditioned payload definitions.
- No empty `anyOf`/`oneOf`/`allOf` arrays were found in this schema.

Strict preflight may reject malformed schema annotations; lazy reference resolution may defer missing-reference failures until affected branches are evaluated. Distinguish those failures from vendor behavior.

## Runtime and Recovery Limits
A library timeout may leave the remote outcome unknown. Establish product-approved line/link restoration before destructive changes or object deletion.

## Source References
- [xDSL schema](https://github.com/rdkcentral/xdsl-manager/blob/main/hal_schema/xdsl_hal_schema.json)
- [Manager configuration](https://github.com/rdkcentral/xdsl-manager/blob/main/config/xdsl_manager_conf.json)
