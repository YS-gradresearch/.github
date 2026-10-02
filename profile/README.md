# YS-gradresearch

電車内の安全監視システム（SOS検知・不審動作検知）の研究用 Organization です。
カメラ映像から姿勢推定を行い、手を上げる SOS ポーズや不審な動作を検知します。

## 全体の流れ

```
[車載エッジ端末]  ──映像──▶  [S3]  ──▶  [Lambda]  ──▶  [DynamoDB]
 edge-raildevice                        aws                │
                                                           ▼
                             [Lambda] 署名付きURL発行 ──▶ [Webアプリ]
                               aws                     user-frontend-app
```

## リポジトリ一覧

### システム本体
| リポジトリ | 役割 |
|---|---|
| [server-backend](https://github.com/YS-gradresearch/server-backend) | 監視システム本体（Python）。SOS検知（ルールベース）・不審動作検知（LSTM）・複数人物トラッキング・アラート生成 |
| [user-frontend-app](https://github.com/YS-gradresearch/user-frontend-app) | 検知結果・映像を確認する Web フロントエンド（React + Vite + TypeScript） |
| [edge-raildevice](https://github.com/YS-gradresearch/edge-raildevice) | 車載エッジ端末側のプログラム。MMPose で姿勢推定し、映像を S3 へアップロード。環境構築手順は `setup.md` |
| [aws](https://github.com/YS-gradresearch/aws) | AWS Lambda 関数。S3 に上がった映像の情報を DynamoDB に登録／Web 向けに署名付き URL を発行 |

### モデル作成・実験
| リポジトリ | 役割 |
|---|---|
| [makemodel](https://github.com/YS-gradresearch/makemodel) | 不審動作／正常動作を判別する LSTM モデルの学習。動画からデータセット作成 → 学習 → ONNX 変換 |
| [mycode](https://github.com/YS-gradresearch/mycode) | 作成したモデル（ONNX）での推論テスト、カメラキャリブレーション、可視化 |
| [openpose](https://github.com/YS-gradresearch/openpose) | OpenPose を使った初期の検知実験（骨格→JSON、SOS検知、魚眼カメラ補正など） |
| [yolo-mediapipe](https://github.com/YS-gradresearch/yolo-mediapipe) | YOLO + MediaPipe による姿勢推定の比較・実験 |
| [mmpose](https://github.com/YS-gradresearch/mmpose) | MMPose 関連（現在は空） |

### ドキュメント・その他
| リポジトリ | 役割 |
|---|---|
| [documents](https://github.com/YS-gradresearch/documents) | 参考資料、不審動作メモ、画像など |
| [diary](https://github.com/YS-gradresearch/diary) | 研究日記・タスク管理 |
| [tools](https://github.com/YS-gradresearch/tools) | 補助ツール置き場 |
| [docker](https://github.com/YS-gradresearch/docker) | Docker 環境（現在は空） |
| [oldfiles](https://github.com/YS-gradresearch/oldfiles) | 過去のファイルの保管 |

## やりたいこと別の入口
- **映像をクラウドに上げる仕組みを触りたい** → `edge-raildevice`（送信側）と `aws`（受信・DB登録側）
- **Web から映像を見られるようにしたい** → `aws`（署名付きURL発行）と `user-frontend-app`
- **検知ロジックを変えたい** → `server-backend` の `src/detectors/`
- **不審動作モデルを作り直したい** → `makemodel` で学習・ONNX 変換 → `mycode/predictionmodel` でテスト
