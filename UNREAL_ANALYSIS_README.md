**언리얼 엔진 스파르타 게임 분석 요약: 기본 문법 및 게임 설계**

이 문서는 `SpartaGame` 프로젝트의 소스 코드 분석을 통해 언리얼 엔진의 핵심 C++ 문법과 게임 설계 패턴을 요약합니다.

**1. 언리얼 엔진 핵심 C++ 문법 및 개념**

*   **리플렉션 시스템 매크로**:
    *   `UCLASS()`, `UPROPERTY()`, `UFUNCTION()`, `USTRUCT()`: 언리얼 엔진의 리플렉션 시스템에 클래스, 프로퍼티, 함수, 구조체를 등록하여 에디터에서 접근 가능하게 하거나, 블루프린트에서 사용 가능하게 합니다.
    *   `GENERATED_BODY()`: 각 `UCLASS`, `USTRUCT`, `UINTERFACE` 선언 내부에 필수적으로 포함되어야 하는 매크로로, 언리얼 엔진이 생성하는 보일러플레이트 코드를 삽입합니다.
    *   `SPARTAGAME_API`: 모듈의 가시성을 제어하는 매크로로, 다른 모듈에서 해당 클래스/함수에 접근할 수 있도록 합니다.
*   **기본 클래스**:
    *   `AActor`: 월드에 배치될 수 있는 모든 게임 오브젝트의 기본 클래스입니다. (예: `ABaseItem`, `ASpawnVolume`)
    *   `ACharacter`: 플레이어 또는 AI가 제어하는 캐릭터의 기본 클래스로, 이동, 점프 등 캐릭터 특유의 기능을 내장합니다. (예: `ASpartaCharacter`)
    *   `APlayerController`: 플레이어의 입력을 처리하고, 플레이어의 시점, UI 상호작용 등을 제어하는 클래스입니다. (예: `ASpartaPlayerController`)
    *   `AGameMode`: 게임의 규칙을 정의하는 클래스로, 서버에서만 존재합니다. (예: `ASpartaGameMode`)
    *   `AGameState`: 게임의 전역적인 상태를 관리하고 모든 클라이언트에게 복제하는 클래스입니다. (예: `ASpartaGameState`)
*   **컴포넌트 시스템**:
    *   `USceneComponent`: 액터의 위치, 회전, 스케일을 담당하는 기본 컴포넌트입니다.
    *   `UStaticMeshComponent`: 시각적인 스태틱 메시를 표현하는 컴포넌트입니다.
    *   `USphereComponent`, `UBoxComponent`: 충돌 감지 및 영역 정의에 사용되는 콜리전 컴포넌트입니다.
    *   `USpringArmComponent`, `UCameraComponent`: 캐릭터에 카메라를 부착하고 제어하는 데 사용됩니다.
    *   `UCharacterMovementComponent`: 캐릭터의 이동 로직을 처리하는 컴포넌트입니다.
    *   `CreateDefaultSubobject<T>()`: 액터의 생성자에서 컴포넌트를 생성하고 초기화하는 데 사용됩니다.
    *   `SetupAttachment()`: 컴포넌트를 다른 컴포넌트에 부착하여 계층 구조를 만듭니다.
*   **입력 시스템 (Enhanced Input System)**:
    *   `UInputMappingContext`: 입력 액션과 실제 입력 장치 입력을 연결하는 매핑 컨텍스트입니다.
    *   `UInputAction`: 특정 게임 내 액션(예: "점프", "이동")을 나타냅니다.
    *   `UEnhancedInputLocalPlayerSubsystem`: 로컬 플레이어의 Enhanced Input System 서브시스템입니다.
    *   `FInputActionValue`: 입력 액션의 값을 전달하는 구조체입니다.
    *   `AddMappingContext()`: `UInputMappingContext`를 활성화하여 입력 매핑을 적용합니다.
    *   `SetupPlayerInputComponent()`: 플레이어 입력을 설정하는 함수로, `UInputComponent`를 통해 입력 액션을 함수에 바인딩합니다.
*   **데이터 타입 및 유틸리티**:
    *   `FName`: 언리얼 엔진에서 문자열을 효율적으로 관리하기 위한 타입입니다.
    *   `TSubclassOf<T>`: 특정 `UObject` 클래스 또는 그 자식 클래스만 참조할 수 있도록 제한하는 템플릿 클래스입니다.
    *   `FTimerHandle`: 언리얼 엔진에서 타이머를 관리하는 데 사용되는 구조체입니다.
    *   `UDataTable`: 구조화된 데이터를 관리하는 에셋으로, `FTableRowBase`를 상속받는 `USTRUCT`와 함께 사용됩니다.
    *   `FMath::Clamp()`, `FMath::FRandRange()`, `FMath::RandRange()`: 수학 관련 유틸리티 함수입니다.
    *   `GEngine->AddOnScreenDebugMessage()`: 화면에 디버그 메시지를 출력합니다.
    *   `UE_LOG()`: 언리얼 엔진의 로깅 시스템입니다.
    *   `Cast<T>()`, `IsA<T>()`: 타입 캐스팅 및 타입 확인 함수입니다.
    *   `Super::FunctionName()`: 부모 클래스의 함수를 호출합니다.
    *   `virtual`, `override`: 다형성을 구현하는 데 사용됩니다.
*   **델리게이트**:
    *   `OnComponentBeginOverlap.AddDynamic()`: 컴포넌트의 오버랩 이벤트에 함수를 동적으로 바인딩하는 언리얼 엔진의 델리게이트 시스템입니다.

**2. 게임 설계 패턴 및 원칙**

*   **컴포넌트 기반 설계**:
    *   액터의 기능을 여러 컴포넌트로 분리하여 모듈화하고 재사용성을 높입니다. (예: `ASpartaCharacter`의 `SpringArm`, `Camera` 컴포넌트)
*   **상속을 통한 재사용성 및 확장성**:
    *   `ABaseItem`을 부모 클래스로 하여 모든 아이템이 공통 기능을 공유하고, `ACoinItem`, `AHealingItem`, `AMineItem` 등 자식 클래스에서 특정 기능을 확장합니다.
    *   `ACoinItem`을 상속받아 `ABigCoinItem`, `ASmallCoinItem`을 구현하여 코인 종류를 세분화합니다.
*   **인터페이스 기반 설계 (다형성)**:
    *   `IItemInterface`를 정의하여 아이템으로서 가져야 할 필수 동작을 명시하고, 이를 구현하는 클래스들이 해당 함수를 반드시 구현하도록 강제합니다.
    *   이를 통해 다양한 아이템을 일반화된 방식으로 처리할 수 있습니다. (예: `AcitvateItem` 함수)
*   **데이터 주도 설계 (Data-Driven Design)**:
    *   `UDataTable`과 `FItemSpawnRow` 구조체를 사용하여 아이템 스폰 로직을 데이터화하여 관리합니다.
    *   코드를 수정하지 않고도 아이템 스폰 규칙이나 속성을 쉽게 변경하고 밸런싱할 수 있습니다.
*   **모듈화된 게임 시스템**:
    *   `AGameMode`, `AGameState`, `ACharacter`, `APlayerController`, `ABaseItem`, `ASpawnVolume` 등 각 클래스가 명확한 역할을 가지고 상호작용합니다.
    *   `ASpartaGameMode`는 게임의 기본 클래스를 설정하고, `ASpartaGameState`는 게임의 전역 상태를 관리하며, `ASpawnVolume`은 아이템 스폰을 담당합니다.
*   **타이머 기반 이벤트**:
    *   `FTimerHandle`과 `GetWorldTimerManager().SetTimer()`를 사용하여 지뢰의 지연 폭발(`AMineItem`)이나 레벨 시간 제한(`ASpartaGameState`)과 같은 시간 기반 이벤트를 구현합니다.
*   **영역 기반 상호작용/데미지**:
    *   `USphereComponent`나 `UBoxComponent`를 사용하여 특정 영역 내의 액터들을 감지하고 상호작용하거나 데미지를 가합니다. (예: `ABaseItem`의 오버랩, `AMineItem`의 폭발)
*   **블루프린트 연동**:
    *   `UPROPERTY(EditAnywhere, BlueprintReadWrite)` 및 `UFUNCTION(BlueprintCallable, BlueprintPure)` 매크로를 사용하여 C++로 구현된 핵심 로직과 데이터를 언리얼 에디터의 블루프린트에서 쉽게 접근하고 활용할 수 있도록 합니다. 이는 디자이너와 프로그래머 간의 협업을 용이하게 합니다.

이 요약은 `SpartaGame` 프로젝트가 언리얼 엔진의 강력한 기능을 활용하여 모듈화되고 확장 가능한 게임 시스템을 구축하는 방법을 보여줍니다.