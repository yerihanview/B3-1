# L0 — AWS 계정 및 실습 환경 점검

## 1. AWS 계정

- 계정 생성일: 2026-10-28
- 리전: Asia Pacific (Seoul)
- 리전 코드: `ap-northeast-2`
- 플랜: AWS Free Plan
- Free Plan 종료일: 2027-04-28

## 2. 비용 및 크레딧

- 최초 Free Tier 크레딧: $100
- 추가 AWS Budgets 크레딧: $20
- 총 잔여 크레딧: $120 (확인 시점)
- 예산: `B3-1-Zero-Spend`
- 비용 알림: Actual cost > $0.01
- 이메일 수신자 인증: Verified

주의: AWS Budgets는 비용 발생을 차단하지 않고 알림을 제공한다.

## 3. Windows 실습 환경

- 운영체제: Windows 10
- OpenSSH: 9.5p1
- AWS CLI: 2.39.0
- Git: 2.53.0
- VS Code: 1.135.0
- AWS CLI 인증: 미설정 (L1에서 진행)

## 4. 보안 및 증빙 관리

- `.gitignore` 생성
- 원본 `evidence/` Git 추적 제외 설정
- AWS 인증정보 및 SSH 개인 키 제출 금지

## 5. 미완료 및 후속 작업

- IAM 최소권한 사용자/정책 설정: L1
- 도메인 및 HTTPS 준비: L5
- AWS 리소스 생성: 아직 시작하지 않음
- 증빙 수집 자동화: 단계별로 구현·검증 예정
