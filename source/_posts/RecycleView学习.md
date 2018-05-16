---

title: RecycleView学习
date: 2018-05-16 13:54:08
tags: RecycleView
categories: 学习知识
description: RecycleView 学习笔记

---


参考 [Android RecyclerView 使用完全解析 体验艺术般的控件](http://blog.csdn.net/lmj623565791/article/details/45059587)
和
[RecycleView的基本使用](http://www.jianshu.com/p/804790b4c957)

##概述
RecycleView 在 support-v7包下

其作用是用来代替 ListView和GridView

其优点是:

    提供了一种插拔式的体验，高度的解耦，异常的灵活，通过设置它提供的不同LayoutManager，ItemDecoration , ItemAnimator实现令人瞠目的效果。

例如: 

1,你想要控制其显示的方式，请通过布局管理器LayoutManager

2,你想要控制Item间的间隔（可绘制），请通过ItemDecoration

3,你想要控制Item增删的动画，请通过ItemAnimator

缺点是: 

	想要实现点击、长按事件，需要自己定义

##基本使用方法

	mRecyclerView = findView(R.id.id_recyclerview);
	//设置布局管理器
	mRecyclerView.setLayoutManager(layout);
	//设置adapter
	mRecyclerView.setAdapter(adapter)
	//设置Item增加、移除动画
	mRecyclerView.setItemAnimator(new DefaultItemAnimator());
	//添加分割线
	mRecyclerView.addItemDecoration(new DividerItemDecoration(
	                getActivity(), DividerItemDecoration.HORIZONTAL_LIST));

###Just like ListView(主要流程的代码)

主界面:

		recyclerView = (RecyclerView) findViewById(gridRv);

        initData();
        //设置布局适配器
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        //设置adapter
        mAdapter = new RecycleViewListAdapter(this, mDatas);
        recyclerView.setAdapter(mAdapter);

adapter:

	public class RecycleViewListAdapter extends RecyclerView.Adapter {
	    private List<String> mDatas;
	    private Context mContext;
	
	    public RecycleViewListAdapter(Context mContext, List<String> mDatas) {
	        this.mContext = mContext;
	        this.mDatas = mDatas;
	    }
	
	    public void setImages(List<String> mDatas) {
	        this.mDatas = mDatas;
	        notifyDataSetChanged();
	    }
	
	    @Override
	    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
	        View view = LayoutInflater.from(mContext).inflate(R.layout.item_list_recycleview, parent, false);
	        MyViewHolder myViewHolder = new MyViewHolder(view);
	        return myViewHolder;
	    }
	
	    @Override
	    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
	        MyViewHolder myViewHolder = (MyViewHolder) holder;
	        myViewHolder.tv.setText(mDatas.get(position));
	    }
	
	    @Override
	    public int getItemCount() {
	        return mDatas.size();
	    }
	
	    class MyViewHolder extends RecyclerView.ViewHolder {
	        TextView tv;
	        public MyViewHolder(View view) {
	            super(view);
	            tv = (TextView) view.findViewById(R.id.recy_tv);
	        }
	    }
	}

###ItemDecoration(添加分割线)
主要方法为:  mRecyclerView.addItemDecoration(ItemDecoration)

该方法的参数为RecyclerView.ItemDecoration，该类为抽象类,该类的源码为: 

	public static abstract class ItemDecoration {
	
		public void onDraw(Canvas c, RecyclerView parent, State state) {
		            onDraw(c, parent);
		 }
		
		
		public void onDrawOver(Canvas c, RecyclerView parent, State state) {
		            onDrawOver(c, parent);
		 }
		
		public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
		            getItemOffsets(outRect, ((LayoutParams) view.getLayoutParams()).getViewLayoutPosition(),
		                    parent);
		}
		
		@Deprecated
		public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent){
		            outRect.set(0, 0, 0, 0);
		}
	}

当我们调用mRecyclerView.addItemDecoration(ItemDecoration)方法添加decoration的时候，RecyclerView在绘制的时候，去会绘制decorator，即调用该类的onDraw和onDrawOver方法;

1,onDraw方法先于drawChildren

2,onDrawOver在drawChildren之后，一般我们选择复写其中一个即可。

3,getItemOffsets 可以通过outRect.set()为每个Item设置一定的偏移量，主要用于绘制Decorator。

需要自定义 可以参考: [Android 自定义RecyclerView 实现真正的Gallery效果](http://blog.csdn.net/lmj623565791/article/details/38173061)

###LayoutManager

RecyclerView.LayoutManager, 这是一个抽象类，系统提供了3个实现类;

1,LinearLayoutManager 现行管理器，支持横向、纵向。

2,GridLayoutManager 网格布局管理器

3,StaggeredGridLayoutManager 瀑布就式布局管理器

使用GridLayoutManager , 代码为:

	//mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
    mRecyclerView.setLayoutManager(new GridLayoutManager(this,4));

只需要修改LayoutManager即可

但是改为GridLayoutManager以后，对于分割线，前面的DividerItemDecoration就不适用了，主要是因为它在绘制的时候，比如水平线，针对每个child的取值为:

	final int left = parent.getPaddingLeft();
	final int right = parent.getWidth() - parent.getPaddingRight();

因为每个Item一行，这样是没问题的。而GridLayoutManager时，一行有多个childItem，这样就多次绘制了，并且GridLayoutManager时，Item如果为最后一列（则右边无间隔线）或者为最后一行（底部无分割线）。

这样的话需要修改gridview中的分割线item

可以参考: [文章开头的地址](http://blog.csdn.net/lmj623565791/article/details/45059587)

主要在getItemOffsets方法中，去判断如果是最后一行，则不需要绘制底部；如果是最后一列，则不需要绘制右边，整个判断也考虑到了StaggeredGridLayoutManager的横向和纵向，所以稍稍有些复杂。最重要还是去理解，如何绘制什么的不重要。一般如果仅仅是希望有空隙，还是去设置item的margin方便。

StaggeredGridLayoutManager可以有三种用法
使用, StaggeredGridLayoutManager,代码: StaggeredGridLayoutManager.VERTICAL

	// mRecyclerView.setLayoutManager(new GridLayoutManager(this,4));
    mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(4, StaggeredGridLayoutManager.VERTICAL));

这两种写法显示的效果是一致的，但是注意StaggeredGridLayoutManager构造的第二个参数传一个orientation，如果传入的是StaggeredGridLayoutManager.VERTICAL代表有多少列；那么传入的如果是StaggeredGridLayoutManager.HORIZONTAL就代表有多少行，比如本例如果改为：StaggeredGridLayoutManager.HORIZONTAL

	mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(4,StaggeredGridLayoutManager.HORIZONTAL));

固定为4行，变成了左右滑动。有一点需要注意，如果是横向的时候，item的宽度需要注意去设置，毕竟横向的宽度没有约束了，应为控件可以横向滚动了。 

适用在需要横向滚动的情况下的listview 或者 gridview

###条目动画 ItemAnimator

item增加、删除的动画也是可配置的。

ItemAnimator也是一个抽象类，好在系统为我们提供了一种默认的实现类

	// 设置item动画
	mRecyclerView.setItemAnimator(new DefaultItemAnimator());

PS:这里更新数据集不是用adapter.notifyDataSetChanged()

而是 notifyItemInserted(position)与notifyItemRemoved(position) 

否则没有动画效果。

可以为adapter中添加了两个方法：
	public void addData(int position) {
	        mDatas.add(position, "Insert One");
	        notifyItemInserted(position);
	    }
	
	    public void removeData(int position) {
	        mDatas.remove(position);
	        notifyItemRemoved(position);
	    }
	}

因为只有一种动画, 所以可以去网上搜索一些比较好看的动画效果

[RecyclerView的一些动画效果](https://github.com/search?l=Java&o=desc&p=1&q=RecyclerView&s=stars&type=Repositories&utf8=✓)

[一个比较好的动画的library](https://github.com/CymChad/BaseRecyclerViewAdapterHelper)

###Click and LongClick(点击事件,,,recyclerview没有提供,所以需要自己写)

实现的方式比较多，你可以通过mRecyclerView.addOnItemTouchListener去监听然后去判断手势， 
当然你也可以通过adapter中自己去提供回调，这里我们选择后者，前者的方式，大家有兴趣自己去实现。

	class HomeAdapter extends RecyclerView.Adapter<HomeAdapter.MyViewHolder> {
	
	//...
	    public interface OnItemClickLitener {
	        void onItemClick(View view, int position);
	        void onItemLongClick(View view , int position);
	    }
	
	    private OnItemClickLitener mOnItemClickLitener;
	
	    public void setOnItemClickLitener(OnItemClickLitener mOnItemClickLitener) {
	        this.mOnItemClickLitener = mOnItemClickLitener;
	    }
	
	    @Override
	    public void onBindViewHolder(final MyViewHolder holder, final int position) {
	        holder.tv.setText(mDatas.get(position));
	
	        // 如果设置了回调，则设置点击事件
	        if (mOnItemClickLitener != null) {
	            holder.itemView.setOnClickListener(new OnClickListener() {
	                @Override
	                public void onClick(View v) {
	                    int pos = holder.getLayoutPosition();
	                    mOnItemClickLitener.onItemClick(holder.itemView, pos);
	                }
	            });
	
	            holder.itemView.setOnLongClickListener(new OnLongClickListener() {
	                @Override
	                public boolean onLongClick(View v) {
	                    int pos = holder.getLayoutPosition();
	                    mOnItemClickLitener.onItemLongClick(holder.itemView, pos);
	                    return false;
	                }
	            });
	        }
	    }
	//...
	}

adapter中自己定义了个接口，然后在onBindViewHolder中去为holder.itemView去设置相应 
的监听最后回调我们设置的监听。

然后在activity中添加监听

	mAdapter.setOnItemClickLitener(new OnItemClickLitener(){
	
	            @Override
	            public void onItemClick(View view, int position) {
	                Toast.makeText(HomeActivity.this, position + " click",
	                        Toast.LENGTH_SHORT).show();
	            }
	
	            @Override
	            public void onItemLongClick(View view, int position) {
	                Toast.makeText(HomeActivity.this, position + " long click",
	                        Toast.LENGTH_SHORT).show();
	                        mAdapter.removeData(position);
	            }
	        });
	}
