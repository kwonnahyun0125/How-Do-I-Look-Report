# 개발 보고서

**프로젝트명:** How Do I Look?  
**팀명:** 1팀  
**작성자:** 권나현  
**작성일:** 2025.06.19  

---

## 1. 프로젝트 개요

### 1-1 목적
사용자들이 자신의 패션 스타일을 등록하고, 다양한 태그/카테고리/이미지와 함께 공유하며, 타인의 큐레이션(피드백)을 받을 수 있는 스타일 플랫폼 구축

### 1-2 주요 기능
- 스타일 등록/수정/삭제/조회 (Style CRUD)  
- 태그, 이미지, 카테고리(상의/하의/신발 등) 중첩 입력  
- 큐레이션(리뷰) 등록 및 스타일-큐레이션 1:N 관계 관리  
- 프론트엔드/백엔드 분리 개발 및 API 명세에 맞는 데이터 포맷 통일  

---

## 2. 주요 개발 내역

### 2-1 백엔드(Express + Prisma)
- Prisma ORM을 활용한 DB 모델 설계 및 마이그레이션  
- Style, Category, Tag, Image, Curation 등 관계형 모델 정의 및 1:N, N:M 관계 구현  
- REST API 개발  
- 스타일 등록/수정 시 name 기반 태그 자동 생성(findOrCreate)  
- 이미지 URL 등록 시 Image 테이블 자동 저장 및 연결  
- 클라이언트 요청 형태(JSON)에 맞춰 categories 필드를 객체 → 배열로 변환  
- BigInt 타입(price 등) JSON 변환 처리  
- superstruct 기반 DTO 미들웨어로 모든 요청/응답 타입 유효성 검증  
- 시드(Seed) 데이터 구축  
- 스타일 10개, 태그/이미지/카테고리/큐레이션 데이터 대량 삽입 스크립트 작성  
- 마이그레이션 및 시드 자동화(npx prisma migrate reset --force)  
- 에러 핸들링 및 로깅  
  - 글로벌 에러 핸들러(error-middleware.js) 구현  
  - 에러 상세 로그 및 DB 저장, 사용자에겐 일관된 메시지 제공  

### 2-2 프론트엔드/테스트
- Postman 등 API 테스트  
- API 명세서에 따라 다양한 시나리오로 POST/PUT/GET 테스트  
- 프론트엔드 팀원과 연동, 실제 데이터 흐름 점검  
- 프론트엔드 연동 지원  
- 데이터 구조(categories, tags, imageUrls 등) 프론트 요구사항과 맞춰 백엔드 응답 포맷 개선  
- 예상 오류(예: undefined map/length) 등 백엔드 차원에서 사전 방어 처리  

---

## 3. **__Style API 개발 이슈 및 해결 내역__**

**1). 불필요한 텍스트로 인한 문법 오류**  
- 이슈: createCuration 함수 내부에 "Add commentMore actions" 텍스트가 삽입되어 문법 오류 발생  
- 해결: 자동 삽입된 불필요한 텍스트 제거  

**2. 스타일 수정 시 이미지 누락**  
- 이슈: 수정 요청 후 기존 이미지가 삭제되어 응답에서 누락됨  
- 원인: deleteMany → create 방식 사용 시 기존 순서 및 정보 유지 안 됨  
- 해결: 입력된 imageUrls 기준으로 이미지 재생성 및 정렬하여 응답 반환  

**3. API 명세서 불일치 응답 구조**  
- 이슈: 응답 필드명이 명세서와 달라 프론트에서 처리 오류 발생  
- 해결: id, nickname, imageUrls, categories 등 명세서 기준으로 구조 수정  

**4. 스타일 삭제 시 외래키 제약 오류**  
- 이슈: Category_styleId_fkey 제약 조건으로 삭제 불가  
- 해결: 연관 테이블 데이터(category, styleTag, styleImage) 삭제 후 style 삭제  

**5. 에러 로그 구조 개선**  
- 이슈: 에러 로그가 단순 문자열로 저장됨  
- 해결: Log.detail 필드를 JSON.stringify 구조로 저장하여 가독성 및 분석성 향상  

**6. 잘못된 관계 참조로 인한 오류**  
- 이슈: Prisma 쿼리에서 존재하지 않는 comments 관계를 include하여 오류 발생  
- 해결: 해당 관계 제거 및 include 정리  

**7. error-middleware 내 DB 객체 누락**  
- 이슈: db.log.create() 호출 시 db 객체 선언 누락으로 오류 발생  
- 해결: import db from '../config/db.js' 추가  

**8. 스타일 관련 파일명 명확화**  
- 작업: style.js → style-controller.js로 변경하여 역할 명확히 구분  

**9. 함수명 정리 및 오타 수정**  
- 작업: getStyles, createStyle 등 오타 및 불명확한 함수명을 명확하게 정리  

**10. 요청 유효성 검증 미들웨어 적용 및 효과**  
- 작업: styleRouter에 validateRequest() 미들웨어 적용하여 요청 유효성 검증 로직 도입  
- 사용 기술: superstruct 기반 DTO 스키마(createStyleSchema, updateStyleSchema 등)  
- 해결한 문제:  
  - 클라이언트에서 잘못된 파라미터 전송 시 API 오작동  
  - controller 진입 전 오류 차단으로 예외 감소 및 코드 안정성 향상  

**11. Seed 데이터 구성**  
- 작업: tag, style, image, curation, comment 테스트용 Seed 데이터 삽입 스크립트 구성 및 실행  

**12. 스타일 등록/수정 전처리 로직 추가**  
- 작업:  
  - categories를 객체 또는 배열로 유연하게 처리하도록 전처리 로직 구현  
  - 이미지 연동 시 StyleImage 조인 테이블 기반 구조 수정  

**13. 코드 스타일 정리**  
- 작업: ESLint 기반 코드 포맷 정리 및 오타 수정  

**14. 태그 기능 연동**  
- 작업: 태그를 문자열 배열로 입력받아 DB에 findOrCreate 방식으로 연동  

**15. Prisma 모델링 및 초기 마이그레이션**  
- 작업: ERD 기반 schema.prisma 모델 정의 및 마이그레이션 완료  

**16. Style API 전체 구현**  
- 작업:  
  - POST /styles (등록)  
  - GET /styles (목록 조회)  
  - GET /styles/:id (상세 조회 + 조회수 증가)  
  - PUT /styles/:id (수정)  
  - DELETE /styles/:id (삭제)  
  - 전체 API 명세서 구조에 맞게 응답 형식 및 예외 처리 구현 완료  

**17. BigInt 타입 JSON 직렬화 오류**  
- 이슈: Prisma에서 price 등의 필드가 BigInt로 정의되어 JSON 응답 시 직렬화 에러 발생  
- 해결: JSON.stringify() 시 BigInt를 toString()으로 변환하는 jsonBigIntReplacer() 함수 도입 → 프론트엔드에서는 문자열로 받은 후 필요 시 Number() 또는 parseInt()로 처리  

**18. 태그 및 이미지 연결 구조 설계**  
- 이슈: 클라이언트에서 태그 id가 아닌 태그 이름만 전달, 중복 태그/이미지 데이터 생성 가능성  
- 해결: getOrCreateTagIds() 유틸 함수로 태그 이름 기준 findOrCreate 처리  
  - 이미지 URL도 imageUrl만 전달받아 내부적으로 Image 테이블에 생성 및 연결  
  - 중복 저장 없이 자동 연결 구조 구현  

**19. 카테고리 포맷 불일치**  
- 이슈: 클라이언트에서는 categories를 객체로 전송하나, DB에는 배열로 저장 필요  
- 해결:  
  - DTO 미들웨어에서 객체 → 배열로 전처리  
  - 백엔드에서도 객체/배열 모두 처리 가능하도록 유연한 로직 구현  
  - API 명세서와 실제 동작 일치시켜 사용자 혼란 방지  

**20. 팀 협업 전략**  
- 이슈: 여러 명이 동시에 개발하며 merge 충돌 및 코드 관리 어려움  
- 해결:  
  - feature/task-xxx 브랜치 전략 도입 및 Pull Request 리뷰 기반 협업  
  - 커밋 메시지 컨벤션 및 작업 단위 명확화 (feat, fix, refactor 등)  
  - 기능 구현 완료 후 작업 내용 및 이슈를 GitHub Issues에 기록  
  - 프론트/백/DB 역할을 명확히 분리하고 매일 소통하여 병목 없이 진행  

---

## 4. **_성과 및 배운 점_**

- _실전 백엔드 설계-구현-테스트 전주기 경험_  
- _API 명세/DB모델 설계, DTO/유효성 검증, 관계형 DB모델 이해 및 시드 데이터 활용법 숙지_  
- _협업 과정에서 소통의 중요성, 오류/이슈 발생 시 빠른 로그 분석 및 근본적 원인 해결 능력 배양_  
- _실무와 유사한 코드 리뷰, 브랜치 전략, PR/merge 작업 경험_  

---

## 5. 향후 과제 및 개선 방향
- Swagger 기반 자동 API 문서화  
- 대량 데이터/트래픽 테스트 및 최적화  
- 배포 모니터링 적용  
- 배포/CI/CD 경험 확장 및 모니터링 적용  
- 코드 리팩토링, 테스트 코드 추가, 성능 로그 및 에러 추적 강화  

---

## 6. 구체적 개발 예시

### 6.1 스타일(Style) 등록 요청 예시
- ■ API 요청/응답 예시 (POST /styles)
```json
{
  "nickname": "나현",
  "password": "password1234",
  "title": "여름 데일리룩",
  "content": "시원하게 입는게 최고!",
    "categories": {
    "top": {
      "name": "반팔티",
      "brand": "유니클로",
      "price": 19900
    },
    "bottom": {
      "name": "숏팬츠",
      "brand": "지오다노",
      "price": 15900
    }
  },
  "tags": ["캐주얼", "여름", "미니멀"],
  "imageUrls": [
    "https://img.example.com/style1.jpg",
    "https://img.example.com/style2.jpg"
  ]
}
```

■ API 요청/응답 예시 (201 Created)
```json
{
  "id": 1,
  "nickname": "나현",
  "title": "여름 데일리룩",
  "content": "시원하게 입는게 최고!",
  "viewCount": 0,
  "curationCount": 0,
  "createdAt": "2025-06-18T09:12:37.365Z",
  "categories": {
    "top": {
      "name": "반팔티",
      "brand": "유니클로",
      "price": 19900
    },
    "bottom": {
      "name": "숏팬츠",
      "brand": "지오다노",
      "price": 15900
    }
  },
  "tags": [
    "캐주얼",
    "미니멀",
    "여름"
  ],
  "imageUrls": [
    "https://img.example.com/style1.jpg",
    "https://img.example.com/style2.jpg"
  ]
}
```

  

### 6.3 주요 커밋/PR 내역
- feat: Style API 기능 구현
- feat: prisma.
- feat: seed 데이터 및 스타일/큐레이션 10개 샘플 추가  
  - prisma/seed.js에 더미 데이터 추가 및 시딩 자동화
- fix: style 라우터에 DTO 미들웨어 적용(수정)
- fix: Style FE 에 맞춰서 DTO 제한 사항 부분 수정
- feat: 에러 로그 DB 저장 구조 개선 (Log.detail 구조화)
- fix: style 삭제 시 외래키 제약 오류 해결
- fix: style 응답을 API 명세서 형식에 맞게 수정
- fix: style 수정 시 이미지 누락 이슈 수정
- fix: seed 데이터 comment 추가
- fix: seed 데이터 test 이미지 실사용 이미지 URL로 변환
- fix: 패스워드 최소길이 8자로 수정되어 seed 데이터 비밀번호 수정
- 
  

### 6.4 API 시나리오 기반 테스트(일부)
- Postman/프론트에서 실제 등록/수정/조회 반복 테스트 진행  
- 엣지 케이스: 태그 없이 등록, 이미지 없이 등록, 잘못된 price 값(숫자/문자), 비밀번호 틀림, 없는 styleId 등  

---

## 7. 참고자료/문서
- [팀 GitHub Repo 링크](https://github.com/gyunam-bark/nb02-how-do-i-look-team1)  
- [API 명세 Wiki](https://github.com/gyunam-bark/nb02-how-do-i-look-team1/wiki/%5BAPI%5DStyle)  
- [DB ERD](https://github.com/gyunam-bark/nb02-how-do-i-look-team1/blob/main/resource/erd.png)  
- [데이터베이스 Seed](https://github.com/gyunam-bark/nb02-how-do-i-look-team1/wiki/%5B%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%5DSeed)

![스크린샷 2025-06-18 153807](https://github.com/user-attachments/assets/d38021c4-a76c-4879-9137-8ef3ab2ee1fc)
![스크린샷 2025-06-18 153820](https://github.com/user-attachments/assets/ce506635-dab1-463b-8ed6-368776b927d7)

- [Postman 테스트 스크린샷]  
![스크린샷 2025-06-18 181322](https://github.com/user-attachments/assets/ae03d918-8e6d-4588-8e64-8e6bb8e0139e)
![스크린샷 2025-06-13 160019](https://github.com/user-attachments/assets/1b353689-da70-4206-998f-ff469c9e2db4)

