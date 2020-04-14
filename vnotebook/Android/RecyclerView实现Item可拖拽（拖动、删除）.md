# RecyclerView实现Item可拖拽（拖动、删除）
话不多说，先附上效果图：

## ItemTouchHelper
这是一个RecyclerView的工具，提供了drag & swipe 的功能，可以帮助我们处理RecyclerView中的Item的拖拽和滑动事件。
```
        ItemTouchHelper helper = new ItemTouchHelper(new MyItemTouchHelperCallback(mAdapter,this));
        helper.attachToRecyclerView(mRecyclerView);
```

## ItemTouchHelper.Callback
ItemTouchHelper.Callback是一个抽象类，里面有一些抽象方法需要开发者去重写。重写方法如下：
```
1. getMovementFlags(RecyclerView, ViewHolder) 

2. onMove(RecyclerView, ViewHolder, ViewHolder)

3. onSwiped(ViewHolder, int)

4. onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState)

5. clearView(RecyclerView recyclerView,RecyclerView.ViewHolder viewHolder)

6. isLongPressDragEnabled()

7. isItemViewSwipeEnabled()
```
-----
**getMovementFlags(RecyclerView, ViewHolder)**
这个方法是设置那些方向可以拖动和滑动RecyclerView中的item
```
    @Override
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        int dragFlag;    //拖动标记
        int swipeFlags;    //滑动标记
        //如果是表格布局，则可以上下左右的拖动，但是不能滑动
        if (recyclerView.getLayoutManager() instanceof GridLayoutManager) {
            dragFlag = ItemTouchHelper.UP |
                    ItemTouchHelper.DOWN |
                    ItemTouchHelper.LEFT |
                    ItemTouchHelper.RIGHT;
            swipeFlags = 0;
        }
        //如果是线性布局，那么只能上下拖动，只能左右滑动
        else {
            dragFlag = ItemTouchHelper.UP |
                    ItemTouchHelper.DOWN;
            swipeFlags = ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
        }

        //通过makeMovementFlags生成最终结果
        return makeMovementFlags(dragFlag, swipeFlags);
    }
```
**onMove(RecyclerView, ViewHolder, ViewHolder)**
拖动item的回调方法，在这里面实现拖动需要做的事情，比如RecyclerView里面item的顺序交换，
传入的参数是被拖动item的ViewHolder和目标ViewHolder
```
    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
        //被拖动的item位置
        int fromPosition = viewHolder.getLayoutPosition();
        //他的目标位置
        int targetPosition = target.getLayoutPosition();
        //为了降低耦合，使用接口让Adapter去实现交换功能
        mItemPositionListener.onItemSwap(fromPosition, targetPosition);
        return true;
    }
```
**onSwiped(ViewHolder, int)**
滑动item的回调方法，在这里面实现滑动需要做的事情，比如RecyclerView里面item的删除
```
    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
        //为了降低耦合，使用接口让Adapter去实现交换功能
        mItemPositionListener.onItemMoved(viewHolder.getLayoutPosition());
    }
```
**onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState)**
在每次item的状态变成拖拽 (ACTION_STATE_DRAG) 或者 滑动 (ACTION_STATE_SWIPE)的时候被调用。这是把你的item view变成激活状态的最佳地点。
```
    @Override
    public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState) {
        super.onSelectedChanged(viewHolder, actionState);

        //当开始拖拽的时候
        if (actionState != ItemTouchHelper.ACTION_STATE_IDLE) {
            //修改背景颜色
            viewHolder.itemView.setBackgroundColor(Color.LTGRAY);
            //手机振动
            Vibrator vibrator = (Vibrator)mContext.getSystemService(mContext.VIBRATOR_SERVICE);
            vibrator.vibrate(50);
        }
    }
```
**clearView(RecyclerView recyclerView,RecyclerView.ViewHolder viewHolder)**
item被放开或者动画完成的回调
```
    //当手指松开的时候
    @Override
    public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        viewHolder.itemView.setBackgroundColor(Color.TRANSPARENT);
        super.clearView(recyclerView, viewHolder);
    }
```
**isLongPressDragEnabled()**
设置是否长按才进入拖动
**isItemViewSwipeEnabled()**
设置是否支持滑动操作
## 简单Demo
开启振动权限
```
<uses-permission android:name="android.permission.VIBRATE"/>
```
```
public class MainActivity extends AppCompatActivity {

    private RecyclerView mRecyclerView;

    private MyAdapter mAdapter = new MyAdapter();

    private ItemTouchHelper helper;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mRecyclerView = findViewById(R.id.mRecyclerView);
        GridLayoutManager mLinearLayoutManager;
        mLinearLayoutManager = new GridLayoutManager(this, 3);

        mRecyclerView.setLayoutManager(mLinearLayoutManager);
        mRecyclerView.addItemDecoration(new DividerItemDecoration(this, LinearLayout.HORIZONTAL));
        mRecyclerView.setAdapter(mAdapter);

        helper = new ItemTouchHelper(new MyItemTouchHelperCallback(mAdapter,this));
        helper.attachToRecyclerView(mRecyclerView);

    }

    /**
     * 数据默认写死的
     */
    public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyHolder> implements OnItemPositionListener {

        private List<String> mDatas = new ArrayList<>();

        public MyAdapter() {
            for (int i = 0; i < 30; i++) {
                mDatas.add("item:" + i);
            }
        }

        @Override
        public MyHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            return new MyHolder(LayoutInflater.from(parent.getContext()).inflate(R.layout.item_recycle, parent, false));
        }

        @Override
        public int getItemCount() {
            return mDatas.size();
        }

        @Override
        public void onBindViewHolder(MyHolder holder, int position) {
            holder.tv.setText(mDatas.get(position));
        }

        @Override
        public void onItemSwap(int from, int target) {
//            Collections.swap(mDatas, from, target);
            //交换数据
            String s = mDatas.get(from);
            mDatas.remove(from);
            mDatas.add(target,s);
            notifyItemMoved(from, target);
        }

        @Override
        public void onItemMoved(int position) {
            mDatas.remove(position);
            notifyItemRemoved(position);
        }

        protected class MyHolder extends RecyclerView.ViewHolder {

            public TextView tv;

            public ImageView drag;

            public MyHolder(View itemView) {
                super(itemView);
                tv = itemView.findViewById(R.id.item_tv);
                drag = itemView.findViewById(R.id.item_drag);

//                drag.setOnTouchListener(this);

            }

/*          @Override
            public boolean onTouch(View v, MotionEvent event) {

                if (event.getAction() == MotionEvent.ACTION_DOWN) {
                    helper.startDrag(this);
                }
                return false;
            }*/
        }
    }


}
```
ItemTouchHelper.Callback的isLongPressDragEnabled方法，让其返回false，表示我们不需要默认的长按开始拖动功能。然后在我们需要开始拖动的地方调用ItemTouchHelper.startDrag(ViewHolder viewHolder)方法即可开始拖动
```
public class MyItemTouchHelperCallback extends ItemTouchHelper.Callback {

    private OnItemPositionListener mItemPositionListener;
    private Context mContext;

    public MyItemTouchHelperCallback(OnItemPositionListener itemPositionListener, Context context) {
        mItemPositionListener = itemPositionListener;
        mContext = context;
    }

    @Override
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        int dragFlag;
        int swipeFlags;
        //如果是表格布局，则可以上下左右的拖动，但是不能滑动
        if (recyclerView.getLayoutManager() instanceof GridLayoutManager) {
            dragFlag = ItemTouchHelper.UP |
                    ItemTouchHelper.DOWN |
                    ItemTouchHelper.LEFT |
                    ItemTouchHelper.RIGHT;
            swipeFlags = 0;
        }
        //如果是线性布局，那么只能上下拖动，只能左右滑动
        else {
            dragFlag = ItemTouchHelper.UP |
                    ItemTouchHelper.DOWN;
            swipeFlags = ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT;
        }

        //通过makeMovementFlags生成最终结果
        return makeMovementFlags(dragFlag, swipeFlags);
    }

    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
        //被拖动的item位置
        int fromPosition = viewHolder.getLayoutPosition();
        //他的目标位置
        int targetPosition = target.getLayoutPosition();
        //为了降低耦合，使用接口让Adapter去实现交换功能
        mItemPositionListener.onItemSwap(fromPosition, targetPosition);
        return true;
    }

    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
        //为了降低耦合，使用接口让Adapter去实现交换功能
        mItemPositionListener.onItemMoved(viewHolder.getLayoutPosition());
    }

    @Override
    public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState) {
        super.onSelectedChanged(viewHolder, actionState);

        //当开始拖拽的时候
        if (actionState != ItemTouchHelper.ACTION_STATE_IDLE) {
            viewHolder.itemView.setBackgroundColor(Color.LTGRAY);
            Vibrator vibrator = (Vibrator)mContext.getSystemService(mContext.VIBRATOR_SERVICE);
            vibrator.vibrate(50);
        }
    }

    //当手指松开的时候
    @Override
    public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        viewHolder.itemView.setBackgroundColor(Color.TRANSPARENT);
        super.clearView(recyclerView, viewHolder);
    }

    //禁止长按滚动交换，需要滚动的时候使用{@link ItemTouchHelper#startDrag(ViewHolder)}
    @Override
    public boolean isLongPressDragEnabled() {
        //return false;
        return true;
    }
}
```
```
public interface OnItemPositionListener {
    //交换
    void onItemSwap(int from, int target);

    //滑动
    void onItemMoved(int position);
}
```