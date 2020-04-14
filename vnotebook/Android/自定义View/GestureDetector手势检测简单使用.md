# GestureDetector手势检测简单使用(音量、亮度、快进、快退)
## 引言
在 android 开发过程中，我们经常需要对一些手势，如：单击、双击、长按、滑动、缩放等，进行监测。这时也就引出了手势监测的概念，所谓的手势监测，说白了就是对于 GestureDetector 的使用。
GestureDetector 是 Android 中，专门用来进行手势监听的一个对象，在他的监听器中，我们通过传入 MotionEvents 对象，就可以在各种事件的回调方法中各种手势进行监测。举个例子： GestureDetector 的 OnGestureListener 就是一种回调方法，就是说在获得了传入的这个 MotionEvents 对象之后，进行了处理，我们通过重写了其中的各种方法（单击事件、双击事件等等），就可以监听到单击，双击，滑动等事件，然后直接在这些方法内部进行处理。注：由于缩放手势独有的复杂性，在这里就先不讨论。

最近毕业设计涉及到了视频的播放，在这里刚好给大家说说GestureDetector来实现视频的快进、快退、音量、亮度的调节以及双击控制播放。项目采用谷歌提供的ExoPlayer来封装实现。

## GestureDetector 介绍

**构造函数**
```
public GestureDetector(Context context, OnGestureListener listener) 
public GestureDetector(Context context, OnGestureListener listener, Handler handler) 
public GestureDetector(Context context, OnGestureListener listener, Handler handler, boolean unused){this(context, listener, handler);} 
```

GestureDetector中的数据是给GestureHandler内部类进行处理，这个类会使用Handler，由Handler的知识知道，创建Handler必须有Looper，但是在一些新开的线程中没有创建Looper，所以我们需要传入一个带了Looper的Handler变量(或者调用Loop.prepare())，否则，GestureDetector对象会创建失败。

* **OnDoubleTapListener** ：也就是双击事件，双击事件除了 onDoubleTapEvent 这个回调方法之外，还有 SingleTapConfirmed 和 DoubleTap 这两个回调方法
 
* **OnGestureListener** ：这里集合了众多手势的监听器：主要有：按下(Down)、 扔(Fling)、长按(LongPress)、滚动(Scroll)、触摸反馈(ShowPress) 和 单击抬起(SingleTapUp)

* **SimpleOnGestureListener**  ：上述接口的空实现，用的频率比较多

```
//下面的6个方法继承自OnGestureListener
public boolean onSingleTapUp( MotionEvent e)
当用户单击时触发

public void onLongPress (MotionEvent e )
当用户手指在长按屏幕时触发

public boolean onScroll (MotionEvent e1 , MotionEvent e2 ,float distanceX , float distanceY)
当用户手指在屏幕上拖动时触发
后面两个变量时在X,Y上移动的距离

public boolean onFling ( MotionEvent e1 ,MotionEvent e2 , float velocityX , float velocityY)
当用户手指拖动后，手指离开屏幕时触发
这个方法常用来使手指离开后页面仍然可以滑动（速度慢慢变小）
后两个变量表示手指在X,Y两个方向上的速度

public void onShowPress (MotionEvent  e)
当用户手指按下，但没有移动时触发该方法

public boolean onDown ( MotionEvent e)  
当按下时触发该方法，所有手势第一个必定触发该方法

//下面的三个方法继承自OnDoubleTapListener
public boolean onDoubleTap (MotionEvent e)
当用户双击时触发

public boolean onDoubleTapEvent (MotionEvent e)
在双击事件确定发生时会对第二次按下产生的 MotionEvent 信息进行回调。

public boolean onSingleTapConfirmed ( MotionEvent e)
当单击事件确定后进行回调
```
### 单击
```
public boolean onSingleTapUp( MotionEvent e)

public boolean onSingleTapConfirmed ( MotionEvent e)
```
我们进行单击事件的时候，这几个方法响应的顺序是这样的：
> onDown() -> onSingleTapUp() -> onSingleCofirmed()

首先onDown()必定是第一个执行的，但是会发现onSingleTapUp在onSingleComfirmed之前执行,我查阅了相关文档，发现他们虽然同样响应的是当手指离开屏幕的活动，但是onSingleTapUp是立即执行，而onSingleComfirmed却要在离开后300ms后才执行 ，这样的目的是确认我们进行的是单击事件（为了防止我们在300ms内再次进行单击事件），所以他们的名字分别是Up和Comfirmed.

### 双击
```
public boolean onDoubleTap (MotionEvent e)

public boolean onDoubleTapEvent (MotionEvent e)
```
onDoubleTap:  可以确认这是一个双击事件的时候回调
onDoubleTapEvent(MotionEvent e)：onDoubleTap()回调之后的输入事件（DOWN、MOVE、UP）都会回调这个方法（这个方法可以实现一些双击后的控制，如让View双击后变得可拖动等）。

### 长按
```
public void onShowPress (MotionEvent e)

public void onLongPress (MotionEvent e)
```
响应顺序如下：
> onDown -> onShowPress -> onLongPress

也就是说，在长按时，onShowPress在onLongPress前面执行

### 滑动/拖动
```
public void onScroll (MotionEvent e)

public void onFling (MotionEvent e)
```
响应顺序:
> onDown -> onScroll -> 不定量个onScroll ->onScroll -> onFiling

由此可见，在滑动/拖动过程中，不断调用onScroll，最后调用onFiling

## 使用案例
### 引入播放UI布局
首先来引入播放页面的布局。
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <RelativeLayout
        android:id="@+id/root_layout"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <RelativeLayout
            android:id="@+id/gesture_volume_layout"
            android:layout_width="120dp"
            android:layout_height="100dp"
            android:layout_centerInParent="true"
            android:background="@drawable/player_gesture_bg"
            android:gravity="center"
            android:visibility="gone">

            <ImageView
                android:id="@+id/gesture_iv_player_volume"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_centerHorizontal="true"
                android:src="@drawable/player_volume" />

            <TextView
                android:id="@+id/gesture_tv_volume_percentage"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_below="@id/gesture_iv_player_volume"
                android:layout_centerHorizontal="true"
                android:gravity="right"
                android:text="80%"
                android:textColor="#ffececec" />
        </RelativeLayout>

        <RelativeLayout
            android:id="@+id/gesture_bright_layout"
            android:layout_width="120dp"
            android:layout_height="100dp"
            android:layout_centerInParent="true"
            android:background="@drawable/player_gesture_bg"
            android:gravity="center"
            android:visibility="gone">

            <ImageView
                android:id="@+id/gesture_iv_player_bright"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_centerHorizontal="true"
                android:src="@drawable/player_bright" />

            <TextView
                android:id="@+id/gesture_tv_bright_percentage"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_below="@id/gesture_iv_player_bright"
                android:layout_centerHorizontal="true"
                android:gravity="right"
                android:text="80%"
                android:textColor="#ffececec" />
        </RelativeLayout>

        <RelativeLayout
            android:id="@+id/gesture_progress_layout"
            android:layout_width="120dp"
            android:layout_height="100dp"
            android:layout_centerInParent="true"
            android:background="@drawable/player_gesture_bg"
            android:gravity="center"
            android:visibility="gone">

            <ImageView
                android:id="@+id/gesture_iv_progress"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_centerHorizontal="true"
                android:src="@drawable/player_backward" />

            <TextView
                android:id="@+id/gesture_tv_progress_time"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_below="@id/gesture_iv_progress"
                android:layout_centerHorizontal="true"
                android:gravity="right"
                android:text="00:35/24:89"
                android:textColor="#ffececec" />
        </RelativeLayout>
    </RelativeLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/lightGrayBg"
        android:orientation="horizontal">

        <ImageView
            android:id="@id/exo_play"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="8dp"
            android:src="@drawable/video_play" />

        <ImageView
            android:id="@id/exo_pause"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="8dp"
            android:src="@drawable/video_pause" />

        <com.google.android.exoplayer2.ui.DefaultTimeBar
            android:id="@id/exo_progress"
            android:layout_width="0dp"
            android:layout_height="26dp"
            android:layout_weight="1"
            app:buffered_color="@android:color/darker_gray"
            app:layout_constraintBottom_toBottomOf="@id/exo_position"
            app:layout_constraintLeft_toRightOf="@id/exo_position"
            app:layout_constraintRight_toLeftOf="@id/exo_duration"
            app:layout_constraintTop_toTopOf="@id/exo_position"
            app:played_color="@color/tabActive"
            app:unplayed_color="@android:color/black" />

        <TextView
            android:id="@id/exo_position"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="2:00"

            android:textColor="@color/textColor" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="/"

            android:textColor="@color/textColor" />

        <TextView
            android:id="@id/exo_duration"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="12:00"
            android:textColor="@color/textColor"

            />

        <ImageView
            android:id="@+id/exo_fullscreen"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="8dp"
            android:layout_marginRight="8dp"
            android:src="@drawable/video_fullscreen" />


    </LinearLayout>

</LinearLayout>
```
### 初始化view
```
        root_layout = findViewById(R.id.root_layout);
        root_layout.setLongClickable(true);//需要实现OnTouchListener监听
        root_layout.setOnTouchListener(this);

        gesture_volume_layout = findViewById(R.id.gesture_volume_layout);
        gesture_iv_player_volume = findViewById(R.id.gesture_iv_player_volume);
        gesture_tv_volume_percentage = findViewById(R.id.gesture_tv_volume_percentage);

        gesture_bright_layout = findViewById(R.id.gesture_bright_layout);
        gesture_iv_player_bright = findViewById(R.id.gesture_iv_player_bright);
        gesture_tv_bright_percentage = findViewById(R.id.gesture_tv_bright_percentage);

        gesture_progress_layout = findViewById(R.id.gesture_progress_layout);
        gesture_iv_progress = findViewById(R.id.gesture_iv_progress);
        gesture_tv_progress_time = findViewById(R.id.gesture_tv_progress_time);

```
### 获取播放窗口的宽高、初始化数据
```
         /** 获取视频播放窗口的尺寸 */
        ViewTreeObserver viewObserver = root_layout.getViewTreeObserver();
        viewObserver.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                root_layout.getViewTreeObserver().removeGlobalOnLayoutListener(this);
                playerWidth = root_layout.getWidth();
                playerHeight = root_layout.getHeight();
            }
        });

        audiomanager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
        maxVolume = audiomanager.getStreamMaxVolume(AudioManager.STREAM_MUSIC); // 获取系统最大音量
        currentVolume = audiomanager.getStreamVolume(AudioManager.STREAM_MUSIC); // 获取当前值
```
### 播放倍速
```
        //倍速
        PlaybackParameters playbackParameters = new PlaybackParameters(speed, 1.0F);
        simpleExoPlayer.setPlaybackParameters(playbackParameters);
```
### 重写onTouch和SimpleOnGestureListener
```
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        if (event.getAction() == MotionEvent.ACTION_UP) {
            GESTURE_FLAG = 0;// 手指离开屏幕后，重置调节音量或进度的标志
            gesture_volume_layout.setVisibility(View.GONE);
            gesture_bright_layout.setVisibility(View.GONE);
            gesture_progress_layout.setVisibility(View.GONE);
        }
        Log.i("onTouch", v.toString());
        Log.i("onTouch", "onTouch");
        return gestureDetector.onTouchEvent(event);
    }
```

```
    GestureDetector.SimpleOnGestureListener gestureListener = new GestureDetector.SimpleOnGestureListener() {

            @Override
            public boolean onDown(MotionEvent e) {
                firstScroll = true;// 设定是触摸屏幕后第一次scroll的标志
                Log.i("onTouch", "onDown");
                return false;
            }

            @Override
            public boolean onDoubleTapEvent(MotionEvent e) {
                Log.i("onTouch", "onDoubleTapEvent");
                switch (e.getActionMasked()) {
                    case MotionEvent.ACTION_UP:
                        if (playing) {
                            simpleExoPlayer.setPlayWhenReady(false);
                            playing = false;
                        } else {
                            simpleExoPlayer.setPlayWhenReady(true);
                            playing = true;
                        }
                        break;
                }

                firstScroll = false;// 第一次scroll执行完成，修改标志

                return super.onDoubleTapEvent(e);
            }

            @Override
            public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
                Log.i("onTouch", "onScroll");
                float mOldX = e1.getX(), mOldY = e1.getY();
                int y = (int) e2.getRawY();
                if (firstScroll) {// 以触摸屏幕后第一次滑动为标准，避免在屏幕上操作切换混乱
                    // 横向的距离变化大则调整进度，纵向的变化大则调整音量
                    if (Math.abs(distanceX) >= Math.abs(distanceY)) {
                        gesture_progress_layout.setVisibility(View.VISIBLE);
                        gesture_volume_layout.setVisibility(View.GONE);
                        gesture_bright_layout.setVisibility(View.GONE);
                        GESTURE_FLAG = GESTURE_MODIFY_PROGRESS;
                    } else {
                        if (mOldX > playerWidth * 3.0 / 5) {// 音量
                            gesture_volume_layout.setVisibility(View.VISIBLE);
                            gesture_bright_layout.setVisibility(View.GONE);
                            gesture_progress_layout.setVisibility(View.GONE);
                            GESTURE_FLAG = GESTURE_MODIFY_VOLUME;
                        } else if (mOldX < playerWidth * 2.0 / 5) {// 亮度
                            gesture_bright_layout.setVisibility(View.VISIBLE);
                            gesture_volume_layout.setVisibility(View.GONE);
                            gesture_progress_layout.setVisibility(View.GONE);
                            GESTURE_FLAG = GESTURE_MODIFY_BRIGHT;
                        }
                    }
                }

                // 如果每次触摸屏幕后第一次scroll是调节进度，那之后的scroll事件都处理音量进度，直到离开屏幕执行下一次操作
                if (GESTURE_FLAG == GESTURE_MODIFY_PROGRESS) {
                    // distanceX=lastScrollPositionX-currentScrollPositionX，因此为正时是快进
                    if (Math.abs(distanceX) > Math.abs(distanceY)) {// 横向移动大于纵向移动
                        if (distanceX >= DensityUtil.dip2px(VideoPlayActivity.this, STEP_PROGRESS)) {// 快退，用步长控制改变速度，可微调
                            gesture_iv_progress.setImageResource(R.drawable.player_backward);
                            if (playingTime > 3000) {// 避免为负
                                playingTime -= 3000;// scroll方法执行一次快退3秒
                            } else {
                                playingTime = 0;
                            }
                        } else if (distanceX <= -DensityUtil.dip2px(VideoPlayActivity.this, STEP_PROGRESS)) {// 快进
                            gesture_iv_progress.setImageResource(R.drawable.player_forward);
                            if (playingTime < videoTotalTime - 16000) {// 避免超过总时长
                                playingTime += 3000;// scroll执行一次快进3秒
                            } else {
                                playingTime = videoTotalTime - 10000;
                            }
                        }
                        if (playingTime < 0) {
                            playingTime = 0;
                        }

                        simpleExoPlayer.seekTo(playingTime);

                        gesture_tv_progress_time.setText(sdf.format(playingTime) + "/" + sdf.format(videoTotalTime));
                    }
                }

                // 如果每次触摸屏幕后第一次scroll是调节音量，那之后的scroll事件都处理音量调节，直到离开屏幕执行下一次操作
                else if (GESTURE_FLAG == GESTURE_MODIFY_VOLUME) {
                    currentVolume = audiomanager.getStreamVolume(AudioManager.STREAM_MUSIC); // 获取当前值
                    if (Math.abs(distanceY) > Math.abs(distanceX)) {// 纵向移动大于横向移动
                        if (distanceY >= DensityUtil.dip2px(VideoPlayActivity.this, STEP_VOLUME)) {// 音量调大,注意横屏时的坐标体系,尽管左上角是原点，但横向向上滑动时distanceY为正
                            if (currentVolume < maxVolume) {// 为避免调节过快，distanceY应大于一个设定值
                                currentVolume++;
                            }
                            gesture_iv_player_volume.setImageResource(R.drawable.player_volume);
                        } else if (distanceY <= -DensityUtil.dip2px(VideoPlayActivity.this, STEP_VOLUME)) {// 音量调小
                            if (currentVolume > 0) {
                                currentVolume--;
                                if (currentVolume == 0) {// 静音，设定静音独有的图片
                                    gesture_iv_player_volume.setImageResource(R.drawable.player_silence);
                                }
                            }
                        }
                        int percentage = (currentVolume * 100) / maxVolume;
                        gesture_tv_volume_percentage.setText(percentage + "%");
                        audiomanager.setStreamVolume(AudioManager.STREAM_MUSIC, currentVolume, 0);
                    }
                }

                // 如果每次触摸屏幕后第一次scroll是调节亮度，那之后的scroll事件都处理亮度调节，直到离开屏幕执行下一次操作
                else if (GESTURE_FLAG == GESTURE_MODIFY_BRIGHT) {
                    gesture_iv_player_bright.setImageResource(R.drawable.player_bright);
                    if (mBrightness < 0) {
                        mBrightness = getWindow().getAttributes().screenBrightness;
                        if (mBrightness <= 0.00f)
                            mBrightness = 0.50f;
                        if (mBrightness < 0.01f)
                            mBrightness = 0.01f;
                    }
                    WindowManager.LayoutParams lpa = getWindow().getAttributes();
                    lpa.screenBrightness = mBrightness + (mOldY - y) / playerHeight;
                    if (lpa.screenBrightness > 1.0f)
                        lpa.screenBrightness = 1.0f;
                    else if (lpa.screenBrightness < 0.01f)
                        lpa.screenBrightness = 0.01f;
                    getWindow().setAttributes(lpa);
                    gesture_tv_bright_percentage.setText((int) (lpa.screenBrightness * 100) + "%");
                }

                firstScroll = false;// 第一次scroll执行完成，修改标志
                return false;
            }
        };

        gestureDetector = new GestureDetector(VideoPlayActivity.this, gestureListener);
        gestureDetector.setIsLongpressEnabled(true);
```

**Demo下载地址：**