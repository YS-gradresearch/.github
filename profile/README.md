# YS-gradresearch

電車内の安全監視システム（SOS検知・不審動作検知）の研究用 Organization です。
カメラ映像から MMPose で姿勢推定を行い、手を上げる SOS ポーズ（ルールベース）や不審な動作（LSTM）を検知します。

## 全体の流れ

```
[車載エッジ端末]  ──mp4──▶  [S3]  ──(ObjectCreated)──▶  [Lambda]  ──▶  [DynamoDB]
 edge-raildevice                                         aws          dcon-rail
                                                                         │
                                [Lambda] 署名付きURL発行  ◀── API Gateway ┘
                                  aws        │
                                             ▼
                                        [Webアプリ]
                                     user-frontend-app

[検知システム]  映像 → MMPose → SOS検知 / 不審動作検知（ONNX）→ アラート
 server-backend                              ▲
                                  [モデル学習] makemodel（LSTM → ONNX 変換）
```

Web アプリと API の接続は未実装です（現在はモックデータ）。エッジ端末への姿勢推定の組み込みもこれからです。

## リポジトリ一覧

### システム本体
| リポジトリ | 役割 | 主な技術 |
|---|---|---|
| [server-backend](https://github.com/YS-gradresearch/server-backend) | 監視システム本体。SOS検知（ルールベース）・不審動作検知（LSTM）・複数人物トラッキング・アラート生成 | Python, MMPose, ONNX Runtime, pytest |
| [user-frontend-app](https://github.com/YS-gradresearch/user-frontend-app) | 検知結果・映像を確認する Web フロントエンド（監視・通知・検索・管理の 4 画面） | React, Vite, TypeScript, Tailwind CSS |
| [edge-raildevice](https://github.com/YS-gradresearch/edge-raildevice) | 車載エッジ端末側のプログラム。カメラ映像を数秒ごとの mp4 に区切って S3 へアップロード | Python, OpenCV, boto3 |
| [aws](https://github.com/YS-gradresearch/aws) | AWS Lambda 関数。S3 に上がった映像の情報を DynamoDB に登録／Web 向けに署名付き URL を発行 | Python, boto3 |

### モデル作成・実験
| リポジトリ | 役割 | 主な技術 |
|---|---|---|
| [makemodel](https://github.com/YS-gradresearch/makemodel) | 不審動作／正常動作を判別する LSTM モデルの学習。動画からデータセット作成 → 学習 → ONNX 変換（demo1〜7） | TensorFlow, MMPose |
| [mycode](https://github.com/YS-gradresearch/mycode) | 試作コード。骨格推定・SOS検知・ONNX モデルでの推論テスト・カメラキャリブレーション・FastAPI の画面試作 | Python, MMPose, FastAPI |
| [openpose](https://github.com/YS-gradresearch/openpose) | OpenPose を使った初期の検知実験（骨格→JSON、SOS検知、魚眼カメラ補正など） | OpenPose |
| [yolo-mediapipe](https://github.com/YS-gradresearch/yolo-mediapipe) | YOLO + MediaPipe による姿勢推定の比較・実験 | YOLO, MediaPipe |

### ドキュメント・その他
| リポジトリ | 役割 |
|---|---|
| [documents](https://github.com/YS-gradresearch/documents) | 参考資料、不審動作メモ、画像など |
| [diary](https://github.com/YS-gradresearch/diary) | 研究日記・タスク管理 |
| [tools](https://github.com/YS-gradresearch/tools) | 補助ツール置き場 |
| [docker](https://github.com/YS-gradresearch/docker) | Docker 環境（現在は空） |
| [oldfiles](https://github.com/YS-gradresearch/oldfiles) | 過去のファイルの保管 |

## やりたいこと別の入口
- **SOS 検知をカメラで動かしてみたい** → `server-backend` の [docs/sos-camera-demo.md](https://github.com/YS-gradresearch/server-backend/blob/main/docs/sos-camera-demo.md)
- **映像をクラウドに上げる仕組みを触りたい** → `edge-raildevice`（送信側）と `aws`（受信・DB登録側）
- **Web から映像を見られるようにしたい** → `aws`（署名付きURL発行）と `user-frontend-app`
- **検知ロジックを変えたい** → `server-backend` の `src/detectors/`
- **不審動作モデルを作り直したい** → `makemodel` で学習・ONNX 変換 → `mycode/predictionmodel` でテスト

## 主な conda 環境
| 環境名 | 用途 | 手順 |
|---|---|---|
| `train-security` | 検知システム（server-backend / mycode） | `mycode/mycode/code/setup.md` |
| `openmmlab` | MMPose での推論、データセット作成 | `makemodel/readme.md` |
| `tf_210_gpu` | LSTM の学習 | `makemodel/readme.md` |
| `mmpose_aws` | エッジ端末 | `edge-raildevice/setup.md` |
