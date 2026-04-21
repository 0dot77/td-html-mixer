# TD HTML Mixer

TouchDesigner Web Render TOP용 HTML 기반 컨트롤러.

슬라이더, XY 패드, 토글/트리거 버튼을 웹 페이지로 만들어서 TouchDesigner에서 0~1 값으로 받아 사용합니다. GitHub Pages로 호스팅되어 URL만 입력하면 누구나 바로 사용 가능합니다.

## Layouts

| Layout | URL | 구성 |
|--------|-----|------|
| **Basic** | [basic/](https://0dot77.github.io/td-html-mixer/basic/) | Slider x10, XY Pad x3, Toggle x6, Trigger x4, Preset x3 |

## TouchDesigner 설정

### 1. Web Render TOP

- **Source**: `URL or File`
- **URL**: `https://0dot77.github.io/td-html-mixer/basic/`
- **Resolution**: 1920x1080 권장

### 2. 마우스 인터랙션

Web Render TOP은 텍스처(이미지)만 출력하므로, 마우스 이벤트를 직접 전달해야 합니다.

1. **Container COMP** 생성 → Background TOP에 `webrender1` 연결
2. **Panel Execute DAT** 생성 → Panels에 Container 이름 입력
3. Panel Execute DAT 코드:

```python
def onOffToOn(panelValue, prev):
    u = me.panel.u.val
    v = me.panel.v.val
    op('webrender1').interactMouse(u, v, left=True)

def whileOn(panelValue, prev):
    u = me.panel.u.val
    v = me.panel.v.val
    op('webrender1').interactMouse(u, v, left=True)

def onToOff(panelValue, prev):
    u = me.panel.u.val
    v = me.panel.v.val
    op('webrender1').interactMouse(u, v, left=False)

def onValueChange(panelValue, prev):
    return
```

### 3. 값 읽기 (JSON → CHOP)

HTML 컨트롤러는 모든 값을 `document.title`에 JSON으로 실시간 기록합니다.

1. **Info DAT** → Web Render TOP에 연결
2. **Select DAT** → Info DAT에서 `title` 행 선택
3. **Script CHOP** → 아래 코드로 JSON을 CHOP 채널로 변환:

```python
import json

def cook(scriptOp):
    scriptOp.clear()
    try:
        data = json.loads(op('select1')[0, 0].val)
        for key, val in data.items():
            scriptOp.appendChan(key)[0] = float(val)
    except:
        pass
```

### 출력 채널

| 채널 | 범위 | 설명 |
|------|------|------|
| `s1` ~ `s10` | 0.0 ~ 1.0 | 슬라이더 |
| `xy1_x`, `xy1_y` ~ `xy3_x`, `xy3_y` | 0.0 ~ 1.0 | XY 패드 |
| `t1` ~ `t6` | 0 / 1 | 토글 버튼 |
| `trig1` ~ `trig4` | 0 / 1 | 트리거 (누르는 동안만 1) |

### 4. TD → HTML 제어 (선택)

```python
op('webrender1').executeJavaScript("setSlider(0, 0.5)")    # S1을 0.5로
op('webrender1').executeJavaScript("setXY(0, 0.3, 0.7)")   # XY1 설정
op('webrender1').executeJavaScript("setToggle(2, 1)")       # T3 ON
op('webrender1').executeJavaScript('loadPreset({"s1":0.8})') # 프리셋 로드
```

## 공유

tox 파일을 만들어서 공유하면, 받는 사람은 별도 설치 없이 인터넷 연결만으로 동일한 컨트롤러를 사용할 수 있습니다. UI 업데이트 시 GitHub Pages에 push하면 모든 사용자에게 자동 반영됩니다.

## 새 레이아웃 추가

폴더를 만들고 `index.html`을 넣으면 새 URL이 자동 생성됩니다:

```
td-html-mixer/
├── basic/index.html    → /td-html-mixer/basic/
├── minimal/index.html  → /td-html-mixer/minimal/
└── custom/index.html   → /td-html-mixer/custom/
```
