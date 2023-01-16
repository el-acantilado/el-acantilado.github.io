---
title: "UE C++"
description: "Unreal Engine programming C++"
lead: "Unreal Engine programming C++"
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu:
  docs:
    parent: "unreal"
weight: 40
toc: true
---

## General information

Crash course: [Converting Blueprints to C++](https://www.unrealengine.com/en-US/onlinelearning-courses/converting-blueprints-to-c).
Documentation: [Gameplay Architecture](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Reference/index.html).

* [UPROPERTY](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Reference/Properties/index.html)
* [UFUNCTION](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Reference/Functions/index.html)
  * BLUEPRINT PURE / CALLABLE
  * BLUEPRINT IMPLEMENTABLE EVENT
  * BLUEPRINT NATIVE EVENT
* [DELEGATES](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Delegates/index.html)


### Common includes

|Class                | #include                   |
|---------------------|----------------------------|
|UWorld               |Engine/World.h              |
|AActor               |GameFramework/Actor.h       |
|UActorComponent      |Components/ActorComponent.h |
|USceneComponent      |Components/SceneComponent.h |
|UGameplayStatics     |Engine/World.h              |
|UKismetSystemLibrary |Kismet/GameplayStatics.h    |
|UKismetMathLibrary*  | Kismet/KismetMathLibrary.h |
|FMath                |Math/UnrealMathUtility.h    |
|UPhysicsHandleComponent | PhysicsEngine/PhysicsHandleComponent.h |

* (*) KismetMath is usually a wrapper to FMath.

### Mapping types

|Blueprint            | C++                        |
|---------------------|----------------------------|
|String               |FString                     |
|Name                 |FName                       |
|Vector               |FVector                     |
|Rotator              |FRotator                    |
|Transform            |FTransform                  |
|Object               |UObject*                    |
|Actor                |AActor*                     |
|ActorComponent       |UActorComponent*            |
|Float                |float                       |
|-                    |double                      |
|Integer              |int32                       |
|-                    |uint32                      |
|Integer64            |int64                       |
|-                    |uint64                      |
|Bool                 |bool                        |

### Properties Visibility

* Blueprint:
  * BlueprintReadonly
  * BlueprintReadWrite
* Defaults and instance:
  * VisibleAnywhere
  * EditAnywhere
* Only defaults:
  * VisibleDefaultsOnly
  * EditDefaultsOnly
* Only instance:
  * VisibleInstanceOnly
  * EditInstanceOnly


### Tips

* You can create C++ files either manually or using the UE editor, first case requires to regenerate project files (left click on .uproject Generate Visual Studio Project Files).
* After syncing files with Git, it's a good idea to regenerate project files.
* If Hot reloading doesn't seem to take your changes, close editor and Rebuild project.
* How to found BP nodes? Put your mouse over the node, see target and try to lookup for the file and lastly search for some keyword on the function name. For example:

  ![UE find Blueprint nodes on C++](https://equilaterus.github.io/wikilaterus/assets/img/ue4/search-blueprint-code-cpp.png)

### Useful tools

* Recommended IDE: [Rider for UE](https://www.jetbrains.com/es-es/lp/rider-unreal/).
* Recommended Git App: [Github Desktop](https://desktop.github.com/).
* PlasticSCM.

## Examples

### Struct template

Sample UE C++ Struct (note F prefix):

```cpp
// MyStruct.h
#pragma once
#include "MyStruct.generated.h"

USTRUCT(BlueprintType)
struct FMyStruct
{
  GENERATED_BODY()

  UPROPERTY(BlueprintReadOnly)
  AActor* Character;

  FMyStruct();
};

// MyStruct.cpp
FMyStruct::FMyStruct()
{
  Character = nullptr;
}

```

### Enum template

Sample UE C++ enum.

```cpp
#pragma once
#include "Enums.generated.h"

UENUM(BlueprintType)
enum class EDirection : uint8
{
    Left UMETA(DisplayName = "Left"),
    Right UMETA(DisplayName = "Right"),
    Front UMETA(DisplayName = "Front"),
    Back UMETA(DisplayName = "Back")
};

// You can add more enums here...
```

### Interfaces

Sample interface:

```cpp
#pragma once

#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "MyInterface.generated.h"

// This class does not need to be modified.
UINTERFACE()
class UMyInterface : public UInterface
{
  GENERATED_BODY()
};

// Your interface
class YOURPROJECT_API IMyInterface
{
  GENERATED_BODY()

public:
  UFUNCTION(BlueprintNativeEvent)
  void YourMethod();
};
```

To use it:

```cpp
// Class that implements the interface
UCLASS()
class YOURPROJECT_API AMyCharacter : public AActor, public IMyInterface
{
  GENERATED_BODY()
public:
  virtual void YourMethod_Implementation() override;
}


// To use the interface from another function
if (AnActor->Implements<UMyInterface>())
{
  IOHActionEntity::Execute_YourMethod(AnActor);
}
```

More info [here](https://docs.unrealengine.com/en-US/ProgrammingAndScripting/GameplayArchitecture/Interfaces/index.html).


### Delegate samples

```cpp
// Declaration
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FCompletedQuestSignature, int32, index)

UPROPERTY(BlueprintAssignable, BlueprintCallable)
FCompletedQuestSignature CompletedQuest;

// Broadcast
CompletedQuest.Broadcast(parameter);

// Subscribe
UFUNCTION() // Required for dynamic
void FunctionHandler(int32 index)

CompletedQuest.AddDynamic(this, &Class::FunctionHandler);
CompletedQuest.Add // For non dynamic version

```

*NOTE:* Dynamic delegates can be serialized, their functions can be found by name, and they are slower than regular delegates. [More info](https://docs.unrealengine.com/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Dynamic/index.html)


### Vector samples

Forward unit vector from a given rotation.

```cpp
GetComponentRotation().Vector()
```

Calculate vector of size *MaxDistance* starting at actor/component location on the same rotation as the actor/component.

```cpp
FVector UClass::GetMaxGrabLocation() const
{
  // GetComponentLocation or GetLocation depending if you're on an Actor or Component
  // same with Rotation
  return GetComponentLocation() + GetComponentRotation().Vector() * MaxDistance;
}
```


### Linetracing samples

Complete custom sample:

```cpp
// Single by channel
FHitResult HitResult;
if (GetWorld()->LineTraceSingleByChannel(HitResult, Start, End, ECC_Visibility))
{
    // Handle hit!
}

// Linetrace for ECC_PhysicsBody or ECC_WorldDynamic
FHitResult OutHit;
if(GetWorld()->LineTraceSingleByObjectType(
   OutHit, Start, End, FCollisionObjectQueryParams(ECC_TO_BITFIELD(ECollisionChannel::ECC_PhysicsBody) | ECC_TO_BITFIELD(ECollisionChannel::ECC_WorldDynamic)))) {
    // Handle hit!
}
```

Using Kismet (you can open its implementation to see more samples):

```cpp
// LineTraceSingle or LineTraceMulti
if (UKismetSystemLibrary::LineTraceSingle(
    WorldContextObject, StartLocation, EndLocation,
    UEngineTypes::ConvertToTraceType(ECC_Visibility), false, TArray<AActor*>(),
    EDrawDebugTrace::ForOneFrame, OutHitResult, true))
{
    // Handle hit(s)!
}
```

### Sweep samples

Complete custom sample:

```cpp
// Params with ignore self
FCollisionQueryParams TraceParams(FName("SphereTrace"), false, GetOwner());
FHitResult OutHit;

// Sweep for ECC_PhysicsBody or ECC_WorldDynamic (SweepSingle or SweepMulti)
if (GetWorld()->SweepSingleByObjectType(
    OutHit, GetComponentLocation(), GetMaxGrabLocation(), FQuat::Identity,
    FCollisionObjectQueryParams(ECC_TO_BITFIELD(ECollisionChannel::ECC_PhysicsBody) | ECC_TO_BITFIELD(ECollisionChannel::ECC_WorldDynamic)),
    FCollisionShape::MakeSphere(GrabRadius), TraceParams))
{
    // Handle hit!
}
```

Using Kismet (you can open its implementation to see more samples):

```cpp
// SphereTraceSingle or SphereTraceMulti
if (UKismetSystemLibrary::SphereTraceMulti(this, CameraComponent->GetComponentLocation(), GetActorLocation(),
    Radius, UEngineTypes::ConvertToTraceType(ECC_Visibility), false,
    TArray<AActor*>(), EDrawDebugTrace::ForOneFrame, OutHits, true)
{
    // Handle hit(s)!
}
```

### Anim notifiers C++

```cpp
 UCLASS()
 class YOURPROJECT_API UAnimNotify_Name : public UAnimNotify
 {
     GENERATED_BODY()

 protected:

     virtual void Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation) override;
 };

 void UAnimNotify_Name::Notify(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation)
 {
     Super::Notify(MeshComp, Animation);

     // YOUR CODE GOES HERE
 }
```

[More info](https://dawnarc.com/2016/10/UEanimation-notifications-animnotify/).

### Anim instance C++

* UE Animation system:

```cpp
UCLASS()
class YOURPROJECT_API UCharacterAnimInstance : public UAnimInstance
{
  GENERATED_BODY()

public:
  UCharacterAnimInstance();

  UPROPERTY(EditDefaultsOnly)
  float MinWalkSpeed;
}
```

* Pixel2d:

```cpp
UCLASS()
class YOURPROJECT_API UCharacterAnimInstance : public UPixel2DAnimInstance
{
  GENERATED_BODY()

public:
  UCharacterAnimInstance();

  UPROPERTY(EditDefaultsOnly)
  float MinWalkSpeed;
}
```

### BoxCollider

```cpp
// .h
class CLASS_API AMyClass : public AActor
{
....
private:
  UPROPERTY(BlueprintReadOnly, meta = (AllowPrivateAccess = "true"))
  UBoxComponent* BoxCollider;
protected:
  virtual void BeginPlay() override;
public:
  UFUNCTION()
  void BeginOverlapBoxCollider(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult & SweepResult);

  UFUNCTION()
  void EndOverlapBoxCollider(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);

...
}

// .cpp
AMyClass::AMyClass()
{
  BoxCollider= CreateDefaultSubobject<UBoxComponent>(TEXT("BoxCollider"));
  BoxCollider->SetCollisionProfileName(TEXT("Custom"));
  // TODO: Set channel (if nedeed): BoxCollision->SetCollisionObjectType(ECC_Visibility);
  // TODO: Set size:
  BoxCollider->SetBoxExtent(FVector(10, 10, 10));
  BoxCollider->SetRelativeTransform(FTransform(FRotator::ZeroRotator, FVector(0, 0, 10), FVector(1, 1, 1)));
  // TODO: Customize collision config
  BoxCollider->SetCollisionResponseToAllChannels(ECR_Ignore);
  BoxCollider->SetCollisionEnabled(ECollisionEnabled::NoCollision);
  BoxCollider->SetCollisionResponseToChannel(ECC_Pawn, ECR_Overlap);
  BoxCollider->SetCollisionResponseToChannel(ECC_Visibility, ECR_Overlap);
  BoxCollider->SetGenerateOverlapEvents(true);
  BoxCollider->SetupAttachment(RootComponent);
}

void AMyClass::BeginPlay()
{
  Super::BeginPlay();

  BoxCollider->OnComponentBeginOverlap.AddDynamic(this, &AMyClass::BeginOverlapBoxCollider);
  BoxCollider->OnComponentEndOverlap.AddDynamic(this, &AMyClass::EndOverlapBoxCollider);
}

void AMyClass::BeginOverlapBoxCollider(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
  UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
  // YOUR CODE GOES HERE
}

void AMyClass::EndOverlapBoxCollider(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
  UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
  // YOUR CODE GOES HERE
}

```

### Billboards (editor icons)

```cpp
// .h
class CLASS_API AMyClass : public AActor
{
....
private:
#if WITH_EDITORONLY_DATA
  // Editor Billboard
  UPROPERTY()
  UBillboardComponent* BillboardComponent;
#endif
....

// .cpp
AMyClass::AMyClass()
{
  USceneComponent* SceneComponent = CreateDefaultSubobject<USceneComponent>(TEXT("SceneComp"));
  RootComponent = SceneComponent;
  RootComponent->Mobility = EComponentMobility::Static; // Or movable

  #if WITH_EDITORONLY_DATA
  BillboardComponent = CreateEditorOnlyDefaultSubobject<UBillboardComponent>(TEXT("Sprite"));
  if (BillboardComponent)
  {
  BillboardComponent->SetWorldTransform(
    FTransform(FRotator::ZeroRotator, FVector::ZeroVector, FVector::OneVector * 4)
  );
  BillboardComponent->Sprite = ConstructorHelpers::FObjectFinderOptional<UTexture2D>(TEXT("/PATH/TO-TEXTURE")).Get();
    BillboardComponent->SetupAttachment(RootComponent);
  }
  #endif
}
```
