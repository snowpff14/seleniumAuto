# 使い方
## 導入
* pythonをインストール(3.7で確認しています)
* 以下のコマンドでライブラリのインストールを行う
```
pip install numpy
pip install selenium
pip install pandas
pip install openpyxl
pip install xlrd
```

## 使い方
継承します。各クラスの説明は
[qiita](https://qiita.com/snowp/items/fcda8c5d590017858bdb)参照
同じ内容を以下に記載しています。

# 実装説明

## インスタンス作成処理
処理の説明
* driverはクラスのインスタンス作成の際引数で渡すことで呼び出し元で制御します。
* waitは画面の要素が待つ時間を設定します。`time.sleep`で待機させるコードをよく見かけますが、こちらの方法なら指定したものが表示したりクリックできるようになり次第即座に操作を行えるようになります。
* logは独自に作ったlogようのクラスです。loggingを使って作っています。
* screenShotBaseNameはスクリーンショットの接頭辞に使っています。


```
    driver=None
    wait=None
    log=None
    # スクリーンショットを格納するディレクトリ
    screenShotBaseName=None

    # 初期化処理
    def __init__(self,driver,log,screenShotBaseName='screenShot'):
        self.driver=driver
        # 画面描画の待ち時間
        self.wait=WebDriverWait(self.driver,20)
        driver.implicitly_wait(30)
        self.log=log
        self.screenShotBaseName=screenShotBaseName+'/'+datetime.now().strftime("%Y%m%d%H%M%S")+'/'
        os.makedirs(self.screenShotBaseName,exist_ok=True)

```

## 画面操作処理 クリック

画面のクリック処理です。
継承先のクラスでこの処理を呼び出すことで要素がクリックできるまで待ってクリックするようになり、失敗した場合はスクリーンショットを取るようにしています。
指定ミスなどで要素がない場合は20秒ほどでエラーになります。

```
    # 画面要素をクリックする 要素が表示されるまで20秒待つ さらに待ち時間が必要な場合は指定を行う
    def webElementClickWaitDisplay(self,webElement,waitTime=0):
        try:
            time.sleep(waitTime)
            self.wait.until(expected_conditions.element_to_be_clickable((By.XPATH,webElement)))
            self.driver.find_element_by_xpath(webElement).click()
        except RuntimeError as err:
            self.log.error('画面要素押下失敗:'+webElement)
            self.log.error('例外発生 {}'.format(err))
            self.getScreenShot()
            raise
        except:
            self.outputException(webElement)
            raise
```
## 画面操作処理 クリック マウス移動

画面によってはクリック後マウスを移動させないとメニューが出ないなどある場合は
以下のようにクリック後マウスを移動させます。

```
    # 画面要素をクリックしてマウスを移動させる クリックした後のメニューの操作を行いたいときに使用する
    # 待ち時間が必要な場合は指定を行う
    # 画面の要素がクリックできるまでまつ
    def webElementClickAndMoveWaitDisplay(self,webElement,waitTime=0):
        try:
            time.sleep(waitTime)
            self.wait.until(expected_conditions.element_to_be_clickable((By.XPATH,webElement)))
            target = self.driver.find_element_by_xpath(webElement)
            actions = ActionChains(self.driver)
            actions.click(target).move_by_offset(10,10).perform()
        except RuntimeError as err:
            self.log.error('画面要素押下失敗:'+webElement)
            self.log.error('例外発生 {0}'.format(err))
            self.getScreenShot()
            raise
        except:
            self.outputException(webElement)
            raise

```

## 画面操作処理 テキスト送信

テキスト送信処理
クリックできるまで待ってから送信しています。

```
    # テキストを送る 要素が表示されるまで20秒待つ さらに待ち時間が必要な場合は指定を行う
    def sendTextWaitDisplay(self,webElement,sendTexts,waitTime=0):
        try:
            time.sleep(waitTime)
            self.wait.until(expected_conditions.element_to_be_clickable((By.XPATH,webElement)))
            self.driver.find_element_by_xpath(webElement).clear()
            self.driver.find_element_by_xpath(webElement).send_keys(sendTexts)
        except RuntimeError as err:
            self.log.error('テキスト送信失敗:'+webElement)
            self.log.error('例外発生 {}'.format(err))
            self.getScreenShot()
            raise
        except:
            self.outputException(webElement)
            raise
```

## 画面操作処理 スクロール

画面でスクロールしたい時用
FireFoxの場合画面の範囲外の要素を操作しようとするとエラーになることがあるので
こちらの処理でスクロールして実施しました。
またスクリーンショットを取る時に微妙に動かしたいときのためスクロール処理も共通化しました。
正の値で上に負の値で下に移動します。


**FireFoxのみ確認Chromeではまだ未検証**

```
    # 指定した位置までスクロールを行う
    def moveScroll(self,webElement):
        # スクロール確認の処理 指定した位置までスクロールを行う
        try:
            self.wait.until(expected_conditions.visibility_of_element_located((By.XPATH,webElement)))
            inputLabel=self.driver.find_element_by_xpath(webElement)
            self.driver.execute_script("arguments[0].scrollIntoView();", inputLabel)

        except RuntimeError as err:
            self.log.error('画面スクロール失敗:'+webElement)
            self.log.error('例外発生 {0}'.format(err))
            self.getScreenShot()
            raise
        except:
            self.outputException(webElement)
            raise

    # 画面をスクロールさせる
    def adjustScroll(self,offset):
        try:
            self.driver.execute_script("window.scrollTo(0, window.pageYOffset +"+str(offset)+")" )

        except RuntimeError as err:
            self.log.error('画面スクロール失敗:'+str(offset))
            self.log.error('例外発生 {0}'.format(err))
            self.getScreenShot()
            raise
        except:
            self.log.error(traceback.format_exc())
            self.getScreenShot()
            raise
```

# 実際例

## 実装例

以下のように`SeleniumOperationBase`を継承して`super()`で呼び出すことができます。

```
class Login(SeleniumOperationBase):

    def __init__(self,driver,log,screenShotBaseName='screenShotName'):
        super().__init__(driver,log,screenShotBaseName)

    def loginMethod(self,userId,password):

        self.driver.get(TARGET_URL)
        useridForm=self.driver.find_element_by_xpath(LOGIN_USER_ID)
        useridForm.clear()
        super().sendTextWaitDisplay(LOGIN_USER_ID,userId)
        passwordForm=self.driver.find_element_by_xpath(LOGIN_PASSWORD)
        passwordForm.clear()
        super().sendTextWaitDisplay(LOGIN_PASSWORD,userId)
        super().webElementClickWaitDisplay(LGOIN_LOGINBUTTON)
```
