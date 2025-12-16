# 2주차: "Auto-Resizing Chat" - 이론 학습 자료

## 목표
텍스트 길이에 따라 자동으로 크기가 조절되는 채팅 버블 UI를 구현하기 위한 핵심 이론을 학습합니다.
ContentSizeFitter, LayoutGroup, 그리고 동적 콘텐츠 처리 기법을 마스터합니다.

---

## 1. 동적 콘텐츠의 도전

### 정적 UI vs 동적 UI

#### 정적 UI (1주차에서 학습)
- **특징**: 크기가 미리 정해져 있음
- **예시**: 고정 크기 버튼, 코너에 고정된 HUD
- **처리**: Anchor와 Pivot만으로 충분

#### 동적 UI (2주차에서 학습)
- **특징**: 콘텐츠에 따라 크기가 변함
- **예시**: 채팅 버블, 툴팁, 알림 메시지
- **처리**: ContentSizeFitter와 LayoutGroup 필요

### 왜 동적 UI가 어려운가?

**문제 상황**:
```
"안녕"              →  작은 버블 필요
"안녕하세요, 오늘 정말 좋은 날씨네요!"  →  큰 버블 필요
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

**해결책**:
```csharp
text.text = "새로운 텍스트";
LayoutRebuilder.ForceRebuildLayoutImmediate(rectTransform);
Debug.Log(rectTransform.sizeDelta); // 올바른 크기 출력
```

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

## 6. 채팅 UI 아키텍처

### 전체 구조

```
Canvas
└── ChatContainer (Vertical Layout Group + ContentSizeFitter)
    ├── ScrollView (Scroll Rect)
    │   └── Viewport (Mask)
    │       └── Content (Vertical Layout Group + ContentSizeFitter)
    │           ├── ChatBubble_Left (상대방 메시지)
    │           ├── ChatBubble_Right (내 메시지)
    │           ├── ChatBubble_Left
    │           └── ...
    └── InputField (메시지 입력)
```

### 각 컴포넌트의 역할

#### 1. ScrollView (Scroll Rect)

**역할**: 채팅 목록 스크롤 가능하게 함

**설정**:
- Content: Content 오브젝트 연결
- Horizontal: ✗ (가로 스크롤 비활성화)
- Vertical: ✓
- Movement Type: Elastic 또는 Clamped
- Scrollbar Visibility: Auto Hide

#### 2. Viewport (Mask)

**역할**: 보이는 영역 제한 (마스킹)

**설정**:
- Mask 컴포넌트 추가
- Show Mask Graphic: ✗ (마스크 자체는 안 보이게)

#### 3. Content (Vertical Layout Group + ContentSizeFitter)

**역할**: 채팅 버블들을 세로로 정렬, 개수에 따라 크기 조절

**Vertical Layout Group 설정**:
- Padding: 10, 10, 10, 10
- Spacing: 10
- Child Alignment: Upper Center
- Control Child Size: Width ✓, Height ✗
- Child Force Expand: Width ✗, Height ✗

**ContentSizeFitter 설정**:
- Horizontal Fit: Unconstrained
- Vertical Fit: Preferred Size

#### 4. ChatBubble (개별 채팅 버블)

**역할**: 하나의 메시지 표시

**구조**:
```
ChatBubble (Horizontal Layout Group + ContentSizeFitter)
├── Profile (Image) - 프로필 사진
└── BubbleContainer
    ├── Background (Image) - 말풍선 배경
    └── Text (TextMeshPro) - 메시지 내용
```

### 좌측 버블 (상대방) vs 우측 버블 (나)

#### 좌측 버블 구조

```
ChatBubble_Left (Horizontal Layout Group)
├── Child Alignment: Middle Left
├── ProfileImage (LayoutElement: Min Width = 40)
└── BubbleContent
    ├── Anchor: Left
    └── Text
```

#### 우측 버블 구조

```
ChatBubble_Right (Horizontal Layout Group)
├── Child Alignment: Middle Right
├── Spacer (Flexible Width = 1) - 왼쪽 빈 공간
└── BubbleContent
    ├── Anchor: Right
    └── Text
```

**핵심**: Spacer를 사용하여 버블을 오른쪽으로 밀기

### 버블 크기 자동 조절 로직

```
1. 텍스트 입력
      ↓
2. TextMeshPro가 preferredWidth/Height 계산
      ↓
3. BubbleContent의 ContentSizeFitter가 크기 조절
      ↓
4. ChatBubble의 Horizontal Layout Group이 재정렬
      ↓
5. Content의 Vertical Layout Group이 재정렬
      ↓
6. Content의 ContentSizeFitter가 전체 높이 조절
      ↓
7. ScrollView가 스크롤 범위 업데이트
```

---

## 7. 스크롤 자동 이동

### 새 메시지 시 하단으로 스크롤

채팅 앱의 필수 기능: 새 메시지가 오면 자동으로 하단으로 스크롤

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class ChatScrollHandler : MonoBehaviour
{
    [SerializeField] private ScrollRect scrollRect;
    [SerializeField] private RectTransform content;
    
    public void ScrollToBottom()
    {
        // 다음 프레임에 스크롤 (Layout 재계산 후)
        StartCoroutine(ScrollToBottomCoroutine());
    }
    
    private IEnumerator ScrollToBottomCoroutine()
    {
        // Layout 재계산 대기
        yield return new WaitForEndOfFrame();
        
        // 가장 아래로 스크롤
        scrollRect.normalizedPosition = new Vector2(0, 0);
    }
    
    // 부드러운 스크롤 (선택)
    public void ScrollToBottomSmooth(float duration = 0.3f)
    {
        StartCoroutine(SmoothScrollCoroutine(duration));
    }
    
    private IEnumerator SmoothScrollCoroutine(float duration)
    {
        yield return new WaitForEndOfFrame();
        
        float elapsed = 0f;
        float startPos = scrollRect.normalizedPosition.y;
        
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / duration;
            t = 1f - Mathf.Pow(1f - t, 3f); // Ease Out Cubic
            
            scrollRect.normalizedPosition = new Vector2(0, Mathf.Lerp(startPos, 0, t));
            yield return null;
        }
        
        scrollRect.normalizedPosition = new Vector2(0, 0);
    }
}
```

### 스크롤 위치에 따른 동작

```csharp
public class SmartChatScroll : MonoBehaviour
{
    [SerializeField] private ScrollRect scrollRect;
    [SerializeField] private float autoScrollThreshold = 0.1f;
    
    private bool isAtBottom = true;
    
    void Start()
    {
        scrollRect.onValueChanged.AddListener(OnScrollValueChanged);
    }
    
    void OnScrollValueChanged(Vector2 position)
    {
        // 하단에 가까운지 확인
        isAtBottom = position.y <= autoScrollThreshold;
    }
    
    public void OnNewMessageAdded()
    {
        // 이미 하단에 있었다면 자동 스크롤
        if (isAtBottom)
        {
            ScrollToBottom();
        }
        else
        {
            // 새 메시지 알림 표시
            ShowNewMessageNotification();
        }
    }
    
    private void ScrollToBottom()
    {
        StartCoroutine(ScrollToBottomNextFrame());
    }
    
    private IEnumerator ScrollToBottomNextFrame()
    {
        yield return new WaitForEndOfFrame();
        scrollRect.normalizedPosition = Vector2.zero;
    }
    
    private void ShowNewMessageNotification()
    {
        // "새 메시지" 버튼 표시
        Debug.Log("새 메시지가 있습니다!");
    }
}
```

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
LayoutRebuilder.ForceRebuildLayoutImmediate(content);
// 1번만 Layout 재계산!
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

---

## 9. 실전 패턴 모음

### 패턴 1: 기본 채팅 버블

```
ChatBubble
├── Components
│   ├── Horizontal Layout Group
│   │   ├── Padding: 10, 10, 10, 10
│   │   ├── Spacing: 10
│   │   └── Child Alignment: Middle Left
│   ├── ContentSizeFitter
│   │   ├── Horizontal Fit: Unconstrained
│   │   └── Vertical Fit: Preferred Size
│   └── LayoutElement
│       └── Preferred Width: 400 (최대 너비)
│
├── Children
│   ├── ProfileImage (50x50)
│   └── BubbleContent
│       ├── Background (9-slice 이미지)
│       └── Text (TextMeshPro)
```

### 패턴 2: 시스템 메시지

```
SystemMessage
├── Components
│   ├── ContentSizeFitter
│   │   └── Vertical Fit: Preferred Size
│   └── LayoutElement
│       └── Flexible Width: 1
│
├── Children
│   └── Text (TextMeshPro)
│       ├── Alignment: Center
│       └── Font Size: 14 (작게)
```

### 패턴 3: 날짜 구분선

```
DateDivider
├── Components
│   └── Horizontal Layout Group
│       ├── Spacing: 10
│       └── Child Alignment: Middle Center
│
├── Children
│   ├── LeftLine (Image, Flexible Width = 1)
│   ├── DateText (TextMeshPro, Preferred Width = 100)
│   └── RightLine (Image, Flexible Width = 1)
```

### 패턴 4: 입력 중 표시 ("...")

```
TypingIndicator
├── Components
│   ├── Horizontal Layout Group
│   └── ContentSizeFitter
│
├── Children
│   ├── ProfileImage (작게)
│   └── DotsContainer
│       ├── Dot1 (애니메이션)
│       ├── Dot2 (애니메이션, 딜레이)
│       └── Dot3 (애니메이션, 딜레이)
```

---

## 10. 개념 비교표

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

## 11. 자주 하는 실수와 해결책

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
// 방법 1: 강제 재계산
LayoutRebuilder.ForceRebuildLayoutImmediate(rectTransform);

// 방법 2: 다음 프레임 대기
yield return new WaitForEndOfFrame();
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

## 12. 학습 체크리스트

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

### 실전 적용
- [ ] 채팅 UI의 전체 구조를 설계할 수 있는가?
- [ ] 좌측/우측 버블의 차이점을 구현할 수 있는가?
- [ ] 스크롤 자동 이동을 구현할 수 있는가?

### 최적화
- [ ] Layout 재계산 비용을 이해했는가?
- [ ] Object Pooling의 필요성을 알고 있는가?
- [ ] Raycast Target 최적화를 적용할 수 있는가?

---

## 마무리

2주차 "Auto-Resizing Chat" 퀘스트의 핵심은 **Layout 시스템의 완벽한 이해**입니다.

**기억할 것**:
1. **ContentSizeFitter**: 콘텐츠에 맞춰 크기 자동 조절
2. **Layout Group**: 자식 요소 자동 정렬
3. **LayoutElement**: 크기 선호도 명시
4. **조합이 핵심**: 단일 컴포넌트가 아닌 조합으로 동작
5. **Preferred Size**: 대부분의 동적 UI에서 사용
6. **성능 고려**: Layout 재계산은 비용이 큼

**황금 패턴**:
```
동적 크기 컨테이너 = ContentSizeFitter + Layout Group
동적 텍스트 박스 = ContentSizeFitter + TextMeshPro
최대 크기 제한 = LayoutElement (Preferred Width/Height)
```

다음 실습 가이드(`Week2_Practice_Guide.md`)에서 직접 채팅 UI를 구현해봅시다!
