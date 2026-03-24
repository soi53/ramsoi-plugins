# Vanilla HTML/CSS/JS 슬라이드 프레젠테이션 구현 리서치

> 4개 서브에이전트 병렬 리서치 결과 통합 정리

## Agent 1: 슬라이드 레이아웃 & DOM 구조

**채택: CSS Grid 스택 방식**
- `display: grid; grid-template: 1fr / 1fr` 으로 단일 셀 격자 생성
- 모든 `.slide`에 `grid-area: 1 / 1` 지정 → 동일 위치에 겹쳐 배치
- `opacity + visibility` 조합으로 활성 슬라이드만 표시

**16:9 비율 고정**
```css
.slide-deck {
  aspect-ratio: 16 / 9;
  max-width: 100%; max-height: 100%; width: 100%;
}
@media (min-aspect-ratio: 16/9) { .slide-deck { width: auto; height: 100%; } }
```

**브라우저 호환성**
- `aspect-ratio`: Chrome 88+, Firefox 89+, Safari 15+
- `container-type`: Chrome 105+, Safari 16+
- `100dvh`: iOS 15.4+, Chrome 108+

---

## Agent 2: 키보드 & 마우스 네비게이션

- Pointer Events API로 마우스/터치 통합 처리 (touchstart 불필요)
- `history.pushState`로 URL hash 동기화하여 딥링크 및 뒤로가기 지원
- `e.repeat` 체크와 `isAnimating` 플래그로 이중 입력 방지

**프로그레스 바**: `transform: scaleX()` 사용 → 레이아웃 리플로우 없이 GPU 가속

---

## Agent 3: 슬라이드 전환 애니메이션

**채택: CSS Transition + JS 클래스 오케스트레이션**

핵심 패턴:
1. `void element.offsetHeight`로 reflow 강제 후 클래스 추가하여 초기 위치 확정
2. `transitionend` + 안전 타임아웃(800ms) 조합으로 완료 감지
3. `will-change`는 전환 직전에만 설정, 완료 후 즉시 해제
4. double-rAF 패턴으로 동일 프레임 실행 방지

---

## Agent 4: 슬라이드 내 콘텐츠 애니메이션

- `data-step` 속성으로 Fragment 등장 순서 명시
- `classList.toggle('visible', step <= activeStep)` 패턴
- 코드 강조는 `data-highlight-steps="1-2|4-6"` 형식으로 선언적 정의

---

## 주의사항

- iOS Safari `100vh` 버그 → `100dvh` 사용, `100vh` 폴백 병행
- `will-change` 남용 시 GPU 메모리 낭비
- reflow 없이 transition 시작 시 초기 위치 스킵 버그
- `transitionend` 미발화 대비 안전 타임아웃 필수
- HTML 이스케이프를 구문 강조 전에 반드시 처리
