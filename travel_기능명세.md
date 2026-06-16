# Triply 상세 기능 명세

> 작성일: 2026.06.16
> 버전: v0.5
> 기반 레퍼런스: mooda 감성일기 (도트 속지, 스티커, 아날로그 감성) + Journal 앱 (책 스택 홈)
> 연계 문서: triply_기획서.md

---

## 6. 디자인 컨셉 & 색상

| 항목 | 값 |
|------|-----|
| 메인 컬러 | `#6B6B9A` (보라-슬레이트) |
| 포인트 컬러 | `#E8573A` (코럴 레드) |
| 배경 (카드) | `#F5F3EE` (크림 베이지) |
| 도트 속지 | `#FEFEFE` + `radial-gradient` dot pattern |
| 태그 | `#EEEDFE` bg / `#534AB7` text |
| CTA 버튼 | `#6B6B9A` bg / `#fff` text |

**책 색상 팔레트 (총 10종 — 여행 생성 시 랜덤 배정)**

책 색상은 특정 여행지에 고정되지 않고, 여행 기록 생성 시 아래 팔레트에서 랜덤으로 1개가 자동 배정됩니다.
사용자가 원하면 생성 후 책 우상단 설정 뱃지를 통해 직접 변경할 수 있습니다.

| 이름 | 책 색상 (book) | 등 색상 (spine) |
|------|---------------|----------------|
| 코럴 레드 | `#E8573A` | `#C43D22` |
| 살구 | `#E8A87C` | `#C4814E` |
| 연두 | `#8BBF6A` | `#6A9E4A` |
| 스카이 블루 | `#6B9EC4` | `#4A7EA5` |
| 라벤더 | `#A87CC4` | `#8A5CA5` |
| 민트 | `#6ABFB0` | `#4A9E8F` |
| 선샤인 옐로 | `#E8C85A` | `#C4A032` |
| 로즈 핑크 | `#E87CA8` | `#C45A85` |
| 슬레이트 그레이 | `#8A8FA8` | `#6A6F85` |
| 딥 네이비 | `#5A6E9E` | `#3A4E7E` |

**랜덤 배정 로직**

```javascript
const BOOK_COLORS = [
  { name: '코럴 레드',    book: '#E8573A', spine: '#C43D22' },
  { name: '살구',         book: '#E8A87C', spine: '#C4814E' },
  { name: '연두',         book: '#8BBF6A', spine: '#6A9E4A' },
  { name: '스카이 블루',  book: '#6B9EC4', spine: '#4A7EA5' },
  { name: '라벤더',       book: '#A87CC4', spine: '#8A5CA5' },
  { name: '민트',         book: '#6ABFB0', spine: '#4A9E8F' },
  { name: '선샤인 옐로',  book: '#E8C85A', spine: '#C4A032' },
  { name: '로즈 핑크',    book: '#E87CA8', spine: '#C45A85' },
  { name: '슬레이트',     book: '#8A8FA8', spine: '#6A6F85' },
  { name: '딥 네이비',    book: '#5A6E9E', spine: '#3A4E7E' },
]

// 최근 사용 색상과 중복되지 않게 배정
const assignBookColor = (recentColors = []) => {
  const available = BOOK_COLORS.filter(c => !recentColors.includes(c.book))
  const pool = available.length > 0 ? available : BOOK_COLORS
  return pool[Math.floor(Math.random() * pool.length)]
}
```

---

## 7. 화면 목록 (IA)

```
├── 로그인
│
├── 홈 (Home)
│   └── 여행지 슬라이드 (1~5) + 더보기 슬롯 (6번째)
│
├── 여행 기록 생성하기 (Create)          ← FAB(+) 탭 진입
│   ├── 사진 업로드
│   ├── EXIF 자동 파싱 결과 확인
│   ├── 여행지 이름 / 태그 / 책 색상 선택
│   └── 카드 자동 생성하기 →
│
├── 슬라이드 기록 카드 (Card Viewer)
│   ├── 장소별 슬라이드 (1~N장)
│   │   ├── 날씨 + 날짜 (자동)
│   │   ├── 사진 (풀블리드)
│   │   ├── 장소명 (역지오코딩 자동)
│   │   ├── 감성 태그
│   │   └── 스티커
│   └── 마지막 장: 지도 핀 전체 + 저장하기
│
├── 내 여행 기록 (List)                  ← 더보기 슬롯 진입
│   ├── 필터 탭 (시간순 / 지역별 / 카테고리)
│   └── 여행 목록 (연도별 그룹)
│
└── 검색 (Search)                        ← 홈 상단 검색 아이콘
    ├── 검색창 + 카테고리 칩
    ├── 최근 검색어
    └── 검색 결과 리스트 → 장소 상세
```

---

## 8. 화면별 상세 명세

### 8-1. 홈 (Home)

```
[상단바]
  좌: 책 아이콘 (라이브러리)
  우: 검색 아이콘, 햄버거 메뉴

[타이틀]
  "내 여행 기록"
  서브: "N trips"

[도트 인디케이터]
  현재 슬라이드 위치 (활성 = 흰 pill 형태)

[슬라이드 네비게이션]
  좌: < 화살표 (첫 슬라이드에서 비활성)
  중: "N / 6" 카운터
  우: > 화살표 (마지막 슬라이드에서 비활성)

[책 컴포넌트 — 슬라이드 1~5]
  - 여행 생성 시 랜덤 배정된 색상으로 렌더링
  - 책 본문: 여행지명
  - 책 하단: 날짜 범위 · 방문지 수
  - 우상단 설정 뱃지: 탭 시 색상 변경 ColorPicker 노출

[슬라이드 6 — 더보기 카드]
  - 점선 테두리 빈 슬롯
  - "전체 기록 보기" 텍스트
  - "더보기 →" 버튼 → 내 여행 기록 화면 진입

[FAB — 플로팅 액션 버튼]
  - 위치: 우하단 고정
  - 디자인: 흰 원형 + 보라 + 아이콘
  - 하단 레이블: "기록 추가"
  - 탭 시: 여행 기록 생성하기 진입

[하단]
  - 홈 인디케이터 바 (iOS 스타일 pill)
  - 탭바 없음
```

### 8-2. 여행 기록 생성하기 (Create)

**진입 경로:** 홈 FAB(+) 탭

```
[헤더]
  < 뒤로가기 / "여행 기록 생성하기"

[사진 올리기]
  점선 업로드 존 — "사진을 눌러 업로드 / 다중 선택 가능"

[선택된 사진 미리보기]
  가로 스크롤 썸네일 스트립 / 마지막 셀: + 추가 버튼

[자동 인식 정보]   ← EXIF 파싱 결과
  뱃지: 📅 날짜  📍 지역명  ☀ 날씨

[여행지 이름]
  텍스트 필드 (EXIF 지역명 자동 입력, 수동 수정 가능)

[태그]
  텍스트 필드 (직접 입력)

[책 색상]
  랜덤 배정된 색상 미리보기 + "변경" 버튼
  → ColorPicker(팔레트 10종 가로 스크롤) 노출

[CTA]
  "카드 자동 생성하기 →" 버튼
  → 슬라이드 기록 카드 화면으로 이동
```

**카드 자동 생성 로직**

```
1. 사진 업로드 → exifr.js로 EXIF 파싱 (DateTimeOriginal, latitude, longitude)
2. 좌표 → 카카오 Local API 역지오코딩 → place_name
3. 중복 장소 제거 → 장소별 대표 사진 1장 선택
4. Open-Meteo API로 날씨 조회 (날짜 + 위치 기반)
5. 장소별 슬라이드 카드 생성 (slide_order 순서 지정)
6. 마지막 장: 전체 방문 핀 지도 생성
```

### 8-3. 슬라이드 기록 카드 (Card Viewer)

**진입 경로:** 카드 자동 생성 완료 후 자동 진입 / 내 여행 기록 목록 탭

```
[헤더]
  < 뒤로가기 / "MONTH YYYY · 여행지명" / N / 총장수

[다이어리 페이지]   ← 도트 격자 배경 (#FEFEFE + dot pattern)
  ┌──────────────────────────┐
  │  DD  요일                │  ← 날짜 크게
  │                          │
  │  [감정볼] + [스티커들]    │  ← 자유 배치
  │  [무드 레이블 뱃지]        │
  │                          │
  │  📍 장소명                │
  │  메모 텍스트               │
  │                          │
  │  ┌────────────────────┐  │
  │  │       사진          │  │  ← 풀블리드 이미지
  │  └────────────────────┘  │
  │  🎵 ✨ 🚗  (플로팅 스티커)  │
  │                          │
  │  [태그] [태그] [날씨]     │
  └──────────────────────────┘

[하단 액션]
  [수정하기]  [다음 →]

[마지막 장 — 지도]
  - 여행명 + 날짜 헤더
  - 카카오맵: 방문지 핀 전체 표시
  - "총 N곳 방문"
  - [저장하기 ✓] → 홈 슬라이드에 랜덤 색상 책으로 반영
```

### 8-4. 내 여행 기록 (List)

**진입 경로:** 홈 슬라이드 6번째 더보기 탭

```
[타이틀] "내 여행 기록"

[필터 탭]
  [시간순 ●] [지역별] [카테고리]

[목록 — 연도별 그룹]
  2025년
    아이콘  여행지명     날짜범위 · N곳   → 탭 시 카드 뷰어 진입
  2024년
    아이콘  여행지명     날짜범위 · N곳
```

### 8-5. 검색 (Search)

**진입 경로:** 홈 상단 검색 아이콘 탭

```
[검색창]
  🔍 "장소, 태그로 검색"

[카테고리 칩]
  [카페] [맛집] [숙소] [관광]

[최근 검색어]
  칩 형태 나열

[검색 결과]
  썸네일 + 장소명 + 📍 지역 · 카테고리 · 날짜
  → 탭 시 장소 상세 진입
```

---

## 9. 유저 플로우 (User Flow)

```
앱 진입
  └─ 구글 로그인
       └─ 홈 화면
            │
            ├─ [슬라이드 스와이프] 여행지 1~5 탐색
            │       └─ 책 탭 → 슬라이드 카드 뷰어
            │
            ├─ [슬라이드 6번째: 더보기] 탭
            │       └─ 내 여행 기록 (시간순/지역별/카테고리)
            │               └─ 여행 항목 탭 → 슬라이드 카드 뷰어
            │
            ├─ [상단 검색 아이콘] 탭
            │       └─ 검색 화면 → 결과 탭 → 장소 상세
            │
            └─ [FAB +] 탭
                    └─ 여행 기록 생성하기
                            ├─ 사진 업로드 (다중)
                            ├─ EXIF 자동 파싱 확인
                            ├─ 여행지명 / 태그 / 책 색상 확인·변경
                            └─ [카드 자동 생성하기] 탭
                                    └─ 슬라이드 카드 뷰어
                                            ├─ 장소별 슬라이드 확인
                                            ├─ [수정하기] → 편집 모드
                                            └─ 마지막 장: 지도 확인
                                                    └─ [저장하기] → 홈 반영
```

---

## 10. 인증 설계 — Google OAuth + Supabase

**인증 흐름**

```
로그인 화면
  └─ [구글로 로그인] 탭
       └─ Supabase Auth → Google OAuth 팝업
            └─ 구글 계정 선택 & 동의
                 └─ user.id / email / avatar 발급
                      └─ 세션 저장 (localStorage)
                           └─ 홈 화면 → 본인 여행 기록 자동 로드
```

**Supabase 설정 순서**
1. Supabase 프로젝트 생성
2. Authentication → Providers → Google 활성화
3. Google Cloud Console에서 OAuth 2.0 클라이언트 생성
   - 승인된 리디렉션 URI: `https://<project>.supabase.co/auth/v1/callback`
4. Client ID / Secret → Supabase Google Provider에 입력
5. Site URL: `http://localhost:5173` (개발) / Vercel URL (배포)

**환경변수 (.env)**

```env
VITE_SUPABASE_URL=https://xxxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGci...
```

**주요 코드**

```javascript
// src/lib/supabase.js
import { createClient } from '@supabase/supabase-js'
export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
)

// 구글 로그인
export const signInWithGoogle = async () => {
  await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: { redirectTo: window.location.origin }
  })
}

// 로그아웃
export const signOut = async () => {
  await supabase.auth.signOut()
}

// App.jsx — 세션 감지
useEffect(() => {
  supabase.auth.getSession().then(({ data: { session } }) => setSession(session))
  const { data: { subscription } } = supabase.auth.onAuthStateChange(
    (_event, session) => setSession(session)
  )
  return () => subscription.unsubscribe()
}, [])

// session 있으면 <HomeScreen />, 없으면 <LoginScreen />
```

---

## 11. DB 설계 — Supabase PostgreSQL

**테이블 구조**

```sql
-- trips: 여행 단위 기록
CREATE TABLE trips (
  id          UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id     UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  title       TEXT NOT NULL,
  date_start  DATE NOT NULL,
  date_end    DATE NOT NULL,
  cover_color TEXT DEFAULT '#E8573A',   -- 랜덤 배정, 사용자 변경 가능
  spine_color TEXT DEFAULT '#C43D22',
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- places: 장소별 슬라이드 카드 (1장 = 1 row)
CREATE TABLE places (
  id          UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  trip_id     UUID REFERENCES trips(id) ON DELETE CASCADE NOT NULL,
  user_id     UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  place_name  TEXT NOT NULL,
  photo_url   TEXT,
  latitude    FLOAT,
  longitude   FLOAT,
  visited_at  TIMESTAMPTZ,
  weather     TEXT,
  tags        TEXT[],
  stickers    TEXT[],
  memo        TEXT,
  slide_order INT DEFAULT 0,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- photos: Supabase Storage 업로드 메타
CREATE TABLE photos (
  id           UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  place_id     UUID REFERENCES places(id) ON DELETE CASCADE NOT NULL,
  user_id      UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  storage_path TEXT NOT NULL,
  exif_date    TIMESTAMPTZ,
  exif_lat     FLOAT,
  exif_lng     FLOAT,
  created_at   TIMESTAMPTZ DEFAULT NOW()
);
```

**RLS 정책 — 본인 데이터만 조회**

```sql
-- trips
ALTER TABLE trips ENABLE ROW LEVEL SECURITY;
CREATE POLICY "본인만" ON trips FOR ALL USING (auth.uid() = user_id);

-- places
ALTER TABLE places ENABLE ROW LEVEL SECURITY;
CREATE POLICY "본인만" ON places FOR ALL USING (auth.uid() = user_id);

-- photos
ALTER TABLE photos ENABLE ROW LEVEL SECURITY;
CREATE POLICY "본인만" ON photos FOR ALL USING (auth.uid() = user_id);
```

**Storage 버킷**
- 버킷명: `trip-photos`
- 경로 규칙: `{user_id}/{trip_id}/{filename}`

**주요 쿼리**

```javascript
// 홈: 본인 최근 여행 5개
const { data } = await supabase
  .from('trips')
  .select('id, title, date_start, date_end, cover_color, spine_color, places(count)')
  .order('date_start', { ascending: false })
  .limit(5)

// 여행 상세: 장소 카드 전체 (슬라이드 순서)
const { data } = await supabase
  .from('places')
  .select('*')
  .eq('trip_id', tripId)
  .order('slide_order', { ascending: true })

// 색상 변경
await supabase
  .from('trips')
  .update({ cover_color, spine_color })
  .eq('id', tripId)

// 사진 업로드
const path = `${userId}/${tripId}/${file.name}`
await supabase.storage.from('trip-photos').upload(path, file)
```

---

## 12. 기술 스택

| 역할 | 선택 |
|------|------|
| 프론트 | React (Vite) |
| 인증 + DB | Supabase |
| 지도 | Kakao Maps API |
| 사진 EXIF 파싱 | exifr.js |
| 장소명 역지오코딩 | Kakao Local API |
| 날씨 | Open-Meteo |
| 배포 | Vercel |

---

## 13. MVP 범위

**MVP에 포함**
- 구글 소셜 로그인
- 홈 여행지 슬라이드 (책 스택 UI, 랜덤 색상 배정)
- FAB → 사진 업로드 → EXIF 파싱 → 슬라이드 카드 자동 생성
- 슬라이드 카드 뷰어 (도트 속지 + 스티커 + 지도 마지막 장)
- 내 여행 기록 목록 (시간순)
- 검색 (키워드 + 카테고리 필터)

**MVP 이후 2차**
- 카카오 로그인
- 날씨 자동 연동
- 스티커 커스텀
- 카드 수동 편집 고도화
- 지도 핀 클러스터링
- PDF / 포토북 내보내기

---

## 14. 컴포넌트 명세

### HomeSlider
```
props:
  trips: Trip[]         // 최근 5개 여행 데이터
  activeIndex: number   // 현재 슬라이드 인덱스
  onTripTap(id)         // 여행 카드 뷰어 진입
  onMoreTap()           // 내 여행 기록 진입
```

### BookCard
```
props:
  title: string         // 여행지명
  dateRange: string     // 날짜 범위
  placeCount: number    // 방문지 수
  color: string         // 랜덤 배정 책 색상 hex
  spineColor: string    // 등 색상 hex
  onColorChange()       // 색상 변경 트리거
```

### ColorPicker
```
props:
  colors: BookColor[]   // 팔레트 10종
  selected: string      // 현재 선택 색상
  onChange(color)       // 색상 변경 콜백
UI: 가로 스크롤 원형 색상 칩
```

### SlideCardViewer
```
props:
  cards: CardData[]     // 장소별 카드 배열
  tripTitle: string
  onSave()              // 저장 후 홈 반영
```

### DiaryCard
```
props:
  date: string          // "12 WED"
  weather: string       // 날씨 텍스트
  placeName: string     // 장소명
  photo: string         // 이미지 URL
  tags: string[]        // 태그 배열
  stickers: string[]    // 스티커 이모지 배열
  memo: string
```

### MapCard
```
props:
  pins: { lat, lng, label }[]
  tripTitle: string
  totalCount: number
```

### 유틸 함수

**책 색상 랜덤 배정**
```javascript
// assignBookColor(recentColors: string[]) → { name, book, spine }
// 최근 사용 색상 제외 후 랜덤 배정
```

**EXIF 파싱**
```javascript
// parsePhotoMeta(file: File) → { date, lat, lng, placeName, weather }
// 1. exifr.parse → DateTimeOriginal, latitude, longitude
// 2. 카카오 Local API 역지오코딩 → placeName
// 3. Open-Meteo → weather
```

---

*v0.5 — 상세 기능 명세 | 기획 배경은 triply_기획서.md 참고 | 클로드 코드 구현 전달용*
