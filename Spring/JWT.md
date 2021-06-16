## 목차
- [JWT](#jwt)
  - [JWT란?](#jwt란)
  - [JWT의 모양](#jwt의-모양)
    - [Header](#header)
    - [Payload](#payload)
    - [Signature](#signature)
  - [참고 자료](#참고-자료)


# JWT

## JWT란?
JWT는 JSON Web Token의 약자로 [웹표준](https://datatracker.ietf.org/doc/html/rfc7519)으로서 두 개체에서 JSON 객체를 사용하여 정보를 안정성 있게 전달해줍니다.

## JWT의 모양
![](images/2021-05-21-00-54-18.png)

> JWT 토큰을 만들 때는 JWT를 담당하는 라이브러리가 자동으로 인코딩 및 해싱 작업을 해줍니다.

### Header
- Header는 두가지 정보를 지니고 있습니다.
- type : 토큰의 타입을 지정합니다. 여기서는 `JWT` 입니다.
- alg : 해싱 알고리즘을 지정합니다. 위의 그림에서는 HS256(HMAC SHA256)을 사용하고 있습니다. 일반적으로 HS256 또는 RSA를 사용합니다. 이 알고리즘은 토큰을 검증할 때 사용되는 signature 부분에서 사용됩니다.
- 위의 그림에 보이는 것처럼 Json 형태로 되어있는 Header를 `base64`로 인코딩하면 1번 파란색 문자열이 나옵니다.
- 이렇게 JWT 파트 중 첫 번째 파트가 완성이 됩니다.

### Payload
- Payload 부분에는 토큰에 담을 정보가 들어있습니다.
- 그림에서는 3가지 정보가 들어있는데 이 정보 하나를 클레임(claim)이라고 부르고 이는 name/ value의 한쌍으로 이루어져있습니다.
- 토큰에는 여러 개의 클레임을 넣을 수 있습니다.
- 클레임의 종류는 종 3가지로 분류됩니다.
    - registered 클레임
    - public 클레임
    - private 클레임
- 클레임에 대한 자세한 정보는 [이곳](https://velopert.com/2389)

### Signature
- JWT의 마지막 파트인 signature는 헤더의 인코딩 값과, Payload의 인코딩 값을 합쳐 주어진 비밀키로 해쉬하여 생성합니다.
- 주어진 header와 payload를 base64로 인코딩한 값을 비밀키를 이용하여 해쉬합니다.

이 세 부분을 합치면 JWT 토큰이 나오게 됩니다. 각각 파트는 `.`으로 구분합니다.

## 참고 자료
- https://velopert.com/2389