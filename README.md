# Antigravity 連携用 Fusion 360 MCP サーバー 仕様書

## 概要
本ドキュメントは、カレントワークスペースの `d:\fusion\Antigravity_fusion_mcp_server` に配置された Fusion 360 向けの MCP (Model Context Protocol) サーバーと AI エージェント（Antigravity）との連携に関する仕様をまとめたものです。
既存の Gemini MCP 設定を参照し、Antigravity でも同様のツール群を利用できるように環境を構築しました。

## 構成と設定ファイル

### 1. MCP サーバー プログラム
*   **ファイルパス**: `d:\fusion\Antigravity_fusion_mcp_server\fusion_mcp_server.js`
*   **元のソース**: `d:\fusion\gemini_fusion_mcp_server\fusion_mcp_server.js`
*   **動作環境**: Node.js
*   **バージョン**: `v 0.7.80 ベータ版 Beta version 2025.08.08`
*   **名前**: `fusion-mcp-server-complete`

### 2. Antigravity MCP 連携設定
*   **ファイルパス**: `C:\Users\tomo123\.gemini\antigravity\mcp_config.json`
*   **設定内容**:
    Antigravity が起動時に外部の MCP サーバーを読み込むための設定です。
    ```json
    {
      "mcpServers": {
        "fusion": {
          "command": "node",
          "args": [
            "d:\\fusion\\Antigravity_fusion_mcp_server\\fusion_mcp_server.js"
          ]
        }
      }
    }
    ```

## 通信方式 (Transport Layer)
*   Antigravity は標準入出力 (`stdio`) を使用して MCP サーバーと通信します。
*   MCP サーバーは `@modelcontextprotocol/sdk/server/stdio.js` の `StdioServerTransport` を利用しています。

## Fusion 360 側とのコマンド連携の仕組み
MCP サーバー（Node.jsプロセス）と実際の Fusion 360 自体とのデータのやり取りは、ローカルのテキストファイル（JSON形式）を介して行われます。

1.  **コマンド送信**:
    *   MCP サーバーは受け取ったツール実行リクエストを元に、JSON データを生成します。
    *   `%USERPROFILE%\Documents\fusion_command.txt` に対象データを書き込みます。
2.  **結果待機と取得**:
    *   Fusion 360 側のアドインが `fusion_command.txt` の更新を監視し、コマンドを実行します。
    *   実行結果を `%USERPROFILE%\Documents\fusion_response.txt` に出力します。
    *   MCP サーバーは `fusion_response.txt` が更新されるまで待機（最大60秒）し、結果をパースして Antigravity へ返却します。

## 提供される主要なツール群（Capabilities）

Antigravity 経由で呼び出すことができる主な Fusion 360 操作コマンド一覧です。
（引数などの詳細は `fusion_mcp_server.js` の `ListToolsRequestSchema` セクションを参照してください）

### 基本形状作成 (Primitive Creation)
*   **create_cube**: 立方体を作成します。（サイズ、配置、テーパーなどを指定可能）
*   **create_cylinder**: 円柱を作成します。
*   **create_box**: 直方体を作成します。
*   **create_sphere**: 球を作成します。
*   **create_hemisphere**: 半球を作成します。
*   **create_cone**: 円錐を作成します。
*   **create_polygon_prism**: 多角柱を作成します。
*   **create_torus**: トーラス（ドーナツ形状）を作成します。
*   **create_half_torus**: 半分のトーラスを作成します。
*   **create_pipe**: 指定した2点間にパイプを作成します。
*   **create_polygon_sweep**: 多角形プロファイルを円形パスでスイープ（ねじり指定可能）。

### パターン・コピー操作 (Pattern/Copy)
*   **copy_body_symmetric**: ボディを平面に対して対称にコピー（ミラー）。
*   **create_circular_pattern**: ボディの円形状パターンを作成。
*   **create_rectangular_pattern**: ボディの矩形状パターンを作成。

### 形状変更・ブール演算 (Modification)
*   **add_fillet**: エッジにフィレットを追加。
*   **add_chamfer**: エッジに面取りを追加。
*   **combine_selection**: 選択した複数のボディを結合（join/cut/intersect）。
*   **combine_selection_all**: 選択したすべてのボディを結合。
*   **combine_by_name**: 名前で指定した2つのボディを結合。

### 配置・表示に関する操作 (Transformation & Visibility)
*   **hide_body** / **show_body**: ボディの表示/非表示を切り替え。
*   **move_by_name**: ボディを平行移動。
*   **rotate_by_name**: ボディを回転。

### 選択操作 (Selection)
*   **select_body** / **select_bodies** / **select_all_bodies**: ボディを選択。

### ボディ情報取得 (Body Information)
*   **get_bounding_box**: バウンディングボックス情報を取得。
*   **get_body_center**: 中心点情報（幾何学的中心、重心など）を取得。
*   **get_body_dimensions**: 詳細寸法情報（長さ、幅、高さ、体積など）。
*   **get_faces_info**: 面情報（タイプ、面積、法線など）。
*   **get_edges_info**: エッジ情報（タイプ、長さ、方向など）。
*   **get_mass_properties**: 質量特性（体積、質量、慣性モーメントなど）。
*   **get_body_relationships**: 2つのボディ間の位置関係。
*   **measure_distance**: 2つのボディ間の距離を測定。

### その他・ユーティリティ (Utility)
*   **execute_macro**: 複数のモデリングコマンドを順番に連続して実行。
*   **delete_all_features**: タイムライン上のすべてのフィーチャを削除し、デザインを初期化。
*   **debug_coordinate_info**: 座標系や単位に関するデバッグ情報を出力。

## 今後の利用について
このドキュメントで定義された MCP Server が `mcp_config.json` に設定されているため、Antigravity を再起動すると上記のツール群が AI エージェントの拡張機能として利用可能になります。
