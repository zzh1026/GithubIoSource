---
title: RecyclerView中LayoutManager的妙用
comments: false
date: 2018-06-05 11:08:35
tags:
categories:
    - Android
    - 转载学习
    - LayoutManager

---

# 简介 #

RecyclerView在现在都已经非常熟悉了，项目必备，其中的LayoutManager可以自定义出各种炫酷的效果：

[实现炫酷列表动画的最佳姿势 - 你的RecyclerView还可以这样](https://www.jianshu.com/p/20f16a4b4630 "你的RecyclerView还可以这样-钉某人")

其github地址为：[LayoutManagerGroup](https://github.com/DingMouRen/LayoutManagerGroup)

今天，主要是对抖音效果的分析和使用

<!-- more -->

# 效果 #

![抖音](https://upload-images.jianshu.io/upload_images/4876639-50f91a8bb140ad3b.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/223 "抖音gif")

时下最火的莫过抖音了，实现这个效果应该很简单嘛，用ViewPager就可以了。但是等你通过ViewPager来实现的时候，手机内存不够用的情况就会显现出来。

有没有更好的方式呢？？？

自然是有，每个人都会用RecyclerView吧，我们就用RecyclerView来实现这个效果，关于内存的回收利用就交给RecyclerView就好了。

那么我们怎么通过RecyclerView来实现这个效果呢？如果你使用过SnapHelper的话，就会很好理解。

# 自定义过程 #

## 1.自定义LayoutManager，并且继承LinearLayoutManager,这样就得到一个可以水平排向或者竖向排向的布局策略。 ##

如果你接触过SnapHelper应该了解一个LinearSnapHelper的类，可以实现让列表的Item居中显示的效果。但是这里我们不用这个类，我们要的效果是一次只能滑动一个Item，也就是一页。PagerSnapHelper就可以做到哦。

    @Override
    public void onAttachedToWindow(RecyclerView view) {
        super.onAttachedToWindow(view);
        mPagerSnapHelper.attachToRecyclerView(view);
        this.mRecyclerView = view;
    }

## 2.经过第一步基本可以实现抖音的效果，但是写完之后一会发现，不知道哪里来开始播放视频和在哪里释放视频。 ##

不要着急，要监听滑动到哪页，需要我们重写onScrollStateChanged（）函数，这里面有三种状态：

- SCROLL_STATE_IDLE（空闲）
- SCROLL_STATE_DRAGGING（拖动）
- SCROLL_STATE_SETTLING（要移动到最后位置时）

我们需要的就是RecyclerView停止时的状态，我们就可以拿到这个View的Position.注意这里还有一个问题，当你通过这个position去拿Item会报错，这里涉及到RecyclerView的缓存机制，自己去脑补~~。

打印Log,你会发现RecyclerView.getChildCount()一直为1或者会出现为2的情况。好了，我们自己来实现一个接口然后通过接口把状态传递出去。

监听器

    public interface OnViewPagerListener {
        /*释放的监听*/
        void onPageRelease(boolean isNext,int position);
        /*选中的监听以及判断是否滑动到底部*/
        void onPageSelected(int position,boolean isBottom);
        /*布局完成的监听*/
        void onLayoutComplete();
    }

获取到RecyclerView空闲时选中的Item,重写LinearLayoutManager的onScrollStateChanged方法

    @Override
    public void onScrollStateChanged(int state) {
      switch (state) {
          case RecyclerView.SCROLL_STATE_IDLE:
              View viewIdle = mPagerSnapHelper.findSnapView(this);
              int positionIdle = getPosition(viewIdle);
              if (mOnViewPagerListener != null && getChildCount() == 1) {
                  mOnViewPagerListener.onPageSelected(positionIdle,positionIdle == getItemCount() - 1);
              }
              break;
          case RecyclerView.SCROLL_STATE_DRAGGING:
              View viewDrag = mPagerSnapHelper.findSnapView(this);
              int positionDrag = getPosition(viewDrag);
              break;
          case RecyclerView.SCROLL_STATE_SETTLING:
              View viewSettling = mPagerSnapHelper.findSnapView(this);
              int positionSettling = getPosition(viewSettling);
              break;
      }
    }

## 3.列表的选中监听好了 ##

我们看看什么时候释放视频的资源，第二步中的三种状态，去打印getChildCount()的日志，你会发现getChildCount()在: 

- SCROLL_STATE_DRAGGING会为1
- SCROLL_STATE_SETTLING为2
- SCROLL_STATE_IDLE有时为1，有时为2

还是RecyclerView的缓存机制，这里不会去赘述缓存机制，我们要做的是要知道在什么时候去做释放视频的操作，还要分清是释放上一页还是下一页，因为适配器adapter的position在这里不好使嘛，这里有两个方法scrollHorizontallyBy（）和scrollVerticallyBy（）可以拿到滑动偏移量，可以判断滑动方向，好~ 齐活了。

看代码:

        /**
          * 监听竖直方向的相对偏移量
          * @param dy
          * @param recycler
          * @param state
          * @return
          */
         @Override
         public int scrollVerticallyBy(int dy, RecyclerView.Recycler recycler, RecyclerView.State state) {
             this.mDrift = dy;
             return super.scrollVerticallyBy(dy, recycler, state);
         }
         /**
          * 监听水平方向的相对偏移量
          * @param dx
          * @param recycler
          * @param state
          * @return
          */
         @Override
         public int scrollHorizontallyBy(int dx, RecyclerView.Recycler recycler, RecyclerView.State state) {
             this.mDrift = dx;
             return super.scrollHorizontallyBy(dx, recycler, state);
         }
        // 可以释放资源的监听，也就是回收Item的时候
        private RecyclerView.OnChildAttachStateChangeListener mChildAttachStateChangeListener = new RecyclerView.OnChildAttachStateChangeListener() {
             @Override
             public void onChildViewAttachedToWindow(View view) {
             }
             @Override
             public void onChildViewDetachedFromWindow(View view) {
                 if (mDrift >= 0){
                     if (mOnViewPagerListener != null) mOnViewPagerListener.onPageRelease(true,getPosition(view));
                 }else {
                     if (mOnViewPagerListener != null) mOnViewPagerListener.onPageRelease(false,getPosition(view));
                 }
             }
         };

是不是觉得很不可思议就好了，贴一哈具体使用的代码，初始化视频和释放视频的地方：

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_view_pager_layout_manager);
        initView();
        initListener();
    }
    private void initView() {
        mRecyclerView = findViewById(R.id.recycler);
        mLayoutManager = new ViewPagerLayoutManager(this, OrientationHelper.VERTICAL);
        mAdapter = new MyAdapter();
        mRecyclerView.setLayoutManager(mLayoutManager);
        mRecyclerView.setAdapter(mAdapter);
    }
    private void initListener(){
        mLayoutManager.setOnViewPagerListener(new OnViewPagerListener() {
            @Override
            public void onPageRelease(boolean isNext,int position) {
                Log.e(TAG,"释放位置:"+position +" 下一页:"+isNext);
                int index = 0;
                if (isNext){
                    index = 0;
                }else {
                    index = 1;
                }
                releaseVideo(index);
            }
            @Override
            public void onPageSelected(int position,boolean isBottom) {
                Log.e(TAG,"选中位置:"+position+"  是否是滑动到底部:"+isBottom);
                playVideo(0);
            }
            @Override
            public void onLayoutComplete() {
                playVideo(0);
            }
        });
    }

# 源码 #

[LayoutManagerGroup](https://github.com/DingMouRen/LayoutManagerGroup "各种LayoutManager定制的效果")


[作者的其他自定义View - 自定义view](https://www.jianshu.com/nb/25316846)