---
name: version-policy
description: 릴리스 버전 번호·태깅·CHANGELOG 기준. 버전을 올릴 때, 태그를 달 때, 릴리스를 만들 때, CHANGELOG 를 쓸 때 사용. "버전", "버전 올려", "릴리스", "태그", "semver", "CHANGELOG", "몇 버전" 요청에도 사용.
---

# 버전 정책

## 1. 번호 체계
`v<MAJOR>.<MINOR>.<PATCH>` — semver. `v` 접두사를 붙인다 (`git describe`·`gh release` 관례).

| 올리는 자리 | 언제 | 신호 |
|---|---|---|
| MAJOR | 쓰던 쪽이 고쳐야 하는 변경 | `<type>!:` 커밋이 하나라도 있을 때 |
| MINOR | 기능 추가, 하위 호환 | `feat:` |
| PATCH | 버그 수정만 | `fix:` |
| 안 올림 | `docs:` `chore:` 뿐 | 릴리스하지 않는다 |

type 은 `/branch-policy` 1절과 같은 넷. breaking 은 `feat!:` 처럼 `!` 를 붙인다 —
버전 판단에만 쓰이는 표시라 여기서 정의한다.

`v0.x` 동안은 MINOR 가 breaking 을 담는다. MAJOR 를 1 로 올리는 것은
"이제 쉽게 깨지 않는다"는 선언이고, 한 번 하면 되돌릴 수 없다.

## 2. 태깅
| 규칙 | 이유 |
|---|---|
| main 에서만 태그 | 릴리스는 머지된 것만 |
| annotated (`git tag -a`) | 작성자·날짜·메시지가 남는다. lightweight 는 아무것도 안 남긴다 |
| 한 커밋에 버전 태그 하나 | 둘이면 어느 쪽이 그 버전인지 모른다 |
| 태그 push = **X 등급** (사전 확인) | 공개되면 되돌릴 수 없다 |

## 3. CHANGELOG
`git log` 에서 뽑는다 — 커밋 메시지가 이미 `<type>: <요약>` 이라 별도 양식이 필요없다.

```bash
git log v1.2.0..HEAD --oneline --no-merges
```

`CHANGELOG.md` 에 버전별로 `feat` → `fix` → 나머지 순으로 묶는다.
**무엇을 바꿨나가 아니라 쓰던 쪽이 무엇을 해야 하나를 쓴다** — 무엇을 바꿨는지는
diff 가 이미 말한다. MAJOR 릴리스는 "고쳐야 할 것" 을 반드시 적는다.

## 4. 릴리스 절차
```bash
git switch main && git pull --prune            # 머지 완료 상태에서
git log v1.2.0..HEAD --oneline --no-merges     # 올릴 자리 판단 (1절)
# CHANGELOG.md 갱신 → /branch-policy 4절 PR 흐름으로 머지
git tag -a v1.3.0 -m "v1.3.0"                  # W
git push origin v1.3.0                         # X — 사전 확인
gh release create v1.3.0 --notes-from-tag      # X — 사전 확인
```

`/pipeline-policy` 1절 게이트 전부 통과가 전제다.

## 5. 금지
- **태그 이동·삭제·재사용** (`git tag -f`, 태그 force push)
  같은 버전에서 서로 다른 코드를 받는 사람이 생긴다. 잘못 달았으면 다음 번호로 올린다
- 머지 안 된 커밋에 태그
- CHANGELOG 없는 MAJOR 릴리스

## 6. 지금 적용 범위
산출물도 배포 대상도 없어서 **아직 아무것도 태깅하지 않는다.**

정책 저장소(`AgentHub-Policy-*`)는 소비 프로젝트가 서브모듈 SHA 로 핀하므로
태그가 중복이다. 소비 프로젝트가 2개 이상이 되고 breaking 정책 변경이 생기면
그때 붙인다 — 그게 "v2 로 올렸다"는 말에 의미가 생기는 시점이다.
