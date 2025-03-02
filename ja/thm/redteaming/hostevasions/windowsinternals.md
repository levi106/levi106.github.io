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

## Task 3 - スレッド

*What is the thread ID of the first thread created by notepad.exe?*

スレッドが作成されると `Thread Create` のイベントが出ます。
一番最初の `Thread Create` のイベントの `Detail` 欄に表示されているスレッド ID が回答になります。

![fig4.png](images/fig4.png)

*What is the stack argument of the previous thread? *

スタック アーギュメントが何かわからないのですが、前の問題の回答のスレッドを作成したスレッドの ID を答えれば良いようです。

`Thread Create` のイベントをダブルクリックし、`Event` タブの `Thread` 欄をみると、呼び出し元スレッドの ID を確認することができます。

![fig5.png](images/fig5.png)

## Task 4 - Virtual Memory

*What is the total theoretical maximum virtual address space of a 32-bit x86 system?*

Windows 11 にはもう x86 版 (32bit 版) はありませんが、以前のバージョンの Windows には x86 版があり、その仮想アドレス空間のサイズは 4GB です。

- [仮想アドレス空間（メモリ管理）](https://learn.microsoft.com/ja-jp/windows/win32/memory/virtual-address-space)

*What default setting flag can be used to reallocate user process address space?*

これは「x86 版 Windows の仮想アドレス空間のユーザーモードの既定のサイズは 2GB だが、サイズを増やすためのフラグは何か」という質問のようです。

BCDEdit を使って `increaseuserva` オプションを設定すると、最大 3GB までユーザーモードのサイズを増やすことができます。全体で 4GB なのは変わりないので、その分システムで使用できるサイズは減ります。

- [4 GB のチューニング: BCDEdit と Boot.ini](https://learn.microsoft.com/ja-jp/windows/win32/memory/4-gigabyte-tuning)

*What is the base address of "notepad.exe"?*

`C:\Windows\System32\notepad.exe` の `Load Image` のイベントをダブルクリックして開くと、`Image Base` 欄に notepad.exe がロードされた仮想アドレス空間上のアドレス (ベース アドレス) が確認できます。

![fig6.png](images/fig6.png)

## Task 5 - Dynamic Link Libraries

*What is the base address of "ntdll.dll" loaded from "notepad.exe"?*

`ntdll.dll` の `Load Image` のイベントをダブルクリックして開くと、`Image Base` 欄に ntdll.dll がロードされた仮想アドレス空間上のアドレス (ベース アドレス) が確認できます。

![fig7.png](images/fig7.png)

*What is the size of "ntdll.dll" loaded from "notepad.exe"?*

同じダイアログで `Image Size` 欄をみると、ntdll.dll のロードされたサイズがわかります。

