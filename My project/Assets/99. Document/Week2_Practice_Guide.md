# 2주차: "Dynamic Info Box" - 단계별 실습 가이드

## 목표
이론 자료(`Week2_Theory.md`)를 바탕으로 Unity에서 직접 동적 UI를 구현합니다.
텍스트 길이에 따라 자동으로 크기가 조절되는 툴팁과 NPC 대화 말풍선, 동적 목록을 만들어봅니다.

**싱글 인디게임 필수 UI**:
- 아이템 툴팁 (마우스 호버)
- NPC 대화 말풍선 (꼬리 포함)
- 퀘스트 로그 / 인벤토리 목록

**학습 방식**: 이론 확인 → Unity 실습 → 검증 → 정리

---

## 학습 전 준비사항

### 1. Unity 씬 설정

1. Unity 에디터를 엽니다.
2. `Assets/Scenes/` 폴더에 새 씬 생성 → 이름: `Week2_DynamicUI`
3. 씬을 저장합니다.

### 2. 필요한 컴포넌트 확인

이 실습에서 사용할 Unity 컴포넌트:
- **Canvas**: UI의 루트 컨테이너
- **ContentSizeFitter**: 콘텐츠에 맞춰 크기 조절
- **Horizontal/Vertical Layout Group**: 자동 정렬
- **LayoutElement**: 크기 제한
- **ScrollRect**: 스크롤 기능
- **TextMeshPro**: 텍스트 표시

### 3. 폴더 구조 준비

```
Assets/
└── _MyProjects/
    ├── Scripts/
    │   └── Week2/
    ├── Prefabs/
    │   └── Week2/
    └── Sprites/
        └── UI/  (툴팁, 말풍선 이미지)
```

### 4. 이론 자료 읽기 체크리스트

실습 전에 `Week2_Theory.md`의 다음 섹션을 읽어주세요:

- [ ] 2. ContentSizeFitter 완벽 가이드
- [ ] 3. Layout Group 시스템
- [ ] 4. LayoutElement 컴포넌트
- [ ] 5. TextMeshPro와 동적 크기

**팁**: 전체를 다 읽지 않아도 됩니다. 각 실습 전에 해당 섹션만 읽어도 충분합니다.

---

## 실습 1: ContentSizeFitter 기초 이해

### 목표
ContentSizeFitter의 동작 원리를 직접 확인하고, 텍스트에 맞춰 크기가 조절되는 박스를 만듭니다.

### 이론 확인
`Week2_Theory.md`의 **"2. ContentSizeFitter 완벽 가이드"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: 기본 Canvas 설정

1. Hierarchy 창에서 우클릭 → `UI` → `Canvas` 생성
2. Canvas 선택 → Inspector에서 `Canvas Scaler` 확인:
   - UI Scale Mode: `Scale With Screen Size`
   - Reference Resolution: `1920 x 1080`
   - Match: `1` (Height 기준 - PC 최적화)

#### Step 2: 테스트용 박스 생성

1. Canvas 하위에 `UI` → `Image` 생성 → 이름: `TestBox`
2. RectTransform 설정:
   - Anchor Preset: **Middle-Center**, **Alt+Shift** 클릭
   - Width: 200, Height: 100
3. Image 컴포넌트:
   - Color: 파란색 (구분용)

#### Step 3: TextMeshPro 추가

1. `TestBox` 하위에 `UI` → `Text - TextMeshPro` 생성 → 이름: `Text`
2. RectTransform 설정:
   - Anchor Preset: **Stretch-Stretch**, **Alt+Shift** 클릭
   - Left, Right, Top, Bottom: 모두 `10` (패딩)
3. TextMeshPro 설정:
   - Text: "안녕하세요"
   - Font Size: 24
   - Alignment: Center, Middle
   - Wrapping: Enabled

#### Step 4: ContentSizeFitter 없이 동작 확인

1. `Text`의 내용을 변경해보세요:
   - "안녕"
   - "안녕하세요, 반갑습니다!"
   - "안녕하세요, 오늘 날씨가 정말 좋네요. 산책하러 가실래요?"

**관찰**:
- 텍스트가 길어져도 `TestBox`의 크기는 변하지 않음
- 텍스트가 잘리거나 박스 밖으로 나감

#### Step 5: ContentSizeFitter 추가

1. `TestBox` 선택
2. `Add Component` → `ContentSizeFitter` 추가
3. 설정:
   - Horizontal Fit: `Preferred Size`
   - Vertical Fit: `Preferred Size`

**관찰**:
- `TestBox`의 크기가 자동으로 변함!
- 하지만... 예상과 다르게 동작할 수 있음

#### Step 6: 문제 해결

**문제**: 박스가 너무 작거나 크게 변함

**원인**: ContentSizeFitter는 자신의 Preferred Size를 사용하는데, Image 컴포넌트의 Preferred Size가 설정되지 않음

**해결**:
1. `TestBox`에서 Image 컴포넌트의 `Set Native Size` 버튼을 클릭하지 마세요
2. ContentSizeFitter는 **자식의 Preferred Size**를 기준으로 동작
3. 따라서 Text의 Preferred Size가 박스 크기가 됨

**구조 변경**:
```
TestBox (ContentSizeFitter)
└── Text (TextMeshPro)
    └── 여백은 RectTransform의 offset으로 처리
```

#### Step 7: 올바른 구조로 재설정

**올바른 여백 처리 방법**:
- ❌ **잘못된 방법**: TextMeshPro의 Margin 사용
- ✅ **올바른 방법**: RectTransform의 Left/Right/Top/Bottom offset 사용

1. `Text`의 RectTransform:
   - Anchor Preset: **Stretch-Stretch**
   - Left: `10`, Right: `10`, Top: `10`, Bottom: `10` (여백 설정)

2. `Text`의 TextMeshPro 설정:
   - Text: "안녕하세요"
   - Font Size: 24
   - Alignment: Center, Middle
   - Wrapping: Enabled
   - **Margins**: 사용하지 않음 (0으로 유지)

3. `TestBox`의 ContentSizeFitter:
   - Horizontal Fit: `Preferred Size`
   - Vertical Fit: `Preferred Size`

4. 다시 텍스트를 변경해보세요

**관찰**:
- 텍스트 길이에 맞춰 박스 크기가 자동 조절됨!
- 여백은 RectTransform offset으로 일정하게 유지됨

**실무 팁**:
> TextMeshPro의 Margin은 텍스트 렌더링 영역을 제한할 뿐, Layout 시스템과는 무관합니다.
> UI 여백은 항상 **RectTransform의 offset** 또는 **부모의 Layout Group Padding**으로 처리하세요!

### 검증

다음 질문에 답할 수 있어야 합니다:

1. ContentSizeFitter의 역할은?
   - **답**: 콘텐츠(자식 또는 자신)의 Preferred Size에 맞춰 자신의 크기를 조절

2. Preferred Size는 어디서 오는가?
   - **답**: TextMeshPro는 텍스트 내용에 따라 자동 계산, 다른 요소는 LayoutElement로 지정

3. Horizontal/Vertical Fit의 Unconstrained는 언제 사용?
   - **답**: 해당 축의 크기를 고정하고 싶을 때

### 문제 해결

**문제**: ContentSizeFitter를 추가하면 에러가 나요!
- **해결**: Anchor가 Stretch로 되어있으면 충돌 발생. Anchor를 점(예: Middle-Center)으로 변경하세요.

**문제**: 박스 크기가 0이 되어버려요!
- **해결**: 자식 요소에 Preferred Size가 있는지 확인. TextMeshPro에 텍스트가 있어야 함.

### 심화 학습

- Horizontal Fit만 Preferred Size로 설정하고 Vertical Fit은 Unconstrained로 해보세요
- 여러 줄 텍스트에서 어떻게 동작하는지 관찰해보세요

---

## 실습 2: Layout Group 이해하기

### 목표
Vertical/Horizontal Layout Group의 동작을 확인하고, 자식 요소들이 자동 정렬되는 것을 체험합니다.

### 이론 확인
`Week2_Theory.md`의 **"3. Layout Group 시스템"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: Vertical Layout Group 테스트

1. Canvas 하위에 `Image` 생성 → 이름: `VerticalContainer`
2. RectTransform 설정:
   - Anchor Preset: **Middle-Center**
   - Width: 300, Height: 400
3. Image Color: 회색 (배경)

4. `Add Component` → `Vertical Layout Group` 추가
5. 설정:
   - Padding: Left=10, Right=10, Top=10, Bottom=10
   - Spacing: 10
   - Child Alignment: Upper Center
   - Control Child Size: Width ✓, Height ✗
   - Child Force Expand: Width ✗, Height ✗

#### Step 2: 자식 요소 추가

1. `VerticalContainer` 하위에 `Image` 3개 생성:
   - `Child1` (빨간색)
   - `Child2` (파란색)
   - `Child3` (초록색)

2. 각 자식의 RectTransform:
   - Width: 100 (무시됨 - Layout Group이 제어)
   - Height: 50

**관찰**:
- 자식들이 자동으로 세로로 정렬됨
- Width가 부모에 맞춰 늘어남 (Control Child Size: Width ✓)
- Height는 각자 유지됨 (Control Child Size: Height ✗)

#### Step 3: 옵션 변경 실험

**실험 1: Child Force Expand Width 활성화**
1. Vertical Layout Group의 Child Force Expand: Width ✓
2. **관찰**: 변화 없음 (이미 Control Child Size로 늘어나 있음)

**실험 2: Child Force Expand Height 활성화**
1. Child Force Expand: Height ✓
2. **관찰**: 자식들이 세로로 늘어나 빈 공간을 채움

**실험 3: Child Alignment 변경**
1. Child Alignment: Middle Center
2. **관찰**: 자식들이 중앙 정렬됨

**실험 4: Control Child Size Height 활성화**
1. Control Child Size: Height ✓
2. 자식들에 LayoutElement 추가, Preferred Height: 80
3. **관찰**: 자식 높이가 Preferred Height로 조절됨

#### Step 4: Horizontal Layout Group 테스트

1. Canvas 하위에 새 `Image` 생성 → 이름: `HorizontalContainer`
2. RectTransform: Width=400, Height=100
3. `Add Component` → `Horizontal Layout Group` 추가
4. 자식 3개 추가 (각각 다른 색)
5. 설정을 변경하며 동작 관찰

**핵심 차이**: 정렬 방향만 다름 (가로 vs 세로)

#### Step 5: Layout Group + ContentSizeFitter 조합

1. `VerticalContainer`에 `ContentSizeFitter` 추가
2. 설정:
   - Horizontal Fit: Unconstrained
   - Vertical Fit: Preferred Size

3. **중요**: Anchor를 점 앵커로 변경 (예: Middle-Center)

**관찰**:
- 컨테이너 높이가 자식들의 높이 합계 + 패딩 + 스페이싱에 맞춰짐
- 자식을 추가/제거하면 컨테이너 크기가 자동 조절됨

### 검증

다음 질문에 답할 수 있어야 합니다:

1. Control Child Size와 Child Force Expand의 차이는?
   - **답**: Control은 자식 크기를 Layout이 제어할지, Expand는 남은 공간을 채울지

2. Layout Group + ContentSizeFitter 조합의 효과는?
   - **답**: 자식들을 정렬하면서 부모 크기도 자식에 맞춰 조절

3. Padding과 Spacing의 차이는?
   - **답**: Padding은 테두리 여백, Spacing은 자식 간 간격

### 문제 해결

**문제**: 자식 크기가 예상과 다르게 변해요!
- **해결**: Control Child Size 옵션 확인. 체크되어 있으면 Layout Group이 크기를 제어함.

**문제**: 자식들이 겹쳐요!
- **해결**: Spacing 값 확인. 또는 자식들의 높이가 너무 큰지 확인.

---

## 실습 3: LayoutElement 활용

### 목표
LayoutElement를 사용하여 최소/최대 크기를 제한하고, 유연한 크기 조절을 구현합니다.

### 이론 확인
`Week2_Theory.md`의 **"4. LayoutElement 컴포넌트"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: 기본 설정

1. 실습 2의 `VerticalContainer`를 복사 → 이름: `LayoutElementTest`
2. 자식 3개가 있는 상태 확인

#### Step 2: Min Size 테스트

1. `Child1`에 `Add Component` → `LayoutElement` 추가
2. 설정:
   - Min Height: 100

3. `VerticalContainer`의 높이를 줄여보세요 (200으로)

**관찰**:
- `Child1`은 최소 100 높이를 유지
- 다른 자식들은 줄어들거나 겹침

#### Step 3: Preferred Size 테스트

1. `Child2`에 `LayoutElement` 추가
2. 설정:
   - Preferred Height: 80

3. Vertical Layout Group의 Control Child Size: Height ✓

**관찰**:
- `Child2`의 높이가 80이 됨
- Layout Group이 Preferred Size를 참조

#### Step 4: Flexible Size 테스트

1. `Child3`에 `LayoutElement` 추가
2. 설정:
   - Flexible Height: 1

3. `VerticalContainer`의 높이를 늘려보세요 (500으로)

**관찰**:
- `Child3`이 남은 공간을 모두 차지
- `Child1`, `Child2`는 고정 크기 유지

#### Step 5: Flexible 비율 테스트

1. 모든 자식의 LayoutElement:
   - `Child1`: Flexible Height = 1
   - `Child2`: Flexible Height = 2
   - `Child3`: Flexible Height = 1

**관찰**:
- 남은 공간이 1:2:1 비율로 분배됨

#### Step 6: 툴팁용 최대 너비 제한

1. 새 `Image` 생성 → 이름: `BubbleTest`
2. 구조:
   ```
   BubbleTest (ContentSizeFitter + LayoutElement)
   └── Text (TextMeshPro)
   ```

3. `BubbleTest` 설정:
   - ContentSizeFitter:
     - Horizontal Fit: Preferred Size
     - Vertical Fit: Preferred Size
   - LayoutElement:
     - Preferred Width: 300 (최대 너비 제한)

4. TextMeshPro 설정:
   - Wrapping: Enabled
   - 긴 텍스트 입력

**관찰**:
- 텍스트가 300px 너비를 넘어가면 자동 줄바꿈
- 높이는 텍스트 줄 수에 맞춰 자동 조절

### 검증

다음 질문에 답할 수 있어야 합니다:

1. Min Size, Preferred Size, Flexible Size의 우선순위는?
   - **답**: Min → Preferred → Flexible 순서로 적용

2. Flexible Width가 0이면 어떻게 동작?
   - **답**: 남은 공간을 가져가지 않음, Preferred Size 유지

3. 툴팁의 최대 너비를 제한하려면?
   - **답**: LayoutElement의 Preferred Width 사용

---

## 실습 4: 아이템 툴팁 만들기

### 목표
텍스트 길이에 따라 크기가 자동 조절되는 아이템 툴팁을 만듭니다.

### 이론 확인
`Week2_Theory.md`의 **"6. 툴팁 UI 구현"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: 툴팁 기본 구조 생성

1. Canvas 하위에 `Image` 생성 → 이름: `ItemTooltip`
2. RectTransform:
   - Anchor Preset: **Top-Left** (나중에 마우스로 이동할 것)
   - Pos X: 200, Pos Y: -200
3. Image 설정:
   - Color: 반투명 검정 (R=0, G=0, B=0, A=200)
   - Image Type: Sliced (9-slice 이미지가 있다면)

#### Step 2: Vertical Layout Group 추가

1. `ItemTooltip`에 `Add Component` → `Vertical Layout Group`
2. 설정:
   - Padding: Left=15, Right=15, Top=10, Bottom=10
   - Spacing: 5
   - Child Alignment: Upper Left
   - Control Child Size: Width ✓, Height ✗
   - Child Force Expand: Width ✗, Height ✗

#### Step 3: 제목 텍스트 추가

1. `ItemTooltip` 하위에 `Text - TextMeshPro` 생성 → 이름: `TitleText`
2. TextMeshPro 설정:
   - Text: "전설의 검 +10"
   - Font Size: 24
   - Color: 노란색 (Yellow)
   - Font Style: Bold
   - Alignment: Left, Top
   - Wrapping: Enabled

#### Step 4: 구분선 추가 (선택)

1. `ItemTooltip` 하위에 `Image` 생성 → 이름: `Divider`
2. RectTransform:
   - Height: 2
3. Image:
   - Color: 회색
4. `LayoutElement` 추가:
   - Preferred Height: 2

#### Step 5: 설명 텍스트 추가

1. `ItemTooltip` 하위에 `Text - TextMeshPro` 생성 → 이름: `DescriptionText`
2. TextMeshPro 설정:
   - Text: "공격력 +500\n크리티컬 확률 +25%\n\n전설의 용을 쓰러뜨린 검"
   - Font Size: 18
   - Color: 흰색
   - Alignment: Left, Top
   - Wrapping: Enabled

#### Step 6: ContentSizeFitter 추가

1. `ItemTooltip` 선택
2. `Add Component` → `ContentSizeFitter`
3. 설정:
   - Horizontal Fit: `Preferred Size`
   - Vertical Fit: `Preferred Size`

**테스트**:
- `DescriptionText`의 텍스트를 변경해보세요
- 툴팁 크기가 자동으로 변하는지 확인

#### Step 7: 최대 너비 제한

1. `ItemTooltip`에 `Add Component` → `LayoutElement`
2. 설정:
   - Preferred Width: `300`
   - Min Width: `150`
   - 나머지는 체크 해제

**테스트**:
- 긴 설명 입력하여 300px에서 줄바꿈 확인
- 짧은 텍스트 입력하여 최소 150px 유지 확인

**최종 구조**:
```
ItemTooltip (Vertical Layout Group + ContentSizeFitter + LayoutElement)
├── TitleText (TextMeshPro, 노란색, Bold)
├── Divider (Image, Height: 2px)
└── DescriptionText (TextMeshPro, 흰색)
```

### 검증

다음을 확인하세요:

- [ ] 짧은 텍스트: 툴팁이 텍스트에 맞게 작아짐 (최소 150px)
- [ ] 긴 텍스트: 300px에서 줄바꿈
- [ ] 제목과 설명이 세로로 정렬됨
- [ ] 구분선이 제목과 설명 사이에 표시됨

### 문제 해결

**문제**: 툴팁이 너무 작아요!
- **해결**: Vertical Layout Group의 Padding 확인, Min Width 확인

**문제**: 텍스트가 줄바꿈이 안 돼요!
- **해결**: TextMeshPro의 Wrapping: Enabled 확인, LayoutElement의 Preferred Width 확인

**문제**: 제목과 설명이 붙어있어요!
- **해결**: Vertical Layout Group의 Spacing 값 확인 (5 이상 권장)

---

## 실습 5: 마우스 따라다니는 툴팁

### 목표
실습 4에서 만든 툴팁을 마우스 위치에 따라 이동시키고, 화면 밖으로 나가지 않게 처리합니다.

### 이론 확인
`Week2_Theory.md`의 **"6. 툴팁 UI 구현"** 섹션의 "마우스 따라다니는 툴팁"을 읽어주세요.

### 실습 단계

#### Step 1: TooltipController 스크립트 작성

1. `Assets/_MyProjects/Scripts/Week2/` 폴더에 새 C# 스크립트 생성
2. 이름: `TooltipController.cs`

```csharp
using UnityEngine;
using TMPro;

public class TooltipController : MonoBehaviour
{
    [SerializeField] private RectTransform tooltipRect;
    [SerializeField] private TMP_Text titleText;
    [SerializeField] private TMP_Text descriptionText;
    [SerializeField] private Canvas canvas;
    [SerializeField] private Vector2 offset = new Vector2(10, -10);
    
    void Awake()
    {
        // 초기에는 숨김
        Hide();
    }
    
    void Update()
    {
        if (tooltipRect.gameObject.activeSelf)
        {
            UpdatePosition();
        }
    }
    
    void UpdatePosition()
    {
        // 마우스 위치 가져오기
        Vector2 mousePos = Input.mousePosition / canvas.scaleFactor;
        mousePos += offset;
        
        // 툴팁 위치 설정
        tooltipRect.anchoredPosition = mousePos;
        
        // 화면 밖 방지
        ClampToScreen();
    }
    
    void ClampToScreen()
    {
        Vector2 screenSize = new Vector2(Screen.width, Screen.height) / canvas.scaleFactor;
        Vector2 tooltipSize = tooltipRect.sizeDelta;
        Vector2 pos = tooltipRect.anchoredPosition;
        
        // Pivot을 조정하여 화면 밖 방지
        Vector2 pivot = new Vector2(0, 1); // 기본: 좌상단
        
        // 우측 화면 밖으로 나가면 우측 기준으로 전환
        if (pos.x + tooltipSize.x > screenSize.x)
        {
            pivot.x = 1;
        }
        
        // 하단 화면 밖으로 나가면 하단 기준으로 전환
        if (pos.y - tooltipSize.y < 0)
        {
            pivot.y = 0;
        }
        
        tooltipRect.pivot = pivot;
    }
    
    public void Show(string title, string description)
    {
        titleText.text = title;
        descriptionText.text = description;
        tooltipRect.gameObject.SetActive(true);
    }
    
    public void Hide()
    {
        tooltipRect.gameObject.SetActive(false);
    }
}
```

#### Step 2: TooltipController 설정

1. Canvas 하위에 빈 GameObject 생성 → 이름: `TooltipController`
2. `TooltipController` 스크립트 컴포넌트 추가
3. Inspector에서 필드 연결:
   - Tooltip Rect: `ItemTooltip` (실습 4에서 만든 것)
   - Title Text: `TitleText`
   - Description Text: `DescriptionText`
   - Canvas: `Canvas`

#### Step 3: 테스트용 아이템 슬롯 생성

1. Canvas 하위에 `Image` 생성 → 이름: `TestItemSlot`
2. RectTransform:
   - Anchor: Middle-Center
   - Width: 80, Height: 80
3. Image:
   - Color: 회색 (아이템 슬롯 배경)

#### Step 4: ItemSlot 스크립트 작성

1. 새 C# 스크립트 생성: `ItemSlot.cs`

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class ItemSlot : MonoBehaviour, IPointerEnterHandler, IPointerExitHandler
{
    [SerializeField] private string itemName = "체력 물약";
    [SerializeField, TextArea(3, 5)] private string itemDescription = "HP를 50 회복합니다.\n\n희귀 등급";
    
    private TooltipController tooltip;
    
    void Start()
    {
        tooltip = FindObjectOfType<TooltipController>();
    }
    
    public void OnPointerEnter(PointerEventData eventData)
    {
        if (tooltip != null)
        {
            tooltip.Show(itemName, itemDescription);
        }
    }
    
    public void OnPointerExit(PointerEventData eventData)
    {
        if (tooltip != null)
        {
            tooltip.Hide();
        }
    }
}
```

2. `TestItemSlot`에 `ItemSlot` 컴포넌트 추가
3. Inspector에서 아이템 정보 입력

#### Step 5: 테스트

1. Play 모드로 전환
2. `TestItemSlot`에 마우스 올려보기
3. **확인 사항**:
   - [ ] 마우스 호버 시 툴팁 표시
   - [ ] 툴팁이 마우스를 따라다님
   - [ ] 마우스를 벗어나면 툴팁 사라짐

#### Step 6: 화면 경계 테스트

1. `TestItemSlot`을 화면 우측 상단으로 이동
2. Play 모드에서 마우스 올리기
3. **관찰**: 툴팁이 화면 밖으로 나가지 않고 Pivot이 전환됨

4. 화면 4개 모서리에서 모두 테스트:
   - 좌상단: Pivot (0, 1) 유지
   - 우상단: Pivot (1, 1)로 전환
   - 좌하단: Pivot (0, 0)으로 전환
   - 우하단: Pivot (1, 0)으로 전환

### 검증

다음을 확인하세요:

- [ ] 툴팁이 마우스를 따라다님
- [ ] 화면 모서리에서 Pivot 자동 전환
- [ ] 툴팁이 화면 밖으로 나가지 않음
- [ ] 텍스트 길이에 따라 툴팁 크기 조절

### 문제 해결

**문제**: 툴팁이 마우스를 따라다니지 않아요!
- **해결**: Canvas Scaler의 Scale Factor 확인, 마우스 위치를 Scale Factor로 나누기

**문제**: 툴팁이 화면 밖으로 나가요!
- **해결**: ClampToScreen 메서드 확인, Pivot 전환 로직 확인

**문제**: 툴팁이 깜빡거려요!
- **해결**: Update 대신 LateUpdate 사용 고려

---

## 실습 6: NPC 대화 말풍선 기본

### 목표
말풍선 몸통과 꼬리를 분리하여, 몸통은 텍스트에 맞춰 늘어나고 꼬리는 찌그러지지 않는 말풍선을 만듭니다.

### 이론 확인
`Week2_Theory.md`의 **"7. NPC 대화 말풍선"** 및 **"9-1. 말풍선 꼬리 처리법"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: 말풍선 몸통(Body) 생성

1. Canvas 하위에 `Image` 생성 → 이름: `DialogueBubble`
2. RectTransform:
   - Anchor Preset: **Middle-Center**
   - Pos X: 0, Pos Y: 0
3. Image 설정:
   - Color: 흰색 (말풍선 배경)
   - Image Type: Sliced (9-slice 이미지가 있다면)

#### Step 2: 말풍선 내부 구조 생성

1. `DialogueBubble` 하위에 `Text - TextMeshPro` 생성 → 이름: `DialogueText`
2. RectTransform:
   - Anchor Preset: **Stretch-Stretch**
   - Left: `15`, Right: `15`, Top: `10`, Bottom: `10` (여백 설정)
3. TextMeshPro 설정:
   - Text: "안녕하세요!"
   - Font Size: 20
   - Color: 검정
   - Alignment: Left, Top
   - **Wrapping**: Enabled
   - Overflow: Overflow

**실무 팁**: 텍스트와 테두리 사이의 여백은 TextMeshPro의 Margin이 아닌 **RectTransform의 offset**으로 처리합니다!

#### Step 3: 말풍선 몸통에 ContentSizeFitter 추가

1. `DialogueBubble` 선택
2. `Add Component` → `ContentSizeFitter`
3. 설정:
   - Horizontal Fit: `Preferred Size`
   - Vertical Fit: `Preferred Size`

4. `Add Component` → `LayoutElement`
5. 설정:
   - Preferred Width: `250`
   - Min Width: `80`

**테스트**:
- 텍스트를 변경하며 몸통 크기가 자동 조절되는지 확인

#### Step 4: 말풍선 꼬리(Tail) 추가 🔥

1. `DialogueBubble` 하위에 `Image` 생성 → 이름: `BubbleTail`
2. RectTransform:
   - Anchor Preset: **Bottom-Center** (ALT+SHIFT 클릭)
   - Anchor Min: (0.5, 0)
   - Anchor Max: (0.5, 0)
   - Pivot: (0.5, 1) ← **중요!** 위쪽이 몸통에 붙음
   - Anchored Position: (0, 0)
   - Width: 20, Height: 10
3. Image 설정:
   - Color: 흰색 (몸통과 같은 색)
   - Image Type: Simple (찌그러지지 않게)

**핵심 포인트**:
- 꼬리는 **앵커로 고정** (LayoutGroup 사용 안 함)
- 몸통이 늘어나도 꼬리는 고정 크기 유지

#### Step 5: 최종 구조 확인

```
DialogueBubble (ContentSizeFitter + LayoutElement)
├── DialogueText (TextMeshPro)
└── BubbleTail (앵커: Bottom-Center, 고정 크기)
```

**테스트**:
1. `DialogueText`의 내용을 변경
   - "안녕!"
   - "안녕하세요, 여기는 위험한 곳입니다."
   - "안녕하세요, 여기는 위험한 곳입니다. 조심하세요. 괴물들이 많이 나타납니다."

2. **관찰**:
   - 몸통은 텍스트에 맞춰 늘어남
   - 꼬리는 항상 몸통 하단 중앙에 고정
   - 꼬리 크기는 변하지 않음

#### Step 6: 임시 NPC 배치 및 테스트

1. Canvas 하위에 `Image` 생성 → 이름: `TestNPC`
2. RectTransform: 화면 좌측에 배치
3. Image: 아이콘 또는 색상 설정

4. `DialogueBubble`을 `TestNPC` 위쪽에 배치
5. 말풍선 위치 조정

**최종 모습**:
```
  ┌──────────────┐
  │ 안녕하세요! │  ← 말풍선 몸통
  └──────▼───────┘  ← 꼬리 (고정 크기)
       👤            ← NPC
```

### 검증

다음을 확인하세요:

- [ ] 텍스트 길이에 따라 몸통 크기 자동 조절
- [ ] 꼬리가 항상 몸통 하단 중앙에 위치
- [ ] 꼬리 크기는 변하지 않음 (찌그러지지 않음)
- [ ] 최대 너비 250px에서 줄바꿈

### 문제 해결

**문제**: 꼬리가 몸통과 함께 늘어나요!
- **해결**: 꼬리에 ContentSizeFitter나 LayoutGroup이 없는지 확인. 앵커로만 위치 고정해야 함.

**문제**: 꼬리가 몸통 중앙에 안 와요!
- **해결**: BubbleTail의 Anchor가 Bottom-Center (0.5, 0)인지 확인

**문제**: 몸통이 늘어나면 꼬리가 따라오지 않아요!
- **해결**: 꼬리의 Pivot이 (0.5, 1)인지 확인 (위쪽이 몸통에 붙음)

---

## 실습 7: 말풍선 꼬리 방향 전환

### 목표
좌/우 NPC 위치에 따라 말풍선 꼬리 방향을 동적으로 변경하는 스크립트를 작성합니다.

### 이론 확인
`Week2_Theory.md`의 **"7. NPC 대화 말풍선"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: DialogueBubble 스크립트 작성

1. `Assets/_MyProjects/Scripts/Week2/` 폴더에 새 C# 스크립트 생성
2. 이름: `DialogueBubble.cs`

```csharp
using UnityEngine;
using TMPro;

public class DialogueBubble : MonoBehaviour
{
    [SerializeField] private TMP_Text dialogueText;
    [SerializeField] private RectTransform bubbleTail;
    
    /// <summary>
    /// 말풍선 방향 설정 (NPC 위치에 따라)
    /// </summary>
    /// <param name="isLeft">true면 좌측 NPC, false면 우측 NPC</param>
    public void SetDirection(bool isLeft)
    {
        if (isLeft)
        {
            // 좌측 NPC: 꼬리가 좌하단 (20% 위치)
            bubbleTail.anchorMin = new Vector2(0.2f, 0);
            bubbleTail.anchorMax = new Vector2(0.2f, 0);
            bubbleTail.pivot = new Vector2(0.5f, 1);
            bubbleTail.localScale = Vector3.one;
        }
        else
        {
            // 우측 NPC: 꼬리가 우하단 (80% 위치)
            bubbleTail.anchorMin = new Vector2(0.8f, 0);
            bubbleTail.anchorMax = new Vector2(0.8f, 0);
            bubbleTail.pivot = new Vector2(0.5f, 1);
            
            // 좌우 반전
            bubbleTail.localScale = new Vector3(-1, 1, 1);
        }
    }
    
    public void SetText(string text)
    {
        if (dialogueText != null)
        {
            dialogueText.text = text;
        }
    }
    
    public void Show()
    {
        gameObject.SetActive(true);
    }
    
    public void Hide()
    {
        gameObject.SetActive(false);
    }
}
```

3. 실습 6에서 만든 `DialogueBubble`에 이 스크립트 추가
4. Inspector에서 필드 연결:
   - Dialogue Text: `DialogueText`
   - Bubble Tail: `BubbleTail`

#### Step 2: 좌/우 NPC 테스트 슬롯 생성

1. Canvas 하위에 `Image` 2개 생성:
   - `LeftNPC` (화면 좌측)
   - `RightNPC` (화면 우측)

2. 각 NPC 위에 말풍선 배치:
   - `DialogueBubble`을 복제하여 2개 생성
   - `LeftNPC_Bubble`, `RightNPC_Bubble`로 이름 변경

#### Step 3: 방향 테스트 스크립트

테스트 버튼 스크립트:

```csharp
using UnityEngine;

public class BubbleDirectionTest : MonoBehaviour
{
    [SerializeField] private DialogueBubble leftBubble;
    [SerializeField] private DialogueBubble rightBubble;
    
    void Start()
    {
        // 좌측 말풍선은 좌측 방향
        leftBubble.SetDirection(true);
        leftBubble.SetText("안녕하세요, 여행자님!");
        
        // 우측 말풍선은 우측 방향
        rightBubble.SetDirection(false);
        rightBubble.SetText("반갑습니다!");
    }
    
    public void TestLeftDialogue()
    {
        leftBubble.SetDirection(true);
        leftBubble.SetText("좌측 NPC의 대사입니다. 여기는 위험한 곳입니다.");
    }
    
    public void TestRightDialogue()
    {
        rightBubble.SetDirection(false);
        rightBubble.SetText("우측 NPC의 대사입니다. 안전한 길은 이쪽입니다.");
    }
}
```

#### Step 4: 시각적 확인

**좌측 NPC 말풍선**:
```
  ┌──────────────┐
  │ 안녕하세요! │
  └▼─────────────┘  ← 꼬리가 좌하단
     👤 (좌측 NPC)
```

**우측 NPC 말풍선**:
```
   ┌──────────────┐
   │ 반갑습니다! │
   └─────────────▼┘  ← 꼬리가 우하단
       👤 (우측 NPC)
```

#### Step 5: Prefab 저장

1. `DialogueBubble`을 `Assets/_MyProjects/Prefabs/Week2/` 폴더로 드래그
2. Prefab 이름: `DialogueBubble.prefab`
3. 씬에서 테스트용 말풍선은 유지 (테스트용)

### 검증

다음을 확인하세요:

- [ ] `SetDirection(true)` 호출 시 꼬리가 좌하단에 위치
- [ ] `SetDirection(false)` 호출 시 꼬리가 우하단에 위치
- [ ] 꼬리가 좌우 반전됨
- [ ] 텍스트 변경 시 말풍선 크기 자동 조절

### 문제 해결

**문제**: 꼬리 방향이 안 바뀌어요!
- **해결**: BubbleTail 참조가 올바른지 확인

**문제**: 꼬리가 중앙에 있어요!
- **해결**: SetDirection 메서드의 anchorMin/Max 값 확인 (0.2 또는 0.8)

**문제**: 좌우 반전이 이상해요!
- **해결**: localScale의 X 값이 -1인지 확인

### 심화 학습

- 말풍선에 타이핑 효과를 추가해보세요
- 여러 개의 NPC가 동시에 말할 때를 대비하여 Canvas Sort Order를 조정해보세요

---

## 실습 8: 동적 목록 UI (퀘스트 로그/인벤토리)

### 목표
ScrollView와 ContentSizeFitter를 활용하여 동적으로 아이템이 추가/제거되는 목록 UI를 만듭니다.

**실무 활용**: 퀘스트 로그, 인벤토리, 아이템 드롭 로그 등

### 이론 확인
`Week2_Theory.md`의 **"9-2. ScrollView 튀는 현상 해결"** 및 **"9-3. 동적 목록 스크롤 방향"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: ScrollView 생성

1. Canvas 하위에 `UI` → `Scroll View` 생성 → 이름: `DynamicListScrollView`
2. RectTransform:
   - Anchor Preset: **Stretch-Stretch**
   - Left: 20, Right: 20, Top: 100, Bottom: 100

3. Scroll Rect 컴포넌트 설정:
   - Horizontal: ✗
   - Vertical: ✓
   - Movement Type: Clamped
   - Scrollbar Visibility: Auto Hide

4. 불필요한 요소 삭제:
   - `Scrollbar Horizontal` 삭제

#### Step 2: Content 설정

1. `DynamicListScrollView` → `Viewport` → `Content` 선택
2. RectTransform:
   - Anchor Preset: **Top-Stretch**
   - Pivot: (0.5, 1) ← **중요!** 위쪽 기준
   - Left: 0, Right: 0, Top: 0

3. `Add Component` → `Vertical Layout Group`
4. 설정:
   - Padding: 10, 10, 10, 10
   - Spacing: 5
   - Child Alignment: Upper Center
   - Control Child Size: Width ✓, Height ✗
   - Child Force Expand: Width ✗, Height ✗

5. `Add Component` → `ContentSizeFitter`
6. 설정:
   - Horizontal Fit: Unconstrained
   - Vertical Fit: Preferred Size

#### Step 3: 목록 아이템 Prefab 만들기

1. `Content` 하위에 `Image` 생성 → 이름: `ListItem`
2. RectTransform: Height: 60
3. `Add Component` → `Horizontal Layout Group`
   - Padding: 10
   - Spacing: 10
   - Child Alignment: Middle Left

4. 자식 요소 추가:
   - `Icon` (Image, 40x40)
   - `TextContainer` (Vertical Layout Group)
     - `ItemName` (TextMeshPro, Bold, Size: 18)
     - `ItemDescription` (TextMeshPro, Size: 14, Gray)

5. `ListItem`을 Prefab으로 저장

#### Step 4: DynamicListManager 스크립트 작성

```csharp
using UnityEngine;
using System.Collections;

public class DynamicListManager : MonoBehaviour
{
    [SerializeField] private Transform content;
    [SerializeField] private GameObject listItemPrefab;
    
    public void AddItem(string itemName, string description)
    {
        GameObject item = Instantiate(listItemPrefab, content);
        
        // 아이템 정보 설정
        TMPro.TMP_Text[] texts = item.GetComponentsInChildren<TMPro.TMP_Text>();
        if (texts.Length >= 2)
        {
            texts[0].text = itemName;
            texts[1].text = description;
        }
    }
    
    public void ClearList()
    {
        foreach (Transform child in content)
        {
            Destroy(child.gameObject);
        }
    }
    
    // 테스트용
    public void AddTestItem()
    {
        string[] names = { "체력 물약", "마나 물약", "전설의 검", "마법 방패" };
        string[] descs = { "HP +50", "MP +30", "공격력 +500", "방어력 +200" };
        
        int index = Random.Range(0, names.Length);
        AddItem(names[index], descs[index]);
    }
}
```

#### Step 5: 테스트 버튼 추가

1. Canvas 하위에 `Button - TextMeshPro` 생성 → 이름: `AddItemButton`
2. RectTransform: 상단에 배치
3. Button OnClick에 `DynamicListManager.AddTestItem` 연결

#### Step 6: 테스트

1. Play 모드로 전환
2. "아이템 추가" 버튼 여러 번 클릭
3. **확인 사항**:
   - [ ] 아이템이 목록에 추가됨
   - [ ] Content 높이가 자동 조절됨
   - [ ] 스크롤이 정상 동작
   - [ ] Viewport 밖의 아이템은 안 보임 (Mask)

### 검증

다음을 확인하세요:

- [ ] 아이템 동적 추가 가능
- [ ] Content 크기 자동 조절
- [ ] 스크롤 정상 동작
- [ ] 많은 아이템 추가 시 성능 문제 없음

### 문제 해결

**문제**: 스크롤이 안 돼요!
- **해결**: Content의 ContentSizeFitter 확인, Viewport 크기 확인

**문제**: 아이템들이 화면 밖으로 나와요!
- **해결**: Viewport의 Mask 컴포넌트 확인

### 심화: 상단에 아이템 추가 시 스크롤 위치 유지

실무에서 자주 겪는 문제를 해결해봅시다:

```csharp
// DynamicListManager.cs에 추가
public void AddItemToTop(string itemName, string description)
{
    // 1. 추가 전 Content 높이 저장
    float previousHeight = content.GetComponent<RectTransform>().rect.height;
    
    // 2. 아이템을 맨 위에 추가
    GameObject item = Instantiate(listItemPrefab, content);
    item.transform.SetAsFirstSibling();
    
    // 텍스트 설정
    TMPro.TMP_Text[] texts = item.GetComponentsInChildren<TMPro.TMP_Text>();
    if (texts.Length >= 2)
    {
        texts[0].text = itemName;
        texts[1].text = description;
    }
    
    // 3. 스크롤 위치 보정
    StartCoroutine(AdjustScrollPosition(previousHeight));
}

private IEnumerator AdjustScrollPosition(float previousHeight)
{
    // Layout 갱신 대기
    yield return null;
    
    // 추가된 높이 계산
    RectTransform contentRect = content.GetComponent<RectTransform>();
    float newHeight = contentRect.rect.height;
    float addedHeight = newHeight - previousHeight;
    
    // 스크롤 위치 보정
    Vector2 pos = contentRect.anchoredPosition;
    pos.y += addedHeight;
    contentRect.anchoredPosition = pos;
}
```

---

## 최종 검증: Dynamic Info Box 완성

### 체크리스트

#### 기능 요구사항

- [ ] **아이템 툴팁**
  - 텍스트 길이에 따른 크기 자동 조절
  - 마우스 호버 시 표시
  - 화면 밖 방지 (Pivot 전환)
  - 최대/최소 너비 제한

- [ ] **NPC 대화 말풍선**
  - 텍스트 길이에 따른 크기 자동 조절
  - 꼬리가 찌그러지지 않음
  - 좌/우 NPC에 따른 꼬리 방향 전환

- [ ] **동적 목록 UI**
  - 아이템 추가/제거 기능
  - 스크롤 정상 동작
  - Content 크기 자동 조절

#### 해상도 테스트

- [ ] 1920x1080 (16:9)
- [ ] 2560x1440 (2K)
- [ ] 3840x2160 (4K)

### 완성 기준

다음 조건을 모두 만족하면 **"Dynamic Info Box" 완성**입니다:

✅ 툴팁이 텍스트에 맞춰 크기 조절
✅ 툴팁이 마우스를 따라다니며 화면 밖 방지
✅ 말풍선 꼬리가 찌그러지지 않음
✅ 좌/우 NPC에 따른 꼬리 방향 전환
✅ 동적 목록 추가/제거 정상 동작

---

## 학습 정리 및 다음 단계

### 배운 핵심 개념 정리

1. **ContentSizeFitter**
   - 콘텐츠에 맞춰 크기 자동 조절
   - Preferred Size 모드가 가장 유용

2. **Layout Group**
   - 자식 요소 자동 정렬
   - Control Child Size vs Child Force Expand

3. **LayoutElement**
   - 크기 선호도 명시적 지정
   - 최소/최대 크기 제한에 유용

4. **조합 패턴**
   - ContentSizeFitter + Layout Group = 동적 크기 컨테이너
   - ContentSizeFitter + TextMeshPro = 텍스트에 맞는 박스

5. **실무 팁**
   - 말풍선 꼬리는 이미지 분리로 찌그러짐 방지
   - 툴팁은 Pivot 전환으로 화면 밖 방지
   - 동적 목록 상단 추가 시 스크롤 위치 보정

### 다음 주차(Quest 3) 준비

**Quest 3: "Complex Card"**

다음 주차에서는 다음을 학습합니다:
- 중첩 레이아웃 (Nested Layout)
- 가로 확장 + 비율 유지
- 복잡한 UI 카드 구현

**준비사항**:
- [ ] ContentSizeFitter 완전히 이해했는가?
- [ ] Layout Group 옵션들을 구분할 수 있는가?
- [ ] LayoutElement로 크기 제어가 가능한가?

---

## 보너스: Prefab 정리

### 최종 Prefab 목록

```
Assets/_MyProjects/Prefabs/Week2/
├── ItemTooltip.prefab
├── DialogueBubble.prefab
└── ListItem.prefab
```

### DialogueBubble Prefab 구조 (참고용)

```
DialogueBubble
├── Components
│   ├── ContentSizeFitter
│   │   ├── Horizontal Fit: Preferred Size
│   │   └── Vertical Fit: Preferred Size
│   └── LayoutElement
│       ├── Preferred Width: 250
│       └── Min Width: 80
│
├── DialogueText (TextMeshPro)
│   ├── RectTransform: Left=15, Right=15, Top=10, Bottom=10 (여백)
│   ├── Wrapping: Enabled
│   └── Overflow: Overflow
│
└── BubbleTail (Image)
    ├── Anchor: Bottom-Center
    ├── Pivot: (0.5, 1)
    └── Size: 20x10 (고정)
```

---

## 마무리

축하합니다! 2주차 "Dynamic Info Box" 퀘스트를 완료했습니다.

**학습한 핵심 개념**:
- ✅ ContentSizeFitter의 동작 원리
- ✅ Horizontal/Vertical Layout Group
- ✅ LayoutElement로 크기 제어
- ✅ 툴팁 UI (마우스 따라다니기 + 화면 밖 방지)
- ✅ NPC 대화 말풍선 (꼬리 분리 + 방향 전환)
- ✅ 동적 목록 UI (ScrollView + ContentSizeFitter)

**싱글 인디게임에서 바로 써먹기**:
- 아이템 툴팁을 게임에 통합하세요
- NPC 대화 시스템에 말풍선을 적용하세요
- 퀘스트 로그나 인벤토리에 동적 목록을 활용하세요

**다음 단계**:
- 이론 자료(`Week2_Theory.md`)를 다시 읽어보며 개념을 정리하세요
- 만든 UI를 자신의 게임 프로젝트에 적용해보세요
- 다음 주차 Quest 3를 준비하세요

**기억할 것**:
> "Layout 시스템은 조합이다. ContentSizeFitter + LayoutGroup + LayoutElement를 적절히 조합하면 어떤 동적 UI도 만들 수 있다."

**실무 팁**:
> "ForceRebuildLayoutImmediate는 성능 킬러! 코루틴으로 `yield return null` 하나만 넣어도 Layout은 자동으로 갱신됩니다."

화이팅! 🚀
