# Windows Internals

https://tryhackme.com/room/windowsinternals

目的:
- Windows のプロセスとその基盤技術を理解し、触れてみる。
- コアとなるファイル フォーマットとその使用方法について学ぶ。
- Windows の内部構造に触れ、カーネルの動作を理解する。

## Task 2 - プロセス

ここではプロセス モニターを使用してプロセスの情報を確認するというタスクを実施します。

*Open the provided file: "Logfile.PML" in Procmon and answer the questions below.*

デスクトップの ProcessMonitor フォルダの下に Logfile.PML という名前のファイルがあるのでこれを開くのですが、Procmon とのファイル関連付けがされていないので、Procmon を起動して "File" - "Open" から開くか、一度 Procmon を起動した後に Logfile.PML ファイルをダブルクリックしてログ ファイルを開きます。

*What is the process ID of "notepad.exe"?*

メニューの "Filter" - "Filter" を選択するか、漏斗のアイコンをクリックして Filter ダイアログを表示させます。
そして次のようなエントリを追加します ("Add" ボタンを押すこと!)。

- Process Name
- is
- notepad.exe
- Include

![fig1.png](images/fig1.png)

そうすると notepad.exe のログだけが表示され、`PID 列をみると notepad.exe のプロセス ID がわかります。

![fig2.png](images/fig2.png)

*What is the parent process ID of the previous process?*

notepad.exe のログであればどれでもいいのですが、例えば一番先頭の `Process Start` をダブルクリックします。

表示されたダイアログの `Process` タブをクリックし、`Parent PID` 欄をみると、親プロセスの ID がわかります。

![fig3.png](images/fig3.png)

*What is the integrity level of the process?*

先ほどのダイアログの `Integrity` 欄みると整合性レベルがわかります。

通常は、ユーザーが普通にプロセスを起動した場合の整合性レベルは「中」 (ストア アプリなどは別)、管理者として実行したり Administrator ユーザーがプロセスを起動した場合は「高」になります。

参考:
- [必須整合性制御](https://learn.microsoft.com/ja-jp/windows/win32/secauthz/mandatory-integrity-control)
