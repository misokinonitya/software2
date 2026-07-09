# デジタル暗記ノートアプリ (PDF対応版)

画像やPDFを取り込み、指定した特定の色の文字（赤・青・緑）を自動的に白いマスクで隠すことができるWebベースのデジタル暗記ノートアプリです。隠された部分をタップすることで、一時的に文字を再表示して復習することができます。

## 🛠 主な機能
* **マルチファイル対応**: 画像（JPEG/PNGなど）だけでなく、PDFファイルも取り込んでページごとにノート化が可能。
* **特定色自動マスク**: 赤・青・緑のペンで書かれた部分を自動認識して隠蔽。
* **インタラクティブ復習**: マスク部分をタップすると、その部分の文字がサーッと現れる（トグル表示）。
* **ローカル保存**: 作成したノートはブラウザの `localStorage` に保存され、一覧で管理可能（CRUD機能）。

---

## 📐 設計ドキュメント (UML図)

本アプリの設計構造を示すUML図です。

### 1. ユースケース図 
ユーザーがこのアプリを使って何ができるかを表現しています。

```mermaid
left-to-right-direction
actor ユーザー as User

rectangle デジタル暗記ノートアプリ {
    usecase "保存されたノート一覧を見る" as UC_List
    usecase "新規ノートを作成する" as UC_Create
    usecase "画像・PDFを読み込む" as UC_Upload
    usecase "消す（隠す）色を選択する" as UC_Color
    usecase "マスク部分をタップして確認する" as UC_Tap
    usecase "ノートを保存する" as UC_Save
    usecase "ノートを削除する" as UC_Delete
}

User --> UC_List
User --> UC_Create
User --> UC_Upload
User --> UC_Color
User --> UC_Tap
User --> UC_Save
User --> UC_Delete

UC_Create ..> UC_Upload : <<include>>

### 2. クラス図
classDiagram
    class Note {
        +number id
        +string imageSrc
        +string maskSrc
        +string title
        +string createdAt
        +array hiddenColors
        +static getAll() Array
        +static save(noteInstance) void
        +static delete(id) void
    }

    class ImageProcessingEngine {
        +static checkColor(r, g, b, targetColor) boolean
        +static generateMaskCanvas(originalCanvas, targetColor) HTMLCanvasElement
        +static toggleMaskAtPoint(maskCanvas, baseMaskCtx, startX, startY) boolean
    }

    class AppScreen {
        +Note currentNote
        +HTMLCanvasElement baseMaskCanvas
        +Object pdfDoc
        +number currentPdfPage
        +boolean isRendering
        +initElements() void
        +initEvents() void
        +renderList() void
        +openEditScreen(note) void
        +closeEditScreen() void
        +clearWorkspace() void
        +onFileSelected(e) void
        +renderPdfPage(pageNum) void
        +renderImageToCanvas(img) void
        +onColorSelect() void
        +displayImageFromSrc(imgSrc, maskSrc) void
        +onMaskTap(e) void
        +onSavePressed() void
    }

    AppScreen --> Note : 管理・操作
    AppScreen ..> ImageProcessingEngine : マスク処理・タップ判定を依頼

### 3.協調図
graph TD
    User((ユーザー))
    UI[AppScreen]
    Engine[ImageProcessingEngine]
    Model[Note]
    Storage[(localStorage)]

    User -- "1. ファイル選択 / 色変更" --> UI
    UI -- "2. マスク生成の要求 (generateMaskCanvas)" --> Engine
    Engine -. "3. 生成したマスクCanvasを返却" .-> UI
    User -- "4. 保存ボタンを押す" --> UI
    UI -- "5. 保存処理を要求 (save)" --> Model
    Model -- "6. JSONデータ書き込み" --> Storage

### 4.状態遷移図
stateDiagram-v2
    [*] --> ノート一覧画面 : アプリ起動 (renderList)

    ノート一覧画面 --> 編集画面_新規 : 「新規ノート」ボタン押下
    ノート一覧画面 --> 編集画面_既存 : 保存済みのノートカードをクリック
    ノート一覧画面 --> ノート一覧画面 : 削除ボタン押下 (確定後、再描画)

    state 編集画面_新規 {
        [*] --> ファイル未選択状態
        ファイル未選択状態 --> 画像プレビュー状態 : 画像を選択
        ファイル未選択状態 --> PDFプレビュー状態 : PDFを選択
        
        画像プレビュー状態 --> 画像プレビュー状態 : 消す色を変更 (再マスク)
        PDFプレビュー状態 --> PDFプレビュー状態 : ページ切り替え / 消す色を変更
    }

    state 編集画面_既存 {
        [*] --> 復習・編集状態
        復習・編集状態 --> 復習・編集状態 : マスク部分をタップ (toggleMaskAtPoint)
    }

    編集画面_新規 --> ノート一覧画面 : キャンセルボタン押下
    編集画面_既存 --> ノート一覧画面 : キャンセルボタン押下
    
    編集画面_新規 --> ノート一覧画面 : 保存ボタン押下 (Note.save)
    編集画面_既存 --> ノート一覧画面 : 保存ボタン押下 (Note.save)

