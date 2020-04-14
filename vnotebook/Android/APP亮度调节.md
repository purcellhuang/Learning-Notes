# APP亮度调节

```
//获取系统亮度
Settings.System.getInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS);	//3 ~ 1023

//设置系统亮度
Settings.System.putInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS,systemBrightness);

//获取系统模式
Settings.System.getInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS_MODE);

//设置系统亮度模式
Settings.System.putInt(getContentResolver(), Settings.System.SCREEN_BRIGHTNESS_MODE, systemMode);


//获取当前window亮度
Window window = activity.getWindow()
WindowManager.LayoutParams lp = window.getAttributes();
float nowBright = lp.screenBrightness;	// 0.0 ~ 1.0 & -1.0  -1.0表示自动调节

//设置当前window亮度
Window window = activity.getWindow();
WindowManager.LayoutParams lp = window.getAttributes();
lp.screenBrightness = brightness;
window.setAttributes(lp);
```