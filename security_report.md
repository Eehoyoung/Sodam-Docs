# SSRF 보안 이슈 발견

StoreController의 2개의 메소드에서 현재 SSRF 보안 이슈가 포착되었습니다.

## 1번째 이슈

1. 45번 라인 (@Parameter(description = "매장 등록 정보", required = true) @RequestBody StoreRegistrationDto storeDto)

- Source: can return tainted content crafted by a user

2. 48번 라인 (storeDto.getQuery())

- This call propagates tainted data to its return value

3. 48번 라인(geocodingService.getCoordinates())

- This call propagates tainted data to its return value

4. GeocodingService의 30번 라인

- Tainted data is passed as a function parameter "address"

5. GeocodingService의 32번 라인

- Tainted data assigned to variable "encodeAddress" from a call

6. GeocodingService의 34번 라인

- Tainted data assigned to variable "url"

7. GeocodingService의 40번 라인

- Sink SSRF: tainted data lands here

## 2번째 이슈

StoreController의 154번 라인부터 시작하여 1번째 이슈 절차와 똑같습니다.
