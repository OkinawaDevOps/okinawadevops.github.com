---
layout: post
tags : [Android, ]
---
{% include JB/setup %}

### [yamanetoshi](https://yamanetoshi.github.io/)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/138)

### 以下でもごもごします

- [Espresso 2.0とRobolectricを併用したテスト環境の作り方](http://qiita.com/Nkzn/items/d5f30bfe31bdf329860b)

### 記録

#### とりあえず

Android Studio のバージョン上げます。check for update にて 1.2.2 になる模様。て何故か Canary という開発 current なサイトに飛ばされたので http://tools.android.com/download/studio/ から Stable を開きます。1.2.1.1 が安定版 current とのこと (2015.7.19 時点)。 

いつ 1.0 にしたのかよく覚えていないんですがもう 1.2 なのか。

#### む

[なかざんさんエントリ](http://qiita.com/Nkzn/items/d5f30bfe31bdf329860b)に「2015.6.9追記：Android Studio 1.2 + Robolectric 3の組み合わせが良さそうなので、この記事をあまり鵜呑みにしないでください」という記載がありますね。ちょっとググッてみたところ、以下なエントリを見つけたのでなぞってみます。

- [Android : Robolectric 3.0](http://yuki312.blogspot.jp/2015/05/android-robolectric-30.html)

ええと、なぞってみます。

##### Build variants

[unit testing support](http://tools.android.com/tech-docs/unit-testing-support) にある

> 4. Open ths "Build variants" tool window (on the left) and change the test artifact to "Unit tests".

というソレ。ハードコピィがあるので何とかなりました。あとは src/test/java を作って云々で Android Studio から unittest が実行できます、ということなのかな。

とりあえずハロワの UI チェックする試験書いてみます。src/test/java の中に同じパケジを作って MainActivityTest というクラスを作成。で、ハロワな TextView は View のオブジェクトを取れるようにしておいて

```
    <TextView
        android:id="@+id/textview"
        android:text="@string/hello_world"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
```

で、試験が以下なのかどうか。

```
public class MainActivityTest extends ActivityInstrumentationTestCase2<MainActivity> {

    private MainActivity mActivity;
    private TextView mHelloWorld;

    public MainActivityTest() {
        super(MainActivity.class);
    }

    @Override
    protected void setUp() throws Exception {
        super.setUp();

        mActivity = getActivity();
        mHelloWorld = (TextView)mActivity.findViewById(R.id.textview);
    }

    public void testHelloWorld() {
        assertEquals(mHelloWorld.getText(), mActivity.getString(R.string.hello_world));
    }
}
```

実行してみたら setUp の中の findViewById でぬるぽでオチました。ちょっと微妙だったので簡単なテストを書いてみた。

```
public class MainActivityTest extends ActivityTestCase {

    public void testHelloWorld() {
        assertTrue(true);
    }
}
```

これだとパスしてますね。とりあえずそもそも的なナニとして Android で UT なナニから復習します。

#### utils

というパケジを作ってその中に Utility というクラスを作り、以下なメソドを作成。

```
    public static String getPriceString(int price) {
        int size = (int)Math.log10((double)price) + 1;

        if (size > 6) {
            return "invalid size";
        }

        DecimalFormat nfCur;
        nfCur = new DecimalFormat("\u00A5###,###");
        return nfCur.format(price);
    }
```

次に src/test/java 配下に同じ名前のパケジを作って UtilityTest というクラスを作ってテスト実行してみました。動いた。

```
public class UtilityTest {

    @Test
    public void testGetPriceString() {
        final String expected = "¥1,234";

        assertEquals(Utility.getPriceString(1234), expected);
    }
}
```

#### instrumented unit testing

以下を build.gradle の defaultConfig に追加、なのかどうか。

```
testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
```

で、再度 Activity の試験に戻ってるんですが、やはり getActivity が null を戻してて涙目。

### 問題点

二点。

- AndroidJUnit4 が解決できない
- getActivity メソドが null を戻す

引き続き問題解決対応の方向。しかし困った。
