# 🧬 QIIME2本解析マニュアル
（ver. 2024.05対応・共通プロトコル）

---

## 🧩 STEP 0-0｜初回セットアップ（班フォルダの作成）
解析を始める前に、~/qiime/template フォルダをコピーして自分の班名に変更します。

```bash
cd ~/qiime
cp -r template ishikawa    # ← ishikawaの部分を自分の班名に変更
```

作成後のフォルダ構成例：
```bash
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



## 🧩 STEP 0｜解析環境の起動

> 📋コピーは⧉を押し、ペーストはUbuntuに右クリックで行えます。

```bash
conda activate q2-picrust2-amplicon-2024.5
export master=ishikawa   # ← 班名を入力
cd ~/qiime/$master
```
これ以降、すべてのコマンドで `$master` が自動的に班名に置き換わります。  

---

## 📁 STEP 1｜フォルダ構成
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
受け取ったシーケンスデータ（`*_R1_001.fastq`, `*_R2_001.fastq`）を 先ほど作成した$master/raw_data/にコピーします。

例：
```bash
~/qiime/ishikawa/raw_data/39320-04_S1_L001_R1_001.fastq
~/qiime/ishikawa/raw_data/39320-04_S1_L001_R2_001.fastq
```

---

## 🧾 STEP 3｜manifestテンプレートの作成
```bash
bash ~/make_manifest.sh
```
自動で `$master/manifest/manifest.csv` が生成されます。  
生成後、`sample-id` 列を自分のサンプル名（例：NC1～5, HF1～5, RBR1～5）に編集します。

**例（Excel表示）**
| sample-id | absolute-filepath | direction |
|------------|-------------------|------------|
| NC1 | /home/ishikawa/qiime/raw_data/..._R1_001.fastq | forward |
| NC1 | /home/ishikawa/qiime/raw_data/..._R2_001.fastq | reverse |

---

## 🧬 STEP 4｜metadataの作成
```bash
bash ~/make_metadata.sh auto
```
自動で `$master/metadata/metadata.tsv` が生成されます。  
`group` 列（例：NC, HF, BR, RBR）は各班で手動入力してください。  

---

## 🧫 STEP 5｜FASTQ → QIIME2（.qza）へのインポート
```bash
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path $master/manifest/manifest.csv \
  --output-path $master/results_qiime/demux.qza \
  --input-format PairedEndFastqManifestPhred33V2
```

---

## 📊 STEP 6｜シーケンス品質確認（Phredスコア）
```bash
qiime demux summarize \
  --i-data $master/results_qiime/demux.qza \
  --o-visualization $master/results_qiime/demux.qzv
```
生成された `.qzv` ファイルを 👉 https://view.qiime2.org にドラッグ＆ドロップして確認します。

**トリミング長の決め方（目安）**
- 品質スコアが急激に低下する前でカット
- 例：R1 → `--p-trunc-len-f 270`、R2 → `--p-trunc-len-r 220`

---

## 🧮 STEP 7｜DADA2によるノイズ除去とASV化
```bash
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs $master/results_qiime/demux.qza \
  --p-trunc-len-f 270 \
  --p-trunc-len-r 220 \
  --o-table $master/results_qiime/table.qza \
  --o-representative-sequences $master/results_qiime/rep-seqs.qza \
  --o-denoising-stats $master/results_qiime/denoising-stats.qza
```

---

## 🧬 STEP 8｜分類（SILVA分類器）
共通で構築済みの分類器を使用します。  
```bash
qiime feature-classifier classify-sklearn \
  --i-classifier ~/qiime/databases/silva-138.1-nr99-v4-classifier.qza \
  --i-reads $master/results_qiime/rep-seqs.qza \
  --o-classification $master/results_qiime/taxonomy.qza
```

---

## 🧩 STEP 9｜分類結果の可視化
```bash
qiime metadata tabulate \
  --m-input-file $master/results_qiime/taxonomy.qza \
  --o-visualization $master/results_qiime/taxonomy.qzv
```

---

## 🧠 STEP 10｜多様性解析

### α多様性
```bash
qiime diversity alpha \
  --i-table $master/results_qiime/table.qza \
  --p-metric shannon \
  --o-alpha-diversity $master/results_qiime/alpha_shannon.qza
```

### β多様性
```bash
qiime diversity beta \
  --i-table $master/results_qiime/table.qza \
  --p-metric braycurtis \
  --o-distance-matrix $master/results_qiime/braycurtis.qza
```

---

## 🧬 STEP 11｜PICRUSt2解析
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
- 使用環境：Ubuntu 22.04（WSL2）, q2-picrust2-amplicon-2024.5
- 作成：SEROL1（QIIME2共通解析マニュアルプロジェクト）
