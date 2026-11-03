# SIGNAL-007 — Anthropic Claude Managed Agents: Brain·Hands·Session 분리와 장기 실행 에이전트

- **공유 원문(X):** https://x.com/F5Adon/status/2117021112500437426
- **공유 계정:** @F5Adon (계정 실명·원 게시 시각 미확인)
- **아카이브 등록·검증일:** 2026-11-03 (KST)
- **원문 확인:** X 본문·작성자의 후속 댓글·스레드 직접 열람 실패. 사용자가 전달한 워크숍 요약을 2차 단서로 사용.
- **관련 공식 1차 자료:** [Anthropic, Scaling Managed Agents: Decoupling the brain from the hands](https://www.anthropic.com/research/managed-agents) (2026-04-08), [Claude Managed Agents overview](https://platform.claude.com/docs/en/managed-agents/overview)
- **연관 행사:** AI Engineer World's Fair 2026 워크숍 (사용자가 발표자 Priyanka Phatak·Gabe Cemaj 및 약 90분 녹화본이라고 전달; 영상 전체와 시간표는 독립 대조 미완료).
- **주제:** AI agent architecture · harness · durable sessions · sandbox · security · evals · memory
- **중요도:** 매우 높음 (편집 평가)
- **상태:** `provisional` — Anthropic 공식 설계 원칙·문서 일부 교차 확인; X 게시물·전체 영상·댓글 및 일부 워크숍 기능 설명 미확인. 공개 완성 글 목록 게이트 미통과.

## 1. 3분 핵심 요약

Anthropic은 장기 실행 에이전트에 필요한 세 기능을 분리했다. **Session**은 지속적으로 조회 가능한 이벤트 기록, **Brain**은 Claude와 하네스(에이전트 루프·컨텍스트 관리), **Hands**는 샌드박스와 실제 도구 실행을 담당한다. 기존에는 세 기능이 하나의 컨테이너에 결합돼 장애 복구·디버깅·VPC 연동·시작 성능이 제약됐다. 분리 후에는 실행 환경을 필요할 때 호출하고 각 부분을 독립적으로 교체할 수 있다.

**핵심 통찰:** 모델의 컨텍스트와 지속 세션을 동일시하지 말 것. 기억·이벤트를 모델 바깥에 보존하고, 도구 실행의 권한·수명주기를 별도로 관리한다. 이는 특정 모델이 바뀌어도 유지될 수 있는 인터페이스 중심 설계다.

## 2. 기술 상세: 작동 원리

```text
Application / Scheduler / UI
           |
     Session events ──> Durable append-only log
           |
   Brain (Claude + harness)
       |            |
 Context / loop   execute(name,input)
                    |
           Hands: sandbox / MCP / tools
                    |
              Result / event
```

- **Session:** 유저 이벤트·도구 결과·상태 변경을 추적하고 외부에서 조회·재개한다. 영구 이벤트 로그와 모델에 매번 투입할 프롬프트는 다르다. 최신 또는 관련 사건만 모델에 제공하거나 압축할 수 있다.
- **Brain:** 모델·하네스가 에이전트 루프, 호출 순서, 컨텍스트 관리를 담당한다. 하네스의 모델 의존적 휴리스틱은 개선된 모델에서 불필요해질 수 있다(공식 글의 context anxiety 사례).
- **Hands:** 실제 파일·셸·외부 API·MCP 등 실행 경계. `execute(name, input) -> string` 인터페이스로 여러 실행 환경을 호출할 수 있으며 샌드박스는 항상 선기동할 필요가 없다.
- **보안:** 실행되는 코드와 장기 인증 토큰의 경계를 분리하고 권한을 최소화해야 한다. 프록시/비밀 관리에 관한 세부 구현 범위는 출처별로 재확인 필요.
- **실패 복구:** 실패한 도구 인스턴스 교체가 가능하다는 것과 실행 중이던 외부 부작용이 자동으로 되돌려진다는 것은 다르다. 재시도에는 멱등성 키·체크포인트·중복 차단이 필요하다.

## 3. 주장·증거·검증 상태

| 주장 | 근거 | 판정 |
|---|---|---|
| Brain / Hands / Session 분리, 기존 단일 컨테이너 한계 | Anthropic 공식 엔지니어링 글 (2026-04-08) | 공식 확인 |
| 하네스의 모델별 가정은 바뀔 수 있음 | 동일 공식 글, Sonnet 4.5 / Opus 4.5 사례 | 공식 확인 |
| p50 TTFT 약 60%, p95 90% 이상 감소 | 동일 공식 글 | **공급자 자체 보고 수치**, 독립 재현 아님 |
| Agent·Environment·Session·Events 및 일정 실행 | Claude Managed Agents 공식 문서 | 공식 문서 확인 |
| Managed Agents는 Beta, API 키·Beta 헤더 필요 | 공식 플랫폼 문서 | 확인일 기준 문서 확인 |
| Dreaming 및 MCP tunnels는 제한적 연구 프리뷰 | 공식 플랫폼 문서 | 문서 확인 |
| 세션·파일 상태 유지, ZDR 및 HIPAA 부적격 | 공식 플랫폼 문서 | 문서 확인 |
| 90분 영상의 모든 실습 단계와 분 단위 타임스탬프 | 사용자 제공 요약 | 미검증 |
| 발표자 두 사람의 정확한 역할/직책 | 사용자 제공 요약 | 별도 확인 필요 |
| '수천 원짜리 트레이닝보다 낫다'는 평가 | 사용자 전달 반응 | 주관적 의견, 출처 불명 |

**정정:** 'TTFT 90% 감소'는 전체 요청의 보편적 개선치가 아니다. 공식 발표는 **중앙값(p50) 약 60%**, **꼬리 지연(p95) 90% 초과** 감소라고 밝힌다.

## 4. Dreaming / 공유 메모리 / 평가

- **Dreaming:** 기존 메모리 저장소와 이전 세션 기록을 읽어 정리된 새 메모리 저장소를 생성하는 별도 작업. 모델 가중치를 갱신하는 파인튜닝이 아니다. 접근 범위·작동 조건은 공식 기능 문서 및 API 지원 범위로 확인해야 한다.
- **Skills·Subagents·Shared memory:** 재사용 지침, 업무 위임, 세션 간 지식 전달이라는 서로 다른 문제를 다룬다. 워크숍 시연 세부는 전체 영상 대조 전까지 공식 지원·시연 여부를 단정하지 않는다.
- **Outcome grading:** 결과가 요구한 기준을 충족하는지 루브릭으로 평가하고 필요시 수정하는 패턴. 에이전트의 자체 평가만으로 신뢰성을 보증할 수 없으며 독립 테스트와 사람 승인이 필요하다.
- **Event-stream UI:** 모델 출력·도구 실행·진행률을 이벤트로 보여 주는 접근. 누락·중복 이벤트, 연결 끊김 이후 재연결을 고려한다.

## 5. 기존 기술과 차별점

| 기존 개념 | 이번 설계에서의 의미 |
|---|---|
| 이벤트 소싱 / append-only log | 세션을 재개·감사·재구성하는 기반 |
| Stateless worker + queue | Brain을 교체하고 여러 작업으로 확장할 수 있는 단위 |
| Container isolation | Hands별 격리와 수명주기 분리 |
| API gateway / secret proxy | 도구에 과도한 장기 비밀을 주입하지 않는 권한 경계 |
| CI & automated evals | 결과 품질을 실행 성공과 별도로 평가 |

개별 개념은 새로운 발명이 아니다. **장기 실행 AI 에이전트라는 워크로드에서 이들을 결합하고 인터페이스를 분리**한 점이 실질적인 기여다.

## 6. 한계·위험·반론

1. Session이 지속되어도 도구의 외부 부작용(결제, 배포, DB 쓰기)은 자동 롤백되지 않는다.
2. 이벤트 로그가 늘면 요약·검색·민감정보 삭제 정책과 저장 비용이 생긴다.
3. 하나의 Brain이 여러 Hands를 운용하면 실행 환경 선택, 권한 혼동, 컨텍스트 관리의 복잡성이 늘어난다.
4. Anthropic의 TTFT 개선은 해당 서비스의 아키텍처 비교 결과로, 다른 환경의 성능 보장이 아니다.
5. Managed Agents는 공급자 인프라·API 비용과 데이터 보존 정책에 종속될 수 있다. 공개 문서상 ZDR 대상이 아니다.
6. Dreaming이 작성한 기억에는 오류가 포함될 수 있다. 원본 기록과 기억 요약을 구분하고 근거 링크·승인·수정 기록을 남겨야 한다.

## 7. 개발자·커리어 관점 및 최소 실습

이 자료는 프롬프트 작성보다 **오케스트레이션·분산 시스템·상태 복구·비밀 관리·평가 설계** 역량의 가치를 강조한다.

**최소 PoC (일반 예시, 실제 실행하지 않음):**
1. 작업별 session ID를 만들고 이벤트를 append-only로 저장한다.
2. Brain worker는 저장된 이벤트를 읽고 다음 도구 호출을 결정한다.
3. Hands는 읽기 전용 도구로 제한하고 실행 실패 시 새 인스턴스로 재연결한다.
4. 의도적으로 Hands를 종료하고 세션 유지 및 중복 실행 여부를 검증한다.
5. 품질 평가와 운영 변경 승인을 별도 게이트로 둔다.
6. 모델 또는 하네스를 바꾼 뒤에도 기존 세션과 메모리를 올바르게 해석하는지 평가한다.

**학습 순서:** event sourcing → queue/worker & idempotency → sandbox/egress proxy → durable sessions → evals & memory compaction.

## 8. 오늘AI의 독립 분석

모델이 더 좋아질수록 기존 하네스의 모델 능력 가정은 빠르게 노후화될 수 있다. 따라서 실행과 데이터 저장의 경계를 안정된 인터페이스로 유지하는 것이 합리적이다. 다만 '분리했다'는 사실만으로 안전한 자율 운영이 되는 것은 아니다. **복구 가능성 + 최소 권한 + 검증 가능한 결과 + 사람이 통제하는 고위험 승인**을 함께 설계해야 한다.

상용 Managed Agents를 도입하는 것과 분리 원칙을 자체 서비스에 적용하는 것은 다른 결정이다. 저장·보안 정책과 총비용, 실행 환경 제약을 비교해야 한다.

## 9. 근거 출처와 후속 확인

1. [공식 아키텍처 — Anthropic, Scaling Managed Agents](https://www.anthropic.com/research/managed-agents) (2026-04-08)
2. [공식 API 개요 — Claude Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview) (2026-11-03 확인)
3. [공식 기능 목록 — Managed Agents features](https://platform.claude.com/docs/en/build-with-claude/overview)
4. [공유된 X 게시물](https://x.com/F5Adon/status/2117021112500437426) (직접 열람 실패)
5. **추가 필요:** 원본 워크숍 영상 URL 및 전체 자막, @F5Adon 원문·본인 답글·스레드, Dreaming/Outcome grading 실습 코드.

## 10. 지식 연결

- [AI 에이전트](../topics/ai-agents.md) — 장기 실행·권한·세션 복구·평가
- [SIGNAL-002](SIGNAL-002.md) — 에이전트 능력 향상과 성능 검증
- [SIGNAL-003](SIGNAL-003.md) — 누적 지식 관리와 원본/요약의 구분
- [SIGNAL-006](SIGNAL-006.md) — 샌드박스 격리·경계 공격과 최소 권한

## 수정 이력

- 2026-11-03: 사용자 제공 워크숍 요약 및 Anthropic 공식 아키텍처·플랫폼 문서 교차 대조. 원본 X·전체 워크숍 미확인으로 provisional 등록. 공개 사이트 발행하지 않음.
