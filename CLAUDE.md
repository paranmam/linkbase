# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 이 디렉터리의 성격

`~/Workspace/Claude` 는 단일 프로젝트가 아니라 **서로 무관한 웹 프로젝트 3개를 담은 컨테이너**다. 이 폴더 자체는 git 저장소가 아니고 루트 `package.json`도 없다. 반드시 해당 하위 디렉터리로 `cd` 한 뒤 작업할 것.

상위 `~/Workspace/CLAUDE.md` 는 JARVIS(RH850 / Jetson 임베디드) 컨텍스트이며 **이 웹 프로젝트들과는 관련이 없다.** 이 폴더의 프로젝트에 임베디드 제약(크로스 컴파일러, 실기 접근 등)을 적용하지 말 것.

| 디렉터리 | 스택 | 패키지 매니저 | git | 자체 지침 |
|---|---|---|---|---|
| `Splitmate/` | React 19 + Vite + Tailwind 4 (JS, JSX) | npm | 없음 | 없음 |
| `linbase/` | Next.js 16 App Router + Tailwind 4 (TS) | npm | 있음 | `AGENTS.md` |
| `golden-rabbit-yojeum-claude-code-crypto-ranking-board/` | Next.js 15 + shadcn/ui (TS) | pnpm | 있음 | `CLAUDE.md` (상세) |

`.playwright-mcp/` 는 Playwright MCP가 남긴 스냅샷/로그 산출물이므로 무시한다.

## Splitmate

Splitwise 류의 비용 분할 앱. **백엔드 없음 — 전부 localStorage.**

```bash
npm run dev       # vite dev server
npm run build     # dist/ 생성
npm run lint      # oxlint (react/rules-of-hooks=error, only-export-components=warn)
npm run preview
```

테스트 러너는 없다.

### 아키텍처 (여러 파일을 읽어야 보이는 부분)

- **`src/data/storage.js` 가 localStorage를 만지는 유일한 모듈**이다. context / hooks / pages / components 는 반드시 여기 export 함수를 통해서만 데이터에 접근한다. 향후 Supabase로 옮길 때 이 파일만 교체하는 것이 설계 의도이므로 이 경계를 깨지 말 것. 데이터 shape(User/Member/Group/Expense)과 키(`splitmate.*`)는 파일 상단 주석에 정의돼 있다.
- **금액은 항상 정수 cents.** 변환은 `utils/money.js`(`toCents`, `formatCents`, `splitEvenly`), 정산은 `utils/balance.js`. float로 금액을 들고 다니지 말 것.
- **Expense는 soft delete** (`isDeleted: true`). 삭제하지 않으므로 expense를 읽는 모든 계산은 `isDeleted` 를 건너뛰어야 한다. 잔액은 저장하지 않고 매번 live expense 목록에서 `computeNetBalances` → `groupBalances`(최소 송금 정산)로 재계산한다.
- **Member ≠ User.** Group의 member는 `userId|null`, `status: 'active'|'pending'` 을 가진다(아직 가입 안 한 이메일 초대 가능). 로그인 사용자의 member는 `member.userId === user.id` 로 찾는다 (`hooks/useGroup.js` 의 `me`).
- 인증은 `context/AuthContext.jsx` + `context/auth-context.js`(Context 객체 분리, only-export-components 린트 회피) + `hooks/useAuth.js`. 마운트 시 `storage.seed()` 가 데모 계정 4개(`*@test.com` / `password`)를 심는다. 비밀번호는 평문 저장 — 데모용이다.
- 라우팅은 `App.jsx` 에 전부 있음: 공개(`/`, `/login`, `/register`), `ProtectedRoute` 로 감싼 `/dashboard`, `/group/new`, `/group/:id`, 나머지는 `/` 로 리다이렉트.
- 훅(`useGroup`, `useGroups`)은 `reload()` 패턴: 변경 후 storage에서 다시 읽어 state를 갱신한다. 상태를 낙관적으로 직접 수정하지 말고 storage 변경 → reload.

## linbase

`create-next-app` 직후의 거의 빈 스캐폴드 (`app/page.tsx` 는 기본 템플릿 그대로). `npm run dev | build | start | lint`.

**Next.js 16.4.4 / React 19.3 — 학습 데이터의 Next와 다르다.** `AGENTS.md` 지시대로 코드를 쓰기 전에 `linbase/node_modules/next/dist/docs/` (`01-app`, `02-pages`, `03-architecture`) 의 해당 가이드를 먼저 읽고 deprecation 공지를 따를 것. `AGENTS.md` 의 해당 블록은 `next dev` 가 자동으로 다시 써 넣으므로 삭제해도 소용없다.

## golden-rabbit-yojeum-claude-code-crypto-ranking-board

상세한 `CLAUDE.md` 가 해당 디렉터리에 이미 있으니 그쪽을 먼저 읽을 것. 요점: 앱 전체가 루트의 `crypto-ranking-board.tsx` 한 컴포넌트, 데이터는 하드코딩 mock, `pnpm lint` 와 Jest 는 선언만 되어 있고 동작하지 않음, 검증은 `npx tsc --noEmit` 만 유효.
