# 메종와이에스 MAISON YS — SNS 콘텐츠 레퍼런스

> 작업일: 2026-07-27 · 세션: maisonys-homepage-sns-analysis
> 대시보드: https://claude.ai/code/artifact/db9ac83c-df58-4d6f-af1a-4e83d18130ce
> (기존 스킬 산출물 형식: https://leejiho-pslab.github.io/pslab/references.html 과 동일 구조)

## 브랜드 학습 요약

- **운영사**: ㈜와이에스콜렉션 (대표 위인혁, 서울 송파) · maisonys.com
- **브랜드 라인**: SATIN(2000년 론칭, 300억대 캐시카우) · S BLANC · HUNCH(2022년 론칭, 소프트 스트리트룩, 2026년 120억 목표)
- **가격대**: 블라우스 7~9만 · 원피스 5~11만 · 자켓 14~29만 (미들 프라이스)
- **판매 문법**: 상품명에 셀럽 PICK(리라·한빛·지구·지인) 표기 · SEASON-OFF 최대 35% · 온라인단독 · 26 SUMMER 컬렉션 단위 운영
- **인스타그램**: @hunch__official (16K 팔로워, 218게시물, 바이오 "MERIDIAN_ Where Light Defines Form" — 시즌 캠페인형 운영). 유사 핸들 @hunch_official(언더스코어 1개)은 무관한 계정이므로 자동화 설정 시 주의.
- **미확보 데이터**: 게시물별 실측(좋아요·발행 패턴·해시태그 전수) — pslab 저장소의 Graph API business_discovery 워크플로로 수집 필요 (keek 사례 참조: `docs/keek/instagram.md`)

## 학습 레퍼런스 (6건)

| 포맷 | 레퍼런스 | 적용 |
|---|---|---|
| 기획전 에디트형 | @wconceptkorea | 캐러셀 1 |
| 리스트 큐레이션형 | @whereisyourmood_ | 캐러셀 2 |
| 댓글→DM 퍼널형 | @yans__world | (미적용, 향후 옵션) |
| 시즌 룩북 캐러셀 | @thisguykirk | 캐러셀 공통 |
| AI 룩북 노하우 | @onji.lab 릴스 | 영상 제작 파이프라인 |
| 룩 전환 멀티컷 | @ruama 릴스 | 릴스·쇼츠 편집 문법 |

## 산출물

- `content/carousel_01_시의성_여름바캉스에디트.html` — 인스타 캐러셀 8슬라이드 (instagram-carousel-autopost 스킬 규격, 실판매가/할인율 반영)
- `content/carousel_02_베스트_아이템7.html` — 인스타 캐러셀 9슬라이드 (베스트 리스트형)
- `content/blog_naver.md` — 네이버 블로그 원고 (상황별 6종 추천 + 가격표)
- `assets/r_look2.png, r_look3.png` — 릴스 룩2·3 (테라스 씬, 동일 모델 착장 생성)
- `assets/y_look2.png, y_look3.png` — 유튜브 쇼츠 룩2·3 (보드워크 선셋 씬)
- 영상 원본(720p)·전체 슬라이드 PNG·룩1 이미지는 세션 산출물로 전달 (대시보드에 임베드됨)

## 영상 제작 파이프라인 (룩 전환 멀티컷)

1. 자사몰 제품컷 `media_import_url`로 Higgsfield 임포트
2. nano banana: 씬 이미지 + 제품컷을 레퍼런스로 동일 인물·동일 장소 착장 이미지 생성 (룩당 1장)
3. Kling 3.0 turbo: 룩별 5초 클립 생성 (모션 프롬프트: 턴/스핀/워킹)
4. ffmpeg: 클립별 트림(각 ~2.2s) 후 모션 컷 concat → 약 7초 멀티컷
5. 발행 시 앱에서 트렌드 음원 추가, 컷 타이밍(2.2s/4.4s)에 비트 매칭 권장

## 남은 과제

- [ ] `sns-자동운영-레퍼런스-생성` 스킬 파일을 이 저장소 `.claude/skills/`에 설치 (현재 세션엔 없어 기존 산출물 페이지에서 구조 역추출로 대응)
- [ ] @hunch__official Graph API 실측 수집 → keek 수준 계정 분석
- [ ] 캐러셀 실계정 발행 연결 (auto_post 폴더 + .env 셋업)
