# Vimixセッションファイル（.mix）フォーマット仕様

このドキュメントは、Vimixのセッションファイル（`.mix`）のXML構造について記述します。

## 概要

`.mix`ファイルは、Vimixのセッションの状態を保存するためのXMLファイルです。セッションには、使用されるメディアソース、各ソースの配置やエフェクト、タイムライン、UIのレイアウトなど、プロジェクトに関する全ての情報が含まれます。

## ルート要素: `<vimix>`

ルート要素は`<vimix>`です。ファイル全体のコンテナとなります。

```xml
<vimix app_name="vimix" major="0" minor="4" date="YYYYMMDDHHMMSS" resolution="WIDTHxHEIGHT" size="N" total="M">
  <!-- ... child elements ... -->
</vimix>
```

### 属性

- `app_name`: アプリケーション名。常に`vimix`です。
- `major`, `minor`: セッションファイルのフォーマットバージョン。
- `date`: ファイルの保存日時 (例: `20231027103000`)。
- `resolution`: セッションの解像度 (例: `1920x1080`)。
- `size`: セッションに直接含まれる`<Source>`要素の数。
- `total`: グループ内のソースなども含めた、プロジェクト全体の総ソース数。

---

## トップレベル要素

`<vimix>`要素の直下には、以下の主要な要素が含まれます。

- **`<Session>`**: プロジェクトのメインコンテナ。メディアソースやセッション全体の設定を保持します。
- **`<Views>`**: 各ビュー（ミキシング、ジオメトリなど）のカメラ位置やズーム率といったUIの状態を保存します。
- **`<Snapshots>`**: 保存されたセッションのスナップショット（特定の状態のプリセット）のリストです。
- **`<Notes>`**: ユーザーが作成したテキストノートの情報です。
- **`<PlayGroups>`**: 複数のソースをまとめて再生するためのグループ（バッチ）定義です。
- **`<InputCallbacks>`**: MIDIコントローラーやキーボードショートカットへのアクション割り当て（マッピング）を定義します。

---

## `<Session>`要素

セッションのコアとなる情報を含みます。

```xml
<Session id="SESSION_ID" activationThreshold="1.3">
  <Source> ... </Source>
  ...
  <Thumbnail> ... </Thumbnail>
</Session>
```

### 属性

- `id`: セッションの一意なID（Unixタイムスタンプ）。
- `activationThreshold`: ソースがアクティブ（ミキシング対象）になるかの閾値。

### 子要素

- `<Source>`: 各メディアソースの詳細設定。（詳細は後述）
- `<Thumbnail>`: セッションのサムネイル画像。バイナリデータとして格納されます。（オプション）

---

## `<Source>`要素

個々のメディアソースの設定です。`type`属性によって子要素の構成が大きく異なります。

### 共通属性

- `id`: ソースの一意なID。
- `name`: ソースの表示名。
- `type`: ソースの種類。下記「ソースタイプ」を参照。
- `play`: 再生状態 (`true`/`false`)。
- `locked`: パラメータのロック状態 (`true`/`false`)。

### 共通の子要素

- `<Mixing>`, `<Geometry>`, `<Layer>`, `<Texture>`: 各ビューに対応するノード情報。位置、スケール、回転、クロップなどの情報を持つ`<Node>`要素を含みます。
- `<Blending>`: ブレンドモードや色の設定。
- `<Mask>`: マスクの設定。
- `<ImageProcessing>`: 輝度、コントラスト、彩度などの画像処理フィルタの設定。
- `<Audio>`: 音量や有効/無効などの音声設定。
- `<MixingGroup>`: このソースが所属するミキシンググループの情報。

### ソースタイプ (`type`属性)

#### `MediaSource`
動画や画像ファイル。
- **子要素**: `<uri>` (ファイルの絶対パス), `<MediaPlayer>` (再生速度、ループ設定など)

#### `SessionSource` / `SessionFileSource`
別の`.mix`ファイルを読み込むソース。
- **子要素**: `<path>` (`.mix`ファイルのパス)

#### `SessionGroupSource`
ソースのグループ。内部に独自の`<Session>`要素を持ちます。

#### `CloneSource`
他のソースを複製（クローン）するソース。
- **子要素**: `<origin id="ORIGINAL_ID">` (クローン元のソースID), `<Filter>` (クローンにのみ適用するフィルタ)

#### `PatternSource`
カラーバーやチェッカーボードなどのテストパターンを生成。
- **属性**: `pattern` (パターンの種類を示すID)
- **子要素**: `<resolution>` (`<ivec2>`)

#### `DeviceSource`
Webカメラなどのビデオキャプチャデバイス。
- **属性**: `device` (デバイス名)

#### `ScreenCaptureSource`
デスクトップ画面や特定のウィンドウのキャプチャ。
- **属性**: `window` (ウィンドウ名)

#### `TextSource`
テキストを描画するソース。
- **子要素**: `<contents>` (テキスト内容、フォント、色、配置など), `<resolution>`

#### `ShaderSource`
GLSLシェーダーによって映像を生成するソース。
- **子要素**: `<resolution>`, `<Filter>` (シェーダーコードやパラメータ)

---

## 詳細な要素の解説

### `<ImageProcessing>`
輝度、コントラスト、彩度、ガンマ補正などのパラメータを保持します。
```xml
<ImageProcessing enabled="true" follow="0">
  <uniforms brightness="0" contrast="1" saturation="1" ... />
  <gamma> <vec4 x="1" y="1" z="1" w="1"/> </gamma>
  <levels> <vec4 x="0" y="1" z="0" w="1"/> </levels>
</ImageProcessing>
```

### `<Blending>`
描画時のブレンドモードを指定します。
```xml
<Blending type="ImageShader">
    <color> <vec4 x="1" y="1" z="1" w="1"/> </color>
    <blending mode="1" /> <!-- 1: Normal, 2: Add, ... -->
</Blending>
```

### `<Mask>`
ソースを切り抜くためのマスク設定です。
```xml
<Mask type="MaskShader" mode="0" shape="1" source="MASK_SOURCE_ID">
    <uniforms blur="0" option="0">
        <size> <vec2 x="1" y="1"/> </size>
    </uniforms>
    <Image> <array>...</array> </Image> <!-- 画像マスクの場合 -->
</Mask>
```
- `source`: マスクとして利用する別のソースのID。

### `<Audio>`
```xml
<Audio enabled="true" volume="1.0" volume_mix="0" />
```
- `enabled`: 音声を有効にするか。
- `volume`: 音量レベル。
- `volume_mix`: 親グループとの音量ミックス方法。

---

## データ型

### ベクトルと行列 (`glm`型)

`glm`ライブラリのベクトル (`ivec2`, `vec2`, `vec3`, `vec4`) と行列 (`mat4`) は、それぞれ独立したXML要素としてシリアライズされます。

**例: `vec3`**
```xml
<vec3 x="1.0" y="0.5" z="0.0" />
```

**例: `mat4`**
```xml
<mat4>
  <vec4 row="0" x="1" y="0" z="0" w="0" />
  <vec4 row="1" x="0" y="1" z="0" w="0" />
  <vec4 row="2" x="0" y="0" z="1" w="0" />
  <vec4 row="3" x="0" y="0" z="0" w="1" />
</mat4>
```

### バイナリデータ (`<array>`)

サムネイル画像やフォントデータなどのバイナリデータは、まず`zlib`で圧縮され、次に`Base64`でエンコードされます。その結果の文字列が`<array>`要素のテキストノードとして格納されます。

```xml
<array len="ORIGINAL_SIZE" zbytes="COMPRESSED_SIZE">
  BASE64_ENCODED_STRING
</array>
```
- `len`: 圧縮前のオリジナルデータのバイト数。
- `zbytes`: zlib圧縮後のデータのバイト数。もしこの属性がなければ、データは圧縮されていないと見なされます。
