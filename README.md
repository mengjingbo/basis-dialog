DialogFragment在android 3.0时被引入。是一种特殊的Fragment，用于在Activity的内容之上展示一个模态的对话框。
### 1.官方这样介绍的：
 > A fragment that displays a dialog window, floating on top of its  activity's window. This fragment contains a Dialog object, which it displays as appropriate based on the fragment's state.  Control of the dialog (deciding when to show, hide, dismiss it) should be done through the API here, not with direct calls on the dialog.
一个显示Dialog(对话框窗口)的Fragment(碎片，片段)，悬浮在当前Activity(活动)窗口之上，这个Fragment包含一个Dialog(对话框窗口)对象，它根据Fragment的状态显示合适的内容，根据提供的API控制这个Dialog的显示，隐藏，解散。而不是直接调用这个Dialog
### 2.继承以及实现类结构：
```java
  public class DialogFragment extends Fragment implements DialogInterface.OnCancelListener, DialogInterface.OnDismissListener 
```
DialogFragment提供了四种默认样式
```java
    /**
     * 一个基础的,正常样式的dialog.
     */
    public static final int STYLE_NORMAL = 0;

    /**
     * 这个样式不包括标题区。
     */
    public static final int STYLE_NO_TITLE = 1;

    /**
     * 这个样式无任何框架，在onCreateView()中进行绘制视图。
     */
    public static final int STYLE_NO_FRAME = 2;

    /**
     * 禁用所有输入对话框。用户不能触摸它，它的窗口将不能接收输入焦点。
     */
    public static final int STYLE_NO_INPUT = 3;
```
### 3.对DialogFragment进行封装，使用更加方便
- 定义一个默认主题
```html
    <style name="default_dialog_style" parent="@android:style/Theme.Dialog">
        <item name="android:windowFrame">@null</item>
        <item name="android:windowIsFloating">true</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowBackground">@android:color/transparent</item>
    </style>
```
- 这里为了方便直接使用STYLE_NO_FRAME样式，具体封装如下：
```java
package com.common.base;

import android.app.Activity;
import android.app.Dialog;
import android.app.DialogFragment;
import android.app.Fragment;
import android.app.FragmentTransaction;
import android.content.Context;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.view.Window;
import android.view.WindowManager;

import com.common.R;

/**
 * 作者：僧格洛卓
 * 时间：2017/8/14
 * 描述：基类Dialog
 */
public abstract class BaseDialog extends DialogFragment {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 设置自定义样式
        if (setDialogStyle() == 0) {
            setStyle(DialogFragment.STYLE_NO_FRAME, R.style.default_dialog_style);
        } else {
            setStyle(DialogFragment.STYLE_NO_FRAME, setDialogStyle());
        }
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, Bundle savedInstanceState) {
        // 设置进场动画
        if (setWindowAnimationsStyle() != 0) {
            getDialog().getWindow().setWindowAnimations(setWindowAnimationsStyle());
        }
        // return 视图
        View mView = null;
        if (mView == null) {
            mView = inflater.inflate(getLayoutId(), container, false);
        } else {
            if (mView != null) {
                ViewGroup mViewGroup = (ViewGroup) mView.getParent();
                if (mViewGroup != null) {
                    mViewGroup.removeView(mView);
                }
            }
        }
        return mView;
    }


    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        initView(view);
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        loadData(getArguments());
    }

    /**
     * 自定义时添加layout
     *
     * @return
     */
    protected abstract int getLayoutId();

    /**
     * 初始化
     *
     * @param view
     */
    protected abstract void initView(View view);


    /**
     * 加载数据
     *
     * @param bundle 用这个Bundle对象接收传入时的Bundle对象
     */
    protected abstract void loadData(Bundle bundle);

    /**
     * 创建视图
     *
     * @param context
     * @param resId
     * @param <T>
     * @return
     */
    protected <T extends View> T createView(Context context, int resId) {
        return (T) LayoutInflater.from(context).inflate(resId, null);
    }

    /**
     * 设置Dialog点击外部区域是否隐藏
     *
     * @param cancel
     */
    protected void setCanceledOnTouchOutside(boolean cancel) {
        if (getDialog() != null) {
            getDialog().setCanceledOnTouchOutside(cancel);
        }
    }

    /**
     * 设置Dialog gravity
     *
     * @param gravity
     */
    protected void setGravity(int gravity) {
        if (getDialog() != null) {
            Window mWindow = getDialog().getWindow();
            WindowManager.LayoutParams params = mWindow.getAttributes();
            params.gravity = gravity;
            mWindow.setAttributes(params);
        }
    }

    /**
     * 设置Dialog窗口width
     *
     * @param width
     */
    protected void setDialogWidth(int width) {
        setDialogWidthAndHeight(width, LinearLayout.LayoutParams.WRAP_CONTENT);
    }

    /**
     * 设置Dialog窗口height
     *
     * @param height
     */
    protected void setDialogHeight(int height) {
        setDialogWidthAndHeight(LinearLayout.LayoutParams.WRAP_CONTENT, height);
    }

    /**
     * 设置Dialog窗口width，height
     *
     * @param width
     * @param height
     */
    protected void setDialogWidthAndHeight(int width, int height) {
        if (getDialog() != null) {
            getDialog().getWindow().setLayout(width, height);
        }
    }

    /**
     * 设置窗口转场动画
     *
     * @return
     */
    protected int setWindowAnimationsStyle() {
        return 0;
    }

    /**
     * 设置弹出框样式
     *
     * @return
     */
    protected int setDialogStyle() {
        return 0;
    }


    /**
     * 显示Dialog
     *
     * @param activity
     * @param tag      设置一个标签用来标记Dialog
     */
    public void show(Activity activity, String tag) {
        show(activity, null, tag);
    }

    /**
     * 显示Dialog
     *
     * @param activity
     * @param bundle   要传递给Dialog的Bundle对象
     * @param tag      设置一个标签用来标记Dialog
     */
    public void show(Activity activity, Bundle bundle, String tag) {
        if (activity == null && isShowing())
            return;
        FragmentTransaction mTransaction = activity.getFragmentManager().beginTransaction();
        Fragment mFragment = activity.getFragmentManager().findFragmentByTag(tag);
        if (mFragment != null) {
            //为了不重复显示dialog，在显示对话框之前移除正在显示的对话框
            mTransaction.remove(mFragment);
        }
        if (bundle != null) {
            setArguments(bundle);
        }
        show(mTransaction, tag);
    }

    /**
     * 是否显示
     *
     * @return false:isHidden  true:isShowing
     */
    protected boolean isShowing() {
        if (this.getDialog() != null) {
            return this.getDialog().isShowing();
        } else {
            return false;
        }
    }
}
```
- 使用
```java
public class SampleDialog extends BaseDialog {

    public static final String PHONE = "phone";
    private TextView mPhoneText;
    private String mPhone;

    public static SampleDialog newInstance(String phone) {
        SampleDialog mDialog = new SampleDialog ();
        Bundle mBundle = new Bundle();
        mBundle .putString(PHONE, phone);
        mDialog.setArguments(args);
        return mDialog;
    }

    @Override
    protected int getLayoutId() {
        return R.layout.dialog_service_phone;
    }

    @Override
    protected void initView(View view) {
        mPhoneText = (TextView) view.findViewById(R.id.dialog_service_phone_tv);
        /** 其他操作 **/
    }

    @Override
    protected void loadData(Bundle bundle) {
        if (bundle != null) {
            mPhone = bundle.getString(PHONE, "");
        }
        mPhoneText.setText(mPhone);
    }

    public void show(Activity activity) {
        show(activity, "PhoneDialog");
    }
}
```
```java
    // 其他地方调用
    SampleDialog.newInstance("182*****008");
```
#### 4.设置Dialog宽高(Width，Height)
对DialogFragment设置宽高时在View初始完成后或者窗口展示出来时进行设置，也可参考如下：
```java
    @Override
    public void onResume() {
        super.onResume();
        setDialogWidth(DeviceUtils.getScreenWidth(getActivity()) - DeviceUtils.dp2px(getActivity(), 200));
    }
  // DeviceUtils.getScreenWidth(getActivity())为获取到的当前屏幕宽度
```
