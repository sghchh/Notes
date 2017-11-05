#安卓开源项目打对勾的自定义view--TickView学习  
最近看到了一个打对勾的安卓自定义控件，有兴趣所以学习了一波。在这里就说一说它的实现思路。  
![](https://user-gold-cdn.xitu.io/2017/10/22/eebd03f003e9241c2859bfa86eb8bd27?imageView2/0/w/1280/h/960/ignore-error/1)  
说一下实现的思路：  
1.在非选中状态下绘制一个圆环和对勾  
2.在选中状态下绘制一个圆  
3.绘制一个对勾  
4.实现一个放大的效果  
所有以上的动态的实现都是作者**基于属性动画**来实现的，这也是这个自定义控件的特别之处。首先看看自定义的属性：  
 
	<?xml version="1.0" encoding="utf-8"?>
	<resources>
    <declare-styleable name="TickView">
        <attr name="uncheck_base_color" format="color"/>
        <attr name="check_base_color" format="color"/>
        <attr name="check_tick_color" format="color"/>
        <attr name="radius" format="dimension"/>
        <attr name="rate">
            <enum name="slow" value="0"/>
            <enum name="normal" value="1"/>
            <enum name="fast" value="2"/>
        </attr>
        <attr name="clickable" format="boolean"/>
    </declare-styleable>
	</resources>  
我们重点就看一看是怎么通过属性动画实现的：  
###动态绘制的圆环  

	//画圆弧进度
        canvas.drawArc(mRectF, 90, ringProgress, false, mPaintRing);
绘制圆环核心是通过drawArc这个方法来实现的，drawArc的本意是绘制圆弧，作者是怎么通过这个方法来绘制一个圆环的呢？答案就在那个ringProgress参数，作者在自定义view中设置了一个实例变量ringProgress，这个变量代表了绘制的度数，没错，就是通过属性动画来为这个度数设置一个范围，这样就会动态将这个圆环绘制出来了。  

	ObjectAnimator mRingAnimator = ObjectAnimator.ofInt(this, "ringProgress", 0, 360);
        mRingAnimator.setDuration(mRingAnimatorDuration);
        mRingAnimator.setInterpolator(null);  
就是通过自定义view的initAnimator方法中的这段代码实现的。但是属性动画的实现需要对应的属性提供setter和getter方法：  

	private int getRingProgress() {
        return ringProgress;
    }

    private void setRingProgress(int ringProgress) {
        this.ringProgress = ringProgress;
        postInvalidate();
    }  
	这里的作用就是提供setter和getter方法来实现属性动画，而这里的postInvalidate方法的作用就是通知重新刷新界面。  
其他的动画实际上也是通过这种类似的方式实现的。  
###降低代码的耦合  
该项目的作者在降低代码的耦合方面做得也是值得借鉴的：各种自定义属性以及判断控件状态的变量，作者都把他们放到了另一个类中，这样一来就降低了代码的耦合度。大大提高了代码的可读性。  
###尝试使用流式代码方式  
本文精彩的地方还有一个就是初始化属性值的时候，使用了流式模式来进行，一下子可读性和实用性都上升了，直接借鉴。  

	private void setupDefaultValue(Context context)
    {
        this.setUnCheckBaseColor(Color.parseColor("#ffeaeaea"))
                .setCheckBaseColor(Color.parseColor("#fff5d747"))
                .setCheckTickColor(Color.WHITE)
                .setRadius(DisplayUtil.dp2px(context,30))
                .setClickable(true)
                .setTickRadius(DisplayUtil.dp2px(context,12))
                .setTickRadiusOffset(DisplayUtil.dp2px(context,4));
    }
每一个单独的方法都返回该类的对象。