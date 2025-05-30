**@harmoniclabs/plutus-data** • [Readme](../README.md) \| [API](../globals)

***

[@harmoniclabs/plutus-data](../README.md) / DataMap

# Class: DataMap\<DataKey, DataValue\>

## Type parameters

• **DataKey** extends [`Data`](../type-aliases/Data)

• **DataValue** extends [`Data`](../type-aliases/Data)

## Constructors

### new DataMap(map)

> **new DataMap**\<`DataKey`, `DataValue`\>(`map`): [`DataMap`](DataMap)\<`DataKey`, `DataValue`\>

#### Parameters

• **map**: [`DataPair`](DataPair)\<`DataKey`, `DataValue`\>[]

#### Returns

[`DataMap`](DataMap)\<`DataKey`, `DataValue`\>

#### Source

[DataMap.ts:11](https://github.com/HarmonicLabs/plutus-data/blob/911664c/src/DataMap.ts#L11)

## Properties

### \_map

> **`private`** **\_map**: [`DataPair`](DataPair)\<`DataKey`, `DataValue`\>[]

#### Source

[DataMap.ts:8](https://github.com/HarmonicLabs/plutus-data/blob/911664c/src/DataMap.ts#L8)

## Accessors

### map

> **`get`** **map**(): [`DataPair`](DataPair)\<`DataKey`, `DataValue`\>[]

#### Returns

[`DataPair`](DataPair)\<`DataKey`, `DataValue`\>[]

#### Source

[DataMap.ts:9](https://github.com/HarmonicLabs/plutus-data/blob/911664c/src/DataMap.ts#L9)

## Methods

### clone()

> **clone**(): [`DataMap`](DataMap)\<`DataKey`, `DataValue`\>

#### Returns

[`DataMap`](DataMap)\<`DataKey`, `DataValue`\>

#### Source

[DataMap.ts:24](https://github.com/HarmonicLabs/plutus-data/blob/911664c/src/DataMap.ts#L24)

***

### toJson()

> **toJson**(): `Object`

#### Returns

`Object`

##### map

> **map**: `Object`[]

#### Source

[DataMap.ts:33](https://github.com/HarmonicLabs/plutus-data/blob/911664c/src/DataMap.ts#L33)

***

### toString()

> **toString**(): `string`

#### Returns

`string`

#### Source

[DataMap.ts:45](https://github.com/HarmonicLabs/plutus-data/blob/911664c/src/DataMap.ts#L45)
