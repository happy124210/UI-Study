# 1주차: "Absolute Defense" - 단계별 실습 가이드

## 목표
이론 자료(`Week1_Theory.md`)를 바탕으로 Unity에서 직접 실습하며 개념을 체득합니다.

**학습 방식**: 이론 확인 → Unity 실습 → 검증 → 정리

---

## 학습 전 준비사항

### 1. Unity 씬 설정

1. Unity 에디터를 엽니다.
2. `Assets/Scenes/Week1_LayoutTest.unity` 씬을 엽니다 (없다면 새 씬 생성).
3. 씬을 저장합니다.

### 2. 필요한 컴포넌트 확인

이 실습에서 사용할 Unity 컴포넌트:
- **Canvas**: UI의 루트 컨테이너
- **Image**: UI 요소 표시
- **TextMeshPro**: 텍스트 표시 (선택)
- **RectTransform**: 모든 UI 요소에 자동으로 붙음

### 3. 이론 자료 읽기 체크리스트

실습 전에 `Week1_Theory.md`의 다음 섹션을 읽어주세요:

- [ ] 1. RectTransform의 본질
- [ ] 2. Anchors (앵커)의 개념
- [ ] 3. Pivot (피벗)의 역할
- [ ] 4. Canvas Scaler 심화
- [ ] 5. "Absolute Defense" 구현 전략

**팁**: 전체를 다 읽지 않아도 됩니다. 각 실습 전에 해당 섹션만 읽어도 충분합니다.

---

## 실습 1: RectTransform 기초 이해

### 목표
Transform과 RectTransform의 차이를 직접 체감하고, 왜 UI는 RectTransform을 사용하는지 이해합니다.

### 이론 확인
`Week1_Theory.md`의 **"1. RectTransform의 본질"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: 3D 오브젝트 생성

1. Hierarchy 창에서 우클릭 → `3D Object` → `Cube` 생성
2. Inspector에서 `Transform` 컴포넌트 확인
3. 다음 속성들을 확인:
   - **Position**: Vector3 (x, y, z)
   - **Rotation**: Vector3 (오일러 각)
   - **Scale**: Vector3 (x, y, z)

#### Step 2: UI 오브젝트 생성

1. Hierarchy 창에서 우클릭 → `UI` → `Canvas` 생성
   - Canvas가 자동으로 생성되며, EventSystem도 함께 생성됩니다.
2. Canvas 하위에 우클릭 → `UI` → `Image` 생성
3. Inspector에서 `RectTransform` 컴포넌트 확인
4. 다음 속성들을 확인:
   - **Anchored Position**: Vector2 (x, y) - Transform의 Position과 다름!
   - **Size Delta**: Vector2 (width, height) - Transform의 Scale과 다름!
   - **Anchor Min/Max**: Vector2 (0~1 정규화 좌표)
   - **Pivot**: Vector2 (0~1 정규화 좌표)

#### Step 3: 속성 비교표 작성

직접 비교해보세요:

| 속성 | Transform (Cube) | RectTransform (Image) |
|------|-----------------|----------------------|
| 위치 | Position (Vector3) | Anchored Position (Vector2) |
| 크기 | Scale (Vector3) | Size Delta (Vector2) |
| 앵커 | 없음 | Anchor Min/Max 있음 |
| 피벗 | 없음 | Pivot 있음 |

**관찰 포인트**: 
- Transform은 3D 공간의 절대 좌표를 사용
- RectTransform은 2D 공간의 상대 좌표를 사용

#### Step 4: 해상도 변경 시 동작 차이 관찰

1. Game 뷰 상단의 해상도 드롭다운 클릭
2. 해상도를 변경해보세요 (예: 1920x1080 → 1024x768)
3. 관찰:
   - **Cube (3D)**: 화면에서의 위치가 변하지 않음 (카메라 기준)
   - **Image (UI)**: 화면 크기에 따라 위치가 상대적으로 유지됨

**왜 이렇게 동작할까?**
- 3D 오브젝트는 월드 좌표계를 사용하므로 화면 크기와 무관
- UI는 화면 좌표계를 사용하므로 화면 크기에 따라 상대적 위치 유지

### 검증

다음 질문에 답할 수 있어야 합니다:

1. Transform과 RectTransform의 가장 큰 차이는?
   - **답**: RectTransform은 앵커 시스템을 사용하여 부모 기준 상대 위치를 정의

2. 왜 UI는 RectTransform을 사용할까?
   - **답**: 다양한 해상도에서 일관된 레이아웃을 유지하기 위해

3. 해상도를 변경했을 때 3D 오브젝트와 UI 오브젝트의 동작 차이는?
   - **답**: 3D는 절대 좌표, UI는 상대 좌표를 사용

### 문제 해결

**문제**: Canvas가 보이지 않아요!
- **해결**: Scene 뷰에서 Canvas를 선택하고, Game 뷰를 확인하세요. Canvas는 기본적으로 화면 전체를 덮습니다.

**문제**: Image가 너무 작아서 안 보여요!
- **해결**: RectTransform의 Width와 Height를 조정하세요 (예: 200x200).

### 심화 학습

- Canvas의 Render Mode를 변경해보고 차이를 관찰해보세요.
- 여러 개의 Image를 생성하고 부모-자식 관계를 만들어보세요.

---

## 실습 2: 앵커 시스템 마스터하기

### 목표
앵커의 동작 원리를 직접 확인하고, 다양한 Anchor Preset을 실험합니다.

### 이론 확인
`Week1_Theory.md`의 **"2. Anchors (앵커)의 개념"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: 기본 Image 생성

1. Canvas 하위에 `Image` 생성 → 이름: `TestAnchor`
2. Inspector에서 RectTransform 확인
3. 현재 Anchor Min/Max 값 확인 (기본값: (0.5, 0.5))

#### Step 2: Anchor Preset 실험

**실험 1: Top-Left**

1. RectTransform의 Anchor Preset 버튼 클릭 (Inspector 상단)
2. **Top-Left** 선택
3. **Alt + Shift** 키를 누른 채로 클릭 (앵커와 피벗 모두 설정)
4. Inspector에서 다음 값 확인:
   - Anchor Min: (0, 1)
   - Anchor Max: (0, 1)
   - Pivot: (0, 1)
5. Game 뷰에서 해상도를 변경해보세요
   - **관찰**: Image가 항상 왼쪽 상단에 위치

**실험 2: Middle-Center**

1. `TestAnchor` 선택
2. Anchor Preset에서 **Middle-Center** 선택
3. **Alt + Shift** 클릭
4. Inspector에서 다음 값 확인:
   - Anchor Min: (0.5, 0.5)
   - Anchor Max: (0.5, 0.5)
   - Pivot: (0.5, 0.5)
5. 해상도 변경
   - **관찰**: Image가 항상 중앙에 위치

**실험 3: Stretch-Stretch**

1. `TestAnchor` 선택
2. Anchor Preset에서 **Stretch-Stretch** 선택
3. **Alt + Shift** 클릭
4. Inspector에서 다음 값 확인:
   - Anchor Min: (0, 0)
   - Anchor Max: (1, 1)
   - Pivot: (0.5, 0.5)
5. 해상도 변경
   - **관찰**: Image가 화면 전체를 덮음

#### Step 3: Min/Max 앵커 수동 조정

1. `TestAnchor` 선택
2. RectTransform의 Anchor Preset 버튼 클릭
3. **Custom** 선택 (맨 아래)
4. Inspector에서 Anchor Min/Max를 직접 수정:
   - Anchor Min: (0.25, 0.25)
   - Anchor Max: (0.75, 0.75)
5. **관찰**: Image가 부모의 25%~75% 영역을 차지
6. 해상도 변경
   - **관찰**: 비율이 유지되며 크기가 변함

**핵심 이해**:
- Anchor Min = Anchor Max: 고정 크기
- Anchor Min ≠ Anchor Max: 부모 크기에 비례하는 크기

#### Step 4: 해상도 변경 시 UI 위치 변화 관찰

1. `TestAnchor`를 Top-Left로 설정
2. Anchored Position을 (100, -100)으로 설정
3. Game 뷰에서 다음 해상도로 변경하며 관찰:
   - 1920x1080
   - 3840x2160
   - 1024x768
   - 1334x750

**예상 결과**: 
- 모든 해상도에서 왼쪽 상단에서 (100, -100) 위치에 Image가 있음
- 하지만 Canvas Scaler에 의해 실제 픽셀 위치는 다를 수 있음

#### Step 5: 앵커와 Anchored Position의 관계 확인

1. `TestAnchor`를 Middle-Center로 설정
2. Anchored Position을 (0, 0)으로 설정
   - **관찰**: 정중앙에 위치
3. Anchored Position을 (200, 100)으로 설정
   - **관찰**: 중앙에서 오른쪽으로 200px, 위로 100px 이동

**핵심 공식**:
```
실제 위치 = 앵커 위치 + Anchored Position
```

### 검증

다음 질문에 답할 수 있어야 합니다:

1. Anchor Min (0, 1)과 Anchor Max (0, 1)의 의미는?
   - **답**: 부모의 왼쪽 상단 모서리에 고정

2. Anchor Min (0, 0)과 Anchor Max (1, 1)의 의미는?
   - **답**: 부모의 전체 영역을 차지 (Stretch)

3. 해상도를 변경했을 때 앵커가 (0.5, 0.5)인 UI는 어떻게 동작할까?
   - **답**: 항상 중앙에 위치 (상대적 위치 유지)

4. Anchored Position과 앵커의 관계는?
   - **답**: Anchored Position은 앵커 위치에서의 오프셋

### 문제 해결

**문제**: Anchor Preset을 클릭해도 변경이 안 돼요!
- **해결**: Alt+Shift를 누른 채로 클릭해야 앵커와 피벗이 함께 설정됩니다.

**문제**: 해상도를 변경했는데 UI가 이상한 위치로 이동해요!
- **해결**: Canvas에 Canvas Scaler 컴포넌트가 있는지 확인하세요. 다음 실습에서 설정합니다.

### 심화 학습

- Anchor Min과 Max를 다르게 설정하여 부모 크기의 일정 비율을 차지하는 UI를 만들어보세요.
- 여러 개의 UI 요소를 다른 Anchor Preset으로 설정하고 해상도를 변경하며 비교해보세요.

---

## 실습 3: Pivot 이해하기

### 목표
Pivot이 회전과 스케일에 미치는 영향을 직접 확인합니다.

### 이론 확인
`Week1_Theory.md`의 **"3. Pivot (피벗)의 역할"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: 테스트용 Image 3개 생성

1. Canvas 하위에 `Image` 3개 생성:
   - `Pivot_TopLeft`
   - `Pivot_Center`
   - `Pivot_BottomRight`

2. 각 Image의 크기를 200x200으로 설정

#### Step 2: 각 Image에 다른 Pivot 설정

**Image 1: Pivot_TopLeft**
1. Anchor Preset: Top-Left, Alt+Shift 클릭
2. Inspector에서 Pivot 값 확인: (0, 1)
3. Anchored Position: (200, -200) 설정

**Image 2: Pivot_Center**
1. Anchor Preset: Middle-Center, Alt+Shift 클릭
2. Inspector에서 Pivot 값 확인: (0.5, 0.5)
3. Anchored Position: (0, 0) 설정

**Image 3: Pivot_BottomRight**
1. Anchor Preset: Bottom-Right, Alt+Shift 클릭
2. Inspector에서 Pivot 값 확인: (1, 0)
3. Anchored Position: (-200, 200) 설정

#### Step 3: 회전 애니메이션 적용

**방법 1: Inspector에서 직접 회전**

1. 각 Image의 RectTransform에서 Rotation Z 값을 변경
2. Pivot 값에 따라 회전 중심이 다른지 관찰

**방법 2: 간단한 스크립트로 회전**

1. `Assets/_MyProjects/Scripts/` 폴더에 새 C# 스크립트 생성: `PivotRotator.cs`
2. 다음 코드 작성:

```csharp
using UnityEngine;

public class PivotRotator : MonoBehaviour
{
    [SerializeField] private float rotationSpeed = 30f;
    private RectTransform rectTransform;

    void Start()
    {
        rectTransform = GetComponent<RectTransform>();
    }

    void Update()
    {
        rectTransform.Rotate(0, 0, rotationSpeed * Time.deltaTime);
    }
}
```

3. 각 Image에 `PivotRotator` 컴포넌트 추가
4. Play 모드로 전환
5. **관찰**: 각 Image가 자신의 Pivot을 중심으로 회전

**예상 결과**:
- `Pivot_TopLeft`: 왼쪽 상단 모서리를 중심으로 회전
- `Pivot_Center`: 중앙을 중심으로 회전
- `Pivot_BottomRight`: 오른쪽 하단 모서리를 중심으로 회전

#### Step 4: 스케일 애니메이션 적용

1. `PivotRotator.cs`를 수정하여 스케일도 추가:

```csharp
using UnityEngine;

public class PivotRotator : MonoBehaviour
{
    [SerializeField] private float rotationSpeed = 30f;
    [SerializeField] private float scaleSpeed = 0.5f;
    [SerializeField] private float scaleRange = 0.3f;
    
    private RectTransform rectTransform;
    private float baseScale = 1f;

    void Start()
    {
        rectTransform = GetComponent<RectTransform>();
        baseScale = rectTransform.localScale.x;
    }

    void Update()
    {
        // 회전
        rectTransform.Rotate(0, 0, rotationSpeed * Time.deltaTime);
        
        // 스케일 (펄스 효과)
        float scale = baseScale + Mathf.Sin(Time.time * scaleSpeed) * scaleRange;
        rectTransform.localScale = Vector3.one * scale;
    }
}
```

2. Play 모드로 전환
3. **관찰**: 각 Image가 자신의 Pivot을 중심으로 확대/축소

#### Step 5: 차이점 관찰 및 기록

다음 사항을 관찰하고 기록하세요:

1. **회전 중심**:
   - Pivot (0, 1): 왼쪽 상단 모서리
   - Pivot (0.5, 0.5): 중앙
   - Pivot (1, 0): 오른쪽 하단 모서리

2. **스케일 중심**:
   - 동일하게 Pivot을 기준으로 확대/축소

3. **앵커와의 관계**:
   - Pivot은 자신의 기준점
   - Anchor는 부모의 기준점
   - 둘은 독립적으로 동작

### 검증

다음 질문에 답할 수 있어야 합니다:

1. Pivot (0, 0)인 UI를 회전하면 어디를 중심으로 회전할까?
   - **답**: 왼쪽 하단 모서리

2. Pivot과 Anchor의 차이는?
   - **답**: Pivot은 자신의 기준점, Anchor는 부모의 기준점

3. 왜 Anchor Preset을 사용할 때 Alt+Shift를 누르는가?
   - **답**: 앵커와 피벗을 함께 설정하여 일관성 유지

### 문제 해결

**문제**: 회전이 예상과 다른 위치에서 일어나요!
- **해결**: Pivot 값을 확인하세요. Pivot이 회전의 중심입니다.

**문제**: 스크립트를 추가했는데 회전이 안 돼요!
- **해결**: Play 모드로 전환했는지 확인하세요. 또한 RectTransform의 Rotation이 아닌 localRotation을 사용해야 할 수도 있습니다.

### 심화 학습

- Pivot 값을 (0.5, 0.5)로 고정하고 Anchor만 변경해보세요. 어떤 차이가 있을까요?
- 여러 개의 UI 요소를 부모-자식 관계로 만들고, 부모의 Pivot을 변경했을 때 자식들의 동작을 관찰해보세요.

---

## 실습 4: Canvas Scaler 설정 및 테스트

### 목표
다양한 해상도에서 UI 스케일이 어떻게 동작하는지 확인하고, 수학적 공식을 검증합니다.

### 이론 확인
`Week1_Theory.md`의 **"4. Canvas Scaler 심화"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: Canvas Scaler 추가

1. Hierarchy에서 `Canvas` 선택
2. Inspector에서 `Add Component` 클릭
3. `Canvas Scaler` 검색 후 추가
4. 기본 설정 확인:
   - UI Scale Mode: `Constant Pixel Size`

#### Step 2: Constant Pixel Size 테스트

1. Canvas 하위에 `Image` 생성 → 이름: `TestScaler`
2. 크기: 200x200
3. Anchor: Middle-Center
4. Game 뷰에서 해상도를 변경:
   - 1920x1080
   - 3840x2160 (2배)
   - 1024x768 (축소)

**관찰**:
- 해상도가 높아지면 UI가 작아 보임
- 해상도가 낮아지면 UI가 커 보임
- 실제 픽셀 크기는 동일하지만 화면에서의 비율이 변함

#### Step 3: Scale With Screen Size로 변경

1. Canvas Scaler의 UI Scale Mode를 `Scale With Screen Size`로 변경
2. 다음 설정:
   - Reference Resolution: X=1920, Y=1080
   - Screen Match Mode: `Match Width Or Height`
   - Match: `0.5`

3. Game 뷰에서 동일한 해상도로 테스트

**관찰**:
- 해상도가 변해도 UI의 상대적 크기가 유지됨
- 3840x2160에서는 2배로 확대되어 표시
- 1024x768에서는 축소되어 표시

#### Step 4: Match 값 변경하며 스케일 변화 관찰

**테스트 1: Match = 0.0 (가로 우선)**

1. Match 값을 `0.0`으로 설정
2. 해상도 변경:
   - 1920x1080 (기준)
   - 1024x768 (가로는 작아지지만 세로는 더 작음)
   - 3840x2160 (가로는 커지지만 세로는 더 큼)

**관찰**:
- 가로 비율만 고려하여 스케일 계산
- 세로가 짧아도 가로 기준으로 스케일

**테스트 2: Match = 1.0 (세로 우선)**

1. Match 값을 `1.0`으로 설정
2. 동일한 해상도로 테스트

**관찰**:
- 세로 비율만 고려하여 스케일 계산
- 가로가 좁아도 세로 기준으로 스케일

**테스트 3: Match = 0.5 (균형)**

1. Match 값을 `0.5`로 설정
2. 동일한 해상도로 테스트

**관찰**:
- 가로와 세로를 모두 고려하여 중간값으로 스케일
- 대부분의 해상도에서 균형잡힌 결과

#### Step 5: 수학적 공식 검증

**계산 예시**:

Reference Resolution: 1920x1080
Match: 0.5
실제 화면: 3840x2160

**공식**:
```
logWidth = log(3840 / 1920) = log(2) ≈ 0.693
logHeight = log(2160 / 1080) = log(2) ≈ 0.693
logMatch = log(0.5) ≈ -0.693

scale = 2^(logWidth × (1 - 0.5) + logHeight × 0.5)
     = 2^(0.693 × 0.5 + 0.693 × 0.5)
     = 2^0.693
     = 2
```

**Unity에서 검증**:

1. `TestScaler` Image의 크기를 200x200으로 설정
2. 3840x2160 해상도에서 Game 뷰 확인
3. 실제 표시되는 크기 측정 (대략적으로)
4. **예상**: 200 × 2 = 400px 크기로 표시

**간단한 검증 스크립트**:

```csharp
using UnityEngine;

public class ScaleChecker : MonoBehaviour
{
    private RectTransform rectTransform;
    private CanvasScaler canvasScaler;

    void Start()
    {
        rectTransform = GetComponent<RectTransform>();
        canvasScaler = GetComponentInParent<CanvasScaler>();
        
        CheckScale();
    }

    void CheckScale()
    {
        float scaleFactor = canvasScaler.scaleFactor;
        Vector2 size = rectTransform.sizeDelta;
        
        Debug.Log($"Canvas Scale Factor: {scaleFactor}");
        Debug.Log($"UI Size: {size}");
        Debug.Log($"Actual Screen Size: {size * scaleFactor}");
    }
}
```

이 스크립트를 `TestScaler`에 추가하고 Play 모드에서 Console을 확인하세요.

### 검증

다음 질문에 답할 수 있어야 합니다:

1. Constant Pixel Size와 Scale With Screen Size의 차이는?
   - **답**: Constant는 픽셀 크기 고정, Scale With Screen Size는 해상도에 따라 비례 스케일

2. Match 값 0.0, 0.5, 1.0의 차이는?
   - **답**: 0.0=가로 우선, 0.5=균형, 1.0=세로 우선

3. Reference Resolution의 의미는?
   - **답**: 스케일 계산의 기준 해상도

### 문제 해결

**문제**: Canvas Scaler를 추가했는데 UI가 너무 작아졌어요!
- **해결**: Reference Resolution을 현재 작업 해상도에 맞게 설정하세요.

**문제**: Match 값을 변경해도 차이가 안 보여요!
- **해결**: 극단적인 비율(예: 21:9, 4:3)로 테스트해보세요.

### 심화 학습

- 각 UI Scale Mode를 모두 테스트하고 사용 케이스를 정리해보세요.
- Match 값을 0.0에서 1.0까지 0.1씩 변경하며 UI 크기 변화를 관찰해보세요.

---

## 실습 5: 코너 고정 UI 만들기 (Absolute Defense Part 1)

### 목표
4개 코너에 UI를 배치하고, 모든 해상도에서 정상 위치를 유지하는지 확인합니다.

### 이론 확인
`Week1_Theory.md`의 **"5. 'Absolute Defense' 구현 전략"** 섹션의 "코너 고정 UI의 앵커 설정 방법"을 읽어주세요.

### 실습 단계

#### Step 1: Canvas 설정 확인

1. Canvas 선택
2. Canvas Scaler 확인:
   - UI Scale Mode: `Scale With Screen Size`
   - Reference Resolution: `1920 x 1080`
   - Match: `0.5`

#### Step 2: Top-Left UI 생성 및 설정

1. Canvas 하위에 `Image` 생성 → 이름: `HUD_TopLeft`
2. Inspector에서 RectTransform 설정:
   - Anchor Preset: **Top-Left**, **Alt+Shift** 클릭
   - Anchored Position: X=50, Y=-50
   - Width: 200, Height: 100
3. Image 컴포넌트에서 Color를 변경하여 시각적으로 구분 (예: 빨간색)

**설정값 확인**:
- Anchor Min: (0, 1)
- Anchor Max: (0, 1)
- Pivot: (0, 1)
- Anchored Position: (50, -50)

#### Step 3: 나머지 3개 코너 UI 생성

**Top-Right UI**

1. `Image` 생성 → 이름: `HUD_TopRight`
2. RectTransform 설정:
   - Anchor Preset: **Top-Right**, **Alt+Shift** 클릭
   - Anchored Position: X=-50, Y=-50
   - Width: 200, Height: 100
3. Color 변경 (예: 파란색)

**Bottom-Left UI**

1. `Image` 생성 → 이름: `HUD_BottomLeft`
2. RectTransform 설정:
   - Anchor Preset: **Bottom-Left**, **Alt+Shift** 클릭
   - Anchored Position: X=50, Y=50
   - Width: 200, Height: 100
3. Color 변경 (예: 초록색)

**Bottom-Right UI**

1. `Image` 생성 → 이름: `HUD_BottomRight`
2. RectTransform 설정:
   - Anchor Preset: **Bottom-Right**, **Alt+Shift** 클릭
   - Anchored Position: X=-50, Y=50
   - Width: 200, Height: 100
3. Color 변경 (예: 노란색)

#### Step 4: 다양한 해상도에서 테스트

Game 뷰에서 다음 해상도로 테스트:

**표준 해상도**:
- [ ] 1920x1080 (16:9) - Full HD
- [ ] 2560x1440 (16:9) - 2K
- [ ] 3840x2160 (16:9) - 4K

**모바일 해상도**:
- [ ] 1334x750 (16:9) - iPhone 8
- [ ] 1170x2532 (19.5:9) - iPhone 12/13
- [ ] 1080x2400 (20:9) - Galaxy S21

**극단적 비율**:
- [ ] 1024x768 (4:3) - 구형 모니터
- [ ] 2560x1080 (21:9) - 울트라와이드
- [ ] 1080x1920 (9:16) - 세로 모바일

**체크리스트**:
- [ ] 모든 해상도에서 4개 코너 UI가 화면 밖으로 나가지 않음
- [ ] 각 UI가 해당 코너에 정확히 위치
- [ ] UI 요소들이 서로 겹치지 않음

#### Step 5: 문제 발견 시 해결 방법 적용

**문제 1: UI가 화면 밖으로 나감**

**원인**: Anchored Position 값이 너무 큼

**해결**:
1. UI 요소 선택
2. Anchored Position 값을 줄임
3. 또는 UI 크기(Width/Height)를 줄임

**문제 2: 해상도에 따라 UI 위치가 달라짐**

**원인**: Canvas Scaler 설정 문제 또는 앵커 설정 오류

**해결**:
1. Canvas Scaler가 올바르게 설정되었는지 확인
2. Anchor Preset이 정확한지 확인 (Alt+Shift 클릭했는지)
3. Anchored Position이 앵커 기준 오프셋인지 확인

**문제 3: 극단적 비율에서 UI가 겹침**

**원인**: UI 크기가 고정되어 있어서 화면이 좁을 때 겹침

**해결**:
1. UI 크기를 줄이기
2. 또는 Content Size Fitter 사용 (다음 주차에서 학습)

### 검증

다음 조건을 모두 만족해야 합니다:

1. **위치 정확성**: 모든 해상도에서 각 UI가 해당 코너에 정확히 위치
2. **화면 밖 배제**: 어떤 해상도에서도 UI가 화면 밖으로 나가지 않음
3. **겹침 없음**: UI 요소들이 서로 겹치지 않음
4. **일관성**: 해상도를 변경해도 상대적 위치가 유지됨

### 문제 해결

**문제**: 특정 해상도에서만 UI가 이상해요!
- **해결**: 해당 해상도에서 Canvas Scaler의 Match 값을 조정해보세요. 또는 해당 해상도에 맞는 특별한 처리가 필요할 수 있습니다.

**문제**: UI가 너무 작아서 안 보여요!
- **해결**: Canvas Scaler의 Reference Resolution을 조정하거나, UI 크기를 키우세요.

### 심화 학습

- 각 코너 UI에 TextMeshPro를 추가하여 "Top-Left", "Top-Right" 등의 텍스트를 표시해보세요.
- UI에 아이콘이나 이미지를 추가하여 실제 HUD처럼 만들어보세요.
- 여러 해상도에서 스크린샷을 찍어 비교해보세요.

---

## 실습 6: 중앙 팝업 만들기 (Absolute Defense Part 2)

### 목표
항상 화면 중앙에 위치하는 팝업을 구현하고, 모든 해상도에서 정상 동작하는지 확인합니다.

### 이론 확인
`Week1_Theory.md`의 **"5. 'Absolute Defense' 구현 전략"** 섹션의 "중앙 팝업의 앵커 설정 방법"을 읽어주세요.

### 실습 단계

#### Step 1: 팝업 기본 구조 생성

1. Canvas 하위에 `Image` 생성 → 이름: `Popup_Center`
2. Inspector에서 RectTransform 설정:
   - Anchor Preset: **Middle-Center**, **Alt+Shift** 클릭
   - Anchored Position: X=0, Y=0 (중앙이므로 오프셋 불필요)
   - Width: 600, Height: 400
3. Image 컴포넌트:
   - Color: 반투명 검정 (예: R=0, G=0, B=0, A=200)

**설정값 확인**:
- Anchor Min: (0.5, 0.5)
- Anchor Max: (0.5, 0.5)
- Pivot: (0.5, 0.5)
- Anchored Position: (0, 0)

#### Step 2: 팝업 내부 요소 추가

**제목 텍스트**

1. `Popup_Center` 하위에 `TextMeshPro - Text (UI)` 생성 → 이름: `Title`
2. RectTransform 설정:
   - Anchor Preset: **Top-Stretch**, **Alt+Shift** 클릭
   - Anchored Position: X=0, Y=-20
   - Height: 60
3. TextMeshPro 설정:
   - Text: "팝업 제목"
   - Font Size: 32
   - Alignment: Center

**본문 텍스트**

1. `Popup_Center` 하위에 `TextMeshPro - Text (UI)` 생성 → 이름: `Body`
2. RectTransform 설정:
   - Anchor Preset: **Stretch-Stretch**, **Alt+Shift** 클릭
   - Anchored Position: X=0, Y=0
   - Left/Right/Top/Bottom: 모두 20 (여백)
3. TextMeshPro 설정:
   - Text: "이 팝업은 모든 해상도에서 중앙에 위치합니다."
   - Font Size: 24
   - Alignment: Center, Middle

**확인 버튼**

1. `Popup_Center` 하위에 `Button` 생성 → 이름: `ConfirmButton`
2. RectTransform 설정:
   - Anchor Preset: **Bottom-Stretch**, **Alt+Shift** 클릭
   - Anchored Position: X=0, Y=20
   - Height: 60
3. Button의 TextMeshPro 설정:
   - Text: "확인"

#### Step 3: 다양한 해상도에서 중앙 위치 확인

실습 5와 동일한 해상도 목록으로 테스트:

**표준 해상도**:
- [ ] 1920x1080
- [ ] 2560x1440
- [ ] 3840x2160

**모바일 해상도**:
- [ ] 1334x750
- [ ] 1170x2532
- [ ] 1080x2400

**극단적 비율**:
- [ ] 1024x768
- [ ] 2560x1080
- [ ] 1080x1920

**체크리스트**:
- [ ] 모든 해상도에서 팝업이 화면 정중앙에 위치
- [ ] 팝업이 화면 밖으로 나가지 않음
- [ ] 팝업 내부 요소들이 올바르게 배치됨
- [ ] 극단적 비율에서도 정상 동작

#### Step 4: 극단적 비율(21:9, 4:3) 테스트

**21:9 울트라와이드 테스트**

1. Game 뷰 해상도: 2560x1080
2. **관찰**:
   - 팝업이 중앙에 위치하는가?
   - 팝업이 화면을 벗어나지 않는가?
   - 가로가 길어서 팝업이 작아 보이는가?

**4:3 비율 테스트**

1. Game 뷰 해상도: 1024x768
2. **관찰**:
   - 팝업이 중앙에 위치하는가?
   - 세로가 짧아서 팝업이 잘리는가?
   - 필요하다면 팝업 크기를 조정

**세로 모바일 테스트**

1. Game 뷰 해상도: 1080x1920
2. **관찰**:
   - 팝업이 중앙에 위치하는가?
   - 세로가 길어서 팝업이 작아 보이는가?

### 검증

다음 조건을 모두 만족해야 합니다:

1. **중앙 위치**: 모든 해상도에서 팝업이 화면 정중앙에 위치
2. **화면 내 유지**: 어떤 해상도에서도 팝업이 화면 밖으로 나가지 않음
3. **내부 요소 정렬**: 팝업 내부 요소들이 올바르게 배치됨
4. **극단적 비율 대응**: 21:9, 4:3 등에서도 정상 동작

### 문제 해결

**문제**: 극단적 비율에서 팝업이 화면 밖으로 나가요!
- **해결**: 팝업 크기를 줄이거나, Content Size Fitter를 사용하여 화면 크기에 맞게 조정

**문제**: 팝업이 중앙에 있는데 약간 어긋나 보여요!
- **해결**: Anchored Position이 (0, 0)인지 확인. 또한 Pivot이 (0.5, 0.5)인지 확인

**문제**: 세로 모바일에서 팝업이 너무 작아요!
- **해결**: Canvas Scaler의 Match 값을 조정하거나, 팝업 크기를 키우세요.

### 심화 학습

- 팝업에 닫기 버튼을 추가하고, 버튼 클릭 시 팝업이 사라지는 기능을 구현해보세요.
- 여러 개의 팝업을 만들고, 각각 다른 크기로 설정해보세요.
- 팝업에 배경 블러 효과를 추가해보세요 (Image에 Material 사용).

---

## 실습 7: Safe Area 대응 (선택)

### 목표
노치와 홈 인디케이터 영역을 고려하여 Safe Area를 처리합니다.

### 이론 확인
`Week1_Theory.md`의 **"6. 실전 베스트 프랙티스"** 섹션의 "Safe Area 대응"을 읽어주세요.

### 실습 단계

#### Step 1: Safe Area Fitter 스크립트 작성

1. `Assets/_MyProjects/Scripts/` 폴더에 새 C# 스크립트 생성: `SafeAreaFitter.cs`
2. 다음 코드 작성:

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

    void Start()
    {
        ApplySafeArea(Screen.safeArea);
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

#### Step 2: Safe Area 패널 생성

1. Canvas 하위에 `Image` 생성 → 이름: `SafeAreaPanel`
2. `SafeAreaFitter` 컴포넌트 추가
3. RectTransform 설정:
   - Anchor Preset: **Stretch-Stretch**
   - 모든 값 0으로 설정
4. Image 컴포넌트:
   - Color: 반투명 (예: R=100, G=100, B=100, A=100) - Safe Area 시각화용

#### Step 3: Safe Area 내부에 UI 배치

1. `SafeAreaPanel` 하위에 기존 코너 UI들을 이동:
   - `HUD_TopLeft`, `HUD_TopRight`, `HUD_BottomLeft`, `HUD_BottomRight`
2. 또는 `SafeAreaPanel` 하위에 새 UI 요소 생성

**중요**: Safe Area 밖의 UI는 노치나 홈 인디케이터에 가려질 수 있습니다.

#### Step 4: 실제 기기 또는 시뮬레이터에서 테스트

**Unity에서 시뮬레이션**:

1. Game 뷰에서 해상도를 iPhone X 이상으로 설정 (예: 1170x2532)
2. `Screen.safeArea`가 제대로 동작하는지 확인
3. Console에서 `Screen.safeArea` 값을 출력하여 확인

**실제 기기 테스트** (가능한 경우):

1. Build Settings에서 플랫폼 선택 (iOS/Android)
2. 빌드 후 실제 기기에서 테스트
3. 노치나 홈 인디케이터 영역에 UI가 가려지지 않는지 확인

**테스트 스크립트**:

```csharp
using UnityEngine;

public class SafeAreaDebugger : MonoBehaviour
{
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            Rect safeArea = Screen.safeArea;
            Debug.Log($"Safe Area: {safeArea}");
            Debug.Log($"Screen Size: {Screen.width}x{Screen.height}");
            Debug.Log($"Safe Area Position: {safeArea.position}");
            Debug.Log($"Safe Area Size: {safeArea.size}");
        }
    }
}
```

이 스크립트를 임의의 GameObject에 추가하고, Play 모드에서 Space 키를 눌러 Safe Area 정보를 확인하세요.

### 검증

다음 조건을 만족해야 합니다:

1. **Safe Area 인식**: `Screen.safeArea`가 올바르게 동작
2. **UI 배치**: Safe Area 내부에 UI가 배치됨
3. **가려짐 방지**: 노치나 홈 인디케이터 영역에 UI가 가려지지 않음

### 문제 해결

**문제**: `Screen.safeArea`가 전체 화면을 반환해요!
- **해결**: 실제 기기나 시뮬레이터에서만 Safe Area가 제대로 동작합니다. 에디터에서는 전체 화면이 반환될 수 있습니다.

**문제**: Safe Area가 적용되지 않아요!
- **해결**: Canvas의 Render Mode를 확인하세요. Screen Space - Overlay나 Screen Space - Camera에서만 동작합니다.

### 심화 학습

- Safe Area를 시각화하는 디버그 UI를 만들어보세요.
- Safe Area 밖의 영역을 어둡게 처리하여 Safe Area를 명확히 구분해보세요.

---

## 최종 검증: Absolute Defense 완성

### 목표
모든 해상도에서 깨지지 않는 HUD와 중앙 팝업이 완성되었는지 최종 검증합니다.

### 체크리스트

#### 기본 기능

- [ ] **4개 코너 UI가 모든 해상도에서 정상 위치**
  - Top-Left: 왼쪽 상단에 고정
  - Top-Right: 오른쪽 상단에 고정
  - Bottom-Left: 왼쪽 하단에 고정
  - Bottom-Right: 오른쪽 하단에 고정

- [ ] **중앙 팝업이 모든 해상도에서 정상 위치**
  - 화면 정중앙에 위치
  - 내부 요소들이 올바르게 배치됨

- [ ] **UI 요소들이 서로 겹치지 않음**
  - 모든 해상도에서 겹침 없음

#### 해상도 테스트

- [ ] **표준 해상도 (16:9)**
  - 1920x1080 ✓
  - 2560x1440 ✓
  - 3840x2160 ✓

- [ ] **모바일 해상도**
  - 1334x750 (iPhone 8) ✓
  - 1170x2532 (iPhone 12/13) ✓
  - 1080x2400 (Galaxy S21) ✓

- [ ] **극단적 비율**
  - 1024x768 (4:3) ✓
  - 2560x1080 (21:9) ✓
  - 1080x1920 (9:16) ✓

#### 추가 기능 (선택)

- [ ] **Safe Area 적용**
  - 노치 영역에 UI가 가려지지 않음
  - 홈 인디케이터 영역에 UI가 가려지지 않음

### 최종 테스트 시나리오

1. **해상도 변경 테스트**
   - Game 뷰에서 해상도를 빠르게 변경하며 UI가 깨지지 않는지 확인

2. **극단적 비율 테스트**
   - 21:9, 4:3 등 극단적 비율에서 UI가 정상 동작하는지 확인

3. **Play 모드 테스트**
   - Play 모드로 전환하여 런타임에서도 정상 동작하는지 확인

4. **스크린샷 비교**
   - 여러 해상도에서 스크린샷을 찍어 비교
   - UI 위치와 크기가 일관된지 확인

### 문제 발견 시

문제를 발견했다면 다음을 확인하세요:

1. **Canvas Scaler 설정**
   - UI Scale Mode가 올바른가?
   - Reference Resolution이 적절한가?
   - Match 값이 적절한가?

2. **앵커 설정**
   - Anchor Preset이 올바른가?
   - Alt+Shift를 눌러서 설정했는가?
   - Anchored Position이 적절한가?

3. **UI 크기**
   - UI 크기가 화면에 비해 너무 큰가?
   - 극단적 비율에서 크기를 조정해야 하는가?

### 완성 기준

다음 조건을 모두 만족하면 **"Absolute Defense" 완성**입니다:

✅ 모든 해상도에서 코너 UI가 정상 위치  
✅ 모든 해상도에서 중앙 팝업이 정상 위치  
✅ UI 요소들이 겹치지 않음  
✅ 극단적 비율에서도 정상 동작  
✅ (선택) Safe Area 적용

---

## 학습 정리 및 다음 단계

### 실습 중 발견한 문제와 해결 방법 기록

다음 항목을 정리해보세요:

1. **어려웠던 부분**
   - 어떤 개념이 가장 이해하기 어려웠나요?
   - 어떤 실습에서 막혔나요?

2. **해결한 문제**
   - 어떤 문제를 발견했나요?
   - 어떻게 해결했나요?

3. **새롭게 알게 된 것**
   - 이론과 실제의 차이점은?
   - 예상하지 못한 동작은?

### 이론과 실제의 차이점 정리

다음 사항을 비교해보세요:

1. **이론에서 배운 것**
   - 앵커는 0~1 정규화 좌표
   - Canvas Scaler는 해상도에 따라 스케일 계산

2. **실제로 경험한 것**
   - Unity 에디터에서의 실제 동작
   - 예상과 다른 부분

3. **차이점과 이유**
   - 왜 차이가 발생했는가?
   - 어떻게 이해를 정리할 것인가?

### 다음 주차(Quest 2) 준비

**Quest 2: "Auto-Resizing Chat"**

다음 주차에서는 다음을 학습합니다:
- ContentSizeFitter 컴포넌트
- LayoutGroup 컴포넌트
- 텍스트 길이에 따른 자동 크기 조정

**준비사항**:
- [ ] RectTransform과 앵커 시스템을 완전히 이해했는가?
- [ ] 다양한 해상도에서 UI를 테스트하는 방법을 익혔는가?
- [ ] Anchor Preset을 자유롭게 사용할 수 있는가?

### 추가 학습 자료

더 깊이 학습하고 싶다면:

1. **Unity 공식 문서**
   - RectTransform: https://docs.unity3d.com/Manual/class-RectTransform.html
   - Canvas Scaler: https://docs.unity3d.com/Manual/script-CanvasScaler.html

2. **실전 프로젝트**
   - 실제 게임의 UI를 분석해보세요
   - 다양한 해상도에서 어떻게 대응하는지 관찰

3. **커뮤니티 자료**
   - Unity 포럼의 UI 관련 질문과 답변
   - 유튜브 튜토리얼 영상

---

## 마무리

축하합니다! 1주차 "Absolute Defense" 퀘스트를 완료했습니다.

**학습한 핵심 개념**:
- ✅ RectTransform의 본질과 Transform과의 차이
- ✅ 앵커 시스템의 동작 원리
- ✅ Pivot의 역할과 앵커와의 관계
- ✅ Canvas Scaler의 다양한 모드와 설정
- ✅ 코너 고정 UI와 중앙 팝업 구현
- ✅ Safe Area 대응 (선택)

**다음 단계**:
- 이론 자료(`Week1_Theory.md`)를 다시 읽어보며 개념을 정리하세요
- 실습 중 발견한 문제와 해결 방법을 기록하세요
- 다음 주차 Quest 2를 준비하세요

**기억할 것**:
> "앵커를 이해하면 UI의 절반을 이해한 것이다."

화이팅! 🚀

