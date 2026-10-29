# B3-1 미션 완료 검증서

> **목적:** 미션 원문의 필수 요구사항과 제출 산출물을 평가자가 빠르게 검증할 수 있도록 **판정 기준 → 실제 결과 → 직접 증빙**을 한곳에 연결한다. 프로젝트 구성에 대한 자세한 설명은 [README](../README.md) 및 [아키텍처 문서](architecture.md)를 참조한다.
>
> **검증 기준일:** 2026-10-29 · **리전:** `ap-northeast-2` · **근거:** L0~L7 실습 기록 및 [L7 통합 검증표](../evidence/07-final-verification/L7-final-verification.md)
>
> **상태 정의:** ✅ 확인 = 기록과 증빙으로 확인됨 · ⚠️ 부분 확인 = 일부 근거 부족 · ⬜ 미완료 = 아직 완료 근거 없음 · ➕ 선택 = 보너스 과제

## 0. 평가자용 결론

| 범위 | 판정 | 검증 포인트 |
|---|---|---|
| 네트워크·EC2·웹 서버·접근 제어·외부 접속 | ✅ 확인 | 아래 **1~3절** 증빙 링크 |
| IAM 사용자 사용 및 제한 사례 | ⚠️ 부분 확인 | 사용자·MFA·거부 사례는 확인, 정책 전체 권한 범위 미확인 |
| 필수 산출물 4종 | ⚠️ 일부 완료 | 아키텍처·외부 접속 증빙 완료; 트러블슈팅 보고서·리소스 정리 체크리스트 미완료 |
| HTTPS 및 Docker 보너스 | ✅ 확인 | 아래 **4절** 증빙 링크 |
| 실습 리소스 종료·삭제 | ⬜ 미완료 | 비용 방지를 위한 종료 후 삭제 확인 필요 |

**현재 종합 판정: 미션 최종 완료 확정 전.** 구축 및 외부 접속은 검증되었지만, IAM 정책 범위·미작성 산출물·리소스 정리 완료 확인이 남아 있다.

## 1. 제출 산출물 4종 검증

미션 원문 **§2 최종 결과물** 기준.

| 필수 산출물 | 원문 최소 기준 / 합격 판단 | 제출 경로 / 증빙 | 상태 |
|---|---|---|---|
| ① 아키텍처 다이어그램 | PNG/PDF 1장; VPC, Subnet, IGW, EC2, SG와 외부→서비스 트래픽 흐름 | [다이어그램](architecture.png) · [상세 설명](architecture.md) | ✅ 작성됨 — 도면 구성요소 최종 육안 점검 권장 |
| ② 외부 접속 증빙 | 방식 A 또는 B 명시; URL/IP 기재; 정상 표시/200 스크린샷 1장 이상 | **방식 A:** `http://3.38.97.202/` · [README의 방식 명시](../README.md#1-서비스-접속-및-외부-검증) · [외부 브라우저 캡처](../evidence/07-final-verification/L7-external-http.png) | ✅ 확인 |
| ③ 트러블슈팅 보고서 | `docs/troubleshooting.md` 또는 PDF; 증상→가설→검증→조치→결과→재발 방지 | [오류 로그](../evidence/06-troubleshooting/L6-nginx-error-log.png) 등 기초 증빙 확보; **`docs/troubleshooting.md` 미작성** | ⬜ 문서 미완료 |
| ④ 리소스 정리 체크리스트 | `docs/cleanup-checklist.md` 및 실습 종료 후 정리 완료 근거 | **체크리스트 미작성, 리소스 삭제 완료 미확인** | ⬜ 미완료 |

### 외부 접속 증빙 — 필수 ②

- **선택 방식:** (A) 외부 브라우저에서 `http://<퍼블릭IP>` 접속
- **실제 접속 URL:** `http://3.38.97.202/`
- **관찰 결과:** 웹페이지 정상 표시 및 HTTP 200 (L3/L7 기록)
- **증빙:** [L3 외부 HTTP](../evidence/03-ec2-nginx/L3-external-http.png) · [L7 외부 HTTP](../evidence/07-final-verification/L7-external-http.png)

![외부 브라우저 HTTP 접속 검증](../evidence/07-final-verification/L7-external-http.png)

> 접속 URL은 실습 당시 값이다. 향후 리소스 삭제 후에는 실제 접속이 되지 않아도 당시 검증 스크린샷으로 확인한다.

## 2. 기능 요구사항 검증

미션 원문 **§4 기능 요구 사항**을 빠짐없이 대응한다.

### 2.1 네트워크 구성 (§4-1)

| 원문 요구사항 | 구축/검증 결과 | 증빙 | 판정 |
|---|---|---|---|
| VPC 1개 생성 | VPC `10.0.0.0/16`, available | [VPC](../evidence/07-final-verification/L7-vpc.png) | ✅ |
| Public Subnet 1개 생성 | Subnet `10.0.1.0/24`, 해당 VPC 소속 | [Subnet](../evidence/07-final-verification/L7-subnet.png) | ✅ |
| IGW를 VPC에 연결 | IGW 연결 상태 확인 | [IGW](../evidence/07-final-verification/L7-igw.png) | ✅ |
| Public Subnet에 `0.0.0.0/0 → IGW` | Subnet은 **VPC Main Route Table을 상속**; 해당 테이블에 활성 기본 경로 | [라우팅 테이블](../evidence/07-final-verification/L7-route-table.png) · [서브넷 라우팅](../evidence/07-final-verification/L7-subnet-routing.png) | ✅ |
| 인스턴스 인터넷 아웃바운드 | L3에서 외부 HTTP 요청 성공 기록 | [아웃바운드 HTTP](../evidence/03-ec2-nginx/L3-outbound-http.png) | ✅ |

### 2.2 컴퓨트 및 웹 서버 (§4-2)

| 원문 요구사항 | 구축/검증 결과 | 증빙 | 판정 |
|---|---|---|---|
| Public Subnet에 EC2 1대 | `t3.micro`, Amazon Linux 2023, 퍼블릭 IPv4 부여, running | [EC2](../evidence/07-final-verification/L7-ec2.png) · [네트워크](../evidence/03-ec2-nginx/L3-ec2-network.png) | ✅ |
| SSH 접속 가능 | `ec2-user` SSH 접속 성공 | [SSH 접속](../evidence/07-final-verification/L7-ssh.png) | ✅ |
| Nginx 설치·실행 | 호스트 Nginx `active`, `enabled` | [서비스 상태](../evidence/07-final-verification/L7-nginx-service.png) | ✅ |
| 인스턴스 `curl http://localhost`가 200 | 호스트 Nginx 경유 로컬 HTTP 200 | [로컬 프록시 HTTP](../evidence/07-final-verification/L7-nginx-proxy-http.png) | ✅ |

### 2.3 보안 그룹 (§4-3)

| 원문 요구사항 | 구축/검증 결과 | 증빙 | 판정 |
|---|---|---|---|
| 필요한 포트만 인바운드 허용 | SSH 22, HTTP 80, HTTPS 443 규칙 확인 | [전체 SG](../evidence/07-final-verification/L7-security-group.png) | ✅ |
| HTTP 80을 `0.0.0.0/0`에 공개 | 인바운드 규칙 및 외부 HTTP 접속 확인 | [SG](../evidence/07-final-verification/L7-security-group.png) · [외부 HTTP](../evidence/07-final-verification/L7-external-http.png) | ✅ |
| SSH 22는 개인 IP 또는 지정 대역 | 특정 공인 IPv4 `/32` 제한 | [SSH 소스 제한](../evidence/07-final-verification/L7-security-group-ssh.png) | ✅ |
| 전체 포트(0~65535) 공개 금지 | 수집한 SG 규칙에서 전체 공개 없음 | [SG](../evidence/07-final-verification/L7-security-group.png) | ✅ |

### 2.4 IAM 최소권한 (§4-4)

| 원문 요구사항 | 확인 결과 | 증빙 | 판정 |
|---|---|---|---|
| 실습 IAM 사용자/Role 사용 | `b3-1-operator` 인증·사용 기록 | [CLI 신원](../evidence/01-account-iam/L1-cli-identity.png) · [L1 요약](../evidence/01-account-iam/L1-summary.md) | ✅ |
| 실습 범위(EC2/VPC/SG)에 필요한 권한으로 제한 | EC2 관련 접근 사례 및 IAM 조회 일부 거부 확인. **정책 JSON 전체 권한 범위는 조회·대조하지 못함** | [L1 요약](../evidence/01-account-iam/L1-summary.md) · [IAM 거부](../evidence/07-final-verification/L7-iam-access-denied.png) | ⚠️ |
| `AdministratorAccess` 미부여 | L1 요약에 미부여로 기록되어 있으나 L7에서 정책 직접 대조 불가 | [L1 요약](../evidence/01-account-iam/L1-summary.md) | ⚠️ 기록 수준 |

> `AccessDenied`는 **해당 요청이 거부된 증거**이지 IAM 전체 권한이 최소권한이라는 완전한 증거는 아니다. 별도 권한 있는 계정에서 사용자·그룹에 연결된 정책 및 정책 내용을 확인해 보완해야 한다.

### 2.5 외부 접속 (§4-5)

- 요구: A(외부 HTTP 브라우저) 또는 B(`/health` 200) 중 하나.
- 선택: **A**, README에 방식 및 IP/URL 기재.
- 판정: ✅ 확인 — [외부 HTTP 스크린샷](../evidence/07-final-verification/L7-external-http.png) 참조.

### 2.6 운영 안정성 / 과금 방지 (§4-6)

- 요구: EC2, EBS, Elastic IP(있는 경우), IGW, VPC 정리 및 종료·삭제 근거; 실제 사용한 기타 유료 리소스도 점검.
- 현재: 실습 중 EC2/EBS/IGW/VPC **존재·실행** 기록만 확인; 종료/삭제 완료 기록은 없음.
- 판정: **⬜ 미완료.** 체크리스트 작성 후 실제 자원 정리 시각, 리소스별 결과 및 Billing 확인을 기입해야 한다. **기록이 없는데 삭제 완료로 표시하지 말 것.**

## 3. 장애 재현·복구 검증 (트러블슈팅 보고서의 근거)

L6 기록으로 다음 사실이 입증되어 있다. 다만 **필수 제출 문서인 `docs/troubleshooting.md`는 별도 작성해야 한다.**

| 흐름 | 확인 내용 | 바로 보기 |
|---|---|---|
| 장애 전 정상 | HTTP/HTTPS 200/200 | [정상 화면](../evidence/06-troubleshooting/L6-before-failure.png) |
| 장애 유발·증상 | Nginx `proxy_pass` 목적지 포트 `8080 → 8081`; HTTP/HTTPS 502/502 | [502 오류](../evidence/06-troubleshooting/L6-failure-502.png) |
| 로그로 원인 검증 | `connect() failed (111: Connection refused)`; upstream `127.0.0.1:8081` | [Nginx 오류 로그](../evidence/06-troubleshooting/L6-nginx-error-log.png) |
| 복구·재검증 | 백업 설정 복원, 문법 검사·reload, HTTP/HTTPS 200/200 | [복구 화면](../evidence/06-troubleshooting/L6-after-recovery.png) |
| 재발 방지 | 향후 `proxy_pass`와 실제 Docker 바인딩 포트 일치 여부 검사 절차 작성 필요 | 보고서 작성 시 기재 |

## 4. 보너스 과제 검증 (§5)

| 보너스 | 요구조건 / 실제 결과 | 증빙 | 판정 |
|---|---|---|---|
| ① HTTPS | DuckDNS 도메인 연결, Let's Encrypt 인증서, TLS 검증·외부 HTTPS 성공 | [인증서](../evidence/05-domain-https/L5-certificate-details.png) · [TLS 검증](../evidence/07-final-verification/L7-https-tls-1.png) · [외부 HTTPS](../evidence/07-final-verification/L7-external-https.png) | ✅ ➕ |
| ② Docker 배포 | EC2 Docker, `nginx:stable`, 컨테이너 `b3-1-nginx` Up; `127.0.0.1:8080 → 80/tcp`; 로컬 HTTP 200, 외부 HTTP 접속 | [docker ps](../evidence/07-final-verification/L7-docker-ps.png) · [컨테이너 HTTP](../evidence/07-final-verification/L7-docker-http.png) · [외부 HTTP](../evidence/04-docker/L4-external-http.png) | ✅ ➕ |

**Docker 실행 방식:** EC2에서 `nginx:stable` 컨테이너 실행 → 호스트의 `127.0.0.1:8080`에만 바인딩 → 호스트 Nginx가 HTTP/HTTPS 요청을 역방향 프록시. [README의 이미지/실행방식/포트/증빙 명시](../README.md#3-docker-배포-보너스).

![Docker 컨테이너 상태 및 포트 매핑](../evidence/07-final-verification/L7-docker-ps.png)

## 5. 제약조건 및 학습 목표 확인

미션 원문 **§3 과제 목표 / §7 제약 사항** 기준.

| 항목 | 현재 근거 / 확인 사항 | 판정 |
|---|---|---|
| 서울 리전 사용 | EC2·VPC 구성의 리전 `ap-northeast-2` | ✅ |
| micro급 EC2와 적정 EBS | `t3.micro`, `gp3` 8GiB | ✅ |
| 별도 IAM 사용자 사용 및 MFA | 사용자 인증, MFA 스크린샷·L1 요약 | ✅ 사용 기록; 정책 범위 ⚠️ |
| 보안 그룹 최소 개방 | 22 특정 `/32`, 80·443 외부 공개, 전체 포트 개방 없음 | ✅ |
| 프리티어/비용 관리 | Budget·크레딧 설정 기록은 있으나 **실제 전체 기간 무과금은 단정 불가** | ⚠️ 비용 확인 필요 |
| 트래픽 경로 설명 가능 | VPC, 서브넷, IGW, 메인 라우팅, SG, EC2, 호스트 Nginx, Docker 요청 흐름 문서화 | ✅ 문서화 |
| 장애 진단 설명 가능 | 502 오류·로그·복구 기록 | ✅ 기록; 보고서는 ⬜ |
| 리소스 삭제/비용 방지 | 종료·삭제 이후 근거 없음 | ⬜ |

## 6. 미완료 항목 — 최종 제출 전 체크

- [ ] `docs/troubleshooting.md` 작성: **증상 → 가설 → 검증 → 조치 → 결과 → 재발 방지** 및 L6 이미지 연결.
- [ ] IAM 정책의 실제 연결 대상/정책 JSON/관리자 권한 부재를 권한 있는 계정으로 확인하고 증빙 추가 (민감정보 마스킹 후).
- [ ] `docs/cleanup-checklist.md` 작성. **실습 종료 시** EC2, EBS, Elastic IP, IGW, VPC, Subnet/라우팅/SG 및 기타 생성 자원의 삭제 상태 기록.
- [ ] 실제 리소스 정리 완료 후 Billing 확인 기록을 첨부하고 본 문서의 미완료 판정을 갱신.
- [ ] 외부 공유 전 증빙 이미지와 Markdown에서 토큰·비밀번호·개인정보가 남아 있지 않은지 최종 점검.

---

### 검증서 사용 방법

1. 평가자는 **1절 산출물 4종**으로 제출 형식·파일 존재를 먼저 확인한다.
2. **2절 기능 요구사항**의 각 증빙 링크를 열어 실제 실행·설정 결과를 대조한다.
3. **3~4절**로 문제 해결과 선택 과제 수행 여부를 확인한다.
4. **6절의 미완료 항목이 모두 해소된 후** 미션 최종 완료로 판정한다.

이 문서는 **검증 증빙을 연결한 평가용 색인**이다. 과거에 수집된 기록을 바탕으로 작성했으며, 링크된 모든 이미지의 내용에 대해 이번 작성 과정에서 새로 육안 검증한 것은 아니다.
