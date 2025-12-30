# PythonのSeleniumでCAPTCHAを回避する

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/)

本ガイドでは、SeleniumでCAPTCHAを回避する方法を解説します。

- [一般的なCAPTCHAの種類](#common-captcha-types)
- [SeleniumでのCAPTCHA対処：ステップバイステップチュートリアル](#selenium-captcha-handling-step-by-step-tutorial)
  - [ステップ #1：新しいPythonプロジェクトを作成する](#step-1-create-a-new-python-project)
  - [ステップ #2：Seleniumをインストールする](#step-2-install-selenium)
  - [ステップ #3：Seleniumスクリプトをセットアップする](#step-3-set-up-your-selenium-script)
  - [ステップ #4：ブラウザ自動化ロジックを追加する](#step-4-add-the-browser-automation-logic)
  - [ステップ #5：Selenium Stealthプラグインをインストールする](#step-5-install-the-selenium-stealth-plugin)
  - [ステップ #6：CAPTCHAを回避するためにStealth設定を構成する](#step-6-configure-the-stealth-settings-to-avoid-captchas)
  - [ステップ #7：ボット検出テストを再実行する](#step-7-repeat-the-bot-detection-test)
 - [上記の解決策が機能しない場合](#what-if-the-above-solution-does-not-work)


## CAPTCHAとは？


[**CAPTCHA**](https://brightdata.jp/blog/web-data/what-is-a-captcha)（Completely Automated Public Turing test to tell Computers and Humans Apart）は、人間のユーザーとボットを区別するために使用されます。人間には簡単ですが機械には難しい課題を提示します。代表的なプロバイダーには、Google reCAPTCHA、hCaptcha、BotDetect などがあります。

## Common CAPTCHA Types

- **Text-based**: 歪んだ文字/数字を入力します。  
- **Image-based**: グリッド内の特定オブジェクトを識別します。  
- **Audio-based**: 音声クリップの単語を入力します。  
- **Puzzle-based**: 簡単なパズル（例：迷路）を解きます。  

![Text CAPTCHA example](https://github.com/luminati-io/bypass-captcha-with-selenium/blob/main/images/Text-CAPTCHA-example.png)

多くの場合、CAPTCHAはフォーム送信の最終ステップで表示されます。

![CAPTCHA in form submission](https://github.com/luminati-io/bypass-captcha-with-selenium/blob/main/images/CAPTCHA-as-a-step-of-a-form-submission-process-example.png)

これは、自動化ボットがアクションを完了するのを防ぐためです。CAPTCHA解決サービスは存在しますが、**ユーザー体験が悪い**ため、ハードコードされたCAPTCHAは稀です。

CAPTCHAは、**Web Application Firewalls (WAFs)** のようなより広範なセキュリティ対策の一部です。

![Web Application Firewall](https://github.com/luminati-io/bypass-captcha-with-selenium/blob/main/images/Example-of-a-Web-Application-Firewall-1024x488.png)

これらのシステムは、不審なアクティビティを検知するとCAPTCHAをトリガーします。これらを回避するには、ボットが**人間の行動を模倣**する必要があり、そのためにはスクリプトの頻繁な更新が必要です。 

## Selenium CAPTCHA Handling: Step-By-Step Tutorial

ブラウザを制御しながら人間の行動を模倣するための最良のツールの1つが、人気のブラウザ自動化ライブラリである [Selenium](https://www.selenium.dev/) です。Pythonスクリプトを使って、SeleniumでCAPTCHAを回避する方法を学びましょう。

### Step #1: Create a New Python Project

本ガイドに従うには、ローカルに Python 3 と Crhome がインストールされている必要があります。

すでにSeleniumのWebスクレイピングまたはテスト用スクリプトをお持ちの場合は、最初の3ステップをスキップしてください。そうでない場合は、SeleniumのCAPTCHA回避デモプロジェクト用のフォルダを作成し、ターミナルウィンドウでそこに移動します。

```bash
mkdir selenium_demo

cd selenium_demo
```

次に、その中に新しい [Python virtual environment](https://docs.python.org/3/library/venv.html) を追加します。

```bash
python -m venv venv
```

お好みのPython IDEでプロジェクトフォルダを開き、`script.py` という名前の新しいファイルを作成します。

### Step #2: Install Selenium

[Python virtual environment](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/#activate-a-virtual-environment) を有効化します。

Windowsの場合：

```powershell
venv\Scripts\activate
```

LinuxまたはmacOSの場合：

```bash
source venv/bin/activate
```

Seleniumをインストールします。

```bash
pip install selenium
```

### Step #3: Set Up Your Selenium Script

`script.py` に次の行を追加してSeleniumをインポートします。

```python
from selenium import webdriver
```

Chromeをヘッドレスモードで起動するように構成するため、[`ChromeOptions`](https://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.chrome.options) オブジェクトを作成します。

```python
options = webdriver.ChromeOptions()

options.add_argument("--headless")
```

これらのオプションで [Chrome WebDriver](https://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.chrome.webdriver) インスタンスを初期化し、最後に `quit()` で閉じます。現時点の `script.py` は次のようになるはずです。

```python
from selenium import webdriver

# configure Chrome to start in headless mode

options = webdriver.ChromeOptions()

options.add_argument("--headless")

# start a Chrome instance

driver = webdriver.Chrome(options=options)

# browser automation logic...

# close the browser and release its resources

driver.quit()
```

上記スクリプトは、ヘッドレスモードで新しいChromeインスタンスを起動し、その後ブラウザを閉じます。

### Step #4: Add the Browser Automation Logic

SeleniumのCAPTCHA回避ロジックをテストするために、スクリプトは **[bot.sannysoft.com](https://bot.sannysoft.com/)** に接続してスクリーンショットを取得します。このページでは、訪問者が人間かボットかを検出するためのさまざまなテストが実行されます。標準的なブラウザでアクセスすると、すべてのテストに合格します。

Chromeに対象ページへアクセスさせるには、[`get()`](https://selenium-python.readthedocs.io/api.html#selenium.webdriver.remote.webdriver.WebDriver.get) メソッドを使用します。

```python
driver.get("https://bot.sannysoft.com/")
```

Seleniumにはページ全体のスクリーンショットを取得する組み込み関数がありません。回避策として、スクリーンショットを撮る前にブラウザウィンドウサイズを `<body>` のサイズに合わせて調整します。


```python
# get the body width and height

full_width = driver.execute_script("return document.body.parentNode.scrollWidth")

full_height = driver.execute_script("return document.body.parentNode.scrollHeight")

# set the browser window to the body width and height

driver.set_window_size(full_width, full_height)

# take a screenshot of the entire page

driver.save_screenshot("screenshot.png")

# restore the original window size

driver.set_window_size(original_size["width"], original_size["height"])
```

上記の工夫で十分であり、`screenshot.png` にはページ全体のスクリーンショットが含まれます。

これらをまとめると、次のロジックになります。

```python
from selenium import webdriver

# configure Chrome to start in headless mode

options = webdriver.ChromeOptions()

options.add_argument("--headless")

# start a Chrome instance

driver = webdriver.Chrome(options=options)

# connect to the target page

driver.get("https://bot.sannysoft.com/")

# get the current window size

original_size = driver.get_window_size()

# get the body width and height

full_width = driver.execute_script("return document.body.parentNode.scrollWidth")

full_height = driver.execute_script("return document.body.parentNode.scrollHeight")

# set the browser window to the body width and height

driver.set_window_size(full_width, full_height)

# take a screenshot of the entire page

driver.save_screenshot("screenshot.png")

# restore the original window size

driver.set_window_size(original_size["width"], original_size["height"])

# close the browser and release its resources

driver.quit()
```

上記の `script.py` をこのコマンドで起動します。

```bash
python script.py
```

スクリプトは **ヘッドレスモードのChromiumインスタンス** を起動し、対象ページに移動してスクリーンショットを取得し、その後ブラウザを閉じます。実行後、プロジェクトのルートフォルダに `screenshot.png` ファイルが作成されます。

![screenshot.png file example](https://github.com/luminati-io/bypass-captcha-with-selenium/blob/main/images/screenshot.png-file-example-206x1024.png)

赤枠で示されているとおり、ヘッドレスモードのChromeは複数の検出テストに失敗します。つまり、スクリプトがボットとしてフラグ付けされる可能性が高く、保護されたサイトではCAPTCHAチャレンジにつながります。

検出を回避してCAPTCHAを防ぐには、Stealthプラグインを使用します。

### Step #5: Install the Selenium Stealth Plugin

[Selenium Stealth](https://github.com/diprajpatra/selenium-stealth) は、Seleniumで制御されたChrome/Chromiumがボットとして検出される可能性を下げるPythonパッケージです。ブラウザのプロパティを変更して自動化の痕跡（automation leaks）を防ぐことで、アンチボット検出の回避に役立ちます。

Selenium Stealthは、人間の行動を模倣して検出を回避するために、さまざまなブラウザ属性を変更します。Puppeteer Stealthと似たように機能しますが、Selenium向けに設計されています。

`pip` でSelenium Stealthをインストールします。

```sh
pip install selenium-stealth
```

次に、`script.py` ファイルにこの行を追加してライブラリをインポートします。

```python
from selenium_stealth import stealth
```

### Step #6: Configure the Stealth Settings to Avoid CAPTCHAs

Selenium Stealthを登録し、CAPTCHAを回避するようにChrome WebDriverを構成するには、`stealth()` 関数を呼び出します。

```python
stealth(

driver,

user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36",

languages=["en-US", "en"],

vendor="Google Inc.",

platform="Win32",

webgl_vendor="Intel Inc.",

renderer="Intel Iris OpenGL Engine",

fix_hairline=True,

)
```

必要に応じて関数引数を設定してください。ただし、上記の値でほとんどのアンチボット防御を回避するには十分である点に注意してください。

これらの設定により、Seleniumで制御されるブラウザは実ユーザーのブラウザを模倣します。

### Step #7: Repeat the Bot Detection Test

以下が最終的な `script.js` ファイルです。

```python
from selenium import webdriver

from selenium_stealth import stealth

# configure Chrome to start in headless mode

options = webdriver.ChromeOptions()

options.add_argument("--headless")

# start a Chrome instance

driver = webdriver.Chrome(options=options)

# configure the WebDriver to avoid bot detection

# with Selenium Stealth

stealth(

driver,

user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36",

languages=["en-US", "en"],

vendor="Google Inc.",

platform="Win32",

webgl_vendor="Intel Inc.",

renderer="Intel Iris OpenGL Engine",

fix_hairline=True,

)

# connect to the target page

driver.get("https://bot.sannysoft.com/")

# get the current window size

original_size = driver.get_window_size()

# get the body width and height

full_width = driver.execute_script("return document.body.parentNode.scrollWidth")

full_height = driver.execute_script("return document.body.parentNode.scrollHeight")

# set the browser window to the body width and height

driver.set_window_size(full_width, full_height)

# take a screenshot of the entire page

driver.save_screenshot("screenshot.png")

# restore the original window size

driver.set_window_size(original_size["width"], original_size["height"])

# close the browser and release its resources

driver.quit()
```

CAPTCHA回避のSelenium Pythonスクリプトを再度実行します。

```bash
python script.py
```

` screenshot.png` を確認すると、すべてのボット検出テストに合格していることがわかります。

![All bot detection tests passed on the new screenshot.png](https://github.com/luminati-io/bypass-captcha-with-selenium/blob/main/images/All-bot-detction-tests-passed-on-the-new-screenshot-249x1024.png)

## What If the Above Solution Does Not Work?

アンチボットシステムはブラウザ設定だけを分析するわけではありません。IPレピュテーションが重要です。無料ライブラリでIPを切り替えるだけではうまくいきません。Seleniumへのプロキシ統合が必要です。

最適に構成されたブラウザでも、CAPTCHAが表示されることがあります。基本的な reCAPTCHA v2 チャレンジであれば、[selenium-recaptcha-solver](https://pypi.org/project/selenium-recaptcha/) を試せますが、これらのライブラリは古く、制限があります。

基本的な手法は、Cloudflareのような複雑なアンチボットシステムに対しては通用しません。堅牢な解決策としては、reCAPTCHA、hCaptcha、px_captcha、SimpleCaptcha、GeeTest、FunCaptcha、Cloudflare Turnstile、AWS WAF Captcha、KeyCAPTCHA などに対応する Bright Data のWebスクレイピングツールを使用してください。

[Bright Data’s CAPTCHA Solver](https://brightdata.jp/products/web-unlocker/captcha-solver) は、Seleniumを含む任意のHTTPクライアントまたはブラウザ自動化ツールで動作します。

## Conclusion

Selenium Stealthライブラリのおかげで、Chromeのデフォルト設定を上書きしてボット検出を抑制できます。しかし、このアプローチは決定的な解決策ではありません。高度なボット検出ツールは、依然としてブロックできてしまいます。本当の解決策は、CAPTCHAなしの任意のWebページのHTMLを返せるアンブロッキングAPIを介して対象サイトに接続することです。

そのような解決策の1つが [Web Unlocker](https://brightdata.jp/products/web-unlocker) です。これはスクレイピングAPIで、プロキシ統合によりリクエストごとに出口IPを自動でローテーションし、ブラウザフィンガープリント、自動リトライ、CAPTCHA解決を処理します。CAPTCHA対応がこれまでになく簡単になります。

今すぐサインアップして、無料トライアルを開始してください。