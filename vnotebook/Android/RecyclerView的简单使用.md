# RecyclerView的简单使用

## 1. 简介
* 定义Google用于代替ListView的滑动组件
* 特点：功能更强大、支持定制样式更丰富、扩展性更高
* 缺点： RecyclerView实现控制点击、长按事件较为麻烦，需要自己写

**与ListView作对比**
![](_v_images/20200202104339079_6415.png =640x)

## 2. 使用步骤
* 1. 定义主xml布局
* 2. 根据需要定义RecyclerView每行所实现的xml布局
* 3. 定义一个Adapter类继承RecyclerView.Adapter，重写里面的方法。
* 4. 定义一个HashMap构成的列表，将数据以键值对的方式存放在里面。
* 5. 构造Adapter对象，设置适配器。
* 6. 将RecyclerView绑定到Adapter上。


## 3. 案例
### 3.1 导入依赖
```
    implementation 'com.google.android.material:material:1.1.0-alpha09'
```
### 3.2 在布局文件中添加布局
```
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
```
### 3.3 编写item_detail.xml布局文件
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="wrap_content"

    android:layout_margin="6dp">

    <ImageView
        android:id="@+id/detail_iv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        />
    <TextView
        android:id="@+id/detail_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:maxLines="2"
        android:text="Android：这是一份全面  详细的RecylerView学习攻略"/>

</LinearLayout>
```
### 3.4 继承RecyclerView.Adapter类
```
import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import org.gpnu.recyclerview.R;
import org.gpnu.recyclerview.model.Bean;

import java.util.List;


public class ImageAdapter extends RecyclerView.Adapter<ImageAdapter.MyViewHolder> {

    private Context mContext;    
    private List<Bean> mList;    //数据集

    public ImageAdapter(Context mContext, List<Bean> mList) {
        this.mContext = mContext;
        this.mList = mList;
    }

    @NonNull
    @Override
    public MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {

        View view = LayoutInflater.from(mContext).inflate(R.layout.detail_item,parent,false);    
        MyViewHolder viewHolder = new MyViewHolder(view);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(@NonNull MyViewHolder holder, int position) {
        //绑定数据到item中
       // Bean bean = mList.get(position);
        //holder.textView.setText(bean.getTitle());
        
    }

    @Override
    public int getItemCount() {
        return 50;
    }

    //复用
    class MyViewHolder extends RecyclerView.ViewHolder {

        ImageView imageView;
        TextView textView;

        public MyViewHolder(@NonNull View itemView) {
            super(itemView);
            //绑定到布局
            imageView = itemView.findViewById(R.id.detail_iv);
            textView = itemView.findViewById(R.id.detail_tv);
        }
    }
}

```

### 3.5 Activity中调用
```
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.DefaultItemAnimator;
import androidx.recyclerview.widget.DividerItemDecoration;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import android.os.Bundle;

import org.gpnu.recyclerview.adapter.ImageAdapter;
import org.gpnu.recyclerview.model.Bean;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private RecyclerView mRecyclerView;
    private List<Bean> mList;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mRecyclerView = findViewById(R.id.recycler_view);

        mList = new ArrayList<>();
        ImageAdapter adapter = new ImageAdapter(this,mList);

        //设置布局管理器
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(layoutManager);
        /*
        布局管理器，设置为LinearLayoutManager、GridLayoutManager、StaggeredGridLayoutManager
        可以轻易地实现ListView，GridView以及流式布局的列表效果。
        */

        //添加分割线
        mRecyclerView.addItemDecoration(new DividerItemDecoration(this, layoutManager.getOrientation()));//用类设置分割线

        //设置Item增加、移除动画
        mRecyclerView.setItemAnimator(new DefaultItemAnimator());

        //设置adapter
        mRecyclerView.setAdapter(adapter);
    }
}

```

**Demo下载地址：[https://github.com/PurcellHH/RecyclerViewDemo](https://github.com/PurcellHH/RecyclerViewDemo)**

## 4. 参考
**简书：[https://www.jianshu.com/p/c821f9bb370d](https://www.jianshu.com/p/c821f9bb370d)**