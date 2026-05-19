# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

React 19 + TypeScript + Vite 기반 노트 앱 실습 프로젝트. json-server를 백엔드 목업으로 사용하며, 강의용으로 점진적으로 기능을 추가하는 구조다 (예: `Note` 타입에 `tags` 필드 추가 예정).

## 명령어

```bash
npm run dev        # Vite 개발 서버 + json-server 동시 실행 (concurrently)
npm run build      # tsc + vite build
npm run lint       # ESLint --fix
npm run format     # Prettier --write
npm test           # vitest run (단발 실행)
npm run test:watch # vitest (watch 모드)
npm run server     # json-server만 단독 실행 (포트 3001)
```

- 앱: http://localhost:5173
- API: http://localhost:3001/notes

## 아키텍처

데이터 흐름: `json-server (db.json)` → `src/api/notes.ts` → `NotesContext` → 컴포넌트

### 레이어 역할

| 레이어 | 파일 | 책임 |
|--------|------|------|
| API | `src/api/notes.ts` | fetch 래퍼 (상태 없음). CRUD 함수 4개: `fetchNotes`, `createNote`, `updateNote`, `deleteNote` |
| 전역 상태 | `src/context/NotesContext.tsx` | API 호출 + React state 관리. `useNotes()` 훅으로 소비 |
| 타입 | `src/types/note.ts` | `Note` 인터페이스 단일 정의 |
| 컴포넌트 | `src/components/` | `useNotes()`로 상태 소비. 선택/생성 UI 상태는 `App.tsx`가 소유 |

### 상태 설계

- **서버 상태** (`notes`, `loading`, `error`): `NotesContext`가 소유, API 응답으로 낙관적 업데이트
- **UI 상태** (`selectedNoteId`, `isCreating`): `App.tsx`가 소유하고 prop으로 전달
- `NotesProvider`는 `App.tsx` 최상단에서 한 번 감싼다

### 데이터 모델

```ts
interface Note {
  id: string;
  title: string;
  content: string;
  createdAt: string; // ISO 8601
  updatedAt: string; // ISO 8601
}
```

`createNote`/`updateNote` 호출 시 `createdAt`/`updatedAt`은 API 레이어에서 자동 주입.

## 구현 패턴

### 컴포넌트 패턴
- Named export만 사용 (`export function ComponentName`) — default export 없음
- Props 인터페이스는 컴포넌트 바로 위에 `interface ComponentNameProps`로 정의
- 얼리 리턴 순서: `loading` → `error` → `empty` → 메인 렌더
- 순수 표시 컴포넌트(`NoteItem`, `Layout`)는 Context를 직접 소비하지 않고 props만 받음

### 상태 관리 패턴
- 파생 상태는 별도 state 없이 인라인 계산: `const selectedNote = notes.find(...)`
- 비동기 작업 플래그: `setSaving(true)` → `try/catch/finally` → `setSaving(false)`
- Context 액션은 API 호출 성공 후 로컬 state를 직접 갱신 (낙관적 업데이트)

### API 호출 패턴
- 컴포넌트는 API 함수를 직접 import하지 않고 Context 액션만 호출
- `createdAt`/`updatedAt` 타임스탬프는 API 레이어(`src/api/notes.ts`)에서 주입
- Context의 에러: `error` state에 저장 → UI에 표시. 컴포넌트 핸들러 에러: `try/catch` + `alert`

### 네이밍 패턴

| 대상 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 내 이벤트 핸들러 | `handleXxx` | `handleSave`, `handleNewNote` |
| 콜백 props | `onXxx` | `onSelect`, `onDelete`, `onDone` |
| Context 액션 | 동사+명사 | `addNote`, `editNote`, `removeNote` |
| API 함수 | 동사+명사 | `fetchNotes`, `createNote`, `updateNote` |
| 불리언 state | 상태 설명형 | `loading`, `saving`, `isCreating`, `isSelected` |

## 테스트

- **프레임워크**: Vitest + @testing-library/react, jsdom 환경
- **설정**: `vite.config.ts`의 `test` 섹션, setupFiles: `src/test-setup.ts`
- json-server가 실행 중이지 않아도 단위/컴포넌트 테스트는 가능 (API는 모킹)

## 스타일

- Tailwind CSS v4 (`@tailwindcss/vite` 플러그인 방식, 별도 config 파일 없음)
- Prettier + ESLint 설정 완료 (`.prettierrc`, `eslint.config.js`)
