## String.contains

string이 특정 sequence of char를 포함하는지의 여부를 반환하는 메소드

n = source string 길이, m = target string 길이

* 시간복잡도 : O(n\*m)
* 공간복잡도 : O(n+m)

```java
    /**
     * Returns true if and only if this string contains the specified
     * sequence of char values.
     *
     * @param s the sequence to search for
     * @return true if this string contains {@code s}, false otherwise
     * @since 1.5
     */
    public boolean contains(CharSequence s) {
        return indexOf(s.toString()) > -1;
    }
```

특정 string이 포함하는 가장 첫 index를 반환하는 indexOf(String str) 메소드를 호출하여 반환값이 -1이상이면 해당 string이 포함되어 있는것이므로 true를 반환

```java
    /**
     * Returns the index within this string of the first occurrence of the
     * specified substring.
     *
     * <p>The returned index is the smallest value <i>k</i> for which:
     * <blockquote><pre>
     * this.startsWith(str, <i>k</i>)
     * </pre></blockquote>
     * If no such value of <i>k</i> exists, then {@code -1} is returned.
     *
     * @param   str   the substring to search for.
     * @return  the index of the first occurrence of the specified substring,
     *          or {@code -1} if there is no such occurrence.
     */

	public int indexOf(String str) {
        return indexOf(str, 0);
    }
```

특정 string인 str이 포함하는 첫번쨰 index를 반환하는 메소드

몇번째 index에서 시작할지를 정해주지 않았으므로 0번째에서 시작하는것으로 간주하여 index(String str, int fromIndex) 메소드 호출

```java
    /**
     * Returns the index within this string of the first occurrence of the
     * specified substring, starting at the specified index.
     *
     * <p>The returned index is the smallest value <i>k</i> for which:
     * <blockquote><pre>
     * <i>k</i> &gt;= fromIndex {@code &&} this.startsWith(str, <i>k</i>)
     * </pre></blockquote>
     * If no such value of <i>k</i> exists, then {@code -1} is returned.
     *
     * @param   str         the substring to search for.
     * @param   fromIndex   the index from which to start the search.
     * @return  the index of the first occurrence of the specified substring,
     *          starting at the specified index,
     *          or {@code -1} if there is no such occurrence.
     */

	public int indexOf(String str, int fromIndex) {
        return indexOf(value, 0, value.length,
                str.value, 0, str.value.length, fromIndex);
    }
```

```java
    /**
     * Code shared by String and StringBuffer to do searches. The
     * source is the character array being searched, and the target
     * is the string being searched for.
     *
     * @param   source       the characters being searched.
     * @param   sourceOffset offset of the source string.
     * @param   sourceCount  count of the source string.
     * @param   target       the characters being searched for.
     * @param   targetOffset offset of the target string.
     * @param   targetCount  count of the target string.
     * @param   fromIndex    the index to begin searching from.
     */

	static int indexOf(char[] source, int sourceOffset, int sourceCount,
            char[] target, int targetOffset, int targetCount,
            int fromIndex) {
        if (fromIndex >= sourceCount) {
            return (targetCount == 0 ? sourceCount : -1);
        }
        if (fromIndex < 0) {
            fromIndex = 0;
        }
        if (targetCount == 0) {
            return fromIndex;
        }

        char first = target[targetOffset];
        int max = sourceOffset + (sourceCount - targetCount);

        for (int i = sourceOffset + fromIndex; i <= max; i++) {
            /* Look for first character. */
            if (source[i] != first) {
                while (++i <= max && source[i] != first);
            }

            /* Found first character, now look at the rest of v2 */
            if (i <= max) {
                int j = i + 1;
                int end = j + targetCount - 1;
                for (int k = targetOffset + 1; j < end && source[j]
                        == target[k]; j++, k++);

                if (j == end) {
                    /* Found whole string. */
                    return i - sourceOffset;
                }
            }
        }
        return -1;
    }
```

sourceOffset+fromIndex의 index에서 sourceOffset+fromIndex+sourceCount 까지의 source string에 포함되는 targetOffset의 index에서 targetOffset+targetCount까지의 target string의 첫번째 index번호를 반환한다.

내부적으로 이중for문을 통해서 source string에 포함되는 target string의 index번호를 구하므로 시간복잡도는 O(n\*m)이며, 공간복잡도는 문자열의 길이만큼만 필요하므로 O(n+m)이다.

(여기서 n = sourceCount, m=targetCount로 정의한다.)