#  Android SoundTrigger

（2022/07/07-2022/00/00，张道远）

## 一、SoundTrigger简介

> 代码路径：**/frameworks/base/core/java/android/hardware/soundtrigger**
>
> 此路径下主要是一些**类（class）**以及**方法（method）**的实现。

声音触发器（SoundTrigger）能使应用程序以低功耗和隐私敏感的方式监听某些声音事件，

目的是在ADSP上运行DNN来识别唤醒词

利用SoundTrigger主要有两条路径：1）关键词唤醒（hotword）；2）语音交互应用（VIA方案）

【**相关文件**】

1. ***ConversionUtil.java***
1. ***IrecognitionStatusCallback.aidl***
1. ***KeyphraseEnrollmentInfo.java***
1. ***KeyphraseMetadata.java***
1. ***ModelParams.aidl***
1. ***OWNERS***
1. ***SoundTrigger.aidl***
1. ***SoundTrigger.java***
1. ***SoundTriggerMudule.java***

当应用收到具体的触发事件后，app可以通过音频框架（audio framework）中实例化的Audio Record对象来获得触发事件（trigger event）时间点附近的音频数据流（audio stream）。



## 二、SoundTrigger软件堆栈

> **声音触发器子系统按层构建**

![](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202207071550393.png)



### 2.1 SoundTrigger HAL Interface

> 代码路径：**/hardware/interfaces/soundtrigger/2.3/**（共有2.0~2.3四个版本）

SoundTrigger HAL Interface（**STHAL接口**）是SoundTrigger软件堆栈的**供应商**特定部分。它处理**启动指令**和其他声音的**硬件识别**。

STHAL提供一个或多个引擎，每个引擎运行不同的算法，旨在检测特定类别的声音。当STHAL检测到触发器时，它会向框架层发送一个事件，然后停止检测。

【**相关文件**】

1. ***ISoundTriggerHw.hal***（`patform/hardware/interfaces/soundtrigger/2.3`）
2. ***SoundTriggerHw.cpp***（`patform/hardware/interfaces/soundtrigger/2.3/default`）
3. ***SoundTriggerHw.h***（`patform/hardware/interfaces/soundtrigger/2.3/default`）

`ISpundTriggerHw`接口支持在给定时间内运行一个或多个检测会话并侦听声学事件的能力。



### 2.2 SoundTriggerMiddleware

**中间件**位于HAL接口之上，与HAL通信并负责在不同的客户端之间共享HAL、记录、强制执行权限以及处理与旧HAL版本的兼容性等功能。

> 代码路径：**frameworks/base/services/core/java/com/android/server/soundtrigger_middleware/**

【相关文件】

1. ***SoundTriggerHw2Enforcer.java***
2. ***SoundTriggerModule.java***
3. ***SoundTriggerMiddlewareLogging.java***
4. ***SoundTriggerMiddlewareValidation.java***



### 2.3 SoundTriggerService

位于中间件之上，有助于其他系统功能集成，例如电话和电池事件。它还维护一个由**唯一ID索引**的**声音模型数据库**。

> 代码路径：
>
> 1. **frameworks/base/core/java/com/android/internal/apps/**
>2. **frameworks/base/services/voiceinteraction/java/com/android/server/soundtrigger/**

【相关文件】

1. ***ISoundTriggerService.aidl***（路径1）
1. ***SoundTriggerService.java***（路径2）
1. ***SoundTriggerHelper.java***（路径2）



### 2.4 SoundTriggerManager

OK Google语音唤醒属于左半部具有Assistant APP的唤醒方案，不需要调用SoundTriggerManager。



### 2.5 VoiceInteractionManagerService

灭屏唤醒的log：

![](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202207251537685.png)



### 2.6 VoiceInteractionService

* **==VIA==**

    **语音交互应用**（**Voice Interactive Application，VIA**）是充当语音控件的Android应用（也称为“助理”：Assistant）。可以通过在此类应用的清单中添加**`VoiceInteractionServer`**进行标识。每次只能将一个应用设为系统中的默认应用。只有默认应用处于活动状态（与系统服务绑定），且是按住开始讲话（PTT）或点按后开始说话（TTT）事件的接收方。

灭屏唤醒log：

![](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202207251538266.png)



## 三、日志解读

### 3.1 模型训练





### 3.2 灭屏唤醒

<img src="https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202207201921556.png"  />

#### SoundTriggerHw2Enforcer

包括名为SoundTriggerHw2Enforcer的class与method

* 源码：***SoundTriggerHw2Enforcer.java***
* 路径：frameworks/base/services/core/java/com/android/server/**soundtrigger_middleware**/

```java
        public void phraseRecognitionCallback(ISoundTriggerHwCallback.PhraseRecognitionEvent event,
                int cookie) {
            int model = event.common.header.model;
            synchronized (mModelStates) {
                if (!mModelStates.getOrDefault(model, false) &&
                event.common.header.status != RecognitionStatus.FIRST_SUCCESS &&
                event.common.header.status != RecognitionStatus.SS_CNN_FAILURE &&
                event.common.header.status != RecognitionStatus.SS_VOP_FAILURE &&
                event.common.header.status != RecognitionStatus.SS_ALL_FAILURE) {
                    Log.wtfStack(TAG, "Unexpected recognition event for model: " + model);
                    rebootHal();
                    return;
                }
                int memoryToByteArrayLength = 0;
                if (HidlMemoryUtil.hidlMemoryToByteArray(event.common.data) != null) {
                    memoryToByteArrayLength
                      = HidlMemoryUtil.hidlMemoryToByteArray(event.common.data).length;
                }
                if (event.common.header.status !=
                       android.media.soundtrigger_middleware.RecognitionStatus.FORCED &&
                       event.common.header.status != RecognitionStatus.FIRST_SUCCESS &&
                       event.common.header.status != RecognitionStatus.SS_CNN_FAILURE &&
                       event.common.header.status != RecognitionStatus.SS_VOP_FAILURE &&
                       event.common.header.status != RecognitionStatus.SS_ALL_FAILURE &&
                       (memoryToByteArrayLength == 0 ||
                             HidlMemoryUtil.hidlMemoryToByteArray(event.common.data)[0] != 1)) {
                             Log.i(TAG, "[phraseRecognitionCallback] ModelStates.replace ");
                    mModelStates.replace(model, false);
                }
            }
            mUnderlying.phraseRecognitionCallback(event, cookie);
        }
```



#### SoundtriggerModule

* 路径：
    1. **frameworks/base/services/core/java/com/android/server/soundtrigger_middleware/**
    2. frameworks/base/core/java/android/hardware/soundtrigger/
* 源码：
    1. ***SoundTriggerModule.java***
    2. SoundTriggerModule.java（未找到关键log：`ModelState.LOADED`）

原生代码：

```java
public void phraseRecognitionCallback(
                    @NonNull ISoundTriggerHwCallback.PhraseRecognitionEvent phraseRecognitionEvent,
                    int cookie) {
                PhraseRecognitionEvent aidlEvent =
                        ConversionUtil.hidl2aidlPhraseRecognitionEvent(phraseRecognitionEvent);
                aidlEvent.common.captureSession = mSession.mSessionHandle;

                synchronized (SoundTriggerModule.this) {
                    if (aidlEvent.common.status != RecognitionStatus.FORCED) {
                        setState(ModelState.LOADED);
                    }
                    //此处加入定制化代码，if语句块
                }

                // The callback must be invoked outside of the lock.
                try {
                    mCallback.onPhraseRecognition(mHandle, aidlEvent);
                } catch (RemoteException e) {
                    // We're not expecting any exceptions here.
                    throw e.rethrowAsRuntimeException();
                }
            }
```

定制化代码：

```java
//在函数phraseRecognitionCallback中
if (aidlEvent.common.status != RecognitionStatus.FORCED &&
			aidlEvent.common.status != RecognitionStatus.FIRST_SUCCESS &&
                        aidlEvent.common.status != RecognitionStatus.SS_CNN_FAILURE &&
                        aidlEvent.common.status != RecognitionStatus.SS_VOP_FAILURE &&
                        aidlEvent.common.status != RecognitionStatus.SS_ALL_FAILURE &&
                          (aidlEventCommonDataLength == 0 ||
                          (aidlEventCommonDataLength > 0 && aidlEvent.common.data[0] != 1))) {
                        Log.i(TAG, "[phraseRecognitionCallback] ModelState.LOADED ");
                        setState(ModelState.LOADED);
                    }
```



#### SoundTriggerHelper

* 路径：frameworks/base/services/voiceinteraction/java/com/android/**server**/soundtrigger/
* 源码：***SoundTriggerHelper.java***

> 主要输出log：
>
> 1. "Recognition success"
> 2. "startRecognition successful"
> 3. "Recognition aborted"

源码：

```java
private void onKeyphraseRecognitionSuccessLocked(KeyphraseRecognitionEvent event) {
        Slog.i(TAG, "Recognition success");
        MetricsLogger.count(mContext, "sth_keyphrase_recognition_event", 1);
        int keyphraseId = getKeyphraseIdFromEvent(event);
        ModelData modelData = getKeyphraseModelDataLocked(keyphraseId);

        if (modelData == null || !modelData.isKeyphraseModel()) {
            Slog.e(TAG, "Keyphase model data does not exist for ID:" + keyphraseId);
            return;
        }

        if (modelData.getCallback() == null) {
            Slog.w(TAG, "Received onRecognition event without callback for keyphrase model.");
            return;
        }

        if (event.status != SoundTrigger.RECOGNITION_STATUS_GET_STATE_RESPONSE) {
            modelData.setStopped();
        }

        try {
            modelData.getCallback().onKeyphraseDetected((KeyphraseRecognitionEvent) event);
        } catch (DeadObjectException e) {
            forceStopAndUnloadModelLocked(modelData, e);
            return;
        } catch (RemoteException e) {
            Slog.w(TAG, "RemoteException in onKeyphraseDetected", e);
        }

        RecognitionConfig config = modelData.getRecognitionConfig();
        if (config != null) {
            // Whether we should continue by starting this again.
            modelData.setRequested(config.allowMultipleTriggers);
        }
        // TODO: Remove this block if the lower layer supports multiple triggers.
        if (modelData.isRequested()) {
            updateRecognitionLocked(modelData, true);
        }
    }
```



#### SoundTriggerMiddlewareLogging

* 路径：frameworks/base/services/core/java/com/android/server/soundtrigger_middleware/
* 源文件：***SoundTriggerMiddlewareLogging.java***

> 输出log："onPhraseRecognition[...]"

```java
 public void onPhraseRecognition(int modelHandle, PhraseRecognitionEvent event)
                    throws RemoteException {
                try {
                    mCallbackDelegate.onPhraseRecognition(modelHandle, event);
                    logVoidReturn("onPhraseRecognition", modelHandle, event);
                } catch (Exception e) {
                    logException("onPhraseRecognition", e, modelHandle, event);
                    throw e;
                }
            }
```



### 3.3 唤醒后询问天气

![](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202207201941828.png)



###### 

## 四、关键函数

### 4.1 StartRecognition

【路径】frameworks/base/services/core/java/com/android/server/soundtrigger_middleware/SoundTriggerMiddlewareValidation.java

灭屏唤醒OK Google：

![](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202207261500267.png)

