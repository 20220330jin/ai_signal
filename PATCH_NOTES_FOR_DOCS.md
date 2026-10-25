# docs/index.html 갱신 안내 (SIGNAL-006)

## 왜 Markdown만으로는 부족한가
`docs/index.html`의 `get(path)`는 **`bundled` 객체에 해당 경로가 있으면 그것을 먼저 반환**하고, 없을 때만 `raw.githubusercontent.com/.../main/`에서 가져온다.
- `INDEX.md`는 `bundled`에 내장돼 있으므로, 저장소의 INDEX.md만 고쳐서는 **사이트 목록에 SIGNAL-006이 나타나지 않는다.**
- 따라서 최소한 내장 `INDEX.md` 문자열을 갱신해야 한다. 본문은 내장하지 않아도 main 병합 후 raw GitHub에서 불러올 수 있지만, 일관성을 위해 내장한다.

## 변경 내용 (이 초안의 `docs/index.html`에 이미 적용됨)
1. `const bundled` 의 `"INDEX.md"` 값을 새 INDEX.md 전문으로 교체(SIGNAL-006 행, 연결 지식 갱신).
2. `const bundled` 에 `"posts/SIGNAL-006.md"` 키 추가(값 = posts/SIGNAL-006.md 전문, JSON 이스케이프).
3. `displayTitles` 에 `'SIGNAL-006':'AI가 AI 격리를 공격하다 — OpenAI 훈련 환경 보안'` 추가.
4. `categories` 에 `'SIGNAL-006':['AI 안전·보안','AI 에이전트']` 추가.
그 외 코드는 변경하지 않았다. 기존 게시물 내장 데이터는 그대로 유지(JSON 재직렬화로 바이트 순서 동일 키, 내용 불변).

## 재생성 방법
```python
import json,re
h=open('docs/index.html').read()
m=re.search(r'const bundled=(\{.*?\});\n',h,re.S)
b=json.loads(m.group(1))
b['INDEX.md']=open('INDEX.md').read()
b['posts/SIGNAL-006.md']=open('posts/SIGNAL-006.md').read()
h=h[:m.start(1)]+json.dumps(b,ensure_ascii=False)+h[m.end(1):]
open('docs/index.html','w').write(h)
```

## 병합 후 확인 (미실시)
- GitHub Pages 배포 완료 → `https://20220330jin.github.io/ai_signal/?post=SIGNAL-006.md` 열람
- 주제 보기 'AI 안전·보안'/'AI 에이전트'에 표시, 최신 글 자동 표시(SIGNAL-006), 모바일, 기존 001~005 정상.
