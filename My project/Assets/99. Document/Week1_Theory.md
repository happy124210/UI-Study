# 1주차: "Absolute Defense" - 이론 학습 자료

## 목표
어떤 해상도(19:9, 4:3 등)에서도 깨지지 않는 HUD와 중앙 팝업을 만들기 위한 핵심 이론을 학습합니다.

---

## 1. RectTransform의 본질

### Transform vs RectTransform

Unity의 UI 시스템은 **RectTransform**을 사용합니다. 일반 3D 오브젝트의 `Transform`과는 근본적으로 다릅니다.

#### Transform (3D 공간)
- **위치**: `Vector3` (x, y, z) - 월드/로컬 좌표계
- **회전**: `Quaternion` 또는 `Vector3` (오일러 각)
- **크기**: `Vector3` (scale)
- **좌표계**: 절대적, 픽셀 단위가 아님

#### RectTransform (UI 공간)
- **위치**: `Vector2` (x, y) - 부모 RectTransform 기준
- **크기**: `Rect` (width, height) - 픽셀 단위
- **앵커**: 부모의 어느 지점에 "고정"할지 결정
- **좌표계**: 상대적, 픽셀 단위

### 왜 UI는 RectTransform인가?

1. **해상도 독립성**: 다양한 화면 크기에서 일관된 레이아웃 유지
2. **앵커 시스템**: 부모의 특정 지점에 상대적으로 배치 가능
3. **레이아웃 그룹**: 자동 정렬 및 크기 조정 지원
4. **픽셀 퍼펙트**: 디자이너가 의도한 정확한 크기 유지

**핵심**: RectTransform은 "부모의 어느 부분에 얼마나 떨어져 있는가"를 정의합니다.

---

## 2. Anchors (앵커)의 개념

### 앵커란 무엇인가?

앵커(Anchor)는 **부모 RectTransform 내에서의 정규화된 위치(0~1)**를 나타냅니다. UI 요소가 부모의 어느 지점에 "고정"될지를 결정합니다.

### Min/Max 앵커의 의미

RectTransform의 앵커는 두 개의 점으로 정의됩니다:
- **Anchor Min**: 왼쪽 하단 모서리의 정규화 좌표 (0~1)
- **Anchor Max**: 오른쪽 상단 모서리의 정규화 좌표 (0~1)

```
부모 RectTransform (1920x1080 기준)

(0,1) ──────────────── (1,1)
  │                     │
  │   Anchor Min        │
  │   (0.5, 0.5)        │
  │         ┌───┐       │
  │         │UI │       │
  │         └───┘       │
  │   Anchor Max        │
  │   (0.75, 0.75)      │
  │                     │
(0,0) ──────────────── (1,0)
```

#### 정규화 좌표 (0~1)의 의미

- `(0, 0)`: 부모의 왼쪽 하단 모서리
- `(1, 1)`: 부모의 오른쪽 상단 모서리
- `(0.5, 0.5)`: 부모의 정중앙
- `(0, 1)`: 부모의 왼쪽 상단 모서리

**중요**: 앵커는 **부모의 크기가 변해도** 같은 비율 위치를 유지합니다.

### Anchor Presets의 동작 원리

Unity 에디터의 Anchor Presets는 Min/Max 앵커를 빠르게 설정하는 도구입니다.

#### 주요 Preset 예시

**1. Top-Left (Alt+Shift 클릭)**
```
Anchor Min: (0, 1)
Anchor Max: (0, 1)
Pivot: (0, 1)
```
- 부모의 왼쪽 상단 모서리에 고정
- 해상도가 변해도 항상 왼쪽 상단에 위치

**2. Middle-Center (Alt+Shift 클릭)**
```
Anchor Min: (0.5, 0.5)
Anchor Max: (0.5, 0.5)
Pivot: (0.5, 0.5)
```
- 부모의 정중앙에 고정
- 해상도가 변해도 항상 중앙에 위치

**3. Stretch-Stretch (Alt+Shift 클릭)**
```
Anchor Min: (0, 0)
Anchor Max: (1, 1)
Pivot: (0.5, 0.5)
```
- 부모의 전체 영역에 늘어남
- 해상도가 변해도 부모 크기에 맞춰 자동 확장

**4. Top-Stretch (Alt+Shift 클릭)**
```
Anchor Min: (0, 1)
Anchor Max: (1, 1)
Pivot: (0.5, 1)
```
- 상단에 고정, 가로로만 늘어남
- 높이는 고정, 너비는 부모에 맞춤

### 앵커가 위치와 크기에 미치는 영향

#### 위치 (Position)

앵커가 위치에 미치는 영향은 **Anchored Position**으로 표현됩니다.

```
실제 위치 = 앵커 위치 + Anchored Position
```

**예시**: 
- 앵커가 `(0.5, 0.5)` (중앙)
- Anchored Position이 `(100, 50)`
- → 부모 중앙에서 오른쪽으로 100px, 위로 50px 떨어진 위치

#### 크기 (Size)

앵커 Min과 Max가 다를 때, UI 요소의 크기는 **앵커 간 거리**에 비례합니다.

```
크기 = (Anchor Max - Anchor Min) × 부모 크기 + Size Delta
```

**예시**:
- 부모 크기: 1920x1080
- Anchor Min: (0.25, 0.25)
- Anchor Max: (0.75, 0.75)
- → UI 크기: 960x540 (부모의 50% × 50%)

**핵심 원리**: 
- Min = Max일 때: 고정 크기 (Size Delta로 조절)
- Min ≠ Max일 때: 부모 크기에 비례하는 크기

---

## 3. Pivot (피벗)의 역할

### 피벗이란?

피벗(Pivot)은 UI 요소의 **회전, 스케일, 위치 계산의 기준점**입니다. 정규화 좌표(0~1)로 표현됩니다.

### 피벗의 위치

```
UI 요소 내부 좌표계

(0,1) ──────────── (1,1)
  │                 │
  │                 │
  │      (0.5,0.5)  │  ← Pivot이 (0.5, 0.5)면 중앙
  │        ●        │
  │                 │
  │                 │
(0,0) ──────────── (1,0)
```

#### 주요 피벗 위치

- `(0, 0)`: 왼쪽 하단 모서리
- `(0.5, 0.5)`: 정중앙 (기본값)
- `(1, 1)`: 오른쪽 상단 모서리
- `(0, 1)`: 왼쪽 상단 모서리

### 회전/스케일의 기준점

#### 회전 (Rotation)
```
Pivot (0, 0) - 왼쪽 하단 기준 회전
┌─────┐
│  UI │  → 회전 시 왼쪽 하단이 고정
└─────┘
  ●

Pivot (0.5, 0.5) - 중앙 기준 회전
┌─────┐
│  UI │  → 회전 시 중앙이 고정
│  ●  │
└─────┘
```

#### 스케일 (Scale)
스케일도 피벗을 기준으로 확대/축소됩니다.

### 앵커와 피벗의 관계

**중요한 혼동 포인트**: 앵커와 피벗은 **다른 개념**입니다!

- **앵커**: 부모의 어느 지점에 고정할지 (부모 기준)
- **피벗**: 자신의 어느 지점을 기준으로 할지 (자신 기준)

#### 일반적인 조합

**코너 UI (예: Top-Left)**
```
Anchor: (0, 1)  ← 부모의 왼쪽 상단에 고정
Pivot: (0, 1)   ← 자신의 왼쪽 상단을 기준점으로
```
→ 해상도가 변해도 왼쪽 상단 모서리가 항상 같은 위치

**중앙 UI (예: 팝업)**
```
Anchor: (0.5, 0.5)  ← 부모의 중앙에 고정
Pivot: (0.5, 0.5)   ← 자신의 중앙을 기준점으로
```
→ 해상도가 변해도 항상 화면 중앙에 위치

**주의**: Anchor Preset을 사용할 때 Alt+Shift를 누르면 앵커와 피벗이 함께 설정됩니다.

---

## 4. Canvas Scaler 심화

### Canvas Scaler의 목적

Canvas Scaler는 **다양한 해상도에서 UI의 일관된 크기와 비율을 유지**하기 위한 컴포넌트입니다.

### UI Scale Mode 비교

#### 1. Constant Pixel Size

**동작 방식**:
- UI 요소의 픽셀 크기가 항상 동일
- 해상도가 높아지면 UI가 작아 보임
- 해상도가 낮아지면 UI가 커 보임

**공식**:
```
실제 크기 = 설정된 픽셀 크기 × Scale Factor
```

**사용 케이스**:
- 픽셀 아트 게임
- 고정 해상도 게임
- 레트로 스타일 UI

**단점**:
- 다양한 기기에서 크기 일관성 부족
- 고해상도 기기에서 UI가 너무 작아질 수 있음

#### 2. Scale With Screen Size (권장)

**동작 방식**:
- Reference Resolution을 기준으로 스케일 계산
- 화면 크기에 따라 UI가 비례적으로 확대/축소

**공식**:
```
Scale Factor = Screen Size / Reference Resolution
```

**핵심 설정**:
- **Reference Resolution**: 기준 해상도 (예: 1920x1080)
- **Screen Match Mode**: 가로/세로 중 어느 것을 우선할지
- **Match**: 0.0 (가로 우선) ~ 1.0 (세로 우선)

### Reference Resolution의 의미

Reference Resolution은 **디자이너가 작업한 기준 해상도**입니다.

**예시**:
- Reference Resolution: 1920x1080
- 실제 화면: 3840x2160 (4K)
- → UI는 2배로 확대되어 표시

**중요**: Reference Resolution은 "이 해상도에서 1:1로 보인다"는 의미가 아닙니다. "이 해상도를 기준으로 스케일을 계산한다"는 의미입니다.

### Match Width Or Height의 계산 공식

Match 값에 따라 가로와 세로 중 어느 것을 기준으로 스케일할지 결정합니다.

#### 공식

```
logWidth = log(Screen Width / Reference Width)
logHeight = log(Screen Height / Reference Height)
logMatch = log(Match)

scale = 2^(logWidth × (1 - Match) + logHeight × Match)
```

**간단한 이해**:
- **Match = 0.0**: 가로 비율만 고려 (세로는 무시)
- **Match = 1.0**: 세로 비율만 고려 (가로는 무시)
- **Match = 0.5**: 가로와 세로의 중간값 (균형)

#### 실제 예시

**Reference Resolution: 1920x1080**
**Match: 0.5**

| 실제 화면 | 가로 비율 | 세로 비율 | 최종 스케일 |
|---------|---------|---------|-----------|
| 1920x1080 | 1.0 | 1.0 | 1.0 |
| 3840x2160 | 2.0 | 2.0 | 2.0 |
| 1334x750 | 0.695 | 0.694 | 0.695 |
| 1024x768 | 0.533 | 0.711 | 0.622 |

**Match = 0.0 (가로 우선)**:
- 1024x768 화면: 스케일 = 0.533 (가로 기준)
- 세로가 짧아도 가로 비율만 따름

**Match = 1.0 (세로 우선)**:
- 1024x768 화면: 스케일 = 0.711 (세로 기준)
- 가로가 좁아도 세로 비율만 따름

**Match = 0.5 (균형)**:
- 1024x768 화면: 스케일 = 0.622 (중간값)
- 가로와 세로를 모두 고려

### 다양한 해상도 비율 대응 전략

#### 전략 1: Match = 0.5 (권장)

**장점**:
- 대부분의 해상도에서 균형잡힌 스케일
- 가로/세로 모두 고려

**단점**:
- 극단적인 비율(예: 21:9)에서 약간의 왜곡 가능

**사용 케이스**:
- 일반적인 모바일/PC 게임
- 다양한 기기를 지원해야 하는 경우

#### 전략 2: Match = 0.0 (가로 우선)

**장점**:
- 가로 레이아웃이 중요한 UI에 적합
- 가로 스크롤 게임에 적합

**단점**:
- 세로가 짧은 화면에서 UI가 잘릴 수 있음

**사용 케이스**:
- 가로 스크롤 게임
- 가로 레이아웃 중심 UI

#### 전략 3: Match = 1.0 (세로 우선)

**장점**:
- 세로 레이아웃이 중요한 UI에 적합
- 세로 스크롤 게임에 적합

**단점**:
- 가로가 좁은 화면에서 UI가 잘릴 수 있음

**사용 케이스**:
- 세로 스크롤 게임
- 세로 레이아웃 중심 UI

#### 전략 4: Constant Physical Size

**동작 방식**:
- 물리적 크기(인치)를 기준으로 스케일
- DPI를 고려

**사용 케이스**:
- AR/VR 애플리케이션
- 물리적 크기가 중요한 UI

---

## 5. "Absolute Defense" 구현 전략

### 코너 고정 UI의 앵커 설정 방법

#### 목표
해상도가 변해도 항상 화면의 특정 코너에 고정되어 있는 UI 요소

#### 구현 방법

**1. Top-Left (좌상단)**
```
1. UI 요소 선택
2. Anchor Preset에서 Top-Left 선택
3. Alt+Shift 클릭 (앵커와 피벗 모두 설정)
4. Anchored Position 설정 (예: X=50, Y=-50)
```

**설정값**:
- Anchor Min: (0, 1)
- Anchor Max: (0, 1)
- Pivot: (0, 1)
- Anchored Position: (50, -50) ← 왼쪽 상단에서 오른쪽으로 50px, 아래로 50px

**2. Top-Right (우상단)**
- Anchor Min: (1, 1)
- Anchor Max: (1, 1)
- Pivot: (1, 1)
- Anchored Position: (-50, -50)

**3. Bottom-Left (좌하단)**
- Anchor Min: (0, 0)
- Anchor Max: (0, 0)
- Pivot: (0, 0)
- Anchored Position: (50, 50)

**4. Bottom-Right (우하단)**
- Anchor Min: (1, 0)
- Anchor Max: (1, 0)
- Pivot: (1, 0)
- Anchored Position: (-50, 50)

#### 시각적 다이어그램

```
해상도: 1920x1080                    해상도: 3840x2160

┌─────────────────────┐            ┌─────────────────────────────┐
│●                    ●│            │●                            ●│
│                     │            │                             │
│                     │            │                             │
│                     │            │                             │
│                     │            │                             │
│                     │            │                             │
│●                    ●│            │●                            ●│
└─────────────────────┘            └─────────────────────────────┘

모든 해상도에서 코너 UI는 항상 같은 위치에 고정됨
```

### 중앙 팝업의 앵커 설정 방법

#### 목표
해상도가 변해도 항상 화면 중앙에 위치하는 팝업

#### 구현 방법

```
1. 팝업 UI 요소 선택
2. Anchor Preset에서 Middle-Center 선택
3. Alt+Shift 클릭
4. Anchored Position: (0, 0) ← 중앙이므로 오프셋 불필요
5. 크기 설정 (Width, Height)
```

**설정값**:
- Anchor Min: (0.5, 0.5)
- Anchor Max: (0.5, 0.5)
- Pivot: (0.5, 0.5)
- Anchored Position: (0, 0)
- Size: (600, 400) ← 고정 크기

#### 시각적 다이어그램

```
해상도: 1920x1080                    해상도: 1024x768

┌─────────────────────┐            ┌──────────────┐
│                     │            │              │
│                     │            │              │
│      ┌─────┐        │            │   ┌─────┐   │
│      │팝업 │        │            │   │팝업 │   │
│      └─────┘        │            │   └─────┘   │
│                     │            │              │
│                     │            │              │
└─────────────────────┘            └──────────────┘

모든 해상도에서 팝업은 항상 중앙에 위치
```

### 주의사항과 함정

#### 함정 1: 절대 좌표 사용 금지

**나쁜 예**:
```csharp
rectTransform.anchoredPosition = new Vector2(100, 200);
// 해상도가 변하면 UI가 잘리거나 이상한 위치로 이동
```

**좋은 예**:
```csharp
// 앵커를 먼저 설정하고, 앵커 기준 오프셋 사용
rectTransform.anchorMin = new Vector2(0, 1);
rectTransform.anchorMax = new Vector2(0, 1);
rectTransform.anchoredPosition = new Vector2(50, -50);
// 해상도가 변해도 항상 왼쪽 상단에서 (50, -50) 위치
```

#### 함정 2: 부모의 크기를 직접 참조

**나쁜 예**:
```csharp
float parentWidth = rectTransform.parent.GetComponent<RectTransform>().rect.width;
rectTransform.sizeDelta = new Vector2(parentWidth * 0.5f, 100);
// 부모 크기가 변하면 계산이 틀어짐
```

**좋은 예**:
```csharp
// 앵커를 사용하여 부모 크기의 50%로 설정
rectTransform.anchorMin = new Vector2(0.25f, 0);
rectTransform.anchorMax = new Vector2(0.75f, 0);
rectTransform.sizeDelta = new Vector2(0, 100);
// 부모 크기의 50% 너비, 100px 높이
```

#### 함정 3: Pivot과 Anchor의 혼동

**나쁜 예**:
```csharp
// Anchor를 중앙으로 설정했는데 Pivot을 왼쪽으로 설정
rectTransform.anchorMin = new Vector2(0.5f, 0.5f);
rectTransform.anchorMax = new Vector2(0.5f, 0.5f);
rectTransform.pivot = new Vector2(0, 0); // 혼란스러움
```

**좋은 예**:
```csharp
// Anchor와 Pivot을 일치시키는 것이 일반적
rectTransform.anchorMin = new Vector2(0.5f, 0.5f);
rectTransform.anchorMax = new Vector2(0.5f, 0.5f);
rectTransform.pivot = new Vector2(0.5f, 0.5f); // 일치
```

#### 함정 4: Canvas Scaler 설정 무시

**나쁜 예**:
- Canvas Scaler 없이 작업
- Constant Pixel Size 사용 (다양한 해상도 대응 불가)

**좋은 예**:
- Canvas Scaler 추가
- Scale With Screen Size 사용
- 적절한 Reference Resolution 설정
- Match 값 조정

---

## 6. 실전 베스트 프랙티스

### Safe Area 대응 (노치, 홈 인디케이터)

#### 문제 상황

최신 모바일 기기(iPhone X 이상, 일부 Android)는:
- **노치(Notch)**: 화면 상단의 카메라 영역
- **홈 인디케이터**: 화면 하단의 제스처 영역

이 영역들은 UI가 가려지면 안 되는 "안전 영역"입니다.

#### 해결 방법

**1. Unity의 Safe Area 컴포넌트 사용 (권장)**

```csharp
using UnityEngine;

public class SafeAreaFitter : MonoBehaviour
{
    private RectTransform rectTransform;
    private Rect lastSafeArea = new Rect(0, 0, 0, 0);

    void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
    }

    void Update()
    {
        Rect safeArea = Screen.safeArea;
        
        if (safeArea != lastSafeArea)
        {
            ApplySafeArea(safeArea);
        }
    }

    void ApplySafeArea(Rect safeArea)
    {
        Vector2 anchorMin = safeArea.position;
        Vector2 anchorMax = safeArea.position + safeArea.size;
        
        anchorMin.x /= Screen.width;
        anchorMin.y /= Screen.height;
        anchorMax.x /= Screen.width;
        anchorMax.y /= Screen.height;
        
        rectTransform.anchorMin = anchorMin;
        rectTransform.anchorMax = anchorMax;
        
        lastSafeArea = safeArea;
    }
}
```

**2. Canvas의 Render Mode 확인**

- **Screen Space - Overlay**: Safe Area 자동 적용 안 됨 (수동 처리 필요)
- **Screen Space - Camera**: Safe Area 자동 적용 안 됨 (수동 처리 필요)
- **World Space**: Safe Area 개념 없음

**3. 실전 적용 예시**

```
Canvas (Safe Area Fitter 적용)
├── SafeAreaPanel (전체 Safe Area)
│   ├── TopBar (상단 바)
│   ├── Content (콘텐츠)
│   └── BottomBar (하단 바)
└── FullScreenPanel (전체 화면, 배경용)
```

### 다양한 기기 해상도 테스트 체크리스트

#### 필수 테스트 해상도

**모바일 (세로)**
- iPhone SE (2세대): 750x1334 (16:9)
- iPhone 12/13: 1170x2532 (19.5:9)
- iPhone 14 Pro Max: 1290x2796 (19.5:9)
- Galaxy S21: 1080x2400 (20:9)
- iPad: 1536x2048 (4:3)

**모바일 (가로)**
- iPhone 12 가로: 2532x1170
- iPad 가로: 2048x1536

**PC/콘솔**
- 1920x1080 (16:9) - Full HD
- 2560x1440 (16:9) - 2K
- 3840x2160 (16:9) - 4K
- 2560x1080 (21:9) - 울트라와이드

**극단적 비율**
- 1024x768 (4:3) - 구형 모니터
- 3440x1440 (21:9) - 울트라와이드
- 1080x1920 (9:16) - 세로 모바일

#### 테스트 체크리스트

**레이아웃 테스트**
- [ ] 모든 코너 UI가 화면 밖으로 나가지 않는가?
- [ ] 중앙 팝업이 항상 화면 중앙에 위치하는가?
- [ ] UI 요소들이 서로 겹치지 않는가?
- [ ] 텍스트가 잘리지 않는가?

**크기 테스트**
- [ ] UI 요소의 크기가 너무 작거나 크지 않은가?
- [ ] 버튼이 터치하기 적절한 크기인가? (최소 44x44px)
- [ ] 텍스트가 읽기 쉬운 크기인가?

**Safe Area 테스트**
- [ ] 노치 영역에 UI가 가려지지 않는가?
- [ ] 홈 인디케이터 영역에 UI가 가려지지 않는가?
- [ ] Safe Area가 적용된 기기에서 정상 동작하는가?

**극단적 비율 테스트**
- [ ] 21:9 울트라와이드에서 UI가 정상인가?
- [ ] 4:3 비율에서 UI가 정상인가?
- [ ] 세로 모드에서 UI가 정상인가?

#### Unity Game 뷰에서 테스트하는 방법

1. Game 뷰 상단의 해상도 드롭다운 클릭
2. "+" 버튼으로 커스텀 해상도 추가
3. 주요 해상도들을 모두 테스트
4. Aspect Ratio도 변경하여 테스트

---

## 7. 개념 비교표

### Canvas Scaler UI Scale Mode 비교

| 모드 | 해상도 독립성 | 픽셀 정확도 | 사용 케이스 | 권장도 |
|------|------------|-----------|-----------|--------|
| **Constant Pixel Size** | 낮음 | 높음 | 픽셀 아트, 고정 해상도 | ⭐⭐ |
| **Scale With Screen Size** | 높음 | 중간 | 대부분의 게임 | ⭐⭐⭐⭐⭐ |
| **Constant Physical Size** | 중간 | 낮음 | AR/VR | ⭐⭐⭐ |

### Match 값 비교 (Scale With Screen Size)

| Match 값 | 기준 | 장점 | 단점 | 사용 케이스 |
|---------|------|------|------|-----------|
| **0.0** | 가로 우선 | 가로 레이아웃 보존 | 세로 짧을 때 잘림 | 가로 스크롤 게임 |
| **0.5** | 균형 | 대부분 해상도 적합 | 극단적 비율에서 약간 왜곡 | 일반 게임 (권장) |
| **1.0** | 세로 우선 | 세로 레이아웃 보존 | 가로 좁을 때 잘림 | 세로 스크롤 게임 |

### Anchor Preset별 사용 케이스

| Preset | Anchor Min | Anchor Max | 사용 케이스 | 예시 |
|--------|-----------|-----------|-----------|------|
| **Top-Left** | (0, 1) | (0, 1) | 좌상단 고정 UI | 체력바, 점수 |
| **Top-Right** | (1, 1) | (1, 1) | 우상단 고정 UI | 시간, 설정 버튼 |
| **Bottom-Left** | (0, 0) | (0, 0) | 좌하단 고정 UI | 조이스틱, 버튼 |
| **Bottom-Right** | (1, 0) | (1, 0) | 우하단 고정 UI | 인벤토리, 메뉴 |
| **Middle-Center** | (0.5, 0.5) | (0.5, 0.5) | 중앙 팝업 | 다이얼로그, 메뉴 |
| **Stretch-Stretch** | (0, 0) | (1, 1) | 전체 화면 | 배경, 패널 |
| **Top-Stretch** | (0, 1) | (1, 1) | 상단 바 | 상단 메뉴바 |
| **Left-Stretch** | (0, 0) | (0, 1) | 좌측 사이드바 | 인벤토리 패널 |

---

## 8. 시각적 다이어그램

### 앵커 Min(0,0) Max(1,1)의 의미

```
부모 RectTransform (1920x1080)

Anchor Min (0, 0) = 왼쪽 하단
Anchor Max (1, 1) = 오른쪽 상단

(0,1) ──────────────── (1,1)
  │                       │
  │                       │
  │   Anchor Min          │
  │   (0, 0)              │
  │   ●                   │
  │                       │
  │                       │
  │                       │
  │   Anchor Max          │
  │   (1, 1)              │
  │                       │
  │                       │
(0,0) ──────────────── (1,0)
```

**의미**: UI 요소가 부모의 전체 영역을 차지함 (Stretch-Stretch)

### 다양한 Anchor Preset의 실제 동작

#### Top-Left Preset

```
해상도 변경 전 (1920x1080)        해상도 변경 후 (3840x2160)

┌─────────────────────┐          ┌─────────────────────────────┐
│●                     │          │●                             │
│  UI                  │          │  UI                          │
│                      │          │                              │
│                      │          │                              │
│                      │          │                              │
│                      │          │                              │
│                      │          │                              │
└─────────────────────┘          └─────────────────────────────┘

● = Anchor (0, 1) = 항상 왼쪽 상단에 고정
```

#### Middle-Center Preset

```
해상도 변경 전 (1920x1080)        해상도 변경 후 (1024x768)

┌─────────────────────┐          ┌──────────────┐
│                     │          │              │
│                     │          │              │
│      ┌─────┐        │          │   ┌─────┐   │
│      │ UI │        │          │   │ UI │   │
│      └─────┘        │          │   └─────┘   │
│                     │          │              │
│                     │          │              │
└─────────────────────┘          └──────────────┘

UI는 항상 중앙에 위치 (Anchor = 0.5, 0.5)
```

### 해상도 변경 시 UI 요소의 이동 경로

#### 시나리오: Top-Left UI 요소

```
1920x1080 → 3840x2160 (2배 확대)

위치: (50, -50) → (100, -100)  ← Canvas Scaler에 의해 2배
하지만 앵커 기준 상대 위치는 동일: 왼쪽 상단에서 (50, -50)

1920x1080 → 1024x768 (축소)

위치: (50, -50) → (약 27, -35)  ← Canvas Scaler에 의해 축소
하지만 앵커 기준 상대 위치는 동일: 왼쪽 상단에서 (50, -50)
```

**핵심**: 앵커를 사용하면 **상대적 위치**가 유지됩니다.

---

## 9. 수학적 공식 정리

### Canvas Scaler 스케일 계산 (Match Width Or Height)

```
logWidth = log(Screen Width / Reference Width)
logHeight = log(Screen Height / Reference Height)
logMatch = log(Match)

scale = 2^(logWidth × (1 - Match) + logHeight × Match)
```

**예시 계산**:
- Reference: 1920x1080
- Screen: 3840x2160
- Match: 0.5

```
logWidth = log(3840 / 1920) = log(2) ≈ 0.693
logHeight = log(2160 / 1080) = log(2) ≈ 0.693
logMatch = log(0.5) ≈ -0.693

scale = 2^(0.693 × 0.5 + 0.693 × 0.5)
     = 2^0.693
     = 2
```

### RectTransform 위치 계산

```
월드 위치 = 부모 위치 + (앵커 위치 × 부모 크기) + Anchored Position
```

**예시**:
- 부모 크기: 1920x1080
- Anchor: (0.5, 0.5) = 중앙
- Anchored Position: (100, 50)

```
앵커 월드 위치 = (1920 × 0.5, 1080 × 0.5) = (960, 540)
최종 위치 = (960, 540) + (100, 50) = (1060, 590)
```

### RectTransform 크기 계산

**케이스 1: Anchor Min = Anchor Max (고정 크기)**
```
크기 = Size Delta
```

**케이스 2: Anchor Min ≠ Anchor Max (비례 크기)**
```
크기 = (Anchor Max - Anchor Min) × 부모 크기 + Size Delta
```

**예시**:
- 부모 크기: 1920x1080
- Anchor Min: (0.25, 0.25)
- Anchor Max: (0.75, 0.75)
- Size Delta: (0, 0)

```
너비 = (0.75 - 0.25) × 1920 = 0.5 × 1920 = 960
높이 = (0.75 - 0.25) × 1080 = 0.5 × 1080 = 540
```

---

## 10. 자주 하는 실수와 해결책

### 실수 1: 절대 좌표를 코드에 하드코딩

**나쁜 예**:
```csharp
rectTransform.anchoredPosition = new Vector2(100, 200);
```

**해결책**: 앵커를 먼저 설정하고 상대적 오프셋 사용
```csharp
rectTransform.anchorMin = new Vector2(0, 1);
rectTransform.anchorMax = new Vector2(0, 1);
rectTransform.anchoredPosition = new Vector2(100, -200);
```

### 실수 2: Canvas Scaler 없이 작업

**증상**: 다른 해상도에서 UI가 깨짐

**해결책**: Canvas에 Canvas Scaler 추가, Scale With Screen Size 선택

### 실수 3: Anchor와 Pivot 혼동

**증상**: UI가 예상과 다른 위치에 배치됨

**해결책**: Anchor Preset 사용 시 Alt+Shift 클릭하여 함께 설정

### 실수 4: 부모 크기를 직접 계산

**나쁜 예**:
```csharp
float width = parent.rect.width * 0.5f;
```

**해결책**: 앵커를 사용하여 비율로 설정
```csharp
rectTransform.anchorMin = new Vector2(0.25f, 0);
rectTransform.anchorMax = new Vector2(0.75f, 1);
```

### 실수 5: Safe Area 무시

**증상**: 노치나 홈 인디케이터에 UI가 가려짐

**해결책**: Safe Area Fitter 컴포넌트 사용 또는 Screen.safeArea 활용

---

## 11. 학습 체크리스트

이 문서를 읽은 후 다음을 확인하세요:

- [ ] RectTransform과 Transform의 차이를 설명할 수 있는가?
- [ ] 앵커 Min/Max의 의미를 이해했는가?
- [ ] Anchor Preset의 동작 원리를 설명할 수 있는가?
- [ ] Pivot의 역할과 앵커와의 차이를 이해했는가?
- [ ] Canvas Scaler의 각 모드를 구분할 수 있는가?
- [ ] Match 값에 따른 스케일 계산을 이해했는가?
- [ ] 코너 고정 UI를 만들 수 있는가?
- [ ] 중앙 팝업을 만들 수 있는가?
- [ ] Safe Area를 처리할 수 있는가?
- [ ] 다양한 해상도에서 테스트하는 방법을 알고 있는가?

---

## 마무리

1주차 "Absolute Defense" 퀘스트의 핵심은 **앵커 시스템을 완벽히 이해**하는 것입니다. 

**기억할 것**:
1. 앵커는 부모의 정규화 좌표 (0~1)
2. 앵커를 사용하면 해상도 독립적인 UI 가능
3. Canvas Scaler는 다양한 해상도 대응의 핵심
4. 절대 좌표 대신 앵커 + 오프셋 사용
5. Safe Area를 고려한 디자인

다음 주차에서는 이 이론을 바탕으로 실제 구현을 진행합니다!