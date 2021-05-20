## 목차
- [Acceptance Test (인수 테스트)](#acceptance-test-인수-테스트)
  - [인수 테스트란?](#인수-테스트란)
  - [LineAcceptanceTest 예제](#lineacceptancetest-예제)
  - [vs 통합테스트](#vs-통합테스트)


# Acceptance Test (인수 테스트)

## 인수 테스트란?
인수(Acceptance) 즉, 무엇인가를 받아서 테스트하는 것
무엇을 받을까?? -> __시나리오(요구사항 명세서)!!__

개발자, 기획자, 사장 등... 이 주는 시나리오를 잘 통과하는지 테스트
이 때 환경은 실제 사용자 환경에서 테스트
제품의 __결함__ 을 찾기 보다는 제품의 기능을 테스트하며 완성도를 확인하는 것이 목적 -> 단위 테스트와는 다르다

## LineAcceptanceTest 예제

```java
public class LineAcceptanceTest extends AcceptanceTest {

    private final LineRequest line2Request;
    private final LineRequest line3Request;
    private final StationRequest gangnamStationRequest;
    private final StationRequest yeoksamStationRequest;

    public LineAcceptanceTest() {
        this.line2Request = new LineRequest("2호선", "bg-red-600", 1L, 2L, 4);
        this.line3Request = new LineRequest("3호선", "bg-red-600", 1L, 2L, 5);
        this.gangnamStationRequest = new StationRequest("강남역");
        this.yeoksamStationRequest = new StationRequest("역삼역");
    }

    @Test
    @DisplayName("지하철 노선 목록을 보여준다.")
    void showLines() {
        // given
        ExtractableResponse<Response> createResponse1 = createStation(gangnamStationRequest);
        ExtractableResponse<Response> createResponse2 = createStation(yeoksamStationRequest);
        ExtractableResponse<Response> createResponse3 = createLine(line2Request);

        // when
        ExtractableResponse<Response> response = RestAssured.given().log().all()
                .when()
                .get("/lines")
                .then().log().all()
                .extract();

        // then
        assertThat(response.statusCode()).isEqualTo(HttpStatus.OK.value());
        List<Long> expectedLineIds = Stream.of(createResponse1)
                .map(it -> Long.parseLong(it.header("Location").split("/")[2]))
                .collect(Collectors.toList());
        List<Long> resultLineIds = response.jsonPath().getList(".", LineResponse.class).stream()
                .map(LineResponse::getId)
                .collect(Collectors.toList());
        assertThat(resultLineIds).containsAll(expectedLineIds);
    }

    private ExtractableResponse<Response> createLine(LineRequest lineRequest) {
        return RestAssured.given().log().all()
                .body(lineRequest)
                .contentType(MediaType.APPLICATION_JSON_VALUE)
                .when()
                .post("/lines")
                .then().log().all()
                .extract();
    }

    private ExtractableResponse<Response> createStation(StationRequest stationRequest) {
        return RestAssured.given().log().all()
                .body(stationRequest)
                .contentType(MediaType.APPLICATION_JSON_VALUE)
                .when()
                .post("/stations")
                .then().log().all()
                .extract();
    }
}

```

__지하철 노선 목록을 보여주는 예제__
> 참고: 프론트는 포함하지 않아 RestAssured로 직접 요청을 보내고 있다.

현재 테스트는 역 2개와 노선 1개가 있을 때 노선 목록을 불러오는 시나리오를 테스트하고 있다. 이처럼 기능을 테스트하며 완성도를 테스트하는 것이 인수 테스트이다.

## vs 통합테스트
인수 테스트를 정리하면서 Controller 테스트 또는 통합테스트와 비슷하다는 것을 느꼈다.
결론부터 말하자면 인수 테스트와 통합 테스트에서 테스트 내용은 비슷해질 수 있다.
그러나 통합 테스트는 각 모듈을 합쳐서 잘 작동하는 지 테스트하는 것이다.
즉, 단위 테스트 이후 각 모듈들의 상호 작용이 제대로 이루어지는 지 검증하고, 모듈을 통합하는 과정에서 발생할 수 있는 오류를 찾는 테스트이다.

그러나 인수 테스트는 앞서 말했듯이 __시나리오(요구사항)__ 을 테스트하기 위해 만들어졌기 때문에 __결함__ 보다는 __완성도__ 가 목적이다. 따라서 목적이 다르다고 할 수 있다.

사실 인수 테스트 의미만 보면 제일 큰 범위인 것 같다.
(극단적으로 보면 단위 테스트 또한 시나리오니 인수테스트 일까..? -> 완성도가 목적이니 아닌 듯함!)
