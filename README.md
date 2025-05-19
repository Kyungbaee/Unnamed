# Unnamed

## **주요 구현 내용**


### **1. 오픈월드 환경 구축 및 최적화**

![image.png](attachment:be9f5394-1c62-4025-b832-377183e002de:image.png)

- **지형 생성 및 환경 구축**: Quixel Bridge를 활용한 하이퀄리티 지형과 텍스처 적용
- **조명 및 분위기 설정**: 실시간 라이팅과 포스트 프로세싱 적용으로 게임 환경의 몰입감 강화
- **최적화 작업**: Packed Level Actors 활용하여 Draw Call 감소를 통한 최적화

### **2. 캐릭터 컨트롤 및 전투 시스템 구현**

![image.png](attachment:293ed810-f2fe-4257-be56-c281549f09e7:image.png)

![classdiagram (1).png](attachment:6bb44ee4-d1c6-43ff-9fb0-80841756d1ea:classdiagram_(1).png)

- **`ACharacter` 클래스를 상속받은 `ABaseCharacter`**
    - 기본적인 **이동, 회피, 공격, 피격 반응(GetHit_Implementation)** 등의 기능을 구현
    - `DirectionalHitReact()`를 활용하여 공격 방향에 따른 피격 애니메이션 적용
    - 피격 시 **사운드 및 파티클 효과(PlayHitSound, SpawnHitParticles)** 생성
    - 생존 여부에 따라 `Die()` 함수를 호출하여 사망 처리
- **`ABaseCharacter`를 상속받아 `ASlashCharacter`와 `Enemy` 구현**
    - `ASlashCharacter`는 플레이어 캐릭터로, 피격 시 **무기 충돌 해제**
    - **체력(Attributes->GetHealthPercent)을 확인하여 피격 상태 `EAS_HitReaction` 적용**
    - `Enemy` 클래스에서는 적 AI 동작을 추가하여 별도의 피격 반응 및 전투 로직 구현

![image.png](attachment:4ee0ee14-c120-4264-8c1f-65ce4ae9764c:image.png)

- **전투 시스템 및 무기 장착 시스템 개발**
    - 애니메이션 몽타주(Animation Montages)를 활용한 공격 애니메이션 재생
    - `UBoxComponent`를 활용한 무기 충돌 감지 및 타격 판정
    - `OnComponentBeginOverlap()` 이벤트를 활용하여 적 캐릭터 피격 처리

### **3. Enemy 행동 패턴 구현**

![Enemy순서도 (1).png](attachment:cab98a78-ce0a-49c5-a18f-92f56bd2b2fb:Enemy순서도_(1).png)

- **AI Behavior Tree 및 Blackboard 활용**
    - `AEnemy` 클래스에서 **순찰 → 감지 → 추적 → 공격 → 사망**의 AI 상태 머신 구현
    - `PawnSensingComponent`를 활용하여 **시야 감지 및 플레이어 추적 동작 자동화**
    - `OnSeePawn()` 이벤트를 사용하여 **플레이어 인식 후 Blackboard의 `TargetActor`를 갱신하고 Behavior Tree 실행**
    - `CheckCombatTarget()` 및 `ChaseTarget()`을 통해 **전투 범위 진입 시 추적 및 공격 동작 수행**

### **4. Widget 및 HUD 시스템 구축**

![HUD 캡처.PNG](attachment:6f10f422-b141-460b-bb6d-d71073d05fd6:99c5c751-b59e-40d2-8bc8-d10842ebf03a.png)

![제목 없는 다이어그램 (1).png](attachment:f0cc6bb0-0b2c-49e7-8e4a-4c6adf02379a:제목_없는_다이어그램_(1).png)

- **플레이어 상태 Widget 및 전투 HUD 시스템 구현**
    - `InitializeSlashOverlay()`에서 **플레이어가 시작할 때 HUD 초기화**
    - `ASlashCharacter` 클래스에서 **체력, 스태미나, 골드, 소울 등의 상태를 Widget에 반영**
    - `UUserWidget`을 활용하여 **플레이어 HUD** 동적 업데이트

### **5. 애니메이션 시스템 및 물리 기반 상호작용**

- **Animation Blueprint**를 활용한 캐릭터 움직임 및 전투 모션 구현

![image.png](attachment:b92a3bbb-a0c3-4a4a-ad1b-bba2e6da74c6:image.png)

![image.png](attachment:932837f8-7f64-4ffd-ae89-e570aa0f7e16:image.png)

- **Inverse Kinematics (IK)** 적용하여,
발 위치 조정 및 애니메이션 개선

## **트러블슈팅 및 해결 과정**



### **⚠️ 문제 1: 전투 중 타격 판정이 정확하지 않음**

**원인:** `OnComponentBeginOverlap()` 이벤트가 여러 번 호출되며 중복 충돌 발생

**해결:** `TSet<AActor*>`를 활용하여 이미 충돌한 대상은 다시 판정되지 않도록 개선

### **⚠️ 문제 2: Enemy가 장애물을 넘어가지 못함**

**원인:** NavMesh 설정 미비 및 AI `MoveTo()` 함수 적용 오류

**해결:** `NavModifierVolume`을 활용하여 AI가 장애물을 인식하고 우회하도록 개선

### **⚠️ 문제 3: Player가 공격 중 회피(dodge) 시, 애니메이션이 부자연스럽게 멈춤**

**원인:** `Montage_Play()`가 dodge 애니메이션과 공격 애니메이션을 동시에 실행하면서 충돌 발생

**해결:** `BlendOutTime`을 조정하여 공격 애니메이션이 부드럽게 종료된 후 Dodge 애니메이션이 재생되도록 수정

### **⚠️ 문제 4: Enemy가 목표를 인식하지 못하고 가만히 멈춰있는 현상 발생**

**원인:** `PawnSensingComponent`가 비활성화되어 있거나, `OnSeePawn()` 이벤트가 정상 바인딩되지 않음

**해결:**

1. `BeginPlay()`에서 `PawnSensingComponent`가 활성화되어 있는지 확인 (`bEnableSensingUpdates = true`)
2. `OnSeePawn.AddDynamic(this, &AEnemy::PawnSeen)`이 정상적으로 등록되었는지 디버깅
3. AI가 특정 거리 내에서만 플레이어를 감지하도록 `SightRadius` 및 `PeripheralVisionAngle` 조정
