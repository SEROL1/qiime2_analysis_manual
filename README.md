# 🧬 QIIME2本解析マニュアル（ver. 2024.05対応・共通プロトコル）

本マニュアルは、16S rRNA解析の共通手順をまとめたものです。

解析の一連の流れ（FASTQ → QIIME2 → PICRUSt2 → 可視化）を通して、
再現性・共有性・効率性を確保することを目指しています。

このマニュアルではコピーボタンコピーボタン⧉が登場し

⧉をクリックするだけでそのコードをコピーすることができます。

しかし、マニュアルに載せる都合上、コピペが必要のない箇所にも⧉が登場するため

**コピペが必要な箇所には「📋」を載せておく**ので、各自判断をお願いします。

---
## 🖥️ STEP 1　｜　解析を始める前に準備すること

**💡 目的**

QIIME2解析を始めるためのフォルダ構成と起動準備を整えます。

このステップでは、シーケンスデータを適切な場所に配置し、Ubuntu環境を立ち上げます。

**🧾 用意するもの**

1.配布されたシーケンスデータフォルダ（例：FASTQ_2025_10_30）
　中には「＊_R1_001.fastq.gz」「＊_R2_001.fastq.gz」が含まれています。

2.Ubuntu（WSL2）がインストールされているWindows PC

3.事前に構築済みのQIIME2環境：q2-picrust2-amplicon-2024.5

---
**🪟 1.Windowsでの準備**

1.研究室PCを起動する。

2.配布されたシーケンスデータフォルダをデスクトップにコピーしておく。
　（USBやGoogle Driveから受け取った場合も、**必ず「デスクトップ上」**　に置く）
```
例：
C:\Users\ユーザー名\Desktop\FASTQ_2025_10_30
　├─ Sample1_S1_L001_R1_001.fastq.gz
　├─ Sample1_S1_L001_R2_001.fastq.gz
　└─ …
```
---
**🧩 2.Ubuntuの起動**

1.デスクトップ上の「Ubuntu」アイコンをダブルクリックして開く。

2.Ubuntuが開いたら、解析環境をアクティブにする。

「📋」
```bash
conda activate q2-picrust2-amplicon-2024.5
```
3.ホームディレクトリに移動し、作業場所を確認する。

「📋」
```bash
cd ~
ls
```

**「qiime」フォルダが表示されていればOK。**

---
**🧩 3.班フォルダの作成**

解析を始める前に、~/qiime/template フォルダをコピーして**自分の班名**に変更します。

「📋」**□を消して自分の班名に変更**
```bash
cd ~/qiime
cp -r template　□□□
```

作成後のフォルダ構成例：
```
~/qiime/
├── template/          # 雛形（触らない）
├── ishikawa/          # 自分の班フォルダ（ここで解析を実行）
│   ├── raw_data/
│   ├── manifest/
│   ├── metadata/
│   ├── results_qiime/
│   └── results_picrust2/
```

> ✅ templateは共通構成を保つため削除しないでください。  
> ✅ 各班は自分のフォルダでのみ解析を行います。  

---

**🧩 4.解析を行う班の設定**

「📋」**□を消して自分の班名に変更**
```bash
export master=□□□   
cd ~/qiime/$master
```
これ以降、すべてのコマンドで　**$master**　が自動的に班名に置き換わります。  

---

**📁 5.フォルダ構成の確認**

解析を始める前に、班フォルダが正しく作成されているか確認します。

例：
```bash
~/qiime/
├── ishikawa/
│   ├── raw_data/           # FASTQファイル
│   ├── manifest/           # manifest.csv
│   ├── metadata/           # metadata.tsv
│   ├── results_qiime/      # QIIME2解析結果
│   └── results_picrust2/   # PICRUSt2解析結果
```


---

## 🧬 STEP 2｜FASTQファイルの配置
受け取ったシーケンスデータ

例：39320-04_S1_L001_R1_001.fastq.gz,

  39320-04_S1_L001_R2_001.fastq.gz
    
  を 先ほど作成した$master/raw_data/にコピーします。

例：
```bash
~/qiime/ishikawa/raw_data/39320-04_S1_L001_R1_001.fastq.gz
~/qiime/ishikawa/raw_data/39320-04_S1_L001_R2_001.fastq.gz
```

---

## 🧾 STEP 3｜manifestテンプレートの作成

「📋」
```bash
bash ~/qiime/tools/make_manifest.sh
```
自動で $master/manifest/manifest.csv が生成されます。  
生成後、manifest.csvをexcelで開き

**一列目の「sample-id」 列を自分のサンプル名（例：NC1～5, HF1～5, RBR1～5）に編集します。**

**例（Excel表示）**
| sample-id | absolute-filepath | direction |
|------------|-------------------|------------|
| NC1 | /home/ishikawa/qiime/raw_data/..._R1_001.fastq | forward |
| NC1 | /home/ishikawa/qiime/raw_data/..._R2_001.fastq | reverse |

---

## 🧬 STEP 4｜metadataの作成

「📋」
```bash
bash ~/qiime/tools/make_metadata.sh
```
自動で $master/metadata/metadata.tsv　が生成されます。 
生成後、metadata.tsvをexcelで開き、
**group 列（例：NC, HF, BR, RBR）は各班で手動入力してください。**

---

## 🧫 STEP 5｜FASTQ → QIIME2（.qza）へのインポート

「📋」
```bash
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path $master/manifest/manifest.csv \
  --output-path $master/results_qiime/demux.qza \
  --input-format PairedEndFastqManifestPhred33V2
```
生成後、results_qiimeに**qzaファイル**が生成されているか確認してください。

---

## 📊 STEP 6｜シーケンス品質確認（Phredスコア）

「📋」
```bash
qiime demux summarize \
  --i-data $master/results_qiime/demux.qza \
  --o-visualization $master/results_qiime/demux.qzv
```
**生成された qzvファイルを 👉 https://view.qiime2.org にドラッグ＆ドロップして確認します。**

生成されたグラフを確認し、**トリミング長**を決定します。

**💡 なぜこの作業が必要なのか？**

シーケンサーで得られたリード（配列）は、末端に近づくほど**エラーが増える**という特徴があります。
このエラー部分を残したまま解析を進めると、

＞DADA2 が誤って「ノイズを本物の配列」と判断したり、

＞リード同士の重なり（マージ）が失敗したり、

＞最終的なASV数や分類結果が不正確になる
などの問題が発生します。

そこで、　**品質が急激に低下する部分を切り落とす（トリミング）**　ことで、
データの信頼性を保ち、以降の解析（DADA2・分類・多様性解析など）を正確に行うことができます。

**👀 Phredスコアグラフの見方**

グラフの縦軸は「Phredスコア（品質）」、横軸は「塩基位置（リードの長さ）」です。
スコアが高いほどエラーが少なく、信頼性が高い配列です。
| Phred値 | 意味         | エラー率の目安  |
| ------ | ---------- | -------- |
| ≥30    | 非常に高品質     | 0.1％未満   |
| 25～30  | 許容範囲内      | 0.3～1％程度 |
| ≤20    | 要注意（ノイズ多い） | 1％以上     |

このとき

**黒部分：品質が高く安定した領域（スコアが高い）**

**灰色の部分：品質が低下している領域（スコアが低い）**

となります。

グラフの黒部分を参考に、各トリミング長を判断します。

**✂️ トリミング長の決め方（実践ステップ）**

**1.Forward（R1）・Reverse（R2）の両方を確認**

各サンプルのグラフを開き、右端に向かってスコアが急激に下がる箇所を探します。

**2.品質低下が始まる少し手前をカット位置に設定**

たとえば、R1のスコアが270bp付近で下がり始めたら、

--p-trunc-len-f **270**

とします。

同様に、R2が220bpあたりで下がるなら、

--p-trunc-len-r **220**

とします。

**3.短すぎるトリミングは避ける**

ForwardとReverseの重なりが150bp以上残るようにします。
そうしないと、マージ（重ね合わせ）ができずにエラーが発生します。

このトリミング長をもとに次のSTEP7での数字を変更してください

---

## 🧮 STEP 7｜DADA2によるノイズ除去とASV化

「📋」
```bash
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs $master/results_qiime/demux.qza \
  --p-trunc-len-f 270 \　　 # 設定したForwardトリミング長
  --p-trunc-len-r 220 \     # 設定したReverseトリミング長
  --o-table $master/results_qiime/table.qza \
  --o-representative-sequences $master/results_qiime/rep-seqs.qza \
  --o-denoising-stats $master/results_qiime/denoising-stats.qza
```
この作業は長くて約１日と、とても時間がかかります。

可能なら、

---

## 🧬 STEP 8｜分類（SILVA分類器）
共通で構築済みの分類器を使用します。  

「📋」
```bash
qiime feature-classifier classify-sklearn \
  --i-classifier ~/qiime/databases/silva-138.1-nr99-v4-classifier.qza \
  --i-reads $master/results_qiime/rep-seqs.qza \
  --o-classification $master/results_qiime/taxonomy.qza
```

---

## 🧩 STEP 9｜分類結果の可視化

「📋」
```bash
qiime metadata tabulate \
  --m-input-file $master/results_qiime/taxonomy.qza \
  --o-visualization $master/results_qiime/taxonomy.qzv
```

---

## 🧠 STEP 10｜多様性解析

### α多様性

「📋」
```bash
qiime diversity alpha \
  --i-table $master/results_qiime/table.qza \
  --p-metric shannon \
  --o-alpha-diversity $master/results_qiime/alpha_shannon.qza
```

### β多様性

「📋」
```bash
qiime diversity beta \
  --i-table $master/results_qiime/table.qza \
  --p-metric braycurtis \
  --o-distance-matrix $master/results_qiime/braycurtis.qza
```

---

## 🧬 STEP 11｜PICRUSt2解析

「📋」
```bash
qiime picrust2 full-pipeline \
  --i-table $master/results_qiime/table.qza \
  --i-seq $master/results_qiime/rep-seqs.qza \
  --output-dir $master/results_picrust2/ \
  --p-threads 6 \
  --p-hsp-method pic \
  --p-max-nsti 2
```

---

## 🌈 STEP 12｜可視化
`results_qiime/` および `results_picrust2/` 内の `.qzv` ファイルを  
👉 https://view.qiime2.org にドラッグして開きます。

---

## 📘 参考情報
- QIIME2 Documentation（2024.5）: https://docs.qiime2.org/2024.5/
- SILVA Database (v138.1): https://www.arb-silva.de/
- 使用環境：Ubuntu 24.04（WSL2）, q2-picrust2-amplicon-2024.5
- 作成：SEROL1（QIIME2共通解析マニュアルプロジェクト）
