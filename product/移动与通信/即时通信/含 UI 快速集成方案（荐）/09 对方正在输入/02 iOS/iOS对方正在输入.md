
## 功能描述
`TUIKit` 从 [6.5.2803](https://cloud.tencent.com/document/product/269/1606#6.5.2803-.402022.07.15---.E5.A2.9E.E5.BC.BA.E7.89.88) 版本开始支持单聊会话“对方正在输入”功能。

本功能使用 IMSDK [在线消息](https://cloud.tencent.com/document/product/269/75340) 能力实现。

<table style="text-align:center;vertical-align:middle;width:600px">
  <tr>
    <th style="text-align:center;" width="300px">开启“对方正在输入”<br></th>
    <th style="text-align:center;" width="300px">关闭“对方正在输入”<br></th>
  </tr>
  <tr>
    <td style="text-align:center;"><img style="width:250px" src="https://qcloudimg.tencent-cloud.cn/raw/de7fbbf9b4db230dacf7b3c6b5055ab6.png"  />    </td>
    <td style="text-align:center;"><img style="width:250px" src="https://qcloudimg.tencent-cloud.cn/raw/85e4bbb27a6e74127b0648333fe6b017.png" />     </td>
	 </tr>
</table>

## 关闭对方正在输入

在 `TUIChat` 组件的 [TUIChatConfig](https://github.com/TencentCloud/TIMSDK/blob/master/iOS/TUIKit/TUIChat/CommonModel/TUIChatConfig.h) 配置参数中，提供了“对方正在输入”功能开关 **enableTypingStatus** , 其类型为 BOOL，默认为 `YES` 。

```Java
- (id)init
{
    self = [super init];
    if(self){
        //...其他配置
        self.enableTypingStatus = YES;
    }
    return self;
}
```
如果想关闭对方正在输入功能，只需把 **enableTypingStatus** 的默认值改为 `NO` ，或者在聊天页面初始化之前调用以下方法来关闭。
```java
TUIChatConfig.defaultConfig.enableTypingStatus = NO;
```


## 常见问题

### 为什么开启开关后没有对方正在输入提示？

在单聊会话中显示“对方正在输入...”的规则是：对方在30秒内向您发送过消息且当前正在输入文字。

## 交流与反馈
欢迎加入 QQ 群进行技术交流和反馈问题。
<img src="https://im.sdk.qcloud.com/tools/resource/officialwebsite/pictures/doc_tuikit_qq_group.jpg" style="zoom:50%;"/>