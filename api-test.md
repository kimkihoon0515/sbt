📘 Springdoc

개요
	•	Spring Boot 기반 프로젝트에서 OpenAPI 3 명세를 자동으로 생성해주는 라이브러리
	•	springdoc-openapi는 기존 Spring REST API 또는 Spring Data REST 엔드포인트를 분석하여 Swagger UI와 OpenAPI 명세 (/v3/api-docs)를 자동으로 생성

특징
	•	자동 문서화: Controller, DTO, Request/Response 등을 기반으로 명세 자동 생성
	•	Swagger UI 연동: /swagger-ui.html 주소에서 API 테스트 가능
	•	Spring Data REST 호환: Repository 기반 REST API도 자동 문서화
	•	추가 정보 커스터마이징 가능: @Schema, @Parameter, @Operation, @ApiResponse, @ExampleObject 등을 통해 OpenAPI 문서 보완 가능

단점
	•	example 값, components.schemas 같은 정보는 기본으로 생성되지 않음 → 수동 주석 필요
	•	복잡한 nested 구조나 제네릭 클래스에 대한 표현이 부족할 수 있음

⸻

🛠 MSW (Mock Service Worker)

개요
	•	브라우저/Node 환경에서 네트워크 요청을 가로채서 mock 응답을 반환해주는 라이브러리
	•	주로 프론트엔드 개발에서 실제 백엔드 없이도 API 응답을 테스트하거나 개발할 수 있게 해주는 도구

특징
	•	Service Worker 기반: 브라우저에서 실제 API 요청을 가로채서 지정한 응답 반환
	•	Node 환경에서도 사용 가능 (msw/node)
	•	OpenAPI 명세 기반 핸들러 자동 생성 가능 (msw-auto-mock 등 활용)
	•	프론트엔드와 API 병렬 개발에 적합

장점
	•	사용법이 간단하고 빠르게 mock 서버 구현 가능
	•	테스트에서도 동일한 mock handler를 재사용 가능

단점
	•	OpenAPI만으로는 충분한 mock 응답 생성이 어려움 → example 필드나 스키마 정보 필요
	•	null 반환 문제: response schema가 명확하지 않거나 example이 없을 경우 null 반환이 기본

⸻

🔬 Schemathesis

개요
	•	OpenAPI/Swagger 명세를 기반으로 자동화된 테스트 케이스를 생성해주는 Python 기반 도구
	•	주로 백엔드 API의 유효성, 안정성, 에러 응답 등을 검증하는 데 사용

특징
	•	OpenAPI 2.0 / 3.0 / 3.1 지원
	•	API의 정상/비정상 요청을 자동 생성 → 엣지 케이스 테스트에 유용
	•	pytest와 통합 가능
	•	Property-based Testing (Hypothesis 기반)

장점
	•	API 안정성 확보에 매우 유용 (특히 fuzzing 테스트에 적합)
	•	명세에 기반한 완전 자동화 테스트 가능
	•	CLI 기반이라 CI에 쉽게 통합 가능

단점
	•	Python 기반 → Java/Spring 프로젝트에서는 별도 환경 필요
	•	OpenAPI 명세가 미흡하면 테스트 효과가 떨어짐