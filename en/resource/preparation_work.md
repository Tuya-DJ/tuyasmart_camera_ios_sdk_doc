# Preparation Work

TuyaSmartCamera relies on the Tuya Smart iOS SDK, when accessing this SDK, you need to access TuyaSmartHomeKit (Click [Here](https://tuyainc.github.io/tuyasmart_home_ios_sdk_doc/en/) to view the document) .

Add blow code to AppDelegate.m top :

```objective-c
#import <TuyaSmartCamera/TuyaSmartCameraFactory.h>
```

Adding initial SDK code in [AppDelegate application:didFinishLaunchingWithOptions:]: (Vsesion 3.0.0 did be deprecated)

```objective-c
// 3.0.0 version did deprecated this methord。sdk will initialize when create camera first time
[TuyaSmartCameraFactory initSDKs];
```

## Example Code Appointment

Under below example code，if there is no special statement， all are under ViewController： 

```
@interface ViewController : UIViewController

@end

@implementation ViewController

// all example code in there

@end
```

Like self.prop， we appoint prop property has right realization under ViewController，like：

```
self.device = [[TuyaSmartDevice alloc] initWithDeviceId:@"your_device_id"];
```

if we want it excute correctly，need to add one TuyaSmartDevice property in ViewController：

```
@property (nonatomic, strong) TuyaSmartDevice *device;
```

The rest can be done in the same manner
