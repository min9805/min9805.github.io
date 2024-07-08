---
title: "[Java] String 과 Byte[] 의 차이 by 인코딩! 바이트 파일을 String 으로 읽으면?"
excerpt: "[Java] String 과 Byte[] 의 차이 by 인코딩! 바이트 파일을 String 으로 읽으면?"

categories:
  - java
tags:
  - [JAVA, string, byte]

toc: true
toc_sticky: true

date: 2024-07-06
last_modified_at: 2024-07-06
---

-   정적 파일을 String 으로 읽어 처리했었다. HTML 이나 CSS, SVG 파일같은 경우에는 모두 문자열로 표현할 수 있기 때문에 아무 문제 없었지만
-   ico 같은 이미지 파일같은 경우에는 바이트 단위로 이루어져있어 String 으로 읽으면 파일이 깨지는 현상이 발생했다.
-   추가적으로 이미지 파일은 38505 바이트인데, String 으로 받을 경우 70259 바이트가 된다는 점이 이상했다.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/f5ad8574-81b7-4307-860c-78f942506343)

-   temp1 은 이미지 파일을 String 으로 읽은 것
-   byteArray 는 이미지를 바이트 단위로 읽은 정상 로드된 경우이다.
-   String 도 결국에는 byte\[\] 인데 도대체 왜 깨지는 것인지 하나하나 알아보도록 하자!

# Byte to String

![image](https://github.com/min9805/min9805.github.io/assets/56664567/7121fbf7-5528-4ee0-8b57-0c2e9f571e77)

-   실제 favicon.ico 파일은 PNG 파일의 시그니처를 가지고 있다.
-   따라서 가장 처음에는 아래와 같은 바이트 배열이 나타난다.
-   \[\-119, 80, 78, 71, 13, 10, 26, 10\]
-   해당 배열을 실제 String 으로 담아보자.

![](https://blog.kakaocdn.net/dn/dvp0o2/btsIoxApyVn/j4SCDkqbMu2OZohGjCpPq0/img.png)

-   실제 String 의 바이트 배열을 열어보면
-   \[\-3, \-1, 80, 0, 78, 0, 71, 0, 13, 0, 10, 0, 26, 0, 10, 0\]
-   위와 같이 변한 것을 볼 수 있다.
-   매번 0 이 추가되었고, -119 는 \[-3, -1\] 이 되었다..

-   양수 바이트 배열

![image](https://github.com/min9805/min9805.github.io/assets/56664567/fc8a8d73-a91a-43f1-b21d-b93c76aa02e6)

-   음수 바이트 배열

![image](https://github.com/min9805/min9805.github.io/assets/56664567/a16f854e-3fcd-44d7-8570-95d57c88ed94)

## 음수 Byte 와 양수 Byte

위에서 바이트를 문자열로 담았을 때의 규칙이 보이는가??

-   모든 바이트들이 양수일 때
    -   바이트와 문자열이 정확하게 일치한다.
-   반면 바이트들 중 하나라도 음수가 존재한다면

```
- 음수 바이트는 -> \[-3, -1\] 로 변환되고  
- 양수 바이트 ex) 10 -> \[10, 0\] 으로 변환된다.
```

### 추측 1. ASCII Code

첫 번째 추측은 ASCII 코드이다.

String 은 말 그대로 문자열이다. 바이트가 주어지더라도 문자열로 변환가능해야할 것이다. 아스키코드로 모두 변환 가능한 경우, 즉 모든 바이트들이 양수(0~127) 일 경우에는 문제 없이 문자열과 바이트배열이 동일하다.

하지만 하나의 음수라도 존재하는 경우 모든 음수들은 \[-3, -1\] 로 변하고 모든 양수 바이트 뒤에는 "0" 이 추가된다.

\[-3, -1\] 는 해당 바이트는 읽을 수 없음을 나타내는 바이트라 추측하고

한 바이트라도 2개의 바이트로 표시해야되는 순간 모든 바이트들의 양식을 맞추기 위해 양수 바이트들도 0을 추가해 2개의 바이트로 만든다고 생각한다.

그렇다면 음수 바이트의 경우에는 어떻게 처리할 수 있는 것인가??

## UTF-8

기본적으로 대부분의 경우에서 UTF-8 로 인코딩한다. 또한 UTF-8 은 가변길이를 가지고 있기에 최대 4바이트를 사용해 문자를 나타낼 수 있다!

실제 0~127 까지는 하나의 바이트로 표현 가능하기 때문에 추가 변환 과정이 필요없다. 이는 위에서 양수 바이트만 존재할 경우 String 과 바이트 배열에 차이가 없는 것에 이유가 되지 않을까 싶다.

그렇다면 2 바이트 이상을 사용하는 문자 (128번 이상의 아스키코드) 에 대해서는 어떻게 처리하는 것인가??

![image](https://github.com/min9805/min9805.github.io/assets/56664567/e6717916-63d5-4e50-a987-7742e5a26224)

-   128~2047 의 유니코드들은 2바이트를 사용해서 읽어낼 수 있다.

1.  유니코드를 16비트로 나타낸다. ex) 0x03C0
2.  128 ~ 2047 사이의 값이기 때문에 (15 ~ 11) 번째 비트들은 모두 0일 수 밖에 없다. 즉, (10 ~ 0) 의 11개 비트를 처리해야한다.
3.  UTF-8 에서는 해당 유니코드를 2 바이트를 사용해서 알아낸다.
    1.   첫 번째 바이트는 "110\\\_ \\\_\\\_\\\_\\\_" 을 고정으로 5개의 비트를 가져온다.
    2.  두 번째 바이트는 "10\\\_\\\_ \\\_\\\_\\\_\\\_" 을 고정으로 6개의 비트를 가져온다.

4.  두 바이트를 각각 읽게되면 0xCF, 0x80 으로 나타낼 수 있다.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/dfc0b7ea-18ad-4604-a261-c36171f349c7)

실제로 해당 값들을 바이트 배열에 담고 문자열로 출력해보면 우리가 원했던 파이 문자열이 그대로 나오는 것을 확인할 수 있다!!

![image](https://github.com/min9805/min9805.github.io/assets/56664567/6b771bba-73b9-411d-af3a-ae5066178fa4)

해당 값을 10진수로 변환시켜 넣었을 때도 정상적으로 동작하는 것을 알 수 있다.

### 추측 2. 음수 바이트

이제 바이트에서 음수가 들어오더라도 정상적으로 문자를 반환하는 경우를 찾아냈다.

하지만 대부분의 경우에서는 음수값(128 이상) 을 사용하는 경우 연속된 두 바이트가

110\_ \_\_\_\_, 10\_\_ \_\_\_\_ 다음 포맷을 만족하지 않기 때문에 읽을 수 없는 값이라고 판단하는 것같다.

## 바이트 배열의 순서???

![image](https://github.com/min9805/min9805.github.io/assets/56664567/4b940d35-a57b-40be-bf30-ea437c5ce23e)

가장 의문이 드는 점이었다.

위 계산대로라면 파이라는 문자열은

![image](https://github.com/min9805/min9805.github.io/assets/56664567/fe7898c0-8053-46d6-b488-8fb2897ef5f5)

위와 같기 때문에 8비트씩 잘라본다면

0000 0011, 1100 0000 이다. 이는 \[3, -64\] 로 바꿀 수 있다.

여기서 가장 의문인 점이 발생한다. \[3, -64\] 로 저장되어야할 파이가 \[-64, 3\] 으로 저장되어있기 때문이다.

도대체 왜 순서를 뒤바꿔 저장하는 것인가??

### 추측 3. LE, BE

이렇게 반대로 저장하는 방법은 네트워크 패킷에서 찾아볼 수 있다. 패킷을 통신할 경우에 저장하는 순서에 따라 Big Endian, Little Endian 으로 부른다. 이러한 방식이 String 에 적용되는 것이 아닐까?

![image](https://github.com/min9805/min9805.github.io/assets/56664567/4cd37214-6fb3-48b8-8daa-5749ea283a68)

# String 저장 방법

구글링을 하면서 여러 정보를 알게되고 이 부분들이 섞이니 더 혼동이 왔다. 혼동이 왔던 부분들을 하나하나 짚어보면서 넘어가보자.

## Java 는 UTF-16 을 기본으로 사용한다??

```
public String(byte bytes[], Charset charset) {
    this(bytes, 0, bytes.length, charset);
}

/**
 * Constructs a new {@code String} by decoding the specified subarray of
 * bytes using the platform's default charset.  The length of the new
 * {@code String} is a function of the charset, and hence may not be equal
 * to the length of the subarray.
 *
 * <p> The behavior of this constructor when the given bytes are not valid
 * in the default charset is unspecified.  The {@link
 * java.nio.charset.CharsetDecoder} class should be used when more control
 * over the decoding process is required.
 * ...
 */
```

-   실제 String 생성자를 찾아보면 charset 은 \*\*java.nio.charset.CharsetDecoder\*\* 패키지에서 확인할 수 있다.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/4ffd22fa-7068-4c6a-ad9e-870594f9b2ac)

-   결과적으로 현재 DefaultCharset 은 \*\*UTF-8\*\* 이다.
-   그렇다면 String 에서 UTF-8-Little Endian 을 사용한다면 모든 것이 이해가 가능하다

## UTF-8 과 Little Endian

-   UTF-8은 Byte Order가 없는 인코딩 방식이다. 따라서 Little Endian(LE)이나 Big Endian(BE)와는 관련이 없다..
-   이 부분을 해결하기 위해 더 깊이 살펴보았다.

## String 생성자

```
public String(byte[] bytes, int offset, int length, Charset charset) {
    Objects.requireNonNull(charset);
    checkBoundsOffCount(offset, length, bytes.length);
    if (length == 0) {
        this.value = "".value;
        this.coder = "".coder;
    } else if (charset == UTF_8.INSTANCE) {
        if (COMPACT_STRINGS && !StringCoding.hasNegatives(bytes, offset, length)) {
            this.value = Arrays.copyOfRange(bytes, offset, offset + length);
            this.coder = LATIN1;
        } else {
            int sl = offset + length;
            int dp = 0;
            byte[] dst = null;
            if (COMPACT_STRINGS) {
                dst = new byte[length];
                while (offset < sl) {
                    int b1 = bytes[offset];
                    if (b1 >= 0) {
                        dst[dp++] = (byte)b1;
                        offset++;
                        continue;
                    }
                    if ((b1 & 0xfe) == 0xc2 && offset + 1 < sl) { // b1 either 0xc2 or 0xc3
                        int b2 = bytes[offset + 1];
                        if (!isNotContinuation(b2)) {
                            dst[dp++] = (byte)decode2(b1, b2);
                            offset += 2;
                            continue;
                        }
                    }
                    // anything not a latin1, including the repl
                    // we have to go with the utf16
                    break;
                }
                if (offset == sl) {
                    if (dp != dst.length) {
                        dst = Arrays.copyOf(dst, dp);
                    }
                    this.value = dst;
                    this.coder = LATIN1;
                    return;
                }
            }
            if (dp == 0 || dst == null) {
                dst = new byte[length << 1];
            } else {
                byte[] buf = new byte[length << 1];
                StringLatin1.inflate(dst, 0, buf, 0, dp);
                dst = buf;
            }
            dp = decodeUTF8_UTF16(bytes, offset, sl, dst, dp, true);
            if (dp != length) {
                dst = Arrays.copyOf(dst, dp << 1);
            }
            this.value = dst;
            this.coder = UTF16;
        }
    } else if (charset == ISO_8859_1.INSTANCE) {
        if (COMPACT_STRINGS) {
            this.value = Arrays.copyOfRange(bytes, offset, offset + length);
            this.coder = LATIN1;
        } else {
            this.value = StringLatin1.inflate(bytes, offset, length);
            this.coder = UTF16;
        }
    } 
    ...
}
```

### 0 ~ 127 유니코드

-   우선 바이트의 길이가 1 이상이고 UTF-8 인코딩에 대해서 알아보자.

```
if (COMPACT_STRINGS && !StringCoding.hasNegatives(bytes, offset, length)) {
            this.value = Arrays.copyOfRange(bytes, offset, offset + length);
            this.coder = LATIN1;
        }
```

-   가장 처음에 진행하는 것은 해당 문자열들이 모두 양수인지 검사한다.
-   즉, 모든 바이트들이 하나의 바이트로 표현될 수 있는가? 를 판별한다
-   이를 성립한다면 \*\*LATIN1\*\* 이라는 인코딩 방식을 사용한다.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/62fa9d4f-0dde-4941-aa6c-96e6aefe3307)

-   따라서 양수 바이트들만 존재하는 배열에 대해서는 String 으로 변환해도 내부 바이트 값이 전혀 변하지 않는다!

### 0 ~ 255 유니코드

```
    while (offset < sl) {
            int b1 = bytes[offset];
            if (b1 >= 0) {
                dst[dp++] = (byte)b1;
                offset++;
                continue;
            }
            if ((b1 & 0xfe) == 0xc2 && offset + 1 < sl) { // b1 either 0xc2 or 0xc3
                int b2 = bytes[offset + 1];
                if (!isNotContinuation(b2)) {
                    dst[dp++] = (byte)decode2(b1, b2);
                    offset += 2;
                    continue;
                }
            }
            // anything not a latin1, including the repl
            // we have to go with the utf16
            break;
        }
```

-   ㅇ음수값이 존재한다면 일단 음수값이 나타날때까지 바이트를 계속 읽어낸다.
-   이때 확인해야할 것이 b1 이 음수일 때이다.
    -   b1 이 0xC2 or 0xC3 를 확인하는 작업은 이어진 b2 를 함께 읽은 문자가 유니코드 128~255 사이에 존재하는 지 여부이다.
    -   b1 은 1100 0010 or 1100 0011 이어야하고
    -   isNotContinuation(b2) 를 살펴보면 b2 는 10\_\_ \_\_\_\_ 이어야한다.
    -   위에서 했던 것 처럼 b1 과 b2 를 합친다면 해당 두 바이트가 표현하는 것이 \[000 10\_\_ \_\_\_\_ or 000 11\_\_ \_\_\_\_\] 이어야한다는 뜻이다.
    -   4비트를 표현할 때 1 \_ \_ \_ 과 같이 첫 번째 비트를 1로 고정시킨다면 해당 비트는 1000 ~ 1111 까지 이므로 8 ~ 15까지의 범위로 읽어낼 수 있다.
    -   동일하게 b1 와 b2 로 읽는 것이 128 ~ 255 사이에 존재하는 지 여부를 확인하는 작업이다.
-   음수가 128 ~ 255 범위에 존재한다면 String 은 그 다음 바이트들을 읽어나간다.

```
                    if (offset == sl) {
                        if (dp != dst.length) {
                            dst = Arrays.copyOf(dst, dp);
                        }
                        this.value = dst;
                        this.coder = LATIN1;
                        return;
                    }
```

-   모든 바이트를 읽었다면 (256 이상을 표현하는 음수가 존재해 break 가 걸리지 않았다면) 해당 바이트들은 1바이트로 모두 표현이 가능하다.
-   애초에 바이트는 8비트이므로 256개의 숫자 표현이 가능하다.
-   때문에 LATIN1 이라는 인코딩 방식으로 바이트 배열을 전달한다.

> 왜 이렇게 할까??  
>   
> 
> 원래는 아스키코드만 보고 문자를 변환시켰을 것이다.  
> 하지만 아스키코드는 8비트를 가지고 0 ~ 127 만 표현하고 있었고 사실상 낭비되고 있는 셈이다.
> 
> 또한 이후에 보겠지만 바이트 배열 중 한 바이트라도 255 를 넘어가는 순간 전체 길이를 2배로 만들어버린다.  
> 이 부분이 비용을 급격히 증가시키는데 동조했을 것이다.
> 
> 이를 위해 사실상 1 바이트로 표현할 수 있는 0 ~ 255 영역에 대해서 엄밀하게 검증하고 가능하다면 모두 1 바이트로 표현하는 것이다.  
>   
>   

### Else

-   이외에 모든 경우에 대해서는 break 가 걸리게되고 끝까지 못읽은 상태이다.

```
                if (dp == 0 || dst == null) {
                    dst = new byte[length << 1];
                } else {
                    byte[] buf = new byte[length << 1];
                    StringLatin1.inflate(dst, 0, buf, 0, dp);
                    dst = buf;
                }
                dp = decodeUTF8_UTF16(bytes, offset, sl, dst, dp, true);
                if (dp != length) {
                    dst = Arrays.copyOf(dst, dp << 1);
                }
                this.value = dst;
                this.coder = UTF16;
```

-   그런 경우에 대해서 전체 바이트 크기를 두배로 만들어내고 이미 읽은 바이트들을 순서에 맞게 집어넣는다.
-   만약에 \[1, 2, -3\] 이라는 바이트를 String 으로 변환한다면
    -   dst = \[1, 2, 0\] 인 상태에서 -3 을 읽고 break 가 걸려 다음으로 왔을 것이다.
    -   이후 buf = \[0, 0, 0, 0, 0, 0\] 길이 6의 바이트를 생성해내고 \[1, 0, 2, 0, 0, 0\] 상태로 만들어낸다.
    -   그렇기 때문에 음수가 존재할 때 \*\*양수 바이트들도 뒤에 0 을 가지게\*\* 된 것이다.
    -   음수에 대한 처리는 decodeUTF8\_UTF16() 메서드에서 이루어진다.
-   결과부터 말하자면 메서드 이름에서도 보이지만 UTF8 을 UTF16 방식으로 바꾼다.
-   이제 정말 마지막으로 해당 메서드를 들여다보자

### decodeUTF8\_UTF16

```
private static final char REPL = '\ufffd'

private static int decodeUTF8_UTF16(byte[] src, int sp, int sl, byte[] dst, int dp, boolean doReplace) {
        while (sp < sl) {
            int b1 = src[sp++];
            if (b1 >= 0) {
                StringUTF16.putChar(dst, dp++, (char) b1);
            } else if ((b1 >> 5) == -2 && (b1 & 0x1e) != 0) {
            ...
            }
            ...
        else {
            if (!doReplace) {
                throwMalformed(sp - 1, 1);
            }
            StringUTF16.putChar(dst, dp++, REPL);
        }
    }
    return dp;
```

-   결국 핵심 메서드는 StringUTF16.putChar 이다.
-   생략했지만 아래에서도 분기처리 후 하는 일은 항상 putChar 이다.

```
    private static native boolean isBigEndian();

    static final int HI_BYTE_SHIFT;
    static final int LO_BYTE_SHIFT;
    static {
        if (isBigEndian()) {
            HI_BYTE_SHIFT = 8;
            LO_BYTE_SHIFT = 0;
        } else {
            HI_BYTE_SHIFT = 0;
            LO_BYTE_SHIFT = 8;
        }
    }

...

@IntrinsicCandidate
    // intrinsic performs no bounds checks
    static void putChar(byte[] val, int index, int c) {
        assert index >= 0 && index < length(val) : "Trusted caller missed bounds check";
        index <<= 1;
        val[index++] = (byte)(c >> HI_BYTE_SHIFT);
        val[index]   = (byte)(c >> LO_BYTE_SHIFT);
    }
```

-   여기서 드디어 BE, LE 의 흔적을 찾을 수 있었다.
-   HI\_BYTE 이후 LO\_BYTE 를 집어넣기에 BE 로 착각할 수 있지만 아니다.
-   전역으로 미리 시스템이 LE 인지, BE 인지 파악하고 이에 맞게 처리한다.
    -   시스템이 BE 인지 LE 인지 알아보기 위해 다음 코드를 실행하였고 현재 JVM 은 LE 라는 것을 확인했다.

![image](https://github.com/min9805/min9805.github.io/assets/56664567/b13fb8bc-c109-4741-b531-22a4829a763e)

-   그리고 읽을 수 없는 문자들 (110, 10) 패턴이 지켜지지않은 음수 바이트들에 대해서는 REPL 을 반환한다.
-   REPL 은 0xFFFD 이므로 변환하면 \[-1, -3\] 으로 나타낼 수 있다.
-   우리가 처음에 봤었던 그 값이다!

### getByte()

다시 처음으로 돌아간다면 파일을 String 으로 읽고 바이트로 반환할 때에는 양수 뒤에 0 이나 \[-1, -3\] 같은 패턴을 찾아볼 수 없다. 

![image](https://github.com/min9805/min9805.github.io/assets/56664567/24186c2f-eb2a-400a-b2d2-4baca1612784)

위에서 다시 보자면 -119 에 대해서 \[-17, -65, -67\] 로 바뀌는 모습과 양수에 대해서는 그대로 나타내는 모습을 볼 수 있다. 

![image](https://github.com/min9805/min9805.github.io/assets/56664567/51a0f985-e037-4fcd-8149-dd0d38f93e14)

이를 디버깅을 통해 확인해보자.

-   생성된 String 은 "파이" 에 대해서는 \[-64, 3\], 양수 뒤에는 0, 음수는 \[-3, -1\] 값을 가진다.
-   getByte 에 들어있는 값이 처음에 본 값과 유사하다. getBytes() 를 통해서 바이트들이 아래와 같이 변환된다.
    -   \[-64, 3\] -> \[-49, -128\]
    -   \[1, 0\] -> 1
    -   \[-3, -1\] -> \[-17, -65, -67\]

그러면 진짜 마지막으로 getBytes() 를 열어보자

```
    /**
     * Encodes this {@code String} into a sequence of bytes using the
     * platform's default charset, storing the result into a new byte array.
     *
     * <p> The behavior of this method when this string cannot be encoded in
     * the default charset is unspecified.  The {@link
     * java.nio.charset.CharsetEncoder} class should be used when more control
     * over the encoding process is required.
     *
     * @return  The resultant byte array
     *
     * @since      1.1
     */
     
    public byte[] getBytes() {
        return encode(Charset.defaultCharset(), coder(), value);
    }
    
    ...
    
    private static byte[] encode(Charset cs, byte coder, byte[] val) {
        if (cs == UTF_8.INSTANCE) {
            return encodeUTF8(coder, val, true);
        }
        if (cs == ISO_8859_1.INSTANCE) {
            return encode8859_1(coder, val);
        }
        if (cs == US_ASCII.INSTANCE) {
            return encodeASCII(coder, val);
        }
        return encodeWithEncoder(cs, coder, val, true);
    }
    
    ...
    
    private static byte[] encodeUTF8(byte coder, byte[] val, boolean doReplace) {
        if (coder == UTF16)
            return encodeUTF8_UTF16(val, doReplace);

        if (!StringCoding.hasNegatives(val, 0, val.length))
            return Arrays.copyOf(val, val.length);

        int dp = 0;
        byte[] dst = new byte[val.length << 1];
        for (byte c : val) {
            if (c < 0) {
                dst[dp++] = (byte) (0xc0 | ((c & 0xff) >> 6));
                dst[dp++] = (byte) (0x80 | (c & 0x3f));
            } else {
                dst[dp++] = c;
            }
        }
        if (dp == dst.length)
            return dst;
        return Arrays.copyOf(dst, dp);
    }
```

결국 파고 들어가보면 아까는 본 decodeUTF\_8\_UTF16 과 유사한 메서드가 존재한다. 당연히 이번에는 encode를 해야하기에.

```
    @IntrinsicCandidate
    // intrinsic performs no bounds checks
    static char getChar(byte[] val, int index) {
        assert index >= 0 && index < length(val) : "Trusted caller missed bounds check";
        index <<= 1;
        return (char)(((val[index++] & 0xff) << HI_BYTE_SHIFT) |
                      ((val[index]   & 0xff) << LO_BYTE_SHIFT));
    }
    
    private static byte[] encodeUTF8_UTF16(byte[] val, boolean doReplace) {
        int dp = 0;
        int sp = 0;
        int sl = val.length >> 1;
        byte[] dst = new byte[sl * 3];
        while (sp < sl) {
            // ascii fast loop;
            char c = StringUTF16.getChar(val, sp);
            if (c >= '\u0080') {
                break;
            }
            dst[dp++] = (byte)c;
            sp++;
        }
        while (sp < sl) {
            char c = StringUTF16.getChar(val, sp++);
            if (c < 0x80) {
                dst[dp++] = (byte)c;
            } else if (c < 0x800) {
                dst[dp++] = (byte)(0xc0 | (c >> 6));
                dst[dp++] = (byte)(0x80 | (c & 0x3f));
            } else if (Character.isSurrogate(c)) {
                int uc = -1;
                char c2;
                if (Character.isHighSurrogate(c) && sp < sl &&
                        Character.isLowSurrogate(c2 = StringUTF16.getChar(val, sp))) {
                    uc = Character.toCodePoint(c, c2);
                }
                if (uc < 0) {
                    if (doReplace) {
                        dst[dp++] = '?';
                    } else {
                        throwUnmappable(sp - 1);
                    }
                } else {
                    dst[dp++] = (byte)(0xf0 | ((uc >> 18)));
                    dst[dp++] = (byte)(0x80 | ((uc >> 12) & 0x3f));
                    dst[dp++] = (byte)(0x80 | ((uc >>  6) & 0x3f));
                    dst[dp++] = (byte)(0x80 | (uc & 0x3f));
                    sp++;  // 2 chars
                }
            } else {
                // 3 bytes, 16 bits
                dst[dp++] = (byte)(0xe0 | ((c >> 12)));
                dst[dp++] = (byte)(0x80 | ((c >>  6) & 0x3f));
                dst[dp++] = (byte)(0x80 | (c & 0x3f));
            }
        }
        if (dp == dst.length) {
            return dst;
        }
        return Arrays.copyOf(dst, dp);
    }
```

-   encode 를 시작하면서 ASCII 코드로 변환시킬 수 있는 바이트들은 바로 변환시킨다. 
    -   이때에도 getChar 를 통해 2바이트 \[65, 0\] 을 한 번에 읽어 "65" 로 만들어준다.
-   이후 2바이트, 혹은 3바이트까지 필요한 경우 모두 파악하여 인코딩한다
-   읽지 못하는 값의 경우 \[-3, -1\] 은 \[-17, -65, -67\] 로 변환시키는 것이다.

---

## 결론

1.  바이트 파일을 기본 String 으로 읽는다면 음수 바이트에 대해서 유니코드로 변환시키지 못하기 때문에 \[-3, -1\] 로 변환시킨다.
    -   이후 getBytes() 를 통해 다시 바이트로 변경 시에는 해당 바이트는 \[-17, -65, -67\] 로 변환된다.
    -   한 바이트라도 표현하지 못하거나 표현가능한 유니코드가 255가 넘어가면 UTF-16 으로 디코딩된다. 
    -   이때 UTF-16BE, UTF-16LE 는 시스템에 따라서 결정된다.  
2.  파일이 문자열로 표현 가능한 경우 (html, css, svg ..) 에는 음수 바이트가 존재하지 않기에 정상 동작한다.

-   추가적으로 UTF-8 로 데이터를 읽기에 깨지는 것이므로 1바이트 그대로를 읽어내는 ISO\_8859\_1 방법을 사용하면 바이트 코드들을 따로 해석하지 않고 그대로 사용하기에 파일을 정상적으로 읽어올 수 있다. 

```
try (BufferedReader reader = new BufferedReader(new FileReader(file, StandardCharsets.ISO_8859_1))) {
    String line;
    while ((line = reader.readLine()) != null) {
        contentBuilder.append(line).append("\n");
    }
}
```