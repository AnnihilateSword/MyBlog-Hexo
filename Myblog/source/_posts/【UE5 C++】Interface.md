---
title: 【UE5 C++】Interface
tags: [UE5, C++]
cover: /res/img/post/【UE5 C++】Interface/cover.jpg
top_img: /res/img/post/【UE5 C++】Interface/cover.jpg
date: 2024-02-25 22:00:00
---

<iframe src="https://player.bilibili.com/player.html?aid=1601182894&bvid=BV1R2421T7be&cid=1451083280&p=1&high_quality=1&autoplay=0" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

# 前言

1. 使用接口一般做什么？

    例如我们有一个拾取交互系统，我们可以拾取水，食物，或者弹药，但是拾取水的时候我们可能希望减少我们的饥渴度或者存储到库存中，拾取到食物可能希望检视饥饿或者存储到库存，拾取到弹药可能希望存储起来，一般不会吃弹药，接口给每个继承自接口的类强制实现了拾取函数，但每个继承了该接口的类又可以重写该函数，以实现不同的行为。

2. 为什么使用接口？

    其实不适用接口也能实现相同的功能，但是使用接口有很多有点：
    优点：在开发中利于代码的维护，有利于代码的规范性，保密性等。

3. 为什么不使用基本的C++类？

    C++ 默认情况下没有接口的概念，一般使用纯虚函数，
    Unreal Engine 给我们提供了接口的概念。


# 在 C++ 中声明接口

## 新建 InteractInterface 类

<img src="/res/img/post/【UE5 C++】Interface/1.png" width=800px>

<img src="/res/img/post/【UE5 C++】Interface/2.png" width=800px>

在 C++ 中声明接口与声明普通的 Unreal 类类似。但是，存在一些主要差异：

- 接口类使用 `UINTERFACE` 宏而不是 `UCLASS` 宏
- 接口类继承自 `UInterface` 而不是 `UObject`

> UINTERFACE 类不是实际的接口，而是一个为反射系统提供可见性的空类。

## C++ 声明示例

`InteractInterface.h`

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "InteractInterface.generated.h"

// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class UInteractInterface : public UInterface
{
	GENERATED_BODY()
};

/**
 * 
 */
class INTERFACE_API IInteractInterface
{
	GENERATED_BODY()

	// Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
	// 声明一个虚函数 Interact
	virtual void Interact();
};
```

可以添加一个空实现

`InteractInterface.cpp`

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "Interfaces/InteractInterface.h"

// Add default functionality here for any IInteractInterface functions that are not pure virtual.
void IInteractInterface::Interact()
{
}
```

## 创建一个测试用的 Actor 类

<img src="/res/img/post/【UE5 C++】Interface/3.png" width=800px>

<img src="/res/img/post/【UE5 C++】Interface/4.png" width=800px>

# 在 C++ 实现接口

`TestActorThatPrintLog.h`

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Interfaces/InteractInterface.h"
#include "TestActorThatPrintLog.generated.h"

UCLASS()
class INTERFACE_API ATestActorThatPrintLog : public AActor, public IInteractInterface
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ATestActorThatPrintLog();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// 实现接口函数
	virtual void Interact() override;
};
```

`TestActorThatPrintLog.cpp`

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "Test/TestActorThatPrintLog.h"

// Sets default values
ATestActorThatPrintLog::ATestActorThatPrintLog()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

}

// Called when the game starts or when spawned
void ATestActorThatPrintLog::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void ATestActorThatPrintLog::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void ATestActorThatPrintLog::Interact()
{
	UE_LOG(LogTemp, Warning, TEXT("Interact Actor!!!"));
}
```

在 `InterfaceCharacter.h` 中添加实现，我这里项目名称是 `Interface`，项目不同名称会有所不同

<img src="/res/img/post/【UE5 C++】Interface/5.png" width=800px>

添加实现

```cpp
void AInterfaceCharacter::InteractLineTrace()
{
	FVector Start = GetFollowCamera()->GetComponentLocation();
	FVector End = Start + GetFollowCamera()->GetComponentRotation().Vector() * 800.0f;

	FHitResult HitResult;
	FCollisionQueryParams CollisionQueryParams;
	CollisionQueryParams.AddIgnoredActor(this);
	if (GetWorld()->LineTraceSingleByChannel(HitResult, Start, End, ECC_Visibility, CollisionQueryParams))
	{
		if (IInteractInterface* InteractInterface = Cast<IInteractInterface>(HitResult.GetActor()))
		{
			InteractInterface->Interact();
		}
	}
    DrawDebugLine(GetWorld(), Start, End, FColor::Red, false, 3.0f, 0, 2.0f);
}
```

创建蓝图类

<img src="/res/img/post/【UE5 C++】Interface/6.png" width=800px>

添加网格体组件

<img src="/res/img/post/【UE5 C++】Interface/7.png" width=800px>

角色蓝图中调用 `InteractLineTrace` 函数用作测试

<img src="/res/img/post/【UE5 C++】Interface/8.png" width=800px>

击中打印效果

<img src="/res/img/post/【UE5 C++】Interface/9.png" width=800px>


# 接口说明符

|接口说明符|描述|
|-|-|
|`Blueprintable`|公开此接口，以便可以通过蓝图实现它。如果接口包含 `BlueprintImplementableEvent` 和 `BlueprintNativeEvent` 函数以外的任何内容，则无法向蓝图公开。使用 `NotBlueprintable` 或 `meta=(CannotImplementInterfaceInBlueprint)` 指定接口在蓝图中实现不安全。|
|`BlueprintType`|将此类公开为可用于蓝图中变量的类型。|
|`DependsOn=(ClassName1, ClassName2, ...)`|构建系统在编译此类之前会编译使用此说明符列出的所有类。 `ClassName` 必须指定同一（或先前）包中的类。您可以使用以逗号分隔的单个 `DependsOn` 行指定多个依赖项类，也可以为每个类使用单独的 `DependsOn` 行指定。|
|`MinimalAPI`|导致仅导出类的类型信息以供其他模块使用。可以转换为该类，但不能调用该类的函数，内联方法除外。通过避免导出不需要在其他模块中访问其所有函数的类的所有内容，可以缩短编译时间。|

# 蓝图可调用接口函数

要制作蓝图可调用接口函数，您必须执行以下操作：

- 使用 `BlueprintCallable` 说明符在函数声明中指定 `UFUNCTION` 宏
- 使用 `BlueprintImplementableEvent` 或 `BlueprintNativeEvent` 说明符

> 蓝图可调用接口函数不能是虚拟的。

可以使用对实现接口的对象的引用在 C++ 或蓝图中调用带有 `BlueprintCallable` 说明符的函数。

> 如果您的蓝图可调用函数不返回值，虚幻引擎会将您的函数视为蓝图中的事件。

# Blueprint Implementable Event

带有 `BlueprintImplementableEvent` 说明符的函数不能在 C++ 中被覆盖，但可以在任何实现或继承您的接口的 Blueprint 类中被覆盖。以下是 `BlueprintImplementableEvent` 的 C++ 接口声明示例：

```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "InteractInterface.generated.h"

// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class UInteractInterface : public UInterface
{
	GENERATED_BODY()
};

/**
 * 
 */
class INTERFACE_API IInteractInterface
{
	GENERATED_BODY()

	// Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
	UFUNCTION(BlueprintCallable, BlueprintImplementableEvent, Category = "Interface")
	void Interact();
};
```

> 接口不能有实现

<img src="/res/img/post/【UE5 C++】Interface/10.png" width=800px>

因为也不是虚函数了，所以派生类也不会有重写，需要删除

<img src="/res/img/post/【UE5 C++】Interface/11.png" width=800px>

现在 Build 成功，然后运行

这里按 F 调用 Interact 函数时报错

<img src="/res/img/post/【UE5 C++】Interface/12.png" width=800px>

需要换一种方式调用

<img src="/res/img/post/【UE5 C++】Interface/13.png" width=800px>

Build 运行，然后在蓝图类中添加实现

<img src="/res/img/post/【UE5 C++】Interface/14.png" width=800px>

<img src="/res/img/post/【UE5 C++】Interface/15.png" width=800px>

效果如下：

<img src="/res/img/post/【UE5 C++】Interface/16.png" width=800px>


# Blueprint Native Event

带有 `BlueprintNativeEvent` 说明符的函数可以在 C++ 或蓝图中实现。以下是 `BlueprintNativeEvent` 的 C++ 接口声明示例：


<img src="/res/img/post/【UE5 C++】Interface/17.png" width=800px>

<img src="/res/img/post/【UE5 C++】Interface/18.png" width=800px>

```cpp
void ATestActorThatPrintLog::Interact_Implementation()
{
	UE_LOG(LogTemp, Warning, TEXT("BlueprintNativeEvent!!! C++"));
}
```

因为刚才在测试 Actor 蓝图中重写了函数，所以效果如下

<img src="/res/img/post/【UE5 C++】Interface/19.png" width=800px>

删除重写的蓝图事件

<img src="/res/img/post/【UE5 C++】Interface/20.png" width=800px>

也可以 C++ 和 蓝图一起调用，因为我们的蓝图类是从 C++ 类派生出来的

<img src="/res/img/post/【UE5 C++】Interface/21.png" width=800px>

<img src="/res/img/post/【UE5 C++】Interface/22.png" width=800px>

效果如下：

<img src="/res/img/post/【UE5 C++】Interface/23.png" width=800px>

# 判断一个类是否实现了一个接口

为了与实现接口的 C++ 和 Blueprint 类兼容，请使用以下任意函数来确定类是否实现接口：

```cpp
bool bIsImplemented;

/* bIsImplemented is true if OriginalObject implements UReactToTriggerInterface */
bIsImplemented = OriginalObject->GetClass()->ImplementsInterface(UReactToTriggerInterface::StaticClass());

/* bIsImplemented is true if OriginalObject implements UReactToTriggerInterface */
bIsImplemented = OriginalObject->Implements<UReactToTriggerInterface>();

/* ReactingObject is non-null if OriginalObject implements UReactToTriggerInterface in C++ */
IReactToTriggerInterface* ReactingObject = Cast<IReactToTriggerInterface>(OriginalObject);
```

我们可以更改一下代码

<img src="/res/img/post/【UE5 C++】Interface/24.png" width=800px>

这样更简洁了

# 安全存储对象和接口指针

要存储对实现特定接口的对象的引用，可以使用 `TScriptInterface` 。如果您有一个实现接口的对象，则可以按如下方式初始化 `TScriptInterface` ：

```cpp
UMyObject* MyObjectPtr;
TScriptInterface<IMyInterface> MyScriptInterface;

if (MyObjectPtr->Implements<UMyInterface>())
{
    MyScriptInterface = TScriptInterface<IMyInterface>(MyObjectPtr);
}

// MyScriptInterface holds a reference to MyObjectPtr and MyInterfacePtr
```

要检索指向原始对象的指针，请使用 `GetObject` ：

```cpp
UMyObject* MyRetrievedObjectPtr = MyScriptInterface.GetObject();
```

要检索指向原始对象实现的接口的指针，请使用 `GetInterface` ：

```cpp
IMyInterface* MyRetrievedInterfacePtr = MyScriptInterface.GetInterface();
```

# 蓝图可实现接口

如果您希望蓝图实现此接口，则必须使用 `Blueprintable` 元数据说明符。每个接口函数（静态函数除外）都必须是 `BlueprintNativeEvent` 或 `BlueprintImplementableEvent` 。当蓝图实现用 C++ 声明的接口时，它的工作方式类似于蓝图接口资产。这意味着该 Blueprint 类的实例实际上不会包含该接口的 C++ 版本，因此它不能与 `Cast<>` 一起使用。在 C++ 中，只有 `Execute_` 静态包装函数才能正常工作。

# 参考

UE5.3 DOC: https://docs.unrealengine.com/5.3/en-US/interfaces-in-unreal-engine/

Sneaky Kitty Game Dev: https://www.youtube.com/playlist?list=PLnHeglBaPYu_m0ju9UKVJXyDN1Yo5JvyM

The End.