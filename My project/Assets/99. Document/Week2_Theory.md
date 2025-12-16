# 2주차: "Dynamic Info Box" - 이론 학습 자료

## 목표
텍스트 길이에 따라 자동으로 크기가 조절되는 동적 UI(툴팁, NPC 대화창, 동적 목록)를 구현하기 위한 핵심 이론을 학습합니다.
ContentSizeFitter, LayoutGroup, 그리고 동적 콘텐츠 처리 기법을 마스터합니다.

**싱글 인디게임에서 가장 많이 사용하는 동적 UI**:
- 아이템 툴팁 (마우스 호버 시)
- NPC 대화 말풍선
- 퀘스트 로그 / 인벤토리 목록

---

## 1. 동적 콘텐츠의 도전

### 정적 UI vs 동적 UI

#### 정적 UI (1주차에서 학습)
- **특징**: 크기가 미리 정해져 있음
- **예시**: 고정 크기 버튼, 코너에 고정된 HUD
- **처리**: Anchor와 Pivot만으로 충분

#### 동적 UI (2주차에서 학습)
- **특징**: 콘텐츠에 따라 크기가 변함
- **예시**: 아이템 툴팁, NPC 대화 말풍선, 퀘스트 로그
- **처리**: ContentSizeFitter와 LayoutGroup 필요

### 왜 동적 UI가 어려운가?

**문제 상황 (아이템 툴팁)**:
```
"체력 물약"                    →  작은 툴팁 필요
"전설의 검 +10\n공격력 +500\n크리티컬 확률 +25%"  →  큰 툴팁 필요
```

**문제 상황 (NPC 대화)**:
```
"안녕!"                →  작은 말풍선
"안녕하세요, 여기는 위험한 곳입니다. 조심하세요!"  →  큰 말풍선
```

**단순 접근의 한계**:
```csharp
// 나쁜 예: 텍스트 길이로 크기 계산
float width = text.Length * fontSize;
rectTransform.sizeDelta = new Vector2(width, height);
// 문제: 폰트마다 글자 폭이 다름, 줄바꿈 처리 불가
```

**해결책**: Unity의 Layout 시스템 활용
- ContentSizeFitter: 내용물에 맞춰 크기 자동 조절
- LayoutGroup: 자식 요소들을 자동 정렬
- TextMeshPro: 정확한 텍스트 크기 계산

---

## 2. ContentSizeFitter 완벽 가이드

### ContentSizeFitter란?

ContentSizeFitter는 **자신의 크기를 자식 콘텐츠(또는 자기 자신의 선호 크기)에 맞춰 자동 조절**하는 컴포넌트입니다.

**핵심 개념**:
- "내 안에 뭐가 있는지 보고, 그에 맞춰 크기를 조절해줘"
- 텍스트, 이미지, 자식 요소 등 내용물의 크기를 기준으로 동작

### Fit Mode 옵션

ContentSizeFitter에는 Horizontal Fit과 Vertical Fit 두 가지 축이 있습니다.

| Fit Mode | 동작 | 사용 케이스 |
|:---:|:---|:---|
| **Unconstrained** | 크기 조절 안 함 (기본값) | 해당 축 크기를 고정하고 싶을 때 |
| **Min Size** | 최소 크기로 조절 | 거의 사용 안 함 |
| **Preferred Size** | **선호 크기로 조절 (가장 많이 사용)** | 텍스트, 동적 콘텐츠 |

### Preferred Size의 의미

**Preferred Size**는 각 UI 요소가 "내가 이 크기가 되고 싶어"라고 말하는 값입니다.

**텍스트의 Preferred Size**:
- 텍스트 내용, 폰트 크기, 폰트 종류에 따라 자동 계산
- "안녕" → 약 80px 폭
- "안녕하세요, 반갑습니다" → 약 250px 폭

**이미지의 Preferred Size**:
- 이미지의 원본 크기 (Native Size)

**LayoutGroup의 Preferred Size**:
- 자식 요소들의 Preferred Size 합계 + Spacing + Padding

### ContentSizeFitter 동작 원리

```
┌─────────────────────────────────────┐
│        ContentSizeFitter            │
│                                     │
│   1. 자식/자신의 Preferred Size 계산 │
│              ↓                      │
│   2. Fit Mode에 따라 크기 결정       │
│              ↓                      │
│   3. RectTransform 크기 업데이트    │
│                                     │
└─────────────────────────────────────┘
```

**실제 동작**:
1. 텍스트 내용이 변경됨
2. TextMeshPro가 새로운 Preferred Size 계산
3. ContentSizeFitter가 이를 감지
4. RectTransform의 sizeDelta 업데이트

### 주의사항과 함정

#### 함정 1: 부모 Stretch 앵커와의 충돌

**문제 상황**:
```
ContentSizeFitter (Preferred Size)
    └── Anchor: Stretch-Stretch (0,0) ~ (1,1)
```

**결과**: 에러 발생 또는 예상치 못한 동작

**이유**: 
- Stretch 앵커: "부모 크기에 맞춰져라"
- ContentSizeFitter: "내용물에 맞춰져라"
- 두 명령이 충돌!

**해결책**:
```
ContentSizeFitter 사용 시 → 해당 축의 앵커를 점 앵커로 설정
(예: Middle-Center, Top-Left 등)
```

#### 함정 2: Layout 재계산 타이밍

**문제 상황**:
```csharp
text.text = "새로운 텍스트";
Debug.Log(rectTransform.sizeDelta); // 이전 크기가 출력됨!
```

**이유**: Layout 계산은 프레임 끝에 일괄 처리됨

**⚠️ 잘못된 해결책 (성능 킬러)**:
```csharp
// 나쁜 예: ForceRebuildLayoutImmediate 남용
text.text = "새로운 텍스트";
LayoutRebuilder.ForceRebuildLayoutImmediate(rectTransform);
// 문제: 연결된 모든 상위/하위 레이아웃을 강제 재계산
// 채팅 100개일 때 호출하면 프레임 드랍!
```

**✅ 올바른 해결책 (코루틴 활용)**:
```csharp
// 좋은 예: 코루틴으로 한 프레임 대기
text.text = "새로운 텍스트";
StartCoroutine(DoAfterLayout());

private IEnumerator DoAfterLayout()
{
    yield return null; // 한 프레임 대기 (Layout 자동 갱신)
    Debug.Log(rectTransform.sizeDelta); // 올바른 크기 출력
}
```

**실무 원칙**:
- 대부분의 경우, 레이아웃 갱신이 한 프레임 늦는 건 **눈에 보이지 않음**
- `ForceRebuildLayoutImmediate`는 **정말 즉시 필요한 경우**에만 사용
- 스크롤 위치 조정 등은 `yield return null` 후 처리가 정석

#### 함정 3: 최대 크기 제한 없음

**문제 상황**:
ContentSizeFitter는 내용물이 아무리 커도 그에 맞춰 크기가 커짐

**해결책**:
- LayoutElement 컴포넌트로 최대 크기 제한
- 또는 스크립트로 최대 크기 클램핑

```csharp
[RequireComponent(typeof(ContentSizeFitter))]
public class MaxSizeConstraint : MonoBehaviour
{
    [SerializeField] private float maxWidth = 400f;
    [SerializeField] private float maxHeight = 300f;
    
    private RectTransform rectTransform;
    
    void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
    }
    
    void LateUpdate()
    {
        Vector2 size = rectTransform.sizeDelta;
        size.x = Mathf.Min(size.x, maxWidth);
        size.y = Mathf.Min(size.y, maxHeight);
        rectTransform.sizeDelta = size;
    }
}
```

---

## 3. Layout Group 시스템

### Layout Group이란?

Layout Group은 **자식 요소들을 자동으로 정렬**하는 컴포넌트입니다.

**종류**:
1. **Horizontal Layout Group**: 가로로 정렬
2. **Vertical Layout Group**: 세로로 정렬
3. **Grid Layout Group**: 격자로 정렬

### Horizontal Layout Group

자식 요소들을 **왼쪽에서 오른쪽**으로 배치합니다.

```
┌─────────────────────────────────┐
│ [자식1] [자식2] [자식3] [자식4] │
└─────────────────────────────────┘
```

**주요 속성**:

| 속성 | 설명 | 예시 값 |
|:---|:---|:---:|
| **Padding** | 내부 여백 (Left, Right, Top, Bottom) | 10, 10, 5, 5 |
| **Spacing** | 자식 요소 간 간격 | 5 |
| **Child Alignment** | 자식 정렬 방향 | Middle Left |
| **Control Child Size** | 자식 크기 제어 여부 | Width ✓, Height ✓ |
| **Use Child Scale** | 자식 스케일 고려 여부 | 보통 false |
| **Child Force Expand** | 남은 공간 채우기 | 상황에 따라 |

### Vertical Layout Group

자식 요소들을 **위에서 아래**로 배치합니다.

```
┌─────────────┐
│   [자식1]   │
│   [자식2]   │
│   [자식3]   │
│   [자식4]   │
└─────────────┘
```

**주요 속성**: Horizontal Layout Group과 동일

### Grid Layout Group

자식 요소들을 **격자 형태**로 배치합니다.

```
┌─────────────────────┐
│ [1] [2] [3] [4]     │
│ [5] [6] [7] [8]     │
│ [9] [10][11][12]    │
└─────────────────────┘
```

**주요 속성**:

| 속성 | 설명 | 예시 값 |
|:---|:---|:---:|
| **Cell Size** | 각 셀의 크기 | (100, 100) |
| **Spacing** | 셀 간 간격 | (5, 5) |
| **Start Corner** | 시작 위치 | Upper Left |
| **Start Axis** | 진행 방향 | Horizontal |
| **Constraint** | 제한 조건 | Fixed Column Count |
| **Constraint Count** | 제한 수 | 4 |

### Child Force Expand vs Control Child Size

**가장 혼란스러운 옵션** 두 가지를 명확히 이해해봅시다.

#### Control Child Size

**역할**: "자식의 크기를 Layout Group이 제어할까?"

**Width 체크 시**:
- 자식의 Width를 Layout Group이 결정
- 자식의 preferred width나 flexible width에 따라 크기 결정

**Height 체크 시**:
- 자식의 Height를 Layout Group이 결정
- 자식의 preferred height나 flexible height에 따라 크기 결정

#### Child Force Expand

**역할**: "남은 공간을 자식들이 나눠 가질까?"

**Width 체크 시**:
- 남은 가로 공간을 자식들이 균등하게 나눠 가짐
- flexible width가 0이어도 확장됨

**Height 체크 시**:
- 남은 세로 공간을 자식들이 균등하게 나눠 가짐

#### 조합별 동작 예시

**Vertical Layout Group 기준**:

| Control Child Size (Width) | Child Force Expand (Width) | 결과 |
|:---:|:---:|:---|
| ✗ | ✗ | 자식이 자신의 원래 크기 유지 |
| ✓ | ✗ | 자식이 preferred width로 조절됨 |
| ✓ | ✓ | 자식이 부모 너비에 맞춰 늘어남 |
| ✗ | ✓ | 자식이 자신의 크기 유지 (Force Expand 무시됨) |

**채팅 UI에서의 권장 설정**:
```
Vertical Layout Group (채팅 목록)
├── Control Child Size: Width ✓, Height ✗
├── Child Force Expand: Width ✗, Height ✗
└── 결과: 각 채팅 버블이 텍스트에 맞는 높이를 가지면서
         너비는 부모에 의해 제어됨
```

### Layout Group + ContentSizeFitter 조합

**황금 조합**: 채팅 UI의 핵심 패턴

```
Vertical Layout Group + ContentSizeFitter
├── 자식들을 세로로 정렬 (Vertical Layout Group)
└── 자식 개수에 따라 부모 크기 조절 (ContentSizeFitter)
```

**동작 원리**:
1. Vertical Layout Group이 자식들의 preferred height 합계 계산
2. ContentSizeFitter가 이 합계를 부모의 높이로 설정
3. 자식이 추가/제거되면 자동으로 크기 조절

---

## 4. LayoutElement 컴포넌트

### LayoutElement란?

LayoutElement는 **Layout 시스템에서 자신의 크기 선호도를 명시적으로 지정**하는 컴포넌트입니다.

**비유**: "나는 최소 이 정도 크기는 필요해, 가능하면 이 정도가 좋아, 그리고 이 비율로 늘어날 수 있어"

### 주요 속성

| 속성 | 설명 | 기본값 |
|:---|:---|:---:|
| **Ignore Layout** | Layout 계산에서 제외 | false |
| **Min Width/Height** | 최소 크기 | -1 (무시) |
| **Preferred Width/Height** | 선호 크기 | -1 (무시) |
| **Flexible Width/Height** | 유연성 (확장 가중치) | -1 (무시) |
| **Layout Priority** | 우선순위 | 1 |

### 크기 결정 순서

Layout Group은 다음 순서로 자식 크기를 결정합니다:

```
1단계: Min Size 확보
   ↓
2단계: Preferred Size 충족 (공간이 있다면)
   ↓
3단계: Flexible로 남은 공간 분배 (공간이 남는다면)
```

### Flexible Width/Height의 의미

**Flexible**은 "남은 공간을 얼마나 가져갈지"의 **비율**입니다.

**예시**:
- 자식 A: Flexible Width = 1
- 자식 B: Flexible Width = 2
- 남은 공간 300px

**결과**:
- A는 100px 추가 (1/3)
- B는 200px 추가 (2/3)

### LayoutElement 활용 예시

#### 예시 1: 최소 크기 보장

```
LayoutElement
├── Min Width: 100
├── Min Height: 50
└── 결과: 어떤 상황에서도 100x50 이하로 줄어들지 않음
```

#### 예시 2: 고정 크기

```
LayoutElement
├── Preferred Width: 200
├── Preferred Height: 100
├── Flexible Width: 0
├── Flexible Height: 0
└── 결과: 항상 200x100 크기 유지
```

#### 예시 3: 확장 가능한 요소

```
LayoutElement
├── Min Width: 100
├── Preferred Width: 200
├── Flexible Width: 1
└── 결과: 최소 100, 기본 200, 남은 공간이 있으면 확장
```

### Ignore Layout 활용

**용도**: Layout Group 안에 있지만 정렬에서 제외하고 싶은 요소

**예시**: 채팅 목록 위에 떠 있는 "새 메시지" 알림
```
Vertical Layout Group (채팅 목록)
├── 채팅 버블 1
├── 채팅 버블 2
├── 채팅 버블 3
└── 새 메시지 알림 (Ignore Layout = true)
    └── 정렬에서 제외, 독립적으로 위치 지정 가능
```

---

## 5. TextMeshPro와 동적 크기

### TextMeshPro의 크기 계산

TextMeshPro는 텍스트 내용에 따라 정확한 **Preferred Size**를 계산합니다.

**계산 요소**:
- 텍스트 내용 (글자 수, 줄바꿈)
- 폰트 크기
- 폰트 종류 (글자별 폭이 다름)
- 자간, 줄간격
- 최대 너비 제한 (Wrapping)

### 핵심 속성

#### 1. Auto Size (자동 크기 조절)

텍스트가 주어진 영역에 맞게 폰트 크기를 자동 조절합니다.

**설정**:
- Enable Auto Size: ✓
- Min: 10
- Max: 36

**동작**: 텍스트가 길어지면 폰트 크기가 작아짐

**주의**: ContentSizeFitter와 함께 사용 시 무한 루프 주의!

#### 2. Wrapping & Overflow

| 옵션 | 설명 | 사용 케이스 |
|:---|:---|:---|
| **Wrapping: Disabled** | 줄바꿈 안 함 | 한 줄 제목 |
| **Wrapping: Enabled** | 자동 줄바꿈 | 채팅 버블 (권장) |
| **Overflow: Overflow** | 영역 밖으로 표시 | 특수 효과 |
| **Overflow: Ellipsis** | 말줄임표(...) | 제한된 공간 |
| **Overflow: Truncate** | 잘라냄 | 제한된 공간 |

### ContentSizeFitter와 TextMeshPro 조합

**황금 패턴**: 텍스트에 맞는 버블 크기

```
채팅 버블 (Image + ContentSizeFitter)
├── Horizontal Fit: Preferred Size
├── Vertical Fit: Preferred Size
└── TextMeshPro (자식)
    ├── Wrapping: Enabled
    └── Overflow: Overflow
```

**문제**: 무한히 늘어나는 버블

**해결**: 최대 너비 제한

```
채팅 버블 (Image + ContentSizeFitter + LayoutElement)
├── ContentSizeFitter
│   ├── Horizontal Fit: Preferred Size
│   └── Vertical Fit: Preferred Size
├── LayoutElement
│   └── Preferred Width: 300 (최대 너비)
└── TextMeshPro (자식)
    ├── Wrapping: Enabled
    └── 부모의 너비 제한에 따라 자동 줄바꿈
```

### 텍스트 크기 가져오기 (스크립트)

```csharp
using TMPro;
using UnityEngine;

public class TextSizeGetter : MonoBehaviour
{
    [SerializeField] private TMP_Text tmpText;
    
    public Vector2 GetTextSize()
    {
        // 텍스트의 선호 크기 가져오기
        return new Vector2(
            tmpText.preferredWidth,
            tmpText.preferredHeight
        );
    }
    
    public Vector2 GetRenderedSize()
    {
        // 실제 렌더링된 크기 가져오기
        return new Vector2(
            tmpText.renderedWidth,
            tmpText.renderedHeight
        );
    }
}
```

---

## 6. 툴팁 UI 구현

### 툴팁이란?

**툴팁(Tooltip)**은 아이템이나 UI 요소에 마우스를 올렸을 때 나타나는 정보 상자입니다.

**싱글 게임에서의 활용**:
- 아이템 정보 표시
- 스킬/버튼 설명
- 퀘스트 힌트
- 맵 지역 정보

### 기본 툴팁 구조

```
Tooltip (ContentSizeFitter + LayoutElement)
├── Background (Image, 9-Slice)
└── Content (Vertical Layout Group)
    ├── TitleText (TextMeshPro) - 아이템 이름
    ├── Divider (Image) - 구분선
    └── DescriptionText (TextMeshPro) - 설명
```

**핵심 컴포넌트**:
- **ContentSizeFitter**: 텍스트 길이에 맞춰 크기 자동 조절
- **LayoutElement**: 최대 너비 제한 (화면 밖 방지)
- **Vertical Layout Group**: 제목/설명 세로 정렬

### 툴팁 크기 설정

```csharp
// Tooltip 오브젝트 설정
ContentSizeFitter:
├── Horizontal Fit: Preferred Size
└── Vertical Fit: Preferred Size

LayoutElement:
├── Preferred Width: 300 (최대 너비)
└── Min Width: 100 (최소 너비)

Vertical Layout Group:
├── Padding: 10, 10, 10, 10
├── Spacing: 5
├── Child Alignment: Upper Left
└── Control Child Size: Width ✓, Height ✗
```

### 마우스 따라다니는 툴팁

```csharp
using UnityEngine;

public class TooltipController : MonoBehaviour
{
    [SerializeField] private RectTransform tooltipRect;
    [SerializeField] private Canvas canvas;
    [SerializeField] private Vector2 offset = new Vector2(10, -10);
    
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
            pivot.x = 1; // 우측 기준
        }
        
        // 하단 화면 밖으로 나가면 하단 기준으로 전환
        if (pos.y - tooltipSize.y < 0)
        {
            pivot.y = 0; // 하단 기준
        }
        
        tooltipRect.pivot = pivot;
    }
    
    public void Show(string title, string description)
    {
        // 툴팁 텍스트 설정
        tooltipRect.GetComponentInChildren<TitleText>().text = title;
        tooltipRect.GetComponentInChildren<DescriptionText>().text = description;
        
        tooltipRect.gameObject.SetActive(true);
    }
    
    public void Hide()
    {
        tooltipRect.gameObject.SetActive(false);
    }
}
```

### Pivot을 활용한 방향 전환

**핵심 개념**: 툴팁이 화면 밖으로 나갈 때, **Pivot을 전환**하여 반대편에서 자라나게 함

```
화면 좌측 상단:          화면 우측 상단:
Pivot (0, 1)             Pivot (1, 1)
┌─────────┐                    ┌─────────┐
●         │                    │         ●
│ 툴팁    │                    │    툴팁 │
└─────────┘                    └─────────┘

화면 좌측 하단:          화면 우측 하단:
Pivot (0, 0)             Pivot (1, 0)
●         │                    │         ●
┌─────────┐                    ┌─────────┐
│ 툴팁    │                    │    툴팁 │
└─────────┘                    └─────────┘
```

**장점**: 항상 마우스 근처에 툴팁이 보임, 화면 밖으로 나가지 않음

### 아이템 호버 이벤트 연결

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class ItemSlot : MonoBehaviour, IPointerEnterHandler, IPointerExitHandler
{
    [SerializeField] private string itemName = "체력 물약";
    [SerializeField] private string itemDescription = "HP를 50 회복합니다.";
    
    private TooltipController tooltip;
    
    void Start()
    {
        tooltip = FindObjectOfType<TooltipController>();
    }
    
    public void OnPointerEnter(PointerEventData eventData)
    {
        // 마우스가 아이템 위에 올라감
        tooltip.Show(itemName, itemDescription);
    }
    
    public void OnPointerExit(PointerEventData eventData)
    {
        // 마우스가 아이템에서 벗어남
        tooltip.Hide();
    }
}
```

---

## 7. NPC 대화 말풍선

### 말풍선이란?

**말풍선(Speech Bubble)**은 NPC가 대화할 때 머리 위에 표시되는 텍스트 상자입니다.

**싱글 게임에서의 활용**:
- NPC 대화
- 튜토리얼 힌트
- 캐릭터 생각 (Thought Bubble)
- 퀘스트 힌트

### 말풍선의 특징

**툴팁 vs 말풍선**:

| 특징 | 툴팁 | 말풍선 |
|:---|:---|:---|
| 위치 | 마우스 따라다님 | NPC 머리 위 고정 |
| 꼬리 | 없음 (또는 단순) | **꼬리 필수** |
| 표시 시간 | 호버 중에만 | 일정 시간 또는 클릭 시 |
| 크기 | 작고 간결 | 다양 (짧은 대사~긴 대사) |

### 말풍선 꼬리(Tail) 처리법 🔥

**가장 큰 도전**: 9-Slice 이미지로 말풍선 몸통을 늘리면, **꼬리도 함께 찌그러짐**

```
문제 상황 (9-Slice만 사용):
짧은 대사:          긴 대사:
┌───────┐          ┌──────────────────┐
│ 안녕  │          │ 안녕하세요 반갑습니다 │
└───▼───┘          └──────▼───────────┘
   ↑                      ↑
  정상                  꼬리 늘어남!
```

**해결책: 이미지 분리**

```
DialogueBubble
├── BubbleBody (Image, 9-Slice)
│   ├── ContentSizeFitter (크기 자동 조절)
│   ├── LayoutElement (최대 너비 제한)
│   └── Text (TextMeshPro)
└── BubbleTail (Image, 일반)
    ├── Anchor: Bottom-Center (몸통 하단 중앙에 고정)
    ├── Pivot: (0.5, 1) (위쪽이 몸통에 붙음)
    └── 크기 고정 (늘어나지 않음)
```

### 말풍선 기본 구조

```csharp
// BubbleBody 설정
ContentSizeFitter:
├── Horizontal Fit: Preferred Size
└── Vertical Fit: Preferred Size

LayoutElement:
├── Preferred Width: 250 (최대 너비)
└── Min Width: 80 (최소 너비)

// BubbleTail 설정 (앵커로 위치 고정)
RectTransform:
├── Anchor: Bottom-Center (0.5, 0)
├── Pivot: (0.5, 1)
└── Anchored Position: (0, 0)
```

### 좌/우 NPC에 따른 꼬리 방향 전환

```csharp
using UnityEngine;

public class DialogueBubble : MonoBehaviour
{
    [SerializeField] private RectTransform bubbleBody;
    [SerializeField] private RectTransform bubbleTail;
    
    /// <summary>
    /// 말풍선 방향 설정 (NPC 위치에 따라)
    /// </summary>
    /// <param name="isLeft">true면 좌측 NPC, false면 우측 NPC</param>
    public void SetDirection(bool isLeft)
    {
        if (isLeft)
        {
            // 좌측 NPC: 꼬리가 좌하단
            bubbleTail.anchorMin = new Vector2(0.2f, 0);
            bubbleTail.anchorMax = new Vector2(0.2f, 0);
        }
        else
        {
            // 우측 NPC: 꼬리가 우하단
            bubbleTail.anchorMin = new Vector2(0.8f, 0);
            bubbleTail.anchorMax = new Vector2(0.8f, 0);
            
            // 좌우 반전
            bubbleTail.localScale = new Vector3(-1, 1, 1);
        }
    }
    
    public void SetText(string text)
    {
        bubbleBody.GetComponentInChildren<TMPro.TMP_Text>().text = text;
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

### 말풍선을 NPC 머리 위에 배치

```csharp
using UnityEngine;

public class NPCDialogue : MonoBehaviour
{
    [SerializeField] private DialogueBubble bubblePrefab;
    [SerializeField] private Transform bubbleSpawnPoint; // NPC 머리 위
    
    private DialogueBubble currentBubble;
    
    public void Say(string text, bool isLeftSide = true)
    {
        // 기존 말풍선 제거
        if (currentBubble != null)
        {
            Destroy(currentBubble.gameObject);
        }
        
        // 새 말풍선 생성
        currentBubble = Instantiate(bubblePrefab, bubbleSpawnPoint.position, Quaternion.identity, transform.parent);
        
        // World Space → Screen Space 변환
        Vector3 screenPos = Camera.main.WorldToScreenPoint(bubbleSpawnPoint.position);
        currentBubble.GetComponent<RectTransform>().position = screenPos;
        
        // 방향 및 텍스트 설정
        currentBubble.SetDirection(isLeftSide);
        currentBubble.SetText(text);
        currentBubble.Show();
    }
    
    public void HideBubble()
    {
        if (currentBubble != null)
        {
            currentBubble.Hide();
        }
    }
}
```

### 타이핑 효과 (선택)

한 글자씩 표시되는 타이핑 효과

```csharp
using System.Collections;
using TMPro;
using UnityEngine;

public class TypingEffect : MonoBehaviour
{
    [SerializeField] private TMP_Text textComponent;
    [SerializeField] private float typingSpeed = 0.05f;
    
    private Coroutine typingCoroutine;
    
    public void ShowText(string fullText)
    {
        if (typingCoroutine != null)
        {
            StopCoroutine(typingCoroutine);
        }
        
        typingCoroutine = StartCoroutine(TypeText(fullText));
    }
    
    private IEnumerator TypeText(string fullText)
    {
        textComponent.text = "";
        
        foreach (char c in fullText)
        {
            textComponent.text += c;
            yield return new WaitForSeconds(typingSpeed);
        }
    }
    
    public void SkipTyping()
    {
        if (typingCoroutine != null)
        {
            StopCoroutine(typingCoroutine);
            typingCoroutine = null;
        }
    }
}
```

### 실전 팁

**1. World Space UI vs Screen Space UI**
- World Space: NPC와 함께 움직임, 원근감 있음
- Screen Space: 항상 카메라 정면, 읽기 쉬움
- **권장**: Screen Space + World 좌표 변환

**2. 말풍선 표시 시간**
```csharp
// 짧은 대사: 2초
// 긴 대사: 글자 수 * 0.1초
float displayDuration = Mathf.Max(2f, text.Length * 0.1f);
Invoke(nameof(HideBubble), displayDuration);
```

**3. 여러 NPC가 동시에 말할 때**
- Z-Order로 우선순위 지정
- 또는 Canvas Sort Order 사용

---

## 8. 성능 최적화

### Layout 재계산 비용

**문제**: Layout Group은 자식이 변경될 때마다 전체 재계산

**비용이 큰 작업**:
- 자식 추가/제거
- 자식 크기 변경
- 자식 활성화/비활성화

### 최적화 기법 1: Layout 일괄 처리

```csharp
// 나쁜 예: 매번 Layout 재계산
for (int i = 0; i < 100; i++)
{
    Instantiate(bubblePrefab, content);
    // 100번 Layout 재계산!
}

// 좋은 예: Layout 비활성화 후 일괄 추가
LayoutGroup layoutGroup = content.GetComponent<LayoutGroup>();
layoutGroup.enabled = false;

for (int i = 0; i < 100; i++)
{
    Instantiate(bubblePrefab, content);
}

layoutGroup.enabled = true;
// 코루틴으로 다음 프레임에 처리 (권장)
StartCoroutine(RefreshAfterBatch());

private IEnumerator RefreshAfterBatch()
{
    yield return null; // Layout 자동 갱신 대기
    // 필요한 후처리 (스크롤 이동 등)
}
```

### 최적화 기법 2: Object Pooling

채팅 버블을 매번 생성/삭제하지 않고 재사용합니다.

```csharp
public class ChatBubblePool : MonoBehaviour
{
    [SerializeField] private GameObject bubblePrefab;
    [SerializeField] private int initialPoolSize = 20;
    
    private Queue<GameObject> pool = new Queue<GameObject>();
    
    void Start()
    {
        // 미리 생성
        for (int i = 0; i < initialPoolSize; i++)
        {
            GameObject bubble = Instantiate(bubblePrefab, transform);
            bubble.SetActive(false);
            pool.Enqueue(bubble);
        }
    }
    
    public GameObject GetBubble()
    {
        if (pool.Count > 0)
        {
            GameObject bubble = pool.Dequeue();
            bubble.SetActive(true);
            return bubble;
        }
        
        // 풀이 비었으면 새로 생성
        return Instantiate(bubblePrefab, transform);
    }
    
    public void ReturnBubble(GameObject bubble)
    {
        bubble.SetActive(false);
        pool.Enqueue(bubble);
    }
}
```

### 최적화 기법 3: 가상화 (Virtualization)

화면에 보이는 버블만 실제로 렌더링합니다.

**원리**:
- 1000개의 채팅 메시지가 있어도
- 화면에는 10개만 보임
- 10개의 버블만 생성하고 스크롤 시 재사용

**구현 복잡도**: 높음 (3주차 이후 학습)

### Raycast Target 최적화 (복습)

```
채팅 버블 구조
├── Background (Image)
│   └── Raycast Target: ✗ (클릭 불필요)
├── Profile (Image)
│   └── Raycast Target: ✗ (클릭 불필요)
└── Text (TextMeshPro)
    └── Raycast Target: ✗ (클릭 불필요)
```

**규칙**: 상호작용이 필요한 요소만 Raycast Target 활성화

### 최적화 기법 4: Layout Group 중첩 최소화 ⚠️

**문제**: Layout Group이 중첩될수록 Unity UI 시스템의 **Dirty Flag(변경 감지)** 처리 비용이 기하급수적으로 증가

**나쁜 예 (중첩 지옥)**:
```
ChatBubble (Horizontal Layout Group)        ← 1단계
├── ProfileContainer (Vertical Layout Group) ← 2단계
│   └── ProfileImage
└── BubbleContainer (Vertical Layout Group)  ← 2단계
    ├── NameText
    └── MessageContainer (Horizontal Layout Group) ← 3단계
        └── Message
```

**좋은 예 (앵커와 혼용)**:
```
ChatBubble (Horizontal Layout Group)
├── ProfileImage (앵커로 고정, Layout 불필요)
└── BubbleContainer (ContentSizeFitter만)
    ├── NameText (앵커: Top-Stretch)
    └── Message (앵커: Stretch-Stretch)
```

**실무 원칙**:
- **고정 크기 요소**(프로필 이미지, 아이콘)는 **앵커로 배치**
- **가변 크기 요소**(텍스트, 동적 콘텐츠)만 **Layout Group 사용**
- Layout Group 중첩은 **최대 2단계**까지만
- 모든 것을 Layout Group으로 해결하려는 강박을 버리세요!

---

## 9. 실무 함정과 프로덕션 팁 🔥

### 9-1. 말풍선 꼬리(Tail) 처리법

실제 채팅 UI는 네모난 박스가 아니라 **꼬리가 달린 말풍선**입니다.

**문제**: 9-Slice 이미지를 써도 꼬리 부분이 늘어나면 찌그러짐

```
일반 9-Slice 적용 시:
┌─────────────┐
│   텍스트    │◀── 꼬리가 늘어남!
└─────────────┘
```

**해결책 A: 이미지 분리 (권장)**

```
구조:
ChatBubble
├── BubbleBody (9-Slice 이미지, 늘어나는 부분)
└── BubbleTail (일반 이미지, 고정 크기)
    └── 앵커로 위치 고정 (예: 좌하단)
```

```csharp
// 좌측/우측 버블에 따라 꼬리 위치 변경
public void SetBubbleDirection(bool isLeft)
{
    tailImage.rectTransform.anchorMin = isLeft ? new Vector2(0, 0) : new Vector2(1, 0);
    tailImage.rectTransform.anchorMax = isLeft ? new Vector2(0, 0) : new Vector2(1, 0);
    tailImage.rectTransform.pivot = isLeft ? new Vector2(1, 0.5f) : new Vector2(0, 0.5f);
    
    // 좌우 반전
    tailImage.rectTransform.localScale = isLeft ? Vector3.one : new Vector3(-1, 1, 1);
}
```

**해결책 B: Sprite Editor Border 설정**

1. Sprite Editor에서 이미지 선택
2. Border 설정 시 **꼬리 부분을 Border 밖으로** 설정
3. 꼬리는 늘어나지 않고, 몸통만 늘어남

```
Sprite Border 설정:
┌───┬─────────┬───┐
│ L │  꼬리   │ R │  ← Top Border (꼬리 포함)
├───┼─────────┼───┤
│   │ 늘어남  │   │  ← 9-Slice 영역
├───┼─────────┼───┤
│ L │         │ R │  ← Bottom Border
└───┴─────────┴───┘
```

**디자이너 협업 팁**:
- 말풍선 이미지를 받을 때 **꼬리 분리 여부** 미리 협의
- 9-Slice용 Border 가이드 요청
- 좌/우 버블용 이미지를 따로 받거나, 코드로 Flip 처리

### 9-2. ScrollView 튀는 현상 (Jittering) 해결

**문제**: 동적 목록(퀘스트 로그, 인벤토리)에서 상단에 아이템을 추가할 때, 스크롤 위치가 **팍!** 하고 튀는 현상

**원인**: ContentSizeFitter가 높이를 재계산하면서, 현재 보고 있던 스크롤 위치(Position)가 어긋남

**사용 사례**:
- 퀘스트 로그: 최신 퀘스트를 상단에 추가
- 채팅 로그: 과거 메시지 로딩 (위로 스크롤 시)
- 인벤토리: 정렬 후 아이템 재배치

**해결책: 스크롤 위치 수동 보정**

```csharp
public class DynamicListManager : MonoBehaviour
{
    [SerializeField] private ScrollRect scrollRect;
    [SerializeField] private RectTransform content;
    
    /// <summary>
    /// 상단에 아이템을 추가할 때 스크롤 위치 유지
    /// </summary>
    public void AddItemToTop(GameObject itemPrefab)
    {
        // 1. 추가 전 Content 높이 저장
        float previousHeight = content.rect.height;
        
        // 2. 아이템을 맨 위에 추가
        GameObject item = Instantiate(itemPrefab, content);
        item.transform.SetAsFirstSibling(); // 맨 위로 이동
        
        // 3. 코루틴으로 레이아웃 갱신 후 보정
        StartCoroutine(AdjustScrollPositionAfterAdd(previousHeight));
    }
    
    private IEnumerator AdjustScrollPositionAfterAdd(float previousHeight)
    {
        // Layout 갱신 대기
        yield return null;
        
        // 4. 새로운 Content 높이 계산
        float newHeight = content.rect.height;
        float addedHeight = newHeight - previousHeight;
        
        // 5. 스크롤 위치 보정 (추가된 높이만큼 아래로)
        Vector2 pos = content.anchoredPosition;
        pos.y += addedHeight;
        content.anchoredPosition = pos;
    }
}
```

**핵심 원리**:
- 상단에 콘텐츠 추가 시, **추가된 높이만큼 스크롤 위치를 수동으로 보정**
- 단순히 LayoutGroup만 믿으면 동적 로딩 기능 구현 시 멘탈 붕괴

### 9-3. 동적 목록 스크롤 방향

**퀘스트 로그 / 채팅 로그 구조**:
- 최신 항목이 **아래** 또는 **위**에 표시
- 스크롤 방향 설정 중요

**Bottom-to-Top (최신 항목이 아래)**:

```
Content 설정 (채팅/퀘스트 로그):
├── Anchor: Top-Stretch (위쪽 고정)
├── Pivot: (0.5, 1) ← 위쪽 기준!
└── Vertical Layout Group
    └── Child Alignment: Upper Left/Center
```

**왜 Pivot이 (0.5, 1)인가?**:
- ContentSizeFitter가 높이를 늘릴 때, **위쪽이 고정**되고 **아래로** 늘어남
- 새 항목이 추가되면 Content가 아래로 확장
- ScrollRect의 `normalizedPosition.y = 0`이 **맨 아래**를 의미

**Top-to-Bottom (최신 항목이 위) - 알림 목록 등**:
```
Content 설정:
├── Anchor: Bottom-Stretch (아래쪽 고정)
├── Pivot: (0.5, 0) ← 아래쪽 기준!
└── Vertical Layout Group
    └── Child Alignment: Lower Left/Center
```

### 9-4. 코루틴을 활용한 우아한 갱신 패턴

**ForceRebuild 대신 사용하는 정석 패턴들**:

```csharp
// 패턴 1: 단순 대기 후 처리
private IEnumerator WaitAndProcess()
{
    yield return null; // 1프레임 대기
    // Layout이 자동으로 갱신된 후 실행됨
    DoSomething();
}

// 패턴 2: EndOfFrame 대기 (더 확실)
private IEnumerator WaitEndOfFrameAndProcess()
{
    yield return new WaitForEndOfFrame();
    // 렌더링 직전에 실행
    DoSomething();
}

// 패턴 3: 조건부 대기
private IEnumerator WaitUntilLayoutReady()
{
    // Content 높이가 변경될 때까지 대기
    float lastHeight = content.rect.height;
    yield return null;
    
    while (Mathf.Approximately(content.rect.height, lastHeight))
    {
        yield return null;
    }
    
    // 높이가 변경되면 실행
    DoSomething();
}
```

**언제 ForceRebuild를 써야 하는가?**:
- 같은 프레임 내에서 **반드시 즉시** 크기를 알아야 할 때
- 예: 드래그 중 실시간 위치 계산
- 그 외에는 **거의 사용할 일 없음**

---

## 10. 실전 패턴 모음

### 패턴 1: 아이템 툴팁

```
ItemTooltip
├── Components
│   ├── ContentSizeFitter
│   │   ├── Horizontal Fit: Preferred Size
│   │   └── Vertical Fit: Preferred Size
│   └── LayoutElement
│       ├── Preferred Width: 300 (최대 너비)
│       └── Min Width: 150 (최소 너비)
│
├── Children (Vertical Layout Group)
│   ├── TitleText (TextMeshPro)
│   │   └── Font Size: 20, Color: Yellow
│   ├── Divider (Image, Height: 2px)
│   ├── StatsText (TextMeshPro)
│   │   └── "공격력 +50\n방어력 +20"
│   └── DescriptionText (TextMeshPro)
│       └── Font Size: 14, Color: Gray
```

### 패턴 2: NPC 대화 말풍선

```
DialogueBubble
├── BubbleBody
│   ├── ContentSizeFitter (Preferred Size)
│   ├── LayoutElement (Preferred Width: 250)
│   └── Text (TextMeshPro)
│
└── BubbleTail (앵커로 하단 중앙에 고정)
    ├── Anchor: Bottom-Center
    ├── Pivot: (0.5, 1)
    └── Size: 고정 (20x10)
```

### 패턴 3: 퀘스트 로그 항목

```
QuestLogItem (Horizontal Layout Group)
├── Components
│   ├── Horizontal Layout Group
│   │   ├── Padding: 10
│   │   ├── Spacing: 10
│   │   └── Control Child Size: Height ✓
│   └── LayoutElement
│       └── Min Height: 80
│
├── Children
│   ├── IconImage (60x60, LayoutElement: Min Width = 60)
│   └── TextContainer (Vertical Layout Group)
│       ├── QuestTitle (TextMeshPro, Bold)
│       └── QuestDescription (TextMeshPro, Small)
```

### 패턴 4: 동적 목록 컨테이너

```
DynamicListContainer (ScrollView Content)
├── Components
│   ├── Vertical Layout Group
│   │   ├── Padding: 10, 10, 10, 10
│   │   ├── Spacing: 5
│   │   └── Child Alignment: Upper Center
│   └── ContentSizeFitter
│       └── Vertical Fit: Preferred Size
│
├── Structure
│   ├── Anchor: Top-Stretch
│   ├── Pivot: (0.5, 1)
│   └── 자식 추가/제거 시 자동 크기 조절
```

---

## 11. 개념 비교표

### Layout 컴포넌트 비교

| 컴포넌트 | 역할 | 사용 케이스 |
|:---|:---|:---|
| **ContentSizeFitter** | 자신의 크기를 콘텐츠에 맞춤 | 동적 크기 버블, 툴팁 |
| **Horizontal Layout Group** | 자식을 가로로 정렬 | 버튼 행, 탭 바 |
| **Vertical Layout Group** | 자식을 세로로 정렬 | 채팅 목록, 메뉴 |
| **Grid Layout Group** | 자식을 격자로 정렬 | 인벤토리, 갤러리 |
| **LayoutElement** | 크기 선호도 지정 | 최소/최대 크기 제한 |

### Fit Mode 비교

| Mode | 동작 | 사용 케이스 |
|:---|:---|:---|
| **Unconstrained** | 크기 조절 안 함 | 고정 크기 유지 |
| **Min Size** | 최소 크기로 조절 | 거의 사용 안 함 |
| **Preferred Size** | 선호 크기로 조절 | **동적 콘텐츠 (권장)** |

### Child Control 옵션 비교

| 옵션 조합 | 결과 |
|:---|:---|
| Control ✗, Expand ✗ | 자식이 원래 크기 유지 |
| Control ✓, Expand ✗ | 자식이 preferred size로 조절 |
| Control ✓, Expand ✓ | 자식이 남은 공간까지 확장 |
| Control ✗, Expand ✓ | Control이 없으면 Expand 무시 |

---

## 12. 자주 하는 실수와 해결책

### 실수 1: ContentSizeFitter + Stretch 앵커

**증상**: 에러 메시지 또는 크기가 이상함

**원인**: 두 가지가 동시에 크기를 제어하려 함

**해결**:
```
ContentSizeFitter 사용 시
├── Horizontal Fit: Preferred → Anchor X를 점(0.5)으로
└── Vertical Fit: Preferred → Anchor Y를 점(0.5)으로
```

### 실수 2: Layout이 즉시 반영 안 됨

**증상**: 코드로 변경했는데 크기가 안 바뀜

**원인**: Layout은 프레임 끝에 일괄 처리

**해결**:
```csharp
// ✅ 권장: 코루틴으로 다음 프레임 대기
yield return null;
// 또는
yield return new WaitForEndOfFrame();

// ⚠️ 비권장: 강제 재계산 (성능 저하 주의)
// LayoutRebuilder.ForceRebuildLayoutImmediate(rectTransform);
// → 정말 즉시 필요한 경우에만 사용!
```

### 실수 3: 무한 확장되는 버블

**증상**: 텍스트가 길면 버블이 화면을 넘어감

**원인**: 최대 크기 제한 없음

**해결**:
```
LayoutElement 추가
├── Preferred Width: 300 (최대 너비)
└── 또는 Min/Max Width 설정
```

### 실수 4: ScrollView Content 크기 문제

**증상**: 스크롤이 안 되거나 이상하게 동작

**원인**: Content의 크기가 Viewport보다 작음

**해결**:
```
Content에 다음 설정 확인
├── Vertical Layout Group: ✓
├── ContentSizeFitter
│   └── Vertical Fit: Preferred Size
└── Anchor: Top-Stretch (0, 1) ~ (1, 1)
```

### 실수 5: 텍스트가 잘림

**증상**: 긴 텍스트가 표시 안 됨

**원인**: TextMeshPro Overflow 설정

**해결**:
```
TextMeshPro 설정
├── Wrapping: Enabled
├── Overflow: Overflow (또는 Linked)
└── 부모가 ContentSizeFitter로 크기 조절해야 함
```

---

## 13. 학습 체크리스트

이 문서를 읽은 후 다음을 확인하세요:

### 기본 개념
- [ ] ContentSizeFitter의 역할과 Fit Mode를 설명할 수 있는가?
- [ ] Horizontal/Vertical Layout Group의 차이를 이해했는가?
- [ ] Control Child Size와 Child Force Expand의 차이를 알고 있는가?
- [ ] LayoutElement의 용도를 설명할 수 있는가?

### 조합 패턴
- [ ] ContentSizeFitter + TextMeshPro 조합을 이해했는가?
- [ ] Layout Group + ContentSizeFitter 조합을 이해했는가?
- [ ] ScrollView + Content 구조를 이해했는가?

### 실전 적용 (싱글 인디게임)
- [ ] 툴팁 UI 구조를 설계할 수 있는가?
- [ ] NPC 대화 말풍선 + 꼬리를 구현할 수 있는가?
- [ ] 동적 목록 UI (퀘스트 로그/인벤토리)를 만들 수 있는가?

### 최적화
- [ ] Layout 재계산 비용을 이해했는가?
- [ ] Object Pooling의 필요성을 알고 있는가?
- [ ] Raycast Target 최적화를 적용할 수 있는가?

### 🔥 실무 프로덕션 팁
- [ ] ForceRebuildLayoutImmediate의 위험성을 알고 있는가?
- [ ] 코루틴을 활용한 레이아웃 갱신 패턴을 이해했는가?
- [ ] 9-Slice 말풍선 꼬리 처리법을 알고 있는가?
- [ ] 동적 목록에서 스크롤 튀는 현상 해결법을 이해했는가?
- [ ] Layout Group 중첩 최적화 원칙을 알고 있는가?
- [ ] Pivot을 활용한 툴팁 방향 전환을 구현할 수 있는가?

---

## 마무리

2주차 "Dynamic Info Box" 퀘스트의 핵심은 **Layout 시스템의 완벽한 이해**입니다.

**기억할 것**:
1. **ContentSizeFitter**: 콘텐츠에 맞춰 크기 자동 조절
2. **Layout Group**: 자식 요소 자동 정렬
3. **LayoutElement**: 크기 선호도 명시
4. **조합이 핵심**: 단일 컴포넌트가 아닌 조합으로 동작
5. **Preferred Size**: 대부분의 동적 UI에서 사용
6. **성능 고려**: Layout 재계산은 비용이 큼

**싱글 인디게임 황금 패턴**:
```
아이템 툴팁 = ContentSizeFitter + LayoutElement (최대 너비 제한)
NPC 대화 말풍선 = Body (ContentSizeFitter) + Tail (앵커 고정)
동적 목록 = ScrollView + Vertical Layout Group + ContentSizeFitter
```

**실무 팁**:
- ForceRebuild 남용 금지 → 코루틴으로 `yield return null` 사용
- 말풍선 꼬리는 **이미지 분리**로 찌그러짐 방지
- 툴팁은 **Pivot 전환**으로 화면 밖 방지

다음 실습 가이드(`Week2_Practice_Guide.md`)에서 직접 툴팁과 대화 말풍선을 구현해봅시다!
