---
title: UE5 C++ 加载 Json 文件
tags: [UE5, C++]
cover: /res/img/post/UE5 C++ 加载 Json 文件/1.png
top_img: /res/img/site/samurai.png
date: 2024-02-21 00:00:00
---

<iframe src="https://player.bilibili.com/player.html?aid=1900966805&bvid=BV1pm411S7Hn&cid=1447927173&p=1&high_quality=1&autoplay=0" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

# 添加 Json 模块

<img src="/res/img/post/UE5 C++ 加载 Json 文件/1.png" width=800px>

# 加载并读取 Json 文件

1. 包含头文件

```cpp
#include "Serialization/JsonReader.h"
#include "Dom/JsonObject.h"
#include "Serialization/JsonSerializer.h"
#include "Misc/FileHelper.h"
```

2. 创建函数 `LoadStringFromFile`，用来读取文件中的字符串

```cpp
bool ARPGCharacter::LoadStringFromFile(const FString& FileName, const FString& RelativePath, FString& ResultString)
{
	if (!FileName.IsEmpty())
	{
		// FPaths::ProjectContentDir() 是 Content 目录下
		FString AbsolutePath = FPaths::ProjectContentDir() + RelativePath + FileName;
		if (FPaths::FileExists(AbsolutePath))
		{
			if (FFileHelper::LoadFileToString(ResultString, *AbsolutePath))
			{
				return true;
			}
			else
			{
				UE_LOG(LogTemp, Warning, TEXT("Load Error!"));
			}
		}
		else
		{
			UE_LOG(LogTemp, Warning, TEXT("File Not Exist!"));
		}
	}
	return false;
}
```

3. 读取 Json 文件示例

```cpp
// Called when the game starts or when spawned
void ARPGCharacter::BeginPlay()
{
	Super::BeginPlay();

	FString jsonStr;
	// 加载 Json 文件
	if (LoadStringFromFile("Player.json", "../LoadJsonFileTestConfig/", jsonStr))
	{
		// 创建 Json 阅读器
		TSharedRef<TJsonReader<>> jsonReader = TJsonReaderFactory<TCHAR>::Create(jsonStr);
		// 创建 Json 对象
		TSharedPtr<FJsonObject> jsonObject;
		// 反序列化，将 JsonReader 里的数据，传入 JsonObject 中
		FJsonSerializer::Deserialize(jsonReader, jsonObject);

		// 读取单行数据
		FString timeStr = jsonObject->GetStringField("Time");
		UE_LOG(LogTemp, Warning, TEXT("Time: %s"), *timeStr);

		// 读取数组数据
		TArray<TSharedPtr<FJsonValue>> data = jsonObject->GetArrayField("Data");
		for (int i = 0; i < data.Num(); i++)
		{
			FString dataStr1 = data[i]->AsObject()->GetStringField("key1");
			FString dataStr2 = data[i]->AsObject()->GetStringField("key2");
			UE_LOG(LogTemp, Warning, TEXT("Data:"));
			UE_LOG(LogTemp, Warning, TEXT("key1: %s"), *dataStr1);
			UE_LOG(LogTemp, Warning, TEXT("key2: %s"), *dataStr2);
		}
	}
}
```

4. 示例 Json 文件路径

<img src="/res/img/post/UE5 C++ 加载 Json 文件/2.png" width=800px>

<img src="/res/img/post/UE5 C++ 加载 Json 文件/3.png" width=800px>

`Player.json` 内容如下

```json
{
    "Time":"2024-02-21",
    "Data": [
        {
            "key1": "123",
            "key2": "345"
        },
        {
            "key1": "666",
            "key2": "520"
        }
    ]
}
```

5. 打印效果

<img src="/res/img/post/UE5 C++ 加载 Json 文件/4.png" width=800px>

# 创建并写入 Json 文件

1. 包含头文件

```cpp
#include "Policies/CondensedJsonPrintPolicy.h"
```

2. 新建函数 `SaveStringToFile`，用来保存字符串到文件中

```cpp
bool ARPGCharacter::SaveStringToFile(const FString& FileName, const FString& RelativePath, FString& TargetString)
{
	if (!FileName.IsEmpty())
	{
		// FPaths::ProjectContentDir() 是 Content 目录下
		FString AbsolutePath = FPaths::ProjectContentDir() + RelativePath + FileName;

		if (FFileHelper::SaveStringToFile(TargetString, *AbsolutePath))
		{
			return true;
		}
		else
		{
			UE_LOG(LogTemp, Warning, TEXT("Save Error!"));
		}
	}
	return false;
}
```

3. 写入 Json 文件示例

```cpp
// Called when the game starts or when spawned
void ARPGCharacter::BeginPlay()
{
	Super::BeginPlay();

	FString jsonStr;
	// 创建 Json 写入器
	TSharedRef<TJsonWriter<TCHAR, TCondensedJsonPrintPolicy<TCHAR>>> jsonWritter = TJsonWriterFactory<TCHAR, TCondensedJsonPrintPolicy<TCHAR>>::Create(&jsonStr);

	// 开始写入
	jsonWritter->WriteObjectStart();

	// 写入单行数据
	jsonWritter->WriteValue(TEXT("Time"), TEXT("2024-02-21"));
	// 写入数组数据
	jsonWritter->WriteArrayStart("Data");
	for (int i = 0; i < 2; i++)
	{
		jsonWritter->WriteObjectStart();
		jsonWritter->WriteValue(TEXT("key1"), TEXT("100"));
		jsonWritter->WriteValue(TEXT("key2"), TEXT("200"));
		jsonWritter->WriteObjectEnd();
	}
	jsonWritter->WriteArrayEnd();

	// 停止写入
	jsonWritter->WriteObjectEnd();

	// 关闭写入器
	jsonWritter->Close();
	UE_LOG(LogTemp, Warning, TEXT("%s"), *jsonStr);

	SaveStringToFile("Write.json", "../LoadJsonFileTestConfig/", jsonStr);
}
```

4. 写入效果

<img src="/res/img/post/UE5 C++ 加载 Json 文件/5.png" width=800px>

<img src="/res/img/post/UE5 C++ 加载 Json 文件/6.png" width=800px>

<img src="/res/img/post/UE5 C++ 加载 Json 文件/7.png" width=800px>

The End.