# 我们的解决方案

* 提供一块容易集成同时又功能强大的实时互动白板。
* 白板提供灵活的扩展能力和二次开发能力，提供全平台（iOS、Android、Web、小程序） SDK 。

# 教具

## 切换教具

White SDK 提供多种教具，我们可以通过修改 `memberState` 来切换当前的教具。例如，将当前教具切换成「铅笔」工具可以使用如下代码。
```java
MemberState memberState = new MemberState();
memberState.setCurrentApplianceName("pencil");
room.setMemberState(memberState);
```
可以通过如下代码获取当前房间的教具名称。

```java
room.getMemberState(new Promise<MemberState>() {
    @Override
    public void then(MemberState memberState) {
        memberState.getCurrentApplianceName();
    }

    @Override
    public void catchEx(Exception t) {

    }
});
```

## 教具列表

| 名称 | 字符串 | 描述 |
| :--- | :--- | :--- |
| 选择 | selector | 选择、移动、放缩 |
| 铅笔 | pencil | 画出带颜色的轨迹 |
| 矩形 | rectangle | 画出矩形 |
| 椭圆 | ellipse | 画出正圆或椭圆 |
| 橡皮 | eraser | 删除轨迹 |
| 文字 | text | 编辑、输入文字 |


## 调色盘

通过如下代码可以修改调色盘的颜色。
```java
MemberState memberState = new MemberState();
memberState.setStrokeColor(new int[]{255, 0, 0});
room.setMemberState(memberState);
```
通过将 RGB 写在一个数组中，形如 [255, 0, 0] 来表示调色盘的颜色。

也可以根据如下代码获取当前调色盘的颜色。
```java
room.getMemberState(new Promise<MemberState>() {
    @Override
    public void then(MemberState memberState) {
        memberState.getStrokeColor();
    }

    @Override
    public void catchEx(Exception t) {

    }
});
```
调色盘能影响铅笔、矩形、椭圆、文字工具的效果。

# 插入图片

相关 API：

```Java
public void insertImage(ImageInformation imageInfo);
public void completeImageUpload(String uuid, String url);
```

1. 首先创建 `ImageInformation` 类，配置图片，宽高，以及中心点位置，设置 uuid，确保 uuid 唯一即可。
1. 调用 `insertImage:` 方法，传入 `ImageInformation` 实例。白板此时就先生成一个占位框。
1. 图片通过其他方式上传或者通过其他方式直接获取到图片的网络，在获取图片地址后，调用
`completeImageUpload` 方法，uuid 参数为 `insertImage:` 方法传入的 uuid，src 为实际图片网络地址。

# 翻页与 PPT

White SDK 允许多个页面并在其中进行切换。在白板初次创建时，只有一个空白页面。我们可以通过如下方法来插入/删除页面。

```java
// 插入新页面在指定 index
room.insertNewPage(1);

// 删除 index 位置的页面
room.removePage(1);
```
我们可以通过修改 globalState 来做到翻页效果。

```java
GlobalState globalState = new GlobalState();
globalState.setCurrentSceneIndex(1);
room.setGlobalState(globalState);
```

注意，globalState 是整个房间所有人共用的。通过修改 globalState 的 currentSceneIndex 属性来翻页，将导致整个房间的所有人切换到该页。

White SDK 还支持插入 PPT。插入的 PPT 将变成带有 PPT 内容的页面。我们需要先将 PPT 文件或 PDF 文件的每一页单独转化成一组图片，并将这组图片在互联网上发布（例如上传到某个云存储仓库中，并获取每一张图片的可访问的 URL）。

然后，将这组 URL 通过如下方法插入。

```java
room.pushPptPages(new PptPage[]{
    new PptPage("http://website.com/image-001.png", 600d, 600d),
    new PptPage("http://website.com/image-002.png", 600d, 600d),
    new PptPage("http://website.com/image-003.png", 600d, 600d)
});
```

这个方法将在当前页之后插入 3 个带有 PPT 内容的新页面。

## 插入PPT 与插入图片 的区别

区别| 插入PPT | 插入图片
---------|----------|---------
 调用后结果 | 会自动新建多个白板页面，但是仍然保留在当前页（所以无明显区别），需要通过翻页API进行切换 | 产生一个占位界面，插入真是图片，需要调用 `completeImageUploadWithUuid:src` ,传入占位界面的 uuid，以及图片的网络地址 |
 移动 | 无法移动，所以不需要位置信息 | 可以移动，所以插入时，需要提供图片大小以及位置信息
 与白板页面关系 | 插入 ppt 的同时，白板就新建了一个页面，这个页面的背景就是 PPT 图片 | 是当前白板页面的一部分，同一个页面可以加入多张图片

# 图片替换

图片替换功能，需要在初始化时使用新的初始化 API。 传入实现 `UrlInterrupter` Interface 的类即可。
传入参数为图片原始地址，返回修改后的图片地址即可。

```Java
public WhiteSdk(WhiteBroadView bridge, Context context, WhiteSdkConfiguration whiteSdkConfiguration, UrlInterrupter urlInterrupter) {
```

* 注意点：

1. 该 API 会在渲染时，被频繁调用。如果没有需求，就不需要使用该方法。
1. 该方法会同时对 ppt插入以及图片插入API起效。

# 主播模式

## 跟随主播的视角

White SDK 提供的白板是向四方无限延伸的。同时也允许用户通过鼠标滚轮、手势等方式移动白板。因此，即便是同一块白板的同一页，不同用户的屏幕上可能看到的内容是不一样的。

为此，我们引入「主播模式」这个概念。主播模式将房间内的某一个人设为主播，他/她屏幕上看到的内容即是其他所有人看到的内容。当主播进行视角的放缩、移动时，其他人的屏幕也会自动进行放缩、移动。

主播模式中，主播就好像摄像机，其他人就好像电视机。主播看到的东西会被同步到其他人的电视机上。

可以通过如下方法修改当前视角的模式。

```java
// 主播模式
// 房间内其他人的视角模式会被自动修改成 follower，并且强制观看你的视角。
// 如果房间内存在另一个主播，该主播的视角模式也会被强制改成 follower。
// 就好像你抢了他/她的主播位置一样。
room.setViewMode(ViewMode.broadcaster);

// 自由模式
// 你可以自由放缩、移动视角。
// 即便房间里有主播，主播也无法影响你的视角。
room.setViewMode(ViewMode.freedom);

// 追随模式
// 你将追随主播的视角。主播在看哪里，你就会跟着看哪里。
// 在这种模式中如果你放缩、移动视角，将自动切回 freedom模式。
room.setViewMode(ViewMode.follower);
```

我们也可以通过如下方法获取当前视角的状态。


```java
room.getBroadcastState(new Promise<BroadcastState>() {
    @Override
    public void then(BroadcastState broadcastState) {
        showToast(broadcastState.getMode());
    }
    @Override
    public void catchEx(Exception t) {}
});
```

它的结构如下。

```java
public class BroadcastState {
    private ViewMode mode;
    private Long broadcasterId;
    private MemberInformation broadcasterInformation;
    
    ... getter/setter
}
```

## 视角中心

同一个房间的不同用户各自的屏幕尺寸可能不一致，这将导致他们的白板都有各自不同的尺寸。实际上，房间的其他用户会将白板的中心对准主播的白板中心（注意主播和其他用户的屏幕尺寸不一定相同）。

我们需要通过如下方法设置白板的尺寸，以便主播能同步它的视角中心。
```java
room.setViewSize(100, 100);
```
尺寸应该和白板在产品中的实际尺寸相同（一般而言就是浏览器页面或者应用屏幕的尺寸）。如果用户调整了窗口大小导致白板尺寸改变。应该重新调用该方法刷新尺寸。

# 白板状态管理

White SDK 还提供多种工具，如选择器、铅笔、文字、圆形工具、矩形工具。同时还提供图片展示工具和 PPT 工具。这些功能的展现形式，关系到具体网页应用本身的交互设计、视觉风格。考虑到这一点，白板上没有直接提供这些 UI 组件。你只能通过程序调用的方式，来让白板使用这些功能。

对此，你可能有如下疑问：

* 我的 UI 组件已经做好了，我如何让 UI 组件的操作影响到白板的行为？
* 白板有哪些状态，我的 UI 组件如何监听、获取白板的状态？

本章将解决这 2 个问题。

## 白板状态简介

当你加入一个房间后，你可以获取和修改的房间状态有如下 2 个。

* __GlobalState__：全房间的状态，自房间创建其就存在。所有人可见，所有人可修改。
* __MemberState__：成员状态。每个房间成员都有独立的一份实例，成员加入房间时自动创建，成员离开房间时自动释放。成员只能获取、监听、修改自己的 MemberState。
* __BroadcastState__：视角状态，和主播模式、跟随模式相关。

这 3 个状态都是一个 key-value 对象。

你可以通过如下方式获取它们。
```java
room.getGlobalState(new Promise<GlobalState>() {
    @Override
    public void then(GlobalState globalState) {}
    @Override
    public void catchEx(Exception e) {}
});
room.getMemberState(new Promise<MemberState>() {
    @Override
    public void then(MemberState memberState) {}
    @Override
    public void catchEx(Exception e) {}
});
room.getBroadcastState(new Promise<BroadcastState>() {
    @Override
    public void then(BroadcastState broadcastState) {}
    @Override
    public void catchEx(Exception t) {}
});
```
其中 room 对象需要通过调用 `joinRoom` 方法获取，在之前的篇章中有说明，在此不再赘述。

这 2 个状态可能被动改变，比如 GlobalState 可能被房间其他成员修改。因此，你需要监听它们的变化。具体做法是在 `WhiteSDK` 上注册监听器，可以注册 `RoomCallbacks`  或者 `AbstractRoomCallbacks` ，使用后者可以只覆盖感兴趣的回调方法。
```java
whiteSdk.addRoomCallbacks(new RoomCallbacks() {
    @Override
    public void onPhaseChanged(RoomPhase roomPhase) {}
    @Override
    public void onBeingAbleToCommitChange(boolean b) {}
    @Override
    public void onDisconnectWithError(Exception e) {}
    @Override
    public void onKickedWithReason(String s) {}
    @Override
    public void onRoomStateChanged(RoomState roomState) {}
    @Override
    public void onCatchErrorWhenAppendFrame(long l, Exception e) {}
});
```

当你需要修改白板状态的时候，你可以使用如下方式修改。
```java
MemberState memberState = new MemberState();
memberState.setStrokeColor(new int[]{99, 99, 99});
memberState.setCurrentApplianceName("rectangle");
room.setMemberState(memberState);
```
你不需要在参数中传入修改后的完整 GlobalState 或 MemberState，只需要填入你希望更新的 key-value 对即可。如果你修改了 GlobalState，整个房间的人都会收到你的修改结果。

## GlobalState

```java
public class GlobalState {
    // 当前场景索引，修改它会切换场景
    private Integer currentSceneIndex;
    ... setter/getter
}
```

## MemberState

```java
public class MemberState {
    // 当前工具，修改它会切换工具。有如下工具可供挑选：
    // 1. selector 选择工具
    // 2. pencil 铅笔工具
    // 3. rectangle 矩形工具
    // 4. ellipse 椭圆工具
    // 5. eraser 橡皮工具
    // 6. text 文字工具
    private String currentApplianceName;
    // 线条的颜色，将 RGB 写在一个数组中。形如 [255, 128, 255]。
    private int[] strokeColor;
    // 线条的粗细
    private Double strokeWidth;
    // 文字的字号
    private Double textSize;
    ... setter/getter
}
```

## BroadcastState

```java
public class BroadcastState {
    // 当前视角模式，有如下：
    // 1."freedom" 自由视角，视角不会跟随任何人
    // 2."follower" 跟随视角，将跟随房间内的主播
    // 2."broadcaster" 主播视角，房间内其他人的视角会跟随我
    private ViewMode mode;

    // 房间主播 ID。
    // 如果当前房间没有主播，则为 undefined
    private Long broadcasterId;

    // 主播信息，可以自定义，具体参见下面的数据结构
    private MemberInformation broadcasterInformation;
    ... setter/getter
}

public class MemberInformation {
    // ID
    private Long id;
    // 昵称
    private String nickName;
    // 头像 URL
    private String avatar;
    ... setter/getter
}
```



# 白板生命周期

joinRoom 的回调函数不仅可以监听白板的行为状态，还可以监听白板的生命周期状态和异常原因，具体使用如下

```java
whiteSdk.addRoomCallbacks(new AbstractRoomCallbacks() {
    @Override
    public void onPhaseChanged(RoomPhase phase) {
        // 白板发生状态改变, 具体状态如下:
        // "connecting",
    	// "connected",
    	// "reconnecting",
    	// "disconnecting",
    	// "disconnected",
    }
    @Override
    public void onDisconnectWithError(Exception e) {
        // 出现连接失败后的具体错误
    }

    @Override
    public void onKickedWithReason(String reason) {
        // 被踢出房间的原因
    }
});
```



# 自定义消息

可以使用自定义消息来满足类似 IM 、弹幕、点赞等场景。

```java
public void dispatchMagixEvent(AkkoEvent eventEntry);

public void addMagixEventListener(String eventName, EventListener eventListener) ;

public void removeMagixEventListener(String eventName) ;

```

- dispatchMagixEvent 用来发送 AkkoEvent，AkkoEvent 结构如下，
  -  payload 为任意可被 JSON 序列化的对象
  - eventName 为消息类型名称，同一个房间的所有人都会收到房间内同一个消息类型的消息

```java
public class AkkoEvent {
    private String eventName;
    private Object payload;

    public AkkoEvent(String eventName, Object payload) {
        this.eventName = eventName;
        this.payload = payload;
    }
}
```

- addMagixEventListener 和 removeMagixEventListener 用来增加和删除消息监听器，eventName 为消息类型名称。

# 面板操作开关

你可以通过 `room.disableOperations(true)` 来禁止用户操作白板。

你可以通过 `room.disableOperations(false)` 来恢复用户操作白板的能力。


# 缩放

一方面通过手势可以放缩白板（iOS 和 Android 上使用双指手势、mac os 使用双指手势、windows 使用鼠标中键的滚轮），另一方面也可以通过 `zoomChange` 来缩放白板。

```java
room.zoomChange(10);
```

