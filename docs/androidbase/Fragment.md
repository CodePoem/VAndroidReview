# [Fragmnet](https://developer.android.google.cn/guide/components/fragments?hl=zh_cn)

## 特点

* Fragment 解决 Activity 间的切换不流畅，轻量切换
* 可以从 startActivityForResult 中接收到返回结果，但是View不能
* 只能在 Activity 保存其状态 onSaveInstanceState() 之前使用 commit() 提交事务。如果您试图在该时间点后提交，则会引发异常。 这是因为如需恢复 Activity，则提交后的状态可能会丢失。 对于丢失提交无关紧要的情况，请使用 commitAllowingStateLoss()。

## [生命周期](https://developer.android.google.cn/guide/components/fragments?hl=zh_cn#Creating)

![Fragment 生命周期](https://developer.android.google.cn/images/fragment_lifecycle.png?hl=zh_cn)

## 通信

### 与Activity通信

Fragment -> Activity ：

* getActivity()

Activity -> Fragment ：

* FragmentManager findFragmentById() 或 findFragmentByTag()

#### 接口回调通信

例如，如果某个新闻应用的 Activity 有两个 Fragment，其中一个用于显示文章列表（FragmentA），另一个用于显示文章（FragmentB），则 FragmentA 必须在列表项被选定后告知 Activity，以便它告知 FragmentB 显示该文章。

##### 定义接口

FragmentA 定义接口用于回调。

```kotlin
public class FragmentA : ListFragment() {
    ...
    // Container Activity must implement this interface
    interface OnArticleSelectedListener {
        fun onArticleSelected(articleUri: Uri)
    }
    ...
}
```

##### 实现接口

FragmentA 的宿主 Activity 会实现 OnArticleSelectedListener 接口并重写 onArticleSelected()，将来自 FragmentA 的事件通知 FragmentB。

```kotlin
public class MainActivity : Activity(), FragmentA.OnArticleSelectedListener {
    // ...

    fun onAttachFragment(fragment: Fragment) {
        if (fragment is FragmentA) {
            fragment.onArticleSelected(this)
        }
    }

    fun onArticleSelected(articleUri: Uri) {
            val articleFrag = supportFragmentManager.findFragmentById(R.id.article_fragment) as ArticleFragment?

            if (articleFrag != null) {
                articleFrag.updateArticleView(articleUri)
            } else {
                val newFragment = ArticleFragment()
                val args = Bundle()
                args.putInt(ArticleFragment.ARG_ARTICLE_URI, articleUri)
                newFragment.arguments = args

                val transaction = supportFragmentManager.beginTransaction()

                transaction.replace(R.id.fragment_container, newFragment)
                transaction.addToBackStack(null)

                transaction.commit()
            }
        }
}
```

##### 确保实现接口

```kotlin
public class FragmentA : ListFragment() {

    var listener: OnArticleSelectedListener? = null
    ...
    override fun onAttach(context: Context) {
        super.onAttach(context)
        listener = context as? OnArticleSelectedListener
        if (listener == null) {
            throw ClassCastException("$context must implement OnArticleSelectedListener")
        }

    }
    ...
}
```
