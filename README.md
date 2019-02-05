# mamorio-sdk-andorid
### プロジェクト初期設定手順
* プロジェクトのbuild.gradleにMAMORIOリポジトリを追加する
```javascript
allprojects {
    repositories {
        maven {url "https://raw.githubusercontent.com/otoshimono/mamorio-sdk-android-bin/master/"}
    }
}
```

* アプリのbuild.gradleにMamorioSDKを追加する
```javascript
dependencies {
    implementation 'jp.mamorio:mamorioSDK:8.16'
}
```

### 基本的な使い方
app_tokenをセットします
ApplicationのonStartで実行して下さい

```java
MamorioSDK.setUp(getBaseContext(), "APP_TOKEN");
User.signUp("mailaddress", "password", "password_confirm", new User.UserCallback() {
	@Override
	public void onSuccess(User user) {
		//アカウント登録成功時の処理
		Log.d("APP","ユーザー登録に成功しました。以後、ユーザー情報はローカルに保存されログインの必要はありません。MAMORIOをペアリングし、レンジングを開始しましょう。");
	}

	@Override
	public void onError(Error error) {
		//アカウント登録失敗時の処理
		Log.d("APP","ユーザー登録に失敗しました。エラーメッセージ：" + error.getMessage());
	}
});
```

MAMORIOをペアリングせず、捜索専用ユーザーとして使用を開始するのであれば、メールアドレスとパスワードは不要です

```java

User.signUpAsTemporalUser(new User.UserCallback() {
	@Override
	public void onSuccess(User user) {
		//アカウント登録成功時の処理
		Log.d("APP","捜索専用ユーザーの登録に成功しました。ペアリングは行わず、周囲を高精度で探索し続け「みんなでさがす」に貢献します。");
	}

	@Override
	public void onError(Error error) {
		//アカウント登録失敗時の処理
		Log.d("APP","捜索専用ユーザーの登録に失敗しました。エラーメッセージ：" + error.getMessage());
	}
});

```

ペアリングを開始するとデバイスの一番近く（1m以内）にあるMAMORIOをチェックし、所有者がいない場合はこのユーザーが所有者となります

```java
MamorioSDK.pairingStart(new MamorioSDK.FinderCallback() {
	@Override
	public void onSuccess(Mamorio mamorio) {
		Log.d("APP","ペアリングに成功し、ユーザーはこのMAMORIOの所有者となりました。");
	}

	@Override
	public void onError(Error error) {
		Log.d("APP","ペアリングに失敗しました。エラーメッセージ：" + error.getMessage());
	}
});

//mamorioの情報を更新し、保存しましょう
mamorio.setName("MAMORIO");//名前
mamorio.setNotification(true); //通知をするか否か
mamorio.setExitDuration(40);//紛失後の待機時間。通知が頻繁な時は長めに、通知が遅い場合は短く設定しましょう。
mamorio.update(new Mamorio.Callback() {
	@Override
	public void onSuccess() {
		Log.d("APP","mamorioの情報を更新しました。以後、どこからこのmamorioを参照しても上記の設定を参照できます。");
	}

	@Override
	public void onError(Error error) {
		Log.d("APP","mamorioの情報の更新に失敗しました。エラーメッセージ：" + error.getMessage())
	}
});
```

レンジングを開始し、周囲の自分と他人のMAMORIOを監視しましょう

```java
MamorioSDK.rangingStart(
	new MamorioSDK.RangingInitializeCallback() {
		@Override
		public void onSuccess() {
			Log.d("APP","レンジングの開始に成功しました。以後、SDKはMAMORIOを発見した時にはonMamoriosEnterを、自分のMAMORIOを見失った際はonMamorioExitをコールバックします。");
		}
		@Override
		public void onError(Error error) {
			Log.d("APP","レンジングの開始に失敗しました。エラーメッセージ："+error.getMessage());
		}
	},
	//MAMORIO発見時のコールバック（null可）
	new MamorioSDK.RangingCallbackEnter() {
		@Override
		public void onEnter(List<Mamorio> list) {
			//list:発見したMAMORIOの一覧
			//nullの場合コールバックしない
		}
	},
	//MAMORIO紛失時のコールバック(null可)
	new MamorioSDK.RangingCallbackExit() {
		@Override
		public void onExit(List<Mamorio> list) {
			//list:紛失した自分のMAMORIO一覧
		}
	});
```

### MamorioSDK
* public static User getCurrentUser();

```
現在ログイン中のユーザーオブジェクトを返す
ログイン情報はshared preferenceに保存され、User.signOutメソッドを呼び出すまでログインしたままになる
ログインしていない場合nullを返す
```

* public static void setUp(Context context, String app_token);

```
MamorioSDKの初期設定を行う
ApplicationのonCreateで呼び出すこと
```

* public static void pairingStart(final FinderCallback callback);

```
ペアリングを開始する
自分か他人がペアリング済みのMAMORIOは除外する
対象のMAMORIOを発見した場合、初期設定後にonSuccessが呼ばれる
未ログインの場合onErrorが呼ばれる
一時ユーザーにてログインしている場合onErrorが呼ばれる
ネットワーク接続が不可の場合onErrorが呼ばれる
ペアリングされてないMAMORIOが複数見つかった場合onErrorが呼ばれる
このメソッドを同時に複数実行するとonErrorが呼ばれる
nearestBeaconFinderStartのコールバック待ち時に実行するとonErrorが呼ばれる
近くにMAMORIOが無い場合onErrorが呼ばれる

```

* public static void pairingStop();

```
pairingStartで始めたペアリングを中断する
```

* public static void nearestBeaconFinderStart(final FinderCallback callback);

```
端末近くのMAMORIOを検索する
MAMORIOを見つけると、onSuccessが呼ばれる
近くにMAMORIOが無い場合onSuccessが呼ばれ、Mamorioがnullになる
このメソッドを同時に複数実行するとonErrorが呼ばれる
pairingStartのコールバック待ち時に実行するとonErrorが呼ばれる
ネットワーク接続が不可の場合onErrorが呼ばれる
MAMORIOが複数見つかった場合onErrorが呼ばれる
```

* public static void nearestBeaconFinderStop();

```
nearestBeaconFinderStopで始めたMAMORIO検索を中断する
```

* public static void rangingStart(RangingInitializeCallback initializeCallback,RangingCallbackEnter callbackEnter,RangingCallbackExit callbackExit);

```
レンジングを開始する
端末周辺のMAMORIOを監視し、サーバーへ送信する
「.mamorio」という名称でプロセスを起動し、レンジングを開始する
MamorioSDK使用アプリが複数インストールされている場合、代表して一つのアプリがプロセスを起動する
起動済みのプロセスがある場合、既存プロセスに接続する
アプリケーションが終了し、メモリから削除されてもプロセスは起動し続ける
callbackEnterがnullでない場合、MAMORIO発見時にコールバックされる
callbackExitがnullでない場合、MAMORIO紛失時にコールバックされる
コールバックは別プロセスからIntent経由で呼ばれるため、
レンジング中にOSによってアプリケーションのプロセスが終了する事がある
そういった場合でもコールバックを受け取れるようにApplicationのonCreateで呼び出すことを推奨

例：
  public class MyApplication extends Application{
    @Override
    public void onCreate(){
      super.onCreate();
      Mamorio.setUp(getBaseContext(),"APP_TOKEN");
      if(Mamorio.getCurrentUser() != null){
        MamorioSDK.startRanging(...);
      }
    }
  }

onEnterコールバックについて
一定時間(30秒程度)のうちに発見したMAMORIOを全て配列として渡す
ひとつも発見できなかった場合は呼ばれない
自分のMAMORIOはMamorioSDK.getCurrentUser().getMamorios()に入っているオブジェクトの参照
自分以外のMAMORIOは、getMajor(),getMinor(),isNotYours()以外の値が不定となる
自分以外のMAMORIOの所有者の有無を確認するには、refreshを呼び出してサーバに問い合わせてから、isUnowned()の値を確認する

onExitコールバックについて
自分のMAMORIOのうち、紛失したと判断されたMAMORIOを配列として渡す
一度紛失となったものは再び発見されるまでこのコールバックに渡ってこない

```

* public static void rangingStop();

```
レンジングを停止する
```

* public static int todaysCrossingMamoriosCount();

```
本日すれ違ったMAMORIOのユニーク数を返す
```


### User
MAMORIOのユーザーを表すモデルであり、基本的にシングルトンでMamorioSDK.getCurrentUser()で参照します

* public static void signUp(String email, String password, String password_confirmation, final UserCallback callback);

```
ユーザーを新規登録する
新規登録完了時にonSuccessが呼ばれ、ログイン状態になる
ネットワーク接続不可の場合onErrorが呼ばれる
ユーザー重複、メールフォーマット不正の場合onErrorが呼ばれる
```

* public static void signUpAsTemporalUser(final UserCallback callback);

```
探索専用の一時ユーザーとしてログインする
ログイン完了時にonSuccessが呼ばれ、ログイン状態になる
ネットワーク接続不可の場合onErrorが呼ばれる
```

* public static void signIn(String auth_token, final UserCallback callback);
* public static void signIn(String email, String password, final UserCallback callback);

```
既存ユーザーとしてログインする
ログイン完了時にonSuccessが呼ばれ、ログイン状態になる
メールアドレス未認証の場合、onErrorが呼ばれる。この時error.getErrorKey() の値が"unconfirmed"になる。
authToken不正、ユーザー存在せず、パスワード不正の場合onErrorが呼ばれる
ネットワーク接続不可の場合onErrorが呼ばれる
```

* public static void signOut(final Callback callback);

```
サインアウトし、ログイン情報を削除する
```

* public static void resetPassword(String email,final Callback callback);

```
パスワードリセットURLをメールアドレスに送信する
```

* public static void newPassword(String password,final Callback callback);

```
パスワードを変更する
```

* public String getEmail();

```
メールアドレスを取得する
```

* public void setEmail(String email);

```
メールアドレスを設定する
```

* public String getAuthToken();

```
authTokenを取得する
```

* public int getId();

```
idを取得する
```

* public String getSynchronizeMode();

```
レンジング動作モードを取得する
User.SYN_NORMAL    : 自動調整
User.SYN_DILIGENT  : 高精度
User.SYN_LAZY      : 省電力

```

* public void setSynchronizeMode(String synchronize_mode);

```
レンジング動作モードを設定する
```

* public List<Mamorio> getMamorios();

```
自分のMAMORIOを取得する
最新の情報を取得する時はrefreshMamoriosを呼ぶこと
```

* public boolean isTemporal();

```
一時ユーザーでログインしている場合trueになる
```

* public void refreshMamorios(Callback callback);

```
サーバから所有MAMORIO一覧を取得し、最新のレンジング結果をマージしてgetMamoriosの値を更新する
このメソッドで取得したMamorioオブジェクトのperipheralHistoryは最新1-2件
```

* public void update(final Callback callback);

```
setEmail,setSynchronizeModeで設定した値をサーバーに送信する
設定変更完了時にonSuccessが呼ばれる
ネットワーク接続不可の場合onErrorが呼ばれる
```

* public void refresh(final Callback callback);

```
サーバーからユーザー情報を取得する
```

### Mamorio
MAMORIOそれ自体を表すモデルです
MAMORIOはmajor-minorがキーになっており、major,minorの値域はそれぞれ0-65535

* public int getMajor();
* public int getMinor();

```
majorとminorを取得する
```

* public String getStatus();

```
STATUS_VANILLA  : 誰にも所有されていない状態
STATUS_VERIFIED : 誰かが所有している状態
```

* public boolean isMinsaga();
* public void setMinsaga(boolean minsaga);

```
MAMORIOを「みんなで探す」状態にする
この値はgetPeripheralHistoriesの1件目のstatusがexitの場合のみ有効
```

* public String getName();
* public void setName(String name);

```
MAMORIOの名称を取得・設定する
```

* public boolean isNotification();
* public void setNotification(boolean notification);

```
紛失時の通知を表示するか否かを取得・設定する
通知を鳴らすアプリを実装する場合は、 onExitでコールバックされたMAMORIOのうち
isNotification()がtrueのもののみ通知を鳴らすこと
```

* public double getExitDuration();
* public void setExitDuration(double exit_duration);

```
MAMORIOの通知間隔を秒数で設定・取得する
ENTER状態のMAMORIOは、BLEの電波を最後に受信してからこの時間経過した時にEXITとなる
```

* public boolean isNotYours();

```
この値がtrueの場合、他人のMAMORIOであることを意味する
他人のMAMORIOはgetMajor(),getMinor(),isUnowned()以外の値が不定となる
```

* public boolean isUnowned();

```
この値がtrueの場合、所有者がおらず、ペアリングが可能であることを意味する
```

* public int getCategoryId();
* public void setCategoryId( int category_id );

```
MAMORIOのカテゴリIDを取得・設定する
CATEGORY_WALLET : 財布
CATEGORY_BAG    : カバン
CATEGORY_CAMERA : 電子機器
CATEGORY_KEY    : 鍵
CATEGORY_BIKE   : 自転車
CATEGORY_PET    : ペット
CATEGORY_HUMAN  : 人
CATEGORY_OTHERS : その他
```

* public String getImageUrl();

```
画像がアップロード済みの場合ダウンロード可能なURLを取得できる
httpで始まらない場合、画像未設定
可能な限りアプリ内で画像をキャッシュし、URLへのアクセスを最低限にすること
```

* public Bitmap getImage();
* public void setImage(Bitmap image);

```
MAMORIOの画像用アイコンを取得、設定する
設定した画像はupdateメソッドでサーバーにアップロードできる
```

* public List<PeripheralHistory> getPeripheralHistories();

```
MAMORIOの過去の発見履歴を発見日時降順で取得する
この履歴は次のタイミングで追加される
・自分のMAMORIOを発見し、前回のPeriphearlHistoryから一時間以上経過しているとき
・自分のMAMORIOを紛失したとき
・自分のMAMORIOを紛失し「isMinsaga()==true」時に、他人がMAMORIOを発見したとき
・紛失した自分のMAMORIOを発見したとき
ただし、最初の一件は上記条件に関わりなく直近の発見履歴となる
```

* public static void get(int major, int minor, final MamorioCallback callback);
* public void refresh(Callback c);

```
MAMORIOの情報を更新する
このメソッドで取得/更新したMamorioのperipheralHistoryは過去24時間の最大11件
```

* public void update(final Callback callback);

```
下記メソッドで設定した値をサーバに送信する
ただし、setImageで更新した画像を送信する場合は、事前にsetImageUpdatedを呼び出すこと
setCategoryId
setExitDuration
setName
setNotification
setMinsaga
setImage
```

* public void delete(final Callback callback);

```
MAMORIOのペアリングを解除する
```

### PeripheralHistory
MAMORIOの過去の発見履歴の１つ１つを表します

* public Date getDetectedAt();

```
MAMORIOを発見、紛失した日時
```

* public double getLatitude();

```
MAMORIOを発見した端末の緯度
```

* public double getLongitude();

```
MAMORIOを発見した端末の経度
```

* public double getPrecision();

```
緯度・経度の正確さ(メートル)
```

* public String getStatus();

```
MAMORIOの状態を取得する
STATUS_ENTER    : 自分で発見した
STATUS_CLOUDON  : 自分以外の誰かが発見した
STATUS_EXIT     : 紛失した
STATUS_SUBMIT   : 新規登録が完了した
```

* public String getLocation();

```
発見位置の名称を取得する
MAMORIOアンテナが発見した場合その名称、
それ以外が発見した場合はnullとなる
```
