# 생성 영상 자체 검증 파이프라인

> 작성 2026-07-27
> **문제**: 이 실행 환경은 Higgsfield 결과물 CDN이 차단(403)이라 생성한 영상·이미지를 열어볼 수 없다.
> 결과를 못 본 채 프롬프트만 반복 수정하면 **결과물이 계속 어긋난다.** 실제로 그렇게 됐다.
> **해결**: Higgsfield 샌드박스를 경유해 프레임을 base64로 끌어와 직접 판독한다.

---

## 0. 왜 필요한가

| 경로 | 상태 |
|---|---|
| `d8j0ntlcm91z4.cloudfront.net` (결과물 CDN) | ❌ **CONNECT 403** |
| `d2ol7oe51mr4n9.cloudfront.net` (입력 미디어 CDN) | ❌ 차단 |
| `youtube.com` · `instagram.com` | ❌ 차단 |
| `keek-line.com` · `cafe24img.poxo.com` | ✅ 허용 |

→ 생성물을 볼 수 없으면 **검수 없이 추측으로 반복**하게 된다. 반드시 아래 절차를 쓸 것.

---

## 1. 파이프라인

```
mcp__higgsfield__sandbox_exec  (Higgsfield 클라우드 · 자사 CDN 접근 O · ffmpeg 내장)
   │  curl로 결과물 mp4 다운로드
   │  ffmpeg로 키프레임 추출 → 소형 컨택트시트 (≤13KB)
   │  base64 -w0 출력
   ▼
로컬 Bash: heredoc으로 받아 base64 -d → .jpg
   ▼
Read 도구로 판독  ← 여기서 처음으로 결과물이 보인다
```

## 2. 실행 명령

### 2-1. 샌드박스에서 프레임 추출

```bash
# 다운로드 + 스펙 확인
curl -sSL -o v.mp4 -w "HTTP %{http_code} size=%{size_download}\n" "<결과물 mp4 URL>" \
 && ffprobe -v error -show_entries format=duration -show_entries stream=width,height,r_frame_rate \
    -of default=noprint_wrappers=1 v.mp4

# 특정 프레임 4장을 2x2 타일로 (크기 제약 맞춤)
ffmpeg -v error -i v.mp4 \
  -vf "select='eq(n\,12)+eq(n\,144)+eq(n\,216)+eq(n\,282)',scale=170:-1,tile=2x2" \
  -frames:v 1 -q:v 27 q.jpg -y -vsync 0

# 크기 확인 — base64가 16,000자를 넘으면 안 된다
ls -l q.jpg && base64 -w0 q.jpg | wc -c
```

### 2-2. base64 출력 → 로컬 디코드

```bash
# 샌드박스
base64 -w0 q.jpg

# 로컬 (출력을 heredoc에 붙여넣기)
cat > f.b64 <<'EOFB64'
<base64 문자열>
EOFB64
tr -d '\n' < f.b64 | base64 -d > frames.jpg
```

그다음 `Read` 도구로 `frames.jpg`를 연다.

## 3. 크기 제약 ⚠️

**샌드박스 stdout이 약 18,000자에서 잘린다.** 잘리면 중간이 사라져 디코드가 불가능하다.

| 설정 | JPEG | base64 | 판정 |
|---|---:|---:|---|
| `scale=150 tile=3x4 -q:v 6` | 95 KB | 126,876 | ❌ 잘림 |
| `scale=110 tile=4x3 -q:v 20` | 22 KB | 29,576 | ❌ 잘림 (분할 필요) |
| `scale=210 tile=2x2 -q:v 21` | 20 KB | 27,732 | ❌ 잘림 |
| **`scale=170 tile=2x2 -q:v 27`** | **12 KB** | **16,508** | ✅ **한 번에 통과** |

> 더 큰 시트가 필요하면 `split -b 16000` 으로 나눠 `cat part_aa` / `cat part_ab` 를 각각 호출한다.
> 다만 base64가 컨텍스트를 두 번(읽기+쓰기) 지나가므로 토큰 비용이 크다. **한 장에 다 담는 편이 낫다.**

## 4. 근본 해결책 ⭐

위 우회는 동작하지만 **한 장 볼 때마다 토큰 3만 자**가 든다. 상시 작업에는 비효율적이다.

**환경(`keek-작업환경`)의 허용 도메인에 아래를 추가하면 이 우회가 전부 불필요해진다.**

```
d8j0ntlcm91z4.cloudfront.net     # Higgsfield 결과물 CDN
d2ol7oe51mr4n9.cloudfront.net    # Higgsfield 입력 미디어 CDN
cdn.higgsfield.ai                # Higgsfield 프리셋/프리뷰
```

추가하면 `curl`로 바로 받아 ffmpeg로 프레임을 뽑고 즉시 판독할 수 있다.
영상 검수·클립 접합(concat)·자막 삽입까지 이 세션 안에서 처리 가능해진다.

> 참고: ffmpeg는 이미 로컬에 설치돼 있다 (`pip install imageio-ffmpeg` → `/usr/local/bin/ffmpeg`).
> pypi.org는 프록시 noProxy 목록에 있어 설치가 가능했다.

---

## 5. 첫 검증 결과 — 2번 문어 12초 클립 (`cd6ed934`)

파이프라인을 처음 돌려 확인한 내용. **프롬프트만 보고는 알 수 없던 문제가 드러났다.**

| 항목 | 결과 |
|---|---|
| 제품 분리도 | ❌ **문어와 목베개가 하나로 융합**. 목베개 형태 자체가 몸통이 되고 촉수만 붙어 있다 |
| 퀼팅 리브 챔버 | ❌ 없음 |
| 투톤 절개선 | ❌ 없음 |
| 원형 밸브 캡 | ❌ 없음 |
| 웨빙 스트랩 + 버클 | ❌ 없음 |
| 문어 사실성 | ❌ 실사 다큐가 아니라 봉제인형 톤 |
| 색 분리 | ❌ 화면 전체가 베이지 단색조라 제품/피사체 구분 불가 |

### 진단 — 프롬프트 디테일을 늘린 것이 역효과였다

제품 구조 서술(말굽형·두 로브·퀼팅·투톤·밸브·스트랩)을 길게 넣을수록
모델이 그 형태를 **착용물이 아니라 피사체 자체의 형태**로 해석했다.
"문어가 목베개를 썼다"가 아니라 "목베개처럼 생긴 생물"이 만들어졌다.

**다음 시도 시 방향** (실행 전 반드시 프레임 검증할 것)
1. 제품 묘사를 **짧게** 줄이고, 대신 `separate object worn around the neck, clearly distinct from the animal's body` 같은 **분리 지시**를 앞세운다
2. 문어의 색·질감을 **명시적으로 다르게** 지정한다 (붉은 갈색 / 반점 / 젖은 피부 광택) — 제품 베이지와 대비를 만든다
3. 여러 마리를 한 프레임에 요구할수록 형태가 뭉개진다. **1~2마리 클로즈업**으로 먼저 형태를 확정한 뒤 군집으로 확장한다
4. 매 생성마다 §2 절차로 **프레임을 먼저 본 다음** 다음 프롬프트를 정한다
