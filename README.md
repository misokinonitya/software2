# デジタル暗記ノートアプリ (PDF対応版)

画像やPDFを取り込み、指定した特定の色の文字（赤・青・緑）を自動的に白いマスクで隠すことができるWebベースのデジタル暗記ノートアプリです。隠された部分をタップすることで、一時的に文字を再表示して復習することができます。

## 🛠 主な機能
* **マルチファイル対応**: 画像（JPEG/PNGなど）だけでなく、PDFファイルも取り込んでページごとにノート化が可能。
* **特定色自動マスク**: 赤・青・緑のペンで書かれた部分を自動認識して隠蔽。
* **インタラクティブ復習**: マスク部分をタップすると、その部分の文字がサーッと現れる（トグル表示）。
* **ローカル保存**: 作成したノートはブラウザの `localStorage` に保存され、一覧で管理可能（CRUD機能）。

---

## 📐 設計ドキュメント (UML図)

```mermaid
graph TB
    %% ==========================================
    %% 1. ユースケース図 セクション
    %% ==========================================
    subgraph UC_Sec ["1. ユースケース図 (Use Case Diagram)"]
        direction LR
        User((ユーザー))
        
        subgraph App_Boundary ["デジタル暗記ノートアプリ"]
            UC_List["保存されたノート一覧を見る"]
            UC_Create["新規ノートを作成する"]
            UC_Upload["画像・PDFを読み込む"]
            UC_Color["消す（隠す）色を選択する"]
            UC_Tap["マスク部分をタップして確認する"]
            UC_Save["ノートを保存する"]
            UC_Delete["ノートを削除する"]
        end

        User --> UC_List
        User --> UC_Create
        User --> UC_Upload
        User --> UC_Color
        User --> UC_Tap
        User --> UC_Save
        User --> UC_Delete
        
        UC_Create -.->|include| UC_Upload
    end

    %% ==========================================
    %% 2. クラス関係図 セクション
    %% ==========================================
    subgraph Class_Sec ["2. クラス関係図 (Class Relations)"]
        direction TB
        
        class_Note["Note<br>+id: number<br>+imageSrc: string<br>+maskSrc: string<br>+title: string<br>+createdAt: string<br>+hiddenColors: array<br><br>+static getAll() Array<br>+static save(note) void<br>+static delete(id) void"]
        
        class_Engine["ImageProcessingEngine<br>+static checkColor(r,g,b,target) boolean<br>+static generateMaskCanvas(canvas,target) Canvas<br>+static toggleMaskAtPoint(canvas,ctx,x,y) boolean"]
        
        class_Screen["AppScreen<br>+currentNote: Note<br>+baseMaskCanvas: Canvas<br>+pdfDoc: Object<br>+currentPdfPage: number<br>+isRendering: boolean<br><br>+initElements() void<br>+initEvents() void<br>+renderList() void<br>+openEditScreen(note) void<br>+closeEditScreen() void<br>+clearWorkspace() void<br>+onFileSelected(e) void<br>+renderPdfPage(num) void<br>+renderImageToCanvas(img) void<br>+onColorSelect() void<br>+displayImageFromSrc() void<br>+onMaskTap(e) void<br>+onSavePressed() void"]

        class_Screen -->|管理・操作| class_Note
        class_Screen -.->|処理依頼| class_Engine
    end

    %% ==========================================
    %% 3. 協調図 セクション
    %% ==========================================
    subgraph Col_Sec ["3. 協調図 / コミュニケーション図 (Collaboration)"]
        direction TB
        User_Col((ユーザー))
        UI[AppScreen]
        Engine[ImageProcessingEngine]
        Model[Note]
        Storage[(localStorage)]

        User_Col -- "1. ファイル選択 / 色変更" --> UI
        UI -- "2. マスク生成の要求" --> Engine
        Engine -. "3. マスクCanvasを返却" .-> UI
        User_Col -- "4. 保存ボタンを押す" --> UI
        UI -- "5. 保存処理を要求" --> Model
        Model -- "6. JSONデータ書き込み" --> Storage
    end

    %% ==========================================
    %% 4. 状態遷移図 セクション
    %% ==========================================
    subgraph State_Sec ["4. 状態遷移図 (State Transition)"]
        direction TB
        Init_State((●)) --> ST_List[ノート一覧画面]
        
        ST_List -->|「新規ノート」押下| ST_New_Parent
        ST_List -->|既存ノートクリック| ST_Edit_Parent
        ST_List -->|削除ボタン押下| ST_List
        
        subgraph ST_New_Parent ["編集画面（新規作成）"]
            ST_None[ファイル未選択] -->|画像選択| ST_Img[画像プレビュー]
            ST_None -->|PDF選択| ST_Pdf[PDFプレビュー]
            ST_Img -->|色変更| ST_Img
            ST_Pdf -->|ページ切り替え / 色変更| ST_Pdf
        end
        
        subgraph ST_Edit_Parent ["編集画面（既存復習）"]
            ST_Review[復習・編集状態] -->|マスクタップ| ST_Review
        end

        ST_New_Parent -->|キャンセルボタン| ST_List
        ST_Edit_Parent -->|キャンセルボタン| ST_List
        
        ST_New_Parent -->|保存ボタン| ST_List
        ST_Edit_Parent -->|保存ボタン| ST_List
    end

    %% 全体セクションの配置コントロール（上から順に並べる）
    UC_Sec --> Class_Sec --> Col_Sec --> State_Sec
