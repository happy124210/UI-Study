# 2주차: "Auto-Resizing Chat" - 단계별 실습 가이드

## 목표
이론 자료(`Week2_Theory.md`)를 바탕으로 Unity에서 직접 채팅 UI를 구현합니다.
텍스트 길이에 따라 자동으로 크기가 조절되는 채팅 버블을 만들어봅니다.

**학습 방식**: 이론 확인 → Unity 실습 → 검증 → 정리

---

## 학습 전 준비사항

### 1. Unity 씬 설정

1. Unity 에디터를 엽니다.
2. `Assets/Scenes/` 폴더에 새 씬 생성 → 이름: `Week2_ChatUI`
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
        └── Chat/
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
    └── 패딩은 Text의 Margin으로 처리
```

#### Step 7: 올바른 구조로 재설정

1. `Text`의 RectTransform:
   - Anchor Preset: **Stretch-Stretch**
   - Left, Right, Top, Bottom: 모두 `0`
2. `Text`의 TextMeshPro 설정:
   - Margins: Left=10, Top=10, Right=10, Bottom=10

3. `TestBox`의 ContentSizeFitter:
   - Horizontal Fit: `Preferred Size`
   - Vertical Fit: `Preferred Size`

4. 다시 텍스트를 변경해보세요

**관찰**:
- 텍스트 길이에 맞춰 박스 크기가 자동 조절됨!

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

#### Step 6: 채팅 버블용 최대 너비 제한

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

3. 채팅 버블의 최대 너비를 제한하려면?
   - **답**: LayoutElement의 Preferred Width 사용

---

## 실습 4: 기본 채팅 버블 만들기

### 목표
텍스트 길이에 따라 크기가 자동 조절되는 단일 채팅 버블을 만듭니다.

### 이론 확인
`Week2_Theory.md`의 **"6. 채팅 UI 아키텍처"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: 버블 기본 구조 생성

1. Canvas 하위에 `Image` 생성 → 이름: `ChatBubble`
2. RectTransform:
   - Anchor Preset: **Top-Left**
   - Pos X: 100, Pos Y: -100
3. Image 설정:
   - Color: 연한 파란색 (말풍선 색)
   - Image Type: Sliced (9-slice 이미지가 있다면)

#### Step 2: 텍스트 추가

1. `ChatBubble` 하위에 `Text - TextMeshPro` 생성 → 이름: `Message`
2. RectTransform:
   - Anchor Preset: **Stretch-Stretch**
   - Left, Right, Top, Bottom: 모두 `0`
3. TextMeshPro 설정:
   - Text: "안녕하세요!"
   - Font Size: 20
   - Color: 검정
   - Alignment: Left, Top
   - **Margins**: Left=15, Top=10, Right=15, Bottom=10
   - **Wrapping**: Enabled
   - Overflow: Overflow

#### Step 3: ContentSizeFitter 추가

1. `ChatBubble` 선택
2. `Add Component` → `ContentSizeFitter`
3. 설정:
   - Horizontal Fit: `Preferred Size`
   - Vertical Fit: `Preferred Size`

**테스트**:
- `Message`의 텍스트를 변경해보세요
- 버블 크기가 자동으로 변하는지 확인

#### Step 4: 최대 너비 제한

1. `ChatBubble`에 `Add Component` → `LayoutElement`
2. 설정:
   - Preferred Width: `300`
   - 나머지는 체크 해제 (또는 -1)

**테스트**:
- 긴 텍스트 입력: "안녕하세요, 오늘 정말 좋은 날씨네요. 산책하러 가실래요? 공원에서 만나요!"
- 300px 너비에서 자동 줄바꿈 확인

#### Step 5: 프로필 이미지 추가 (좌측 버블)

1. `ChatBubble`의 이름을 `ChatBubble_Left`로 변경
2. `ChatBubble_Left`에 `Horizontal Layout Group` 추가
3. 설정:
   - Padding: 0
   - Spacing: 10
   - Child Alignment: Upper Left
   - Control Child Size: Width ✗, Height ✗
   - Child Force Expand: Width ✗, Height ✗

4. `ChatBubble_Left` 하위에 `Image` 생성 → 이름: `ProfileImage`
5. 순서 조정: `ProfileImage`를 첫 번째로 (드래그)
6. `ProfileImage` 설정:
   - Width: 40, Height: 40
   - LayoutElement 추가:
     - Min Width: 40
     - Min Height: 40

7. 기존 `Message`를 감싸는 컨테이너 생성:
   - `ChatBubble_Left` 하위에 `Image` 생성 → 이름: `BubbleBackground`
   - `Message`를 `BubbleBackground` 하위로 이동
   - `BubbleBackground`에 `ContentSizeFitter` 추가:
     - Horizontal Fit: Preferred Size
     - Vertical Fit: Preferred Size
   - `BubbleBackground`에 `LayoutElement` 추가:
     - Preferred Width: 300

**최종 구조**:
```
ChatBubble_Left (Horizontal Layout Group + ContentSizeFitter)
├── ProfileImage (40x40)
└── BubbleBackground (ContentSizeFitter + LayoutElement)
    └── Message (TextMeshPro)
```

#### Step 6: 우측 버블 만들기

1. `ChatBubble_Left` 복제 → 이름: `ChatBubble_Right`
2. 위치 변경: Anchor를 **Top-Right**로
3. `ProfileImage` 삭제 (또는 오른쪽으로 이동)
4. Horizontal Layout Group 설정:
   - Child Alignment: Upper Right

5. 버블을 오른쪽으로 밀기 위해 Spacer 추가:
   - `BubbleBackground` 앞에 빈 `RectTransform` 생성 → 이름: `Spacer`
   - `Spacer`에 `LayoutElement` 추가:
     - Flexible Width: 1

6. `BubbleBackground` 색상 변경 (내 메시지 구분)

**최종 구조**:
```
ChatBubble_Right (Horizontal Layout Group + ContentSizeFitter)
├── Spacer (LayoutElement: Flexible Width = 1)
└── BubbleBackground (ContentSizeFitter + LayoutElement)
    └── Message (TextMeshPro)
```

### 검증

다음을 확인하세요:

- [ ] 짧은 텍스트: 버블이 텍스트에 맞게 작아짐
- [ ] 긴 텍스트: 300px에서 줄바꿈
- [ ] 좌측 버블: 프로필 이미지 + 버블이 왼쪽 정렬
- [ ] 우측 버블: 버블이 오른쪽 정렬

### 문제 해결

**문제**: 버블이 너무 작아요!
- **해결**: TextMeshPro의 Margins 확인, ContentSizeFitter의 Fit Mode 확인

**문제**: 텍스트가 줄바꿈이 안 돼요!
- **해결**: TextMeshPro의 Wrapping: Enabled 확인, LayoutElement의 Preferred Width 확인

**문제**: 프로필 이미지가 찌그러져요!
- **해결**: LayoutElement로 Min Width/Height 고정

---

## 실습 5: 채팅 목록 ScrollView 만들기

### 목표
여러 채팅 버블을 담고 스크롤할 수 있는 채팅 목록을 만듭니다.

### 이론 확인
`Week2_Theory.md`의 **"6. 채팅 UI 아키텍처"** 및 **"7. 스크롤 자동 이동"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: ScrollView 생성

1. Canvas 하위에 `UI` → `Scroll View` 생성 → 이름: `ChatScrollView`
2. RectTransform:
   - Anchor Preset: **Stretch-Stretch**
   - Left: 20, Right: 20, Top: 100, Bottom: 100

3. Scroll Rect 컴포넌트 설정:
   - Horizontal: ✗ (가로 스크롤 비활성화)
   - Vertical: ✓
   - Movement Type: Clamped
   - Scrollbar Visibility: Auto Hide And Expand Viewport

4. 불필요한 요소 삭제:
   - `Scrollbar Horizontal` 삭제

#### Step 2: Content 설정

1. `ChatScrollView` → `Viewport` → `Content` 선택
2. RectTransform:
   - Anchor Preset: **Top-Stretch** (위쪽 고정, 가로 늘림)
   - Pivot: (0.5, 1) ← 중요! 위쪽 기준
   - Left: 0, Right: 0
   - Top: 0

3. `Add Component` → `Vertical Layout Group`
4. 설정:
   - Padding: Left=10, Right=10, Top=10, Bottom=10
   - Spacing: 15
   - Child Alignment: Upper Center
   - Control Child Size: Width ✓, Height ✗
   - Child Force Expand: Width ✗, Height ✗

5. `Add Component` → `ContentSizeFitter`
6. 설정:
   - Horizontal Fit: Unconstrained
   - Vertical Fit: Preferred Size

#### Step 3: 버블 Prefab 만들기

1. 실습 4에서 만든 `ChatBubble_Left` 선택
2. `Assets/_MyProjects/Prefabs/Week2/` 폴더로 드래그 → Prefab 생성
3. `ChatBubble_Right`도 동일하게 Prefab 생성
4. 씬에서 원본 삭제 (Prefab으로 대체할 것)

#### Step 4: 버블을 Content에 추가

1. `Content` 하위에 `ChatBubble_Left` Prefab 드래그하여 추가
2. `ChatBubble_Right` Prefab도 추가
3. 번갈아가며 여러 개 추가 (테스트용)

**관찰**:
- 버블들이 세로로 정렬됨
- Content 높이가 자동으로 늘어남
- 스크롤 가능해짐

#### Step 5: 스크롤 테스트

1. 많은 버블 추가 (10개 이상)
2. Play 모드로 전환
3. 마우스 휠 또는 드래그로 스크롤 테스트

**확인 사항**:
- [ ] 스크롤이 부드럽게 동작
- [ ] 맨 위/아래에서 정상적으로 멈춤
- [ ] 버블들이 올바르게 정렬됨

#### Step 6: 스크롤바 스타일링 (선택)

1. `Scrollbar Vertical` 선택
2. `Sliding Area` → `Handle` 선택
3. Image 색상 변경 (채팅 앱 느낌으로)
4. Handle의 너비 조정

### 검증

다음을 확인하세요:

- [ ] Content가 버블 개수에 따라 높이 자동 조절
- [ ] 스크롤이 정상 동작
- [ ] 버블들이 올바르게 정렬 (좌/우 구분)
- [ ] Viewport 밖의 버블은 안 보임 (Mask 동작)

### 문제 해결

**문제**: 스크롤이 안 돼요!
- **해결**: Content의 ContentSizeFitter → Vertical Fit: Preferred Size 확인

**문제**: 버블이 Viewport 밖으로 나와요!
- **해결**: Viewport의 Mask 컴포넌트 확인

**문제**: 스크롤 방향이 반대예요!
- **해결**: Content의 Pivot Y가 1인지 확인 (위쪽 기준)

---

## 실습 6: 동적 메시지 추가 스크립트

### 목표
코드로 채팅 버블을 동적으로 추가하고, 새 메시지 시 자동 스크롤되는 기능을 구현합니다.

### 이론 확인
`Week2_Theory.md`의 **"7. 스크롤 자동 이동"** 섹션을 읽어주세요.

### 실습 단계

#### Step 1: ChatBubble 스크립트 작성

1. `Assets/_MyProjects/Scripts/Week2/` 폴더에 새 C# 스크립트 생성
2. 이름: `ChatBubble.cs`

```csharp
using UnityEngine;
using TMPro;

public class ChatBubble : MonoBehaviour
{
    [SerializeField] private TMP_Text messageText;
    [SerializeField] private UnityEngine.UI.Image backgroundImage;
    
    public void SetMessage(string message)
    {
        if (messageText != null)
        {
            messageText.text = message;
        }
    }
    
    public void SetBackgroundColor(Color color)
    {
        if (backgroundImage != null)
        {
            backgroundImage.color = color;
        }
    }
}
```

3. `ChatBubble_Left` Prefab 열기
4. 루트 오브젝트에 `ChatBubble` 컴포넌트 추가
5. `Message Text` 필드에 TextMeshPro 연결
6. `Background Image` 필드에 BubbleBackground 연결
7. Prefab 저장
8. `ChatBubble_Right`도 동일하게 설정

#### Step 2: ChatManager 스크립트 작성

1. 새 C# 스크립트 생성: `ChatManager.cs`

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections;

public class ChatManager : MonoBehaviour
{
    [Header("References")]
    [SerializeField] private Transform chatContent;
    [SerializeField] private ScrollRect scrollRect;
    
    [Header("Prefabs")]
    [SerializeField] private GameObject leftBubblePrefab;
    [SerializeField] private GameObject rightBubblePrefab;
    
    [Header("Settings")]
    [SerializeField] private bool autoScrollToBottom = true;
    
    public void AddMessage(string message, bool isMyMessage)
    {
        // 적절한 Prefab 선택
        GameObject prefab = isMyMessage ? rightBubblePrefab : leftBubblePrefab;
        
        // 버블 생성
        GameObject bubbleObj = Instantiate(prefab, chatContent);
        
        // 메시지 설정
        ChatBubble bubble = bubbleObj.GetComponent<ChatBubble>();
        if (bubble != null)
        {
            bubble.SetMessage(message);
        }
        
        // 자동 스크롤
        if (autoScrollToBottom)
        {
            StartCoroutine(ScrollToBottomNextFrame());
        }
    }
    
    private IEnumerator ScrollToBottomNextFrame()
    {
        // Layout 재계산을 위해 프레임 대기
        yield return new WaitForEndOfFrame();
        
        // 가장 아래로 스크롤
        scrollRect.normalizedPosition = new Vector2(0, 0);
    }
    
    // 테스트용 메서드
    public void AddTestMessage()
    {
        string[] testMessages = new string[]
        {
            "안녕하세요!",
            "오늘 날씨가 좋네요.",
            "점심 뭐 먹을까요?",
            "저는 파스타가 먹고 싶어요. 근처에 맛있는 이탈리안 레스토랑 있나요?",
            "네, 좋아요!"
        };
        
        string message = testMessages[Random.Range(0, testMessages.Length)];
        bool isMyMessage = Random.value > 0.5f;
        
        AddMessage(message, isMyMessage);
    }
}
```

#### Step 3: ChatManager 설정

1. Canvas 하위에 빈 GameObject 생성 → 이름: `ChatManager`
2. `ChatManager` 스크립트 컴포넌트 추가
3. Inspector에서 필드 연결:
   - Chat Content: `Content` 오브젝트
   - Scroll Rect: `ChatScrollView` 오브젝트
   - Left Bubble Prefab: `ChatBubble_Left` Prefab
   - Right Bubble Prefab: `ChatBubble_Right` Prefab

#### Step 4: 테스트 버튼 추가

1. Canvas 하위에 `UI` → `Button - TextMeshPro` 생성 → 이름: `AddMessageButton`
2. RectTransform:
   - Anchor Preset: **Bottom-Center**
   - Pos Y: 50
   - Width: 200, Height: 50
3. Button의 텍스트: "메시지 추가"
4. Button의 OnClick 이벤트에 `ChatManager.AddTestMessage` 연결

#### Step 5: 테스트

1. Play 모드로 전환
2. "메시지 추가" 버튼 클릭
3. **확인 사항**:
   - [ ] 버블이 동적으로 추가됨
   - [ ] 좌/우 버블이 랜덤하게 나타남
   - [ ] 새 메시지 추가 시 자동으로 아래로 스크롤

#### Step 6: 입력 필드 추가 (선택)

1. Canvas 하위에 `UI` → `Input Field - TextMeshPro` 생성
2. RectTransform: 하단에 배치
3. 스크립트 수정:

```csharp
// ChatManager.cs에 추가
[SerializeField] private TMP_InputField inputField;

public void SendMessage()
{
    if (string.IsNullOrEmpty(inputField.text)) return;
    
    AddMessage(inputField.text, true); // 내 메시지로 추가
    inputField.text = "";
    inputField.ActivateInputField(); // 입력 필드에 포커스 유지
}
```

### 검증

다음을 확인하세요:

- [ ] 버튼 클릭으로 메시지 추가 가능
- [ ] 새 메시지가 목록 하단에 추가됨
- [ ] 자동 스크롤이 정상 동작
- [ ] 많은 메시지 추가해도 성능 문제 없음

### 문제 해결

**문제**: 버블이 안 보여요!
- **해결**: Prefab의 참조가 올바른지 확인, chatContent가 Content를 가리키는지 확인

**문제**: 스크롤이 안 돼요!
- **해결**: ScrollRect 참조 확인, Content의 ContentSizeFitter 확인

**문제**: 버블 크기가 이상해요!
- **해결**: Prefab의 ContentSizeFitter, LayoutElement 설정 확인

---

## 실습 7: 스마트 스크롤 구현

### 목표
사용자가 위로 스크롤한 상태에서는 자동 스크롤하지 않고, "새 메시지" 알림을 표시합니다.

### 이론 확인
`Week2_Theory.md`의 **"7. 스크롤 자동 이동"** 섹션의 "스크롤 위치에 따른 동작"을 읽어주세요.

### 실습 단계

#### Step 1: ChatManager 업그레이드

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using System.Collections;

public class ChatManager : MonoBehaviour
{
    [Header("References")]
    [SerializeField] private Transform chatContent;
    [SerializeField] private ScrollRect scrollRect;
    [SerializeField] private GameObject newMessageNotification;
    
    [Header("Prefabs")]
    [SerializeField] private GameObject leftBubblePrefab;
    [SerializeField] private GameObject rightBubblePrefab;
    
    [Header("Settings")]
    [SerializeField] private float scrollThreshold = 0.1f;
    
    private bool isAtBottom = true;
    private int unreadCount = 0;
    
    void Start()
    {
        // 스크롤 이벤트 리스너 등록
        scrollRect.onValueChanged.AddListener(OnScrollValueChanged);
        
        // 알림 숨기기
        if (newMessageNotification != null)
        {
            newMessageNotification.SetActive(false);
        }
    }
    
    void OnScrollValueChanged(Vector2 position)
    {
        // 하단에 가까운지 확인 (normalizedPosition.y가 0이면 맨 아래)
        isAtBottom = position.y <= scrollThreshold;
        
        // 하단으로 스크롤하면 알림 숨기기
        if (isAtBottom)
        {
            HideNewMessageNotification();
        }
    }
    
    public void AddMessage(string message, bool isMyMessage)
    {
        GameObject prefab = isMyMessage ? rightBubblePrefab : leftBubblePrefab;
        GameObject bubbleObj = Instantiate(prefab, chatContent);
        
        ChatBubble bubble = bubbleObj.GetComponent<ChatBubble>();
        if (bubble != null)
        {
            bubble.SetMessage(message);
        }
        
        // 스마트 스크롤
        if (isAtBottom || isMyMessage)
        {
            // 하단에 있거나 내 메시지면 자동 스크롤
            StartCoroutine(ScrollToBottomNextFrame());
        }
        else
        {
            // 위로 스크롤한 상태면 알림 표시
            ShowNewMessageNotification();
        }
    }
    
    private void ShowNewMessageNotification()
    {
        unreadCount++;
        
        if (newMessageNotification != null)
        {
            newMessageNotification.SetActive(true);
            
            // 알림 텍스트 업데이트 (있다면)
            TMP_Text notificationText = newMessageNotification.GetComponentInChildren<TMP_Text>();
            if (notificationText != null)
            {
                notificationText.text = $"새 메시지 {unreadCount}개";
            }
        }
    }
    
    private void HideNewMessageNotification()
    {
        unreadCount = 0;
        
        if (newMessageNotification != null)
        {
            newMessageNotification.SetActive(false);
        }
    }
    
    // 알림 클릭 시 호출
    public void OnNewMessageNotificationClicked()
    {
        StartCoroutine(ScrollToBottomSmooth());
        HideNewMessageNotification();
    }
    
    private IEnumerator ScrollToBottomNextFrame()
    {
        yield return new WaitForEndOfFrame();
        scrollRect.normalizedPosition = new Vector2(0, 0);
    }
    
    private IEnumerator ScrollToBottomSmooth(float duration = 0.3f)
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
    
    // 테스트용
    public void AddTestMessage()
    {
        string[] testMessages = new string[]
        {
            "안녕하세요!",
            "오늘 날씨가 좋네요.",
            "점심 뭐 먹을까요?",
            "저는 파스타가 먹고 싶어요.",
            "네, 좋아요!"
        };
        
        string message = testMessages[Random.Range(0, testMessages.Length)];
        bool isMyMessage = Random.value > 0.5f;
        
        AddMessage(message, isMyMessage);
    }
}
```

#### Step 2: 새 메시지 알림 UI 만들기

1. `ChatScrollView` 하위에 `Button - TextMeshPro` 생성 → 이름: `NewMessageNotification`
2. RectTransform:
   - Anchor Preset: **Bottom-Center**
   - Pos Y: 20
   - Width: 150, Height: 40
3. Image 설정:
   - Color: 연한 파란색 배경
4. TextMeshPro:
   - Text: "새 메시지"
   - Font Size: 16
5. Button OnClick에 `ChatManager.OnNewMessageNotificationClicked` 연결
6. ChatManager Inspector에서 `New Message Notification` 필드 연결

#### Step 3: 테스트

1. Play 모드로 전환
2. 여러 메시지 추가 (스크롤 가능할 정도로)
3. 위로 스크롤
4. 메시지 추가 버튼 클릭
5. **확인 사항**:
   - [ ] 위로 스크롤한 상태에서 새 메시지 오면 알림 표시
   - [ ] 알림 클릭하면 부드럽게 아래로 스크롤
   - [ ] 수동으로 아래로 스크롤해도 알림 사라짐

### 검증

스마트 스크롤 동작 확인:

| 상황 | 예상 동작 |
|:---|:---|
| 맨 아래에서 새 메시지 | 자동 스크롤 |
| 위로 스크롤 후 새 메시지 | 알림 표시, 스크롤 안 함 |
| 내 메시지 전송 | 항상 자동 스크롤 |
| 알림 클릭 | 부드럽게 아래로 스크롤 |

---

## 실습 8: 채팅 UI 완성 및 스타일링

### 목표
채팅 UI를 완성하고, 실제 메신저 앱처럼 스타일링합니다.

### 실습 단계

#### Step 1: 상단 헤더 추가

1. Canvas 하위에 `Image` 생성 → 이름: `Header`
2. RectTransform:
   - Anchor Preset: **Top-Stretch**
   - Height: 80
   - Left: 0, Right: 0, Top: 0
3. 헤더 내용 추가:
   - 뒤로가기 버튼 (좌측)
   - 상대방 이름 (중앙)
   - 메뉴 버튼 (우측)

#### Step 2: 하단 입력 영역 추가

1. Canvas 하위에 `Image` 생성 → 이름: `InputArea`
2. RectTransform:
   - Anchor Preset: **Bottom-Stretch**
   - Height: 80
   - Left: 0, Right: 0, Bottom: 0
3. Horizontal Layout Group 추가:
   - Padding: 10, 10, 10, 10
   - Spacing: 10
4. 자식 요소:
   - Input Field (Flexible Width = 1)
   - Send Button (Fixed Width = 80)

#### Step 3: ScrollView 영역 조정

1. `ChatScrollView` RectTransform:
   - Top: 80 (헤더 높이)
   - Bottom: 80 (입력 영역 높이)

#### Step 4: 버블 스타일 개선

**좌측 버블 (상대방)**:
- 배경색: 연한 회색 (#E5E5EA)
- 모서리: 둥글게 (Sprite 사용 또는 Rounded Rectangle)

**우측 버블 (나)**:
- 배경색: 파란색 (#007AFF)
- 텍스트 색: 흰색

#### Step 5: 시간 표시 추가

1. 버블 Prefab 수정
2. 버블 아래에 시간 텍스트 추가:
   - Font Size: 12
   - Color: 회색
   - Alignment: 버블 방향에 맞춤

```csharp
// ChatBubble.cs에 추가
[SerializeField] private TMP_Text timeText;

public void SetTime(string time)
{
    if (timeText != null)
    {
        timeText.text = time;
    }
}
```

#### Step 6: 날짜 구분선 추가

1. 새 Prefab 생성: `DateDivider`
2. 구조:
   ```
   DateDivider (Horizontal Layout Group)
   ├── LeftLine (Image, Flexible Width = 1)
   ├── DateText (TextMeshPro)
   └── RightLine (Image, Flexible Width = 1)
   ```

3. ChatManager에 날짜 구분선 추가 메서드:

```csharp
[SerializeField] private GameObject dateDividerPrefab;

public void AddDateDivider(string dateString)
{
    GameObject divider = Instantiate(dateDividerPrefab, chatContent);
    TMP_Text dateText = divider.GetComponentInChildren<TMP_Text>();
    if (dateText != null)
    {
        dateText.text = dateString;
    }
}
```

### 최종 구조

```
Canvas
├── Header (상단 바)
│   ├── BackButton
│   ├── TitleText
│   └── MenuButton
│
├── ChatScrollView
│   └── Viewport
│       └── Content (Vertical Layout Group + ContentSizeFitter)
│           ├── DateDivider ("2024년 1월 15일")
│           ├── ChatBubble_Left
│           ├── ChatBubble_Right
│           ├── ...
│           └── NewMessageNotification
│
├── InputArea (하단 입력)
│   ├── InputField
│   └── SendButton
│
└── ChatManager
```

### 검증

최종 체크리스트:

**기능**:
- [ ] 메시지 입력 및 전송
- [ ] 좌/우 버블 구분
- [ ] 스크롤 정상 동작
- [ ] 새 메시지 알림

**UI/UX**:
- [ ] 헤더/입력 영역 배치
- [ ] 버블 스타일링
- [ ] 시간 표시
- [ ] 날짜 구분선

**반응형**:
- [ ] 다양한 해상도에서 테스트
- [ ] 텍스트 길이에 따른 버블 크기 조절

---

## 최종 검증: Auto-Resizing Chat 완성

### 체크리스트

#### 기능 요구사항

- [ ] **텍스트 길이에 따른 버블 크기 자동 조절**
  - 짧은 텍스트: 작은 버블
  - 긴 텍스트: 줄바꿈 후 높이 증가
  - 최대 너비 제한: 약 300px

- [ ] **채팅 목록 스크롤**
  - 위/아래 스크롤 가능
  - 부드러운 스크롤 동작

- [ ] **동적 메시지 추가**
  - 코드로 버블 추가 가능
  - 새 메시지 시 자동 스크롤

- [ ] **스마트 스크롤**
  - 위로 스크롤 시 자동 스크롤 비활성화
  - "새 메시지" 알림 표시

#### UI 요구사항

- [ ] **좌측 버블**: 상대방 메시지, 프로필 이미지 포함
- [ ] **우측 버블**: 내 메시지, 오른쪽 정렬
- [ ] **헤더**: 상단 바
- [ ] **입력 영역**: 하단 입력 필드 + 전송 버튼

#### 해상도 테스트

- [ ] 1920x1080 (16:9)
- [ ] 1334x750 (모바일)
- [ ] 1080x1920 (세로 모바일)

### 완성 기준

다음 조건을 모두 만족하면 **"Auto-Resizing Chat" 완성**입니다:

✅ 텍스트 길이에 따라 버블 크기 자동 조절
✅ 최대 너비에서 자동 줄바꿈
✅ 채팅 목록 스크롤 가능
✅ 동적 메시지 추가 가능
✅ 새 메시지 시 스마트 스크롤 동작
✅ 좌/우 버블 정상 표시

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
├── ChatBubble_Left.prefab
├── ChatBubble_Right.prefab
├── DateDivider.prefab
└── SystemMessage.prefab (선택)
```

### ChatBubble Prefab 구조 (참고용)

```
ChatBubble_Left
├── Components
│   ├── Horizontal Layout Group
│   │   ├── Padding: 0
│   │   ├── Spacing: 10
│   │   └── Child Alignment: Upper Left
│   └── ContentSizeFitter
│       ├── Horizontal Fit: Unconstrained
│       └── Vertical Fit: Preferred Size
│
├── ProfileImage (40x40)
│   └── LayoutElement: Min Width/Height = 40
│
└── BubbleContainer
    ├── ContentSizeFitter
    │   ├── Horizontal Fit: Preferred Size
    │   └── Vertical Fit: Preferred Size
    ├── LayoutElement
    │   └── Preferred Width: 300
    │
    └── Message (TextMeshPro)
        ├── Margins: 15, 10, 15, 10
        ├── Wrapping: Enabled
        └── Overflow: Overflow
```

---

## 마무리

축하합니다! 2주차 "Auto-Resizing Chat" 퀘스트를 완료했습니다.

**학습한 핵심 개념**:
- ✅ ContentSizeFitter의 동작 원리
- ✅ Horizontal/Vertical Layout Group
- ✅ LayoutElement로 크기 제어
- ✅ ScrollView와 동적 콘텐츠
- ✅ 스마트 스크롤 구현

**다음 단계**:
- 이론 자료(`Week2_Theory.md`)를 다시 읽어보며 개념을 정리하세요
- 채팅 UI를 자신만의 스타일로 커스터마이징해보세요
- 다음 주차 Quest 3를 준비하세요

**기억할 것**:
> "Layout 시스템은 조합이다. 단일 컴포넌트가 아닌 여러 컴포넌트의 조합으로 원하는 동작을 만든다."

화이팅! 🚀
