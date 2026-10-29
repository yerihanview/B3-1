# B3-1 L7 — 최종 통합 검증표 (E23)

- 검증일: 2026-10-29
- AWS 리전: `ap-northeast-2`
- AWS CLI 프로필: `b3-1`
- 검증 방법: AWS CLI 조회, EC2 SSH 명령, HTTP/HTTPS 요청, 외부 브라우저 접속
- 증빙 기준 경로: `evidence/07-final-verification/`
- 상태: **대부분 검증 완료 — IAM 정책의 상세 권한 범위는 미검증**

## 1. 인프라 및 네트워크

| 대상 | 검증 결과 | 판정 | 증빙 파일 |
|---|---|---|---|
| VPC | `vpc-0442e2bd8ce7ff488`, `10.0.0.0/16`, `available`, 비기본 VPC | 통과 | `L7-vpc.png` |
| Subnet | `subnet-0543e9e86dee40c3d`, `10.0.1.0/24`, `available`, 지정 VPC 소속 | 통과 | `L7-subnet.png` |
| Internet Gateway | `igw-067455c9fc1f5aee8`, 지정 VPC에 `available` 상태로 연결 | 통과 | `L7-igw.png` |
| Route Table | `rtb-0d27123bfa1fd03fe`, `0.0.0.0/0 → IGW`, `active` | 통과 | `L7-route-table.png` |
| Subnet 라우팅 | Subnet에 명시적 Route Table 연결 없음(`[]`); VPC Main Route Table 사용 | 통과 | `L7-subnet-routing.png` |
| EC2 | `i-0fabb53623ab701ca`, `t3.micro`, `running`, Public IP `3.38.97.202`, Private IP `10.0.1.191` | 통과 | `L7-ec2.png` |
| EBS | `vol-0c4580477a8d7380c`, `gp3` 8GiB, `in-use`, 지정 EC2에 `attached`; `DeleteOnTermination=true` | 통과 | `L7-ebs.png` |
| Security Group | `sg-0acdbff1f820fee1b`: SSH 22 특정 공인 IPv4 `/32` 제한, HTTP 80·HTTPS 443 `0.0.0.0/0` 허용 | 통과 | `L7-security-group.png`, `L7-security-group-ssh.png` |

> 참고: Subnet의 `MapPublicIpOnLaunch=False`는 기존 EC2에 퍼블릭 IPv4가 할당되어 있다는 검증 결과와 모순되지 않는다. Public Subnet 여부는 Main Route Table의 IGW 경로로 확인했다.

## 2. IAM 및 접근 제어

| 대상 | 검증 결과 | 판정 | 증빙 파일 |
|---|---|---|---|
| CLI 인증 주체 | L1 요약에 `b3-1-operator`로 확인 기록; L7 `aws login` 시 동일 사용자 인증 확인 | 통과 | `../01-account-iam/L1-cli-identity.png` (기존 L1 자료) |
| IAM 그룹·정책 구성 | L1 요약에 `b3-1-operators`, `B3-1-EC2-VPC-Policy`, `SignInLocalDevelopmentAccess`, `IAMUserChangePassword` 기록 | 기록 확인 | `../01-account-iam/L1-summary.md` |
| MFA | L1 요약에 IAM 사용자·루트 MFA 등록 및 IAM MFA 로그인 성공 기록 | 기록 확인 | `../01-account-iam/L1-operator-mfa.png`, `../01-account-iam/L1-root-mfa.png` |
| 불필요한 IAM 조회 제한 | `iam:ListAttachedUserPolicies`, `iam:ListGroupsForUser` 거부. L1에는 다른 IAM 관리 조회 거부 기록 | 통과(거부 사실) | `L7-iam-access-denied.png`, `L7-iam-groups-access-denied.png`, `../01-account-iam/L1-iam-denied.png` |
| 최소권한 정책의 전체 세부 범위 | 정책 JSON/전체 유효 권한을 L7에서 직접 검토하지 못함. L1 요약은 `AdministratorAccess` 미부여를 기록함 | **추가 확인 필요** | `../01-account-iam/L1-summary.md` |

> `AccessDenied`는 특정 IAM 작업이 제한되었다는 증거이며, 그 자체로 전체 정책이 최소권한이라는 완전한 증명은 아니다. 검증 때문에 IAM 권한을 추가하지 않았다.

## 3. 서비스 및 접속

| 대상 | 검증 결과 | 판정 | 증빙 파일 |
|---|---|---|---|
| SSH | `ec2-user@3.38.97.202` 접속 성공 | 통과 | `L7-ssh.png` |
| 호스트 Nginx | `systemctl is-active nginx` → `active`; `is-enabled` → `enabled` | 통과 | `L7-nginx-service.png` |
| Docker | `sudo docker ps`: `b3-1-nginx`, `nginx:stable`, `Up`, `127.0.0.1:8080->80/tcp` | 통과 | `L7-docker-ps.png` |
| Docker 직접 HTTP | EC2에서 `http://127.0.0.1:8080/` → HTTP 200 | 통과 | `L7-docker-http.png` |
| Nginx 프록시 HTTP | EC2에서 `http://localhost/` → HTTP 200 | 통과 | `L7-nginx-proxy-http.png` |
| HTTPS/TLS | 도메인 SAN 일치, OpenSSL 인증서 검증 성공, HTTPS HTTP/1.1 200 | 통과 | `L7-https-tls-1.png`, `L7-https-tls-2.png` |
| 외부 HTTPS | Chrome 브라우저 정상 접속, 인증서 경고 없이 페이지 표시 | 통과 | `L7-external-https.png` |
| 외부 HTTP | `http://3.38.97.202/` 브라우저 접속 성공 | 통과 | `L7-external-http.png` |

> 일반 사용자 `docker ps`는 소켓 권한 때문에 실패했으며, `sudo docker ps`로 컨테이너 상태를 확인했다. Docker 권한이나 서비스를 변경하지 않았다.

## 4. 장애 복구 이력(L6 연계)

- 정상(장애 전): HTTP/HTTPS `200/200`
- 장애 재현: 프록시 대상 포트 `8080 → 8081`, HTTP/HTTPS `502/502`
- 오류 근거: Nginx `connect() failed (111: Connection refused)` 및 upstream `127.0.0.1:8081`
- 복구: 사전 백업된 Nginx 설정 복원, `nginx -t` 및 reload 성공
- 정상(복구 후): HTTP/HTTPS `200/200`
- 근거 폴더: `evidence/06-troubleshooting/`의 전·중·후 및 오류 로그 캡처 4개

## 5. 종합 판정 및 후속 확인

- VPC 네트워크, EC2/EBS, Security Group, SSH, Nginx, Docker, HTTP/HTTPS 및 TLS 검증은 **통과**.
- IAM은 사용자 인증, MFA 구성 기록, 개별 권한 거부, 실습 리소스 조회·운영 이력까지 확인했지만, **정책 세부 권한의 최소 범위는 미검증**.
- 증빙 파일 20개의 존재 및 비어 있지 않은 파일 크기는 PowerShell로 확인했으며, **이미지 본문에 대한 개별 육안 점검은 별개**.
- Phase 4 문서 작성 전에 외부 제출용 증빙의 계정 식별값·개인 IP 등 마스킹을 검토한다.
- EC2 및 연결 자원의 삭제는 **최종 산출물과 증빙을 확보한 이후** 별도 단계에서 시행한다.

