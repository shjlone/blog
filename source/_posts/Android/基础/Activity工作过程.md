---
title: Activity工作过程
tags: Android
toc: true
---


## 概述

Activity是一种展示型组件，具有两种启动方式，一种是显示的，通过intent实现；另一种是隐式的，也需要intent，但还需要在AndroidManifest.xml
中添加intentfilter。在实现Activity时，需要继承Activity抽象类，并且重写onCreat()方法，因此，Activity具有启动和停止的概念。

## 流程分析

### 流程图

![](./1.png)

### Activity的启动

从startActivity开始，代码会运行到Activity的startActivityForResult方法。

```java
Activity.java

public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
        @Nullable Bundle options) {
    if (mParent == null) {
        options = transferSpringboardActivityOptions(options);
        Instrumentation.ActivityResult ar =
            mInstrumentation.execStartActivity(
                this, mMainThread.getApplicationThread(), mToken, this,
                intent, requestCode, options);
        if (ar != null) {
            mMainThread.sendActivityResult(
                mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                ar.getResultData());
        }
        if (requestCode >= 0) {
            // If this start is requesting a result, we can avoid making
            // the activity visible until the result is received.  Setting
            // this code during onCreate(Bundle savedInstanceState) or onResume() will keep the
            // activity hidden during this time, to avoid flickering.
            // This can only be done when a result is requested because
            // that guarantees we will get information back when the
            // activity is finished, no matter what happens to it.
            mStartedActivity = true;
        }
    
        cancelInputsAndStartExitTransition(options);
        // TODO Consider clearing/flushing other event sources and events for child windows.
    } else {
        if (options != null) {
            mParent.startActivityFromChild(this, intent, requestCode, options);
        } else {
            // Note we want to go through this method for compatibility with
            // existing applications that may have overridden it.
            mParent.startActivityFromChild(this, intent, requestCode);
        }
    }
}

```

mParent代表的是ActivityGroup，ActivityGroup最开始被用来在一个界面中嵌入多个子Activity，但是其在API 13中已经被废弃了，
系统推荐采用Fragment来代替ActivityGroup。

mMainThread.getApplicationThread()这个参数，它的类型是ApplicationThread，ApplicationThread是ActivityThread的内部类，
继承IApplicationThread.Stub，也是个Binder对象。

接下来看看execStartActivity方法

```java
Instrumentation.java

@UnsupportedAppUsage
public ActivityResult execStartActivity(
        Context who, IBinder contextThread, IBinder token, Activity target,
        Intent intent, int requestCode, Bundle options) {
    IApplicationThread whoThread = (IApplicationThread) contextThread;
    Uri referrer = target != null ? target.onProvideReferrer() : null;
    if (referrer != null) {
        intent.putExtra(Intent.EXTRA_REFERRER, referrer);
    }
    if (mActivityMonitors != null) {
        synchronized (mSync) {
            final int N = mActivityMonitors.size();
            for (int i=0; i<N; i++) {
                final ActivityMonitor am = mActivityMonitors.get(i);
                ActivityResult result = null;
                if (am.ignoreMatchingSpecificIntents()) {
                    result = am.onStartActivity(intent);
                }
                if (result != null) {
                    am.mHits++;
                    return result;
                } else if (am.match(who, null, intent)) {
                    am.mHits++;
                    if (am.isBlocking()) {
                        return requestCode >= 0 ? am.getResult() : null;
                    }
                    break;
                }
            }
        }
    }
    try {
        intent.migrateExtraStreamToClipData();
        intent.prepareToLeaveProcess(who);
        int result = ActivityTaskManager.getService()
            .startActivity(whoThread, who.getBasePackageName(), intent,
                    intent.resolveTypeIfNeeded(who.getContentResolver()),
                    token, target != null ? target.mEmbeddedID : null,
                    requestCode, 0, null, options);
        //检查Activity启动的结果
        //比如如果没有注册，则会抛出ActivityNotFoundException
        checkStartActivityResult(result, intent);
    } catch (RemoteException e) {
        throw new RuntimeException("Failure from system", e);
    }
    return null;
}

public static void checkStartActivityResult(int res, Object intent) {
    if (!ActivityManager.isStartResultFatalError(res)) {
        return;
    }

    switch (res) {
        case ActivityManager.START_INTENT_NOT_RESOLVED:
        case ActivityManager.START_CLASS_NOT_FOUND:
            if (intent instanceof Intent && ((Intent)intent).getComponent() != null)
                throw new ActivityNotFoundException(
                        "Unable to find explicit activity class "
                        + ((Intent)intent).getComponent().toShortString()
                        + "; have you declared this activity in your AndroidManifest.xml?");
            throw new ActivityNotFoundException(
                    "No Activity found to handle " + intent);
        case ActivityManager.START_PERMISSION_DENIED:
            throw new SecurityException("Not allowed to start activity "
                    + intent);
        case ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT:
            throw new AndroidRuntimeException(
                    "FORWARD_RESULT_FLAG used while also requesting a result");
        case ActivityManager.START_NOT_ACTIVITY:
            throw new IllegalArgumentException(
                    "PendingIntent is not an activity");
        case ActivityManager.START_NOT_VOICE_COMPATIBLE:
            throw new SecurityException(
                    "Starting under voice control not allowed for: " + intent);
        case ActivityManager.START_VOICE_NOT_ACTIVE_SESSION:
            throw new IllegalStateException(
                    "Session calling startVoiceActivity does not match active session");
        case ActivityManager.START_VOICE_HIDDEN_SESSION:
            throw new IllegalStateException(
                    "Cannot start voice activity on a hidden session");
        case ActivityManager.START_ASSISTANT_NOT_ACTIVE_SESSION:
            throw new IllegalStateException(
                    "Session calling startAssistantActivity does not match active session");
        case ActivityManager.START_ASSISTANT_HIDDEN_SESSION:
            throw new IllegalStateException(
                    "Cannot start assistant activity on a hidden session");
        case ActivityManager.START_CANCELED:
            throw new AndroidRuntimeException("Activity could not be started for "
                    + intent);
        default:
            throw new AndroidRuntimeException("Unknown error code "
                    + res + " when starting " + intent);
    }
}


```

这里使用了ActivityTaskManager来启动Activity。ActivityTaskManager是一个Binder。ActivityTaskManager.getService()会返回Activity
服务管理器ActivityManagerService(Android10返回ActivityTaskManagerService)。

> ATMS是在Android10中新增的，分担了之前ActivityManagerService（AMS）的一部分功能（activity task相关）。
在Android10 之前 ，这个地方获取的是服务是AMS。查看Android10的AMS，你会发现startActivity方法内也是调用了ATMS的startActivity方法。
所以在理解上，ATMS就隶属于AMS。

接下来要去ActivityTaskManagerService看看了

### Activity的管理

```java
ActivityTaskManagerService.java

@Override
public final int startActivities(IApplicationThread caller, String callingPackage,
        Intent[] intents, String[] resolvedTypes, IBinder resultTo, Bundle bOptions,
        int userId) {
    final String reason = "startActivities";
    enforceNotIsolatedCaller(reason);
    userId = handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(), userId, reason);
    // TODO: Switch to user app stacks here.
    return getActivityStartController().startActivities(caller, -1, 0, -1, callingPackage,
            intents, resolvedTypes, resultTo, SafeActivityOptions.fromBundle(bOptions), userId,
            reason, null /* originatingPendingIntent */,
            false /* allowBackgroundActivityStart */);
}

@Override
public int startActivityAsUser(IApplicationThread caller, String callingPackage,
        Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
        int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) {
    return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
            resultWho, requestCode, startFlags, profilerInfo, bOptions, userId,
            true /*validateIncomingUser*/);
}

int startActivityAsUser(IApplicationThread caller, String callingPackage,
        Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
        int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId,
        boolean validateIncomingUser) {
    enforceNotIsolatedCaller("startActivityAsUser");

    userId = getActivityStartController().checkTargetUser(userId, validateIncomingUser,
            Binder.getCallingPid(), Binder.getCallingUid(), "startActivityAsUser");

    // TODO: Switch to user app stacks here.
    return getActivityStartController().obtainStarter(intent, "startActivityAsUser")
            .setCaller(caller)
            .setCallingPackage(callingPackage)
            .setResolvedType(resolvedType)
            .setResultTo(resultTo)
            .setResultWho(resultWho)
            .setRequestCode(requestCode)
            .setStartFlags(startFlags)
            .setProfilerInfo(profilerInfo)
            .setActivityOptions(bOptions)
            .setMayWait(userId)
            .execute();

}
```

getActivityStartController().obtainStarter方法获取ActivityStarter实例，进去看看execute。

```java
ActivityStarter.java

/**
 * Starts an activity based on the request parameters provided earlier.
 * @return The starter result.
 */
int execute() {
    try {
        // TODO(b/64750076): Look into passing request directly to these methods to allow
        // for transactional diffs and preprocessing.
        if (mRequest.mayWait) {
        //也会执行到startActivity方法
            return startActivityMayWait(mRequest.caller, mRequest.callingUid,
                    mRequest.callingPackage, mRequest.realCallingPid, mRequest.realCallingUid,
                    mRequest.intent, mRequest.resolvedType,
                    mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                    mRequest.resultWho, mRequest.requestCode, mRequest.startFlags,
                    mRequest.profilerInfo, mRequest.waitResult, mRequest.globalConfig,
                    mRequest.activityOptions, mRequest.ignoreTargetSecurity, mRequest.userId,
                    mRequest.inTask, mRequest.reason,
                    mRequest.allowPendingRemoteAnimationRegistryLookup,
                    mRequest.originatingPendingIntent, mRequest.allowBackgroundActivityStart);
        } else {
            return startActivity(mRequest.caller, mRequest.intent, mRequest.ephemeralIntent,
                    mRequest.resolvedType, mRequest.activityInfo, mRequest.resolveInfo,
                    mRequest.voiceSession, mRequest.voiceInteractor, mRequest.resultTo,
                    mRequest.resultWho, mRequest.requestCode, mRequest.callingPid,
                    mRequest.callingUid, mRequest.callingPackage, mRequest.realCallingPid,
                    mRequest.realCallingUid, mRequest.startFlags, mRequest.activityOptions,
                    mRequest.ignoreTargetSecurity, mRequest.componentSpecified,
                    mRequest.outActivity, mRequest.inTask, mRequest.reason,
                    mRequest.allowPendingRemoteAnimationRegistryLookup,
                    mRequest.originatingPendingIntent, mRequest.allowBackgroundActivityStart);
        }
    } finally {
        onExecutionComplete();
    }
}


private int startActivity(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity, boolean restrictedBgActivity) {
    int result = START_CANCELED;
    final ActivityStack startedActivityStack;
    try {
        mService.mWindowManager.deferSurfaceLayout();
        result = startActivityUnchecked(r, sourceRecord, voiceSession, voiceInteractor,
                startFlags, doResume, options, inTask, outActivity, restrictedBgActivity);
                
...                
}
                    

private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
        IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
        int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
        ActivityRecord[] outActivity, boolean restrictedBgActivity) {
...
if (mDoResume) {
    final ActivityRecord topTaskActivity =
            mStartActivity.getTaskRecord().topRunningActivityLocked();
    if (!mTargetStack.isFocusable()
            || (topTaskActivity != null && topTaskActivity.mTaskOverlay
            && mStartActivity != topTaskActivity)) {
        // If the activity is not focusable, we can't resume it, but still would like to
        // make sure it becomes visible as it starts (this will also trigger entry
        // animation). An example of this are PIP activities.
        // Also, we don't want to resume activities in a task that currently has an overlay
        // as the starting activity just needs to be in the visible paused state until the
        // over is removed.
        mTargetStack.ensureActivitiesVisibleLocked(mStartActivity, 0, !PRESERVE_WINDOWS);
        // Go ahead and tell window manager to execute app transition for this activity
        // since the app transition will not be triggered through the resume channel.
        mTargetStack.getDisplay().mDisplayContent.executeAppTransition();
    } else {
        // If the target stack was not previously focusable (previous top running activity
        // on that stack was not visible) then any prior calls to move the stack to the
        // will not update the focused stack.  If starting the new activity now allows the
        // task stack to be focusable, then ensure that we now update the focused stack
        // accordingly.
        if (mTargetStack.isFocusable()
                && !mRootActivityContainer.isTopDisplayFocusedStack(mTargetStack)) {
            mTargetStack.moveToFront("startActivityUnchecked");
        }
        mRootActivityContainer.resumeFocusedStacksTopActivities(
                mTargetStack, mStartActivity, mOptions);
    }
} else if (mStartActivity != null) {
    mSupervisor.mRecentTasks.add(mStartActivity.getTaskRecord());
}
mRootActivityContainer.updateUserStack(mStartActivity.mUserId, mTargetStack);

...
}
```

startActivityMayWait最终也是会进入startActivity，startActivity调用了startActivityUnchecked，startActivityUnchecked调用了
`mRootActivityContainer.resumeFocusedStacksTopActivities(mTargetStack, mStartActivity, mOptions);`。mRootActivityContainer
是RootActivityContainer，Android10新增到API，分担了ActivityStackSupervisor部分功能。接着看看RootActivityContainer

```java
RootActivityContainer.java
boolean resumeFocusedStacksTopActivities(
        ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {

    if (!mStackSupervisor.readyToResume()) {
        return false;
    }

    boolean result = false;
    if (targetStack != null && (targetStack.isTopStackOnDisplay()
            || getTopDisplayFocusedStack() == targetStack)) {
        result = targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
    }
    ...
    
}
```

接着跳转到了ActivityStack的resumeTopActivityUncheckedLocked方法

```java
ActivityStack.java

@GuardedBy("mService")
boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
    if (mInResumeTopActivity) {
        // Don't even start recursing.
        return false;
    }

    boolean result = false;
    try {
        // Protect against recursion.
        mInResumeTopActivity = true;
        result = resumeTopActivityInnerLocked(prev, options);

        // When resuming the top activity, it may be necessary to pause the top activity (for
        // example, returning to the lock screen. We suppress the normal pause logic in
        // {@link #resumeTopActivityUncheckedLocked}, since the top activity is resumed at the
        // end. We call the {@link ActivityStackSupervisor#checkReadyForSleepLocked} again here
        // to ensure any necessary pause logic occurs. In the case where the Activity will be
        // shown regardless of the lock screen, the call to
        // {@link ActivityStackSupervisor#checkReadyForSleepLocked} is skipped.
        final ActivityRecord next = topRunningActivityLocked(true /* focusableOnly */);
        if (next == null || !next.canTurnScreenOn()) {
            checkReadyForSleep();
        }
    } finally {
        mInResumeTopActivity = false;
    }

    return result;
}

@GuardedBy("mService")
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
...
    boolean pausing = getDisplay().pauseBackStacks(userLeaving, next, false);
    if (mResumedActivity != null) {
        if (DEBUG_STATES) Slog.d(TAG_STATES,
                "resumeTopActivityLocked: Pausing " + mResumedActivity);
         // 暂停上一个Activity
        pausing |= startPausingLocked(userLeaving, false, next, false);
    }
    ...
    //这里next.attachedToProcess()，只有启动了的Activity才会返回true
    if (next.attachedToProcess()) {
        ...
        
        try {
            final ClientTransaction transaction =
                    ClientTransaction.obtain(next.app.getThread(), next.appToken);
            ...
            //启动了的Activity就发送ResumeActivityItem事务给客户端了，后面会讲到
            transaction.setLifecycleStateRequest(
                    ResumeActivityItem.obtain(next.app.getReportedProcState(),
                            getDisplay().mDisplayContent.isNextTransitionForward()));
            mService.getLifecycleManager().scheduleTransaction(transaction);
           ....
        } catch (Exception e) {
            ....
            mStackSupervisor.startSpecificActivityLocked(next, true, false);
            return true;
        }
        ....
    } else {
        ....
        if (SHOW_APP_STARTING_PREVIEW) {
                //这里就是 冷启动时 出现白屏 的原因了：取根Activity的主题背景 展示StartingWindow
                next.showStartingWindow(null , false ,false);
            }
        // 继续当前Activity，普通Activity的正常启动 关注这里即可
        //ActivityStackSupervisor.java
        mStackSupervisor.startSpecificActivityLocked(next, true, true);
    }
    return true;

}

```

接下来到了ActivityStackSupervisor的startSpecificActivityLocked方法

```java
ActivityStackSupervisor.java

void startSpecificActivityLocked(ActivityRecord r, boolean andResume, boolean checkConfig) {
    // Is this activity's application already running?
    final WindowProcessController wpc =
            mService.getProcessController(r.processName, r.info.applicationInfo.uid);

    boolean knownToBeDead = false;
    if (wpc != null && wpc.hasThread()) {
        try {
            realStartActivityLocked(r, wpc, andResume, checkConfig);
            return;
        } catch (RemoteException e) {
            Slog.w(TAG, "Exception when starting activity "
                    + r.intent.getComponent().flattenToShortString(), e);
        }
        knownToBeDead = true;
    }

    ...
    
    try {
        if (Trace.isTagEnabled(TRACE_TAG_ACTIVITY_MANAGER)) {
            Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "dispatchingStartProcess:"
                    + r.processName);
        }
        // 上面的wpc != null && wpc.hasThread()不满足的话，说明没有进程，就会去创建进程
        final Message msg = PooledLambda.obtainMessage(
                ActivityManagerInternal::startProcess, mService.mAmInternal, r.processName,
                r.info.applicationInfo, knownToBeDead, "activity", r.intent.getComponent());
        mService.mH.sendMessage(msg);
    } finally {
        Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
    }
}

boolean realStartActivityLocked(ActivityRecord r, WindowProcessController proc,
        boolean andResume, boolean checkConfig) throws RemoteException {

    ...
    
    // Create activity launch transaction.
    final ClientTransaction clientTransaction = ClientTransaction.obtain(
            proc.getThread(), r.appToken);

    final DisplayContent dc = r.getDisplay().mDisplayContent;
    //添加了启动Activity的Item
    clientTransaction.addCallback(LaunchActivityItem.obtain(new Intent(r.intent),
            System.identityHashCode(r), r.info,
            // TODO: Have this take the merged configuration instead of separate global
            // and override configs.
            mergedConfiguration.getGlobalConfiguration(),
            mergedConfiguration.getOverrideConfiguration(), r.compat,
            r.launchedFromPackage, task.voiceInteractor, proc.getReportedProcState(),
            r.icicle, r.persistentState, results, newIntents,
            dc.isNextTransitionForward(), proc.createProfilerInfoIfNeeded(),
                    r.assistToken));

    // Set desired final state.
    final ActivityLifecycleItem lifecycleItem;
    if (andResume) {
        lifecycleItem = ResumeActivityItem.obtain(dc.isNextTransitionForward());
    } else {
        lifecycleItem = PauseActivityItem.obtain();
    }
    clientTransaction.setLifecycleStateRequest(lifecycleItem);

    // Schedule transaction.
    //ActivityTaskManagerService获取ClientLifecycleManager
    mService.getLifecycleManager().scheduleTransaction(clientTransaction);

    ...

    return true;
}


```

由以上代码可知，ClientTransaction包含一系列的待客户端处理的事务的容器，客户端接收后取出事务并执行。其添加了LaunchActivityItem、ResumeActivityItem等。
然后运行ClientLifecycleManager的scheduleTransaction

```java
ClientLifecycleManager.java

/**
 * Schedule a transaction, which may consist of multiple callbacks and a lifecycle request.
 * @param transaction A sequence of client transaction items.
 * @throws RemoteException
 *
 * @see ClientTransaction
 */
void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
    final IApplicationThread client = transaction.getClient();
    //ClientTransaction
    transaction.schedule();
    if (!(client instanceof Binder)) {
        // If client is not an instance of Binder - it's a remote call and at this point it is
        // safe to recycle the object. All objects used for local calls will be recycled after
        // the transaction is executed on client in ActivityThread.
        transaction.recycle();
    }
}

```

```java
ClientTransaction.java

public void schedule() throws RemoteException {
//ApplicationThread
    mClient.scheduleTransaction(this);
}
```

IApplicationThread是ApplicationThread在系统进程的代理，所以真正执行的地方是客户端的ApplicationThread。

现有流程如下：
启动Activity的操作从客户端跨进程转移到ATMS，ATMS通过ActivityStarter、ActivityStack、ActivityStackSupervisor对Activity任务、
Activity栈、Activity记录管理后，又用过跨进程把正在启动过程又转移到了客户端。

流程图如下：

![](./activity_process_1.jpg)

### 线程切换及消息处理

接下来看看ApplicationThread的scheduleTransaction

scheduleTransaction会发送Message，ActivityThread内部类H处理此消息

```java
//ApplicationThread.java

@Override
public void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
    ActivityThread.this.scheduleTransaction(transaction);
}


//ActivityThread.java

@Override
public void scheduleTransaction(ClientTransaction transaction) throws RemoteException {
//会调用sendMessage
    ActivityThread.this.scheduleTransaction(transaction);
}


private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
    if (DEBUG_MESSAGES) Slog.v(
        TAG, "SCHEDULE " + what + " " + mH.codeToString(what)
        + ": " + arg1 + " / " + obj);
    Message msg = Message.obtain();
    msg.what = what;
    msg.obj = obj;
    msg.arg1 = arg1;
    msg.arg2 = arg2;
    if (async) {
        msg.setAsynchronous(true);
    }
    mH.sendMessage(msg);//对应mH会处理消息，H是ActivityThread的内部类
}

//ClientTransactionHandler.java 是ActivityThread的父类
void scheduleTransaction(ClientTransaction transaction) {
    transaction.preExecute(this);
    sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
}


//
class H extends Handler {
    public void handleMessage(Message msg) {
        if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
                switch (msg.what) {
                ...
                    case EXECUTE_TRANSACTION:
                        final ClientTransaction transaction = (ClientTransaction) msg.obj;
                        //这里处理消息
                        //TransactionExecutor
                        mTransactionExecutor.execute(transaction);
                        if (isSystem()) {
                            // Client transactions inside system process are recycled on the client side
                            // instead of ClientLifecycleManager to avoid being cleared before this
                            // message is handled.
                            transaction.recycle();
                        }
                        // TODO(lifecycler): Recycle locally scheduled transactions.
                        break;
                    case RELAUNCH_ACTIVITY:
                        handleRelaunchActivityLocally((IBinder) msg.obj);
                        break;
                }
    }
}

```

最终到了TransactionExecutor的execute方法

```java
//TransactionExecutor.java

public void execute(ClientTransaction transaction) {
    if (DEBUG_RESOLVER) Slog.d(TAG, tId(transaction) + "Start resolving transaction");

    final IBinder token = transaction.getActivityToken();
    if (token != null) {
        final Map<IBinder, ClientTransactionItem> activitiesToBeDestroyed =
                mTransactionHandler.getActivitiesToBeDestroyed();
        final ClientTransactionItem destroyItem = activitiesToBeDestroyed.get(token);
        if (destroyItem != null) {
            if (transaction.getLifecycleStateRequest() == destroyItem) {
                // It is going to execute the transaction that will destroy activity with the
                // token, so the corresponding to-be-destroyed record can be removed.
                activitiesToBeDestroyed.remove(token);
            }
            if (mTransactionHandler.getActivityClient(token) == null) {
                // The activity has not been created but has been requested to destroy, so all
                // transactions for the token are just like being cancelled.
                Slog.w(TAG, tId(transaction) + "Skip pre-destroyed transaction:\n"
                        + transactionToString(transaction, mTransactionHandler));
                return;
            }
        }
    }

    if (DEBUG_RESOLVER) Slog.d(TAG, transactionToString(transaction, mTransactionHandler));

    executeCallbacks(transaction);

    executeLifecycleState(transaction);
    mPendingActions.clear();
    if (DEBUG_RESOLVER) Slog.d(TAG, tId(transaction) + "End resolving transaction");
}

public void executeCallbacks(ClientTransaction transaction) {
    final List<ClientTransactionItem> callbacks = transaction.getCallbacks();
    if (callbacks == null || callbacks.isEmpty()) {
        // No callbacks to execute, return early.
        return;
    }
    if (DEBUG_RESOLVER) Slog.d(TAG, tId(transaction) + "Resolving callbacks in transaction");

    final IBinder token = transaction.getActivityToken();
    ActivityClientRecord r = mTransactionHandler.getActivityClient(token);

    // In case when post-execution state of the last callback matches the final state requested
    // for the activity in this transaction, we won't do the last transition here and do it when
    // moving to final state instead (because it may contain additional parameters from server).
    final ActivityLifecycleItem finalStateRequest = transaction.getLifecycleStateRequest();
    final int finalState = finalStateRequest != null ? finalStateRequest.getTargetState()
            : UNDEFINED;
    // Index of the last callback that requests some post-execution state.
    final int lastCallbackRequestingState = lastCallbackRequestingState(transaction);

//遍历callbacks，调用ClientTransactionItem的execute方法
//LaunchActivityItem会在这里调用execute
    final int size = callbacks.size();
    for (int i = 0; i < size; ++i) {
        final ClientTransactionItem item = callbacks.get(i);
        ...
        item.execute(mTransactionHandler, token, mPendingActions);
        item.postExecute(mTransactionHandler, token, mPendingActions);
        ...
    }
}

```

继续查看LaunchActivityItem的execute

```java
LaunchActivityItem.java

@Override
public void execute(ClientTransactionHandler client, IBinder token,
        PendingTransactionActions pendingActions) {
    Trace.traceBegin(TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
    ActivityClientRecord r = new ActivityClientRecord(token, mIntent, mIdent, mInfo,
            mOverrideConfig, mCompatInfo, mReferrer, mVoiceInteractor, mState, mPersistentState,
            mPendingResults, mPendingNewIntents, mIsForward,
            mProfilerInfo, client, mAssistToken);
    client.handleLaunchActivity(r, pendingActions, null /* customIntent */);
    Trace.traceEnd(TRACE_TAG_ACTIVITY_MANAGER);
}

```

里面调用了client.handleLaunchActivity方法，client是ClientTransactionHandler的实例，是在TransactionExecutor构造方法传入的，
TransactionExecutor创建是在ActivityThread中,所以，client.handleLaunchActivity方法就是ActivityThread的handleLaunchActivity方法。

流程图如下：

![](./activity_process_2.jpg)

### Activity初始化及生命周期函数回调

```java
ActivityThread.java

public Activity handleLaunchActivity(ActivityClientRecord r,
        PendingTransactionActions pendingActions, Intent customIntent) {
    ...
    final Activity a = performLaunchActivity(r, customIntent);
    ...
    return a;
}

/**  activity 启动的核心实现. */
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    //1、从ActivityClientRecord获取待启动的Activity的组件信息
    ActivityInfo aInfo = r.activityInfo;
    if (r.packageInfo == null) {
        r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                Context.CONTEXT_INCLUDE_CODE);
    }

    ComponentName component = r.intent.getComponent();
    if (component == null) {
        component = r.intent.resolveActivity(
            mInitialApplication.getPackageManager());
        r.intent.setComponent(component);
    }

    if (r.activityInfo.targetActivity != null) {
        component = new ComponentName(r.activityInfo.packageName,
                r.activityInfo.targetActivity);
    }
    //创建ContextImpl对象
    ContextImpl appContext = createBaseContextForActivity(r);
    Activity activity = null;
    try {
        //2、创建activity实例
        java.lang.ClassLoader cl = appContext.getClassLoader();
        activity = mInstrumentation.newActivity(
                cl, component.getClassName(), r.intent);
        StrictMode.incrementExpectedActivityCount(activity.getClass());
        r.intent.setExtrasClassLoader(cl);
        r.intent.prepareToEnterProcess();
        if (r.state != null) {
            r.state.setClassLoader(cl);
        }
    } catch (Exception e) {
        ..
    }
    try {
        //3、创建Application对象（如果没有的话）
        Application app = r.packageInfo.makeApplication(false, mInstrumentation);
        ...
        if (activity != null) {
            CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
            Configuration config = new Configuration(mCompatConfiguration);
            if (r.overrideConfig != null) {
                config.updateFrom(r.overrideConfig);
            }
          
            Window window = null;
            if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                window = r.mPendingRemoveWindow;
                r.mPendingRemoveWindow = null;
                r.mPendingRemoveWindowManager = null;
            }
            appContext.setOuterContext(activity);
            
            //4、attach方法为activity关联上下文环境
            activity.attach(appContext, this, getInstrumentation(), r.token,
                    r.ident, app, r.intent, r.activityInfo, title, r.parent,
                    r.embeddedID, r.lastNonConfigurationInstances, config,
                    r.referrer, r.voiceInteractor, window, r.configCallback,
                    r.assistToken);

            if (customIntent != null) {
                activity.mIntent = customIntent;
            }
            r.lastNonConfigurationInstances = null;
            checkAndBlockForNetworkAccess();
            activity.mStartedActivity = false;
            int theme = r.activityInfo.getThemeResource();
            if (theme != 0) {
                activity.setTheme(theme);
            }

            activity.mCalled = false;
            
            //5、调用生命周期onCreate
            if (r.isPersistable()) {
                mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
            } else {
                mInstrumentation.callActivityOnCreate(activity, r.state);
            }
            if (!activity.mCalled) {
                throw new SuperNotCalledException(
                    "Activity " + r.intent.getComponent().toShortString() +
                    " did not call through to super.onCreate()");
            }
            r.activity = activity;
        }
        r.setState(ON_CREATE);
        
        synchronized (mResourcesManager) {
            mActivities.put(r.token, r);
        }

    } 
    ...

    return activity;
}

```

由以上代码可知，performLaunchActivity主要完成以下事情：

- 从ActivityClientRecord获取待启动的Activity的组件信息
- 通过mInstrumentation.newActivity方法使用类加载器创建activity实例
- 通过LoadedApk的makeApplication方法创建Application对象，内部也是通过mInstrumentation使用类加载器，创建后就调用了instrumentation.callApplicationOnCreate方法，也就是Application的onCreate方法。
- 创建ContextImpl对象并通过activity.attach方法对重要数据初始化，关联了Context的具体实现ContextImpl，attach方法内部还完成了window创建，这样Window接收到外部事件后就能传递给Activity了。
- 调用Activity的onCreate方法，是通过 mInstrumentation.callActivityOnCreate方法完成。

其他生命周期处理也是类似的，先在ActivityStackSupervisor中添加对应的XXXActivityItem，然后在ActivityThread中的handleXXXActivity处理。

## 总结归纳

### 整体流程图如下

![](./activity_process_3.jpg)

### 一些类的介绍

类名 | 作用
--- | ---
ActivityThread | 应用的入口类，系统通过调用main函数，开启消息循环队列。ActivityThread所在线程被称为应用的主线程（UI线程）
ApplicationThread |是ActivityThread的内部类，继承IApplicationThread.Stub，是一个IBinder，是ActiivtyThread和AMS通信的桥梁，AMS则通过代理调用此App进程的本地方法，运行在Binder线程池
H|继承Handler，在ActivityThread中初始化，即主线程Handler，用于主线程所有消息的处理。本片中主要用于把消息从Binder线程池切换到主线程
Intrumentation|具有跟踪application及activity生命周期的功能，用于监控app和系统的交互
ActivityManagerService|Android中最核心的服务之一，负责系统中四大组件的启动、切换、调度及应用进程的管理和调度等工作，其职责与操作系统中的进程管理和调度模块相类似，因此它在Android中非常重要，它本身也是一个Binder的实现类。
ActivityTaskManagerService|管理activity及其容器（task, stacks, displays）的系统服务（Android10中新增，分担了AMS的部分职责）
ActivityStarter|用于解释如何启动活动。该类收集所有逻辑，用于确定Intent和flag应如何转换为活动以及相关的任务和堆栈
ActivityStack|用来管理系统所有的Activity，内部维护了Activity的所有状态和Activity相关的列表等数据
ActivityStackSupervisor|负责所有Activity栈的管理。AMS的stack管理主要有三个类，ActivityStackSupervisor，ActivityStack和TaskRecord
ClientLifecycleManager|客户端生命周期执行请求管理
ClientTransaction|是包含一系列的 待客户端处理的事务 的容器，客户端接收后取出事务并执行
LaunchActivityItem、ResumeActivityItem|继承ClientTransactionItem，客户端要执行的事务信息，启动activity

## 参考

- [https://juejin.cn/post/6847902222294990862](https://juejin.cn/post/6847902222294990862)
- 《Android开发艺术探索》
