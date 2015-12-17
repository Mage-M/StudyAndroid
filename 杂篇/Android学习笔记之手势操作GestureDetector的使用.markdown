# Android学习笔记之手势操作GestureDetector的使用

标签（空格分隔）： Android

---

GestureDetector对外提供了两个接口和一个外部类：
接口：OnGestureListener、OnDoubleTapListener
内部类：SimpleOnGestureListener

但是大致的使用方法是一样的，区别只是如果你实现上面两个接口则需要自己重写其中所有的抽象方法。
继承下面的类的话，只需要重写自己需要用到的方法即可，因为该类也是实现了上面的两个接口，只是在它内部已经将所有的抽象方法全部重写，不用自己重写而已。

##OnGestureListener中有如下方法：

```
private class gesturelistener implements GestureDetector.OnGestureListener{  

    //用户按下屏幕就会触发
    public boolean onDown(MotionEvent e) {  
        // TODO Auto-generated method stub  
        return false;  
    }  
    
    //按下超过瞬时，且按下时没有松开或者拖动，执行该方法。
    //但是超过多少瞬时，查不到
    public void onShowPress(MotionEvent e) {  
        // TODO Auto-generated method stub  
          
    }  
  
    //一次单独的轻击屏幕，即轻击屏幕，立即抬起才会触发，有其他操作则不触发
    public boolean onSingleTapUp(MotionEvent e) {  
        // TODO Auto-generated method stub  
        return false;  
    }  
  
  
    //在屏幕上拖动事件，无论是用手拖动View，或者是拖的动作滚动，都会多次触发该方法
    public boolean onScroll(MotionEvent e1, MotionEvent e2,  
            float distanceX, float distanceY) {  
        // TODO Auto-generated method stub  
        return false;  
    }  
  
    //长按触摸屏，超过一定时长，就会触发这个事件
    public void onLongPress(MotionEvent e) {  
        // TODO Auto-generated method stub  
          
    }  
  
    //滑屛，用户按下触摸屏后，快速移动后松开
    //由一个down，多个move，一个up组成
    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,  
            float velocityY) {  
        // TODO Auto-generated method stub  
        return false;  
    }  
      
}  
```

ps：
1.onSingleTapUp(MotionEvent e)：
触发顺序：
    点击一下非常快的（不滑动）Touchup：
    onDown->onSingleTapUp->onSingleTapConfirmed 
    点击一下稍微慢点的（不滑动）Touchup：
    onDown->onShowPress->onSingleTapUp->onSingleTapConfirmed

2.onLongPress(MotionEvent e)：
触发顺序：
    onDown->onShowPress->onLongPress
3.onScroll(MotionEvent e1, MotionEvent e2,float distanceX, float distanceY)：
 滑屏：手指触动屏幕后，稍微滑动后立即松开
    onDown-----》onScroll----》onScroll----》onScroll----》………----->onFling
    拖动
    onDown------》onScroll----》onScroll------》onFiling

    可见，无论是滑屏，还是拖动，影响的只是中间OnScroll触发的数量多少而已，最终都会触发onFling事件！

##OnDoubleTapListener中有如下方法：
```
private class doubleTapListener implements GestureDetector.OnDoubleTapListener{  
  
    //单击事件
    public boolean onSingleTapConfirmed(MotionEvent e) {  
        // TODO Auto-generated method stub  
        return false;  
    }  
  
    //双击事件
    public boolean onDoubleTap(MotionEvent e) {  
        // TODO Auto-generated method stub  
        return false;  
    }  
  
    //双击间隔中发生的动作，包含down、move和up事件
    public boolean onDoubleTapEvent(MotionEvent e) {  
        // TODO Auto-generated method stub  
        return false;  
    }  
}  
```
##PS:
onSingleTapConfirmed(MotionEvente)：
用来判定该次点击是SingleTap而不是DoubleTap，如果连续点击两次就是DoubleTap手势，如果只点击一次，系统等待一段时间后没有收到第二次点击则判定该次点击为SingleTap而不是DoubleTap，然后触发SingleTapConfirmed事件。
触发顺序是：OnDown->OnsingleTapUp->OnsingleTapConfirmed

##SimpleOnGestureListener中实现了以上两个接口中的所有方法，只不过实现全部为空而已。

##使用步骤：(这里给出一种常用的使用步骤)
###1.在使用的activity中定义一个私有类继承需要的接口，如下：
```
private class gestureListener implements GestureDetector.OnGestureListener{  
  
        public boolean onDown(MotionEvent e) {  
            Log.i("MyGesture", "onDown");     
            Toast.makeText(MainActivity.this, "onDown", Toast.LENGTH_SHORT).show();     
            return false;  
        }  
  
       
        public void onShowPress(MotionEvent e) {  
            Log.i("MyGesture", "onShowPress");     
            Toast.makeText(MainActivity.this, "onShowPress", Toast.LENGTH_SHORT).show();     
        }  
  
      
        public boolean onSingleTapUp(MotionEvent e) {  
            Log.i("MyGesture", "onSingleTapUp");     
            Toast.makeText(MainActivity.this, "onSingleTapUp", Toast.LENGTH_SHORT).show();     
            return true;     
        }  
  
             
        public boolean onScroll(MotionEvent e1, MotionEvent e2,  
                float distanceX, float distanceY) {  
            Log.i("MyGesture22", "onScroll:"+(e2.getX()-e1.getX()) +"   "+distanceX);     
            Toast.makeText(MainActivity.this, "onScroll", Toast.LENGTH_LONG).show();     
              
            return true;     
        }  
  
             
        public void onLongPress(MotionEvent e) {  
             Log.i("MyGesture", "onLongPress");     
             Toast.makeText(MainActivity.this, "onLongPress", Toast.LENGTH_LONG).show();     
        }  
  
             
        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,  
                float velocityY) {  
            Log.i("MyGesture", "onFling");     
            Toast.makeText(MainActivity.this, "onFling", Toast.LENGTH_LONG).show();     
            return true;  
        }  
    }; 
```
###2.在你的Activity中定义一个GestureDetector对象。
```
private GestureDetector mGestureDetector;  

mGestureDetector = new GestureDetector(new gestureListener());
```
在实例化这个对象时使用你自己定义的这个私有类来new出对象。

###3.重写Activity的onTouchEvent()方法，用自定义的手势识别器，也就是自己new出来的这个对象监听event事件，如下：
```
public boolean onTouchEvent(MotionEvent event){   
        mGestureDetector.onTouchEvent(event);  
        // Be sure to call the superclass implementation  
        return super.onTouchEvent(event);  
    }  
```
###4.这样，当我们出发event事件时，手势识别器就会识别我们做出的手势了。

###注：如果使用SimpleOnGestureListener，第一个步骤也可以写成一个匿名内部类，如下：
```
mGestureDetector = new GestureDetector(this, new GestureDetector.SimpleOnGestureListener() {

			//e1代表手指第一次触摸屏幕的事件，e2代表手指离开屏幕一瞬间的事件
			//velocityX水平方向的速度 单位pix/s， velocityY竖直方向的速度
			@Override
			public boolean onFling(MotionEvent e1, MotionEvent e2,
					float velocityX, float velocityY) {
				
				if(Math.abs(velocityX) < 200) {
					Toast.makeText(getApplicationContext(), "无效动作，移动太慢", Toast.LENGTH_SHORT).show();
					return true;
				}
				
				if((e2.getRawX() - e1.getRawX()) > 200) {
					//从左向右滑动屏幕，显示上一个界面
					showPre();
					overridePendingTransition(R.anim.pre_in, R.anim.pre_out);
					return true;
				}
				
				if((e1.getRawX() - e2.getRawX()) > 200) {
					//从右向左滑动屏幕，显示下一个界面
					showNext();
					overridePendingTransition(R.anim.next_in, R.anim.next_out);
					return true;
				}
				
				return super.onFling(e1, e2, velocityX, velocityY);
			}
			
		});
```
上面我们使用了SimpleOnGestureListener，只重写了其中的onFling()方法,其他方法不用重写，因为SimpleOnGestureListener已经帮我们重写好了所有的类。
然后直接按照第三步去监听即可。