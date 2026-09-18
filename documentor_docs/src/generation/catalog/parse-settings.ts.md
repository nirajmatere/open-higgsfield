# Technical Documentation: `src/generation/catalog/parse-settings.ts`

## Overview

The `src/generation/catalog/parse-settings.ts` module provides a utility function for parsing, validating, and normalizing raw user-provided settings against the setting definitions declared in a `ModelEntry`. 

Its primary purpose is to ensure that incoming key-value settings conform to expected types (`enum`, `range`, or `boolean`) and value constraints (allowed values or numeric ranges), falling back to default values when input is invalid or missing, and throwing errors when values fail schema validation.

---

## Dependencies

- **`ModelEntry`** (imported from `./types`): Type definition representing a model configuration entry, which includes a `settings` schema map.

---

## Function Reference

### `parseSettings(model, raw)`

Parses a raw settings object against the setting definitions specified on the given `ModelEntry`.

#### Signature

```typescript
export function parseSettings(
  model: ModelEntry,
  raw: Record<string, unknown>,
): Record<string, unknown>
```

#### Parameters

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `model` | `ModelEntry` | The model catalog entry containing the target `settings` definition schema. |
| `raw` | `Record<string, unknown>` | Unparsed, raw input key-value pairs to validate and parse. |

#### Return Value

- **Type**: `Record<string, unknown>`
- **Description**: A new object containing validated and normalized settings matching the schema defined in `model.settings`.

---

## Logic & Parsing Rules

The function iterates over every entry defined in `model.settings`. For each setting key and its associated field configuration, it checks the raw value (`raw[key]`) and processes it according to `field.type`:

### 1. Enum Settings (`field.type === "enum"`)
- **Type Check**: Evaluates if the raw value is of type `string`.
- **Fallback**: If the raw value is not a string, it defaults to `field.default`.
- **Validation**: Checks if the chosen value (`picked`) is included in the `field.values` array.
- **Error**: Throws `Error("Invalid <key>")` if `picked` is not contained in `field.values`.
- **Output**: Sets `out[key]` to `picked`.

### 2. Range Settings (`field.type === "range"`)
- **Type Check**: Evaluates if the raw value is of type `number`.
- **Fallback**: If the raw value is not a number, it defaults to `field.default`.
- **Validation**: Checks if the chosen numeric value (`picked`) satisfies `picked >= field.min` and `picked <= field.max`.
- **Error**: Throws `Error("Invalid <key>")` if `picked` is strictly less than `field.min` or strictly greater than `field.max`.
- **Output**: Sets `out[key]` to `picked`.

### 3. Boolean / Default Settings (All other field types)
- **Type Check**: Evaluates if the raw value is of type `boolean`.
- **Fallback**: If the raw value is not a boolean, it falls back to `field.default`.
- **Output**: Sets `out[key]` to the raw boolean value or `field.default`.

---

## Error Handling

The function throws a standard JavaScript `Error` in the following scenarios:

1. **Enum Validation Failure**: An enum setting value (or its default fallback) is not included within `field.values`.
   - Error message format: `Invalid <key>`
2. **Range Validation Failure**: A range setting value (or its default fallback) falls outside the closed interval `[field.min, field.max]`.
   - Error message format: `Invalid <key>`