## BitSet

#### BitSet이란?

* boolean element로 이루어진 Array
* BitSet instance를 출력하면 true인 bit의 index가 Array처럼 출력됨



#### Reference

[Java8 BistSet](https://docs.oracle.com/javase/8/docs/api/java/util/BitSet.html)



#### set

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `void`            | `set(int bitIndex)`Sets the bit at the specified index to `true`. |
| `void`            | `set(int bitIndex, boolean value)`Sets the bit at the specified index to the specified value. |
| `void`            | `set(int fromIndex, int toIndex)`Sets the bits from the specified `fromIndex` (inclusive) to the specified `toIndex` (exclusive) to `true`. |
| `void`            | `set(int fromIndex, int toIndex, boolean value)`Sets the bits from the specified `fromIndex` (inclusive) to the specified `toIndex` (exclusive) to the specified value. |
| `int`             | `previousSetBit(int fromIndex)`Returns the index of the nearest bit that is set to `true` that occurs on or before the specified starting index. |
| `int`             | `nextSetBit(int fromIndex)`Returns the index of the first bit that is set to `true` that occurs on or after the specified starting index. |

#### clear

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `void`            | `clear()`Sets all of the bits in this BitSet to `false`.     |
| `void`            | `clear(int bitIndex)`Sets the bit specified by the index to `false`. |
| `void`            | `clear(int fromIndex, int toIndex)`Sets the bits from the specified `fromIndex` (inclusive) to the specified `toIndex` (exclusive) to `false`. |
| `int`             | `previousClearBit(int fromIndex)`Returns the index of the nearest bit that is set to `false` that occurs on or before the specified starting index. |
| `int`             | `nextClearBit(int fromIndex)`Returns the index of the first bit that is set to `false` that occurs on or after the specified starting index. |

#### get

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `boolean`         | `get(int bitIndex)`Returns the value of the bit with the specified index. |
| `BitSet`          | `get(int fromIndex, int toIndex)`Returns a new `BitSet` composed of bits from this `BitSet` from `fromIndex` (inclusive) to `toIndex` (exclusive). |

#### flip

complrement(보수)로 설정

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `void`            | `flip(int bitIndex)`Sets the bit at the specified index to the complement of its current value. |
| `void`            | `flip(int fromIndex, int toIndex)`Sets each bit from the specified `fromIndex` (inclusive) to the specified `toIndex` (exclusive) to the complement of its current value. |



#### Bitwise Operation

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `void`            | `and(BitSet set)`Performs a logical **AND** of this target bit set with the argument bit set. |
| `void`            | `andNot(BitSet set)`Clears all of the bits in this `BitSet` whose corresponding bit is set in the specified `BitSet`. |
| `void`            | `or(BitSet set)`Performs a logical **OR** of this bit set with the bit set argument. |
| `void`            | `xor(BitSet set)`Performs a logical **XOR** of this bit set with the bit set argument. |
| `boolean`         | `intersects(BitSet set)`Returns true if the specified `BitSet` has any bits set to `true` that are also set to `true` in this `BitSet`. |
| `int`             | `cardinality()`Returns the number of bits set to `true` in this `BitSet`. |

#### related on length

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `boolean`         | `isEmpty()`Returns true if this `BitSet` contains no bits that are set to `true`. |
| `int`             | `length()`Returns the "logical size" of this `BitSet`: the index of the highest set bit in the `BitSet` plus one. |
| `int`             | `size()`Returns the number of bits of space actually in use by this `BitSet` to represent bit values. |

#### convert BitSet To Object

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `IntStream`       | `stream()`Returns a stream of indices for which this `BitSet` contains a bit in the set state. |
| `byte[]`          | `toByteArray()`Returns a new byte array containing all the bits in this bit set. |
| `long[]`          | `toLongArray()`Returns a new long array containing all the bits in this bit set. |
| `String`          | `toString()`Returns a string representation of this bit set. |

#### convert Object To BitSet

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| `static BitSet`   | `valueOf(byte[] bytes)`Returns a new bit set containing all the bits in the given byte array. |
| `static BitSet`   | `valueOf(ByteBuffer bb)`Returns a new bit set containing all the bits in the given byte buffer between its position and limit. |
| `static BitSet`   | `valueOf(long[] longs)`Returns a new bit set containing all the bits in the given long array. |
| `static BitSet`   | `valueOf(LongBuffer lb)`Returns a new bit set containing all the bits in the given long buffer between its position and limit. |