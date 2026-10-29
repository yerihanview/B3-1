# L1 — IAM 최소권한 설정 및 검증 결과

## 1. 실습 목표

- 루트 계정 대신 IAM 사용자로 AWS 리소스 관리
- B3-1 실습에 필요한 EC2·VPC 권한 부여
- 불필요한 IAM 관리 권한 제한
- MFA를 통한 로그인 보안 강화
- AWS CLI 브라우저 인증 및 권한 검증

## 2. IAM 구성

| 항목 | 설정 |
|---|---|
| IAM 사용자 | `b3-1-operator` |
| IAM 그룹 | `b3-1-operators` |
| 사용자 지정 정책 | `B3-1-EC2-VPC-Policy` |
| CLI 로그인 정책 | `SignInLocalDevelopmentAccess` |
| 비밀번호 변경 정책 | `IAMUserChangePassword` |
| AWS 리전 | `ap-northeast-2` |

정책 연결 구조:

- `B3-1-EC2-VPC-Policy`: 그룹에 연결
- `SignInLocalDevelopmentAccess`: 그룹에 연결
- `IAMUserChangePassword`: 사용자에게 직접 연결

사용자가 그룹에 정상적으로 소속되어 있는 것을 확인했다.

## 3. MFA 설정

- 루트 계정: Passkey MFA 등록 완료
- IAM 사용자: Passkey MFA 등록 완료
- IAM 사용자 MFA 로그인 성공
- 루트 계정 액세스 키: 없음

## 4. 권한 검증

| 검증 항목 | 결과 |
|---|---|
| AWS 콘솔 EC2 Instances 조회 | 성공 |
| IAM `ListUsers` | Access denied |
| IAM `GetAccountSummary` | Access denied |
| IAM `ListAccountAliases` | Access denied |
| IAM 사용자 MFA 로그인 | 성공 |
| AWS CLI `sts get-caller-identity` | 성공 |
| AWS CLI `ec2 describe-instances` | 성공 |
| EC2 인스턴스 목록 | 비어 있음 |

IAM 관리 정보 조회가 거부되어, 불필요한 IAM 권한이 부여되지 않았음을 일부 확인했다.

EC2·VPC 생성·수정·삭제 권한은 실제 리소스 작업이 이루어지는 L2~L3에서 추가 검증한다.

## 5. AWS CLI 설정

인증 방식: `aws login` 브라우저 기반 인증

프로필 이름: `b3-1`

확인된 인증 주체: `b3-1-operator`

검증 명령:

```powershell
aws sts get-caller-identity --profile b3-1
aws ec2 describe-instances --region ap-northeast-2 --profile b3-1
```

두 명령 모두 정상적으로 실행되었다.

## 6. 실습 중 발생한 문제

**문제 1 — IAM Security credentials 접근 거부**

- 원인: IAM 사용자에게 일부 본인 자격 증명 조회 권한이 없었음
- 조치: 루트 계정에서 IAM 사용자 MFA 등록
- 결과: MFA 등록 및 로그인 성공

**문제 2 — IAM 사용자 그룹 소속 누락**

- 증상: 사용자의 Groups 탭에 소속 그룹 없음
- 조치: `b3-1-operators` 그룹에 사용자 추가
- 결과: 그룹 정책 2개 연결 확인

**문제 3 — AWS CLI 인증 오류**

- 초기 인증 주체: 루트 계정
- 조치: `aws logout` 후 IAM 사용자로 재인증
- 추가 문제: Windows 로컬 인증 콜백 연결 중단
- 해결: `aws login --remote` 사용
- 결과: CLI 인증 주체가 `b3-1-operator`임을 확인

## 7. 증빙 파일

- `L1-root-mfa.png`
- `L1-operator-mfa.png`
- `L1-ec2-read.png`
- `L1-iam-denied.png`
- `L1-cli-identity.png`
- `L1-cli-ec2-read.png`

## 8. 남은 확인 사항

- EC2·VPC 실제 생성·수정·삭제 권한 검증
- 리소스별 최소권한 정책의 추가 제한 검토
- 최종 증빙의 민감정보 마스킹

## 9. 결론

IAM 사용자 기반 콘솔 로그인, MFA, AWS CLI 인증 및 기본 조회 권한 검증을 완료했다.

관리자 권한을 부여하지 않고 실습용 IAM 환경을 구성했으며, 실제 인프라 관리 권한은 L2 이후 단계에서 검증한다.
