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

### 💡目的

QIIME2解析を始めるためのフォルダ構成と起動準備を整えます。

このステップでは、シーケンスデータを適切な場所に配置し、Ubuntu環境を立ち上げます。

### 🧾 用意するもの

1.配布されたシーケンスデータフォルダ（例：FASTQ_2025_10_30）

　中には「＊_R1_001.fastq.gz」「＊_R2_001.fastq.gz」が含まれています。

2.Ubuntu（WSL2）がインストールされているWindows PC

3.事前に構築済みのQIIME2環境：q2-picrust2-amplicon-2024.5

---
### 🪟 1.Windowsでの準備

**1.研究室PCを起動する。**

**2.配布されたシーケンスデータフォルダをUSBなどに移しておく。**

**3.USBをPCに接続し、フォルダを確認する。**
 
```
例：
D:\FASTQ_2025_10_30
　├─ Sample1_S1_L001_R1_001.fastq.gz
　├─ Sample1_S1_L001_R2_001.fastq.gz
　└─ …
```
---

デスクトップ上にあるqiimeフォルダを開き、
中に以下のファイルが有ることを確認してください。

```
~/qiime/
├── tools/      ←解析用スクリプトフォルダ（触らない）
├── template/   ←各班が使用する雛形フォルダ（コピペして使用する）
├── databases/ 　←解析用機器（触らない）
```

以上の/templateフォルダをフォルダをコピーして**自分の班名**に変更します。
```
~/qiime/
├── tools/      ←解析用スクリプトフォルダ（触らない）
├── template/   ←各班が使用する雛形フォルダ（消さないように注意）
├── databases/ 　←解析用機器（触らない）
├── 自分の班名/  ←コピペして名前を変更、この名前を今後の解析で使います。
```

その後、以下のようになっているか確認してください。

作成後のフォルダ構成例：
```
~/qiime/
├── template/          # 雛形（触らない）
├── 自分の班名/         # 自分の班フォルダ（ここで解析を実行）
│   ├── raw_data/
│   ├── manifest/
│   ├── metadata/
│   ├── results_qiime/
│   └── results_picrust2/
├── tools/
├── databases/
```

```
~/qiime/自分の班名/
├── raw_data/
├── manifest/
├── metadata/
├── results_qiime/
│   ├── results_dada2/       
│   ├── results_taxonomy/    
│   └── results_coremetrics/  
└── results_picrust2/
    ├── results_pipeline/     
    ├── results_visualization/
    └── results_export/       
```

> ✅ templateは共通構成を保つため削除しないでください。  
> ✅ 各班は自分のフォルダでのみ解析を行います。

---

### 🧩 2.Ubuntuの起動

**1.デスクトップ上の「Ubuntu」アイコンをダブルクリックして開く。**

**2.Ubuntuが開いたら、解析環境をアクティブにする。**

「📋」
```bash
conda activate q2-picrust2-amplicon-2024.5
```
**3.ホームディレクトリに移動し、作業場所を確認する。**

「📋」
```bash
cd ~
ls
```

**「qiime」フォルダが表示されていればOK。**

---

### 🧩 3.解析を行う班の設定

「📋」**□を消して自分の班名に変更**
```bash
export master="$HOME/qiime/□□□"
```

「📋」
```bash
cd "$master"
```

これ以降、すべてのコマンドで　**$master**　が自動的に班名に置き換わります。  

---

### 📁 4.フォルダ構成の確認

解析を始める前に、班フォルダが正しく作成されているか確認します。

「📋」
```bash
ls
```

出力が以下のようになっていれば問題ないです。
```bash
manifest  metadata  raw_data  results_picrust2  results_qiime
```

---

## 🧬 STEP 2｜FASTQファイルの配置
受け取ったシーケンスデータ

例：39320-04_S1_L001_R1_001.fastq.gz,

  39320-04_S1_L001_R2_001.fastq.gz
    
  を 先ほど作成した$master/raw_data/にUSBのフォルダから**ドラッグ＆ドロップ**し、コピーします。

先ほど作成した自分の班のraw_dateフォルダに正しくfastqファイルがコピーできているか確認してください。

例：
```bash
~/qiime/ishikawa/raw_data/39320-04_S1_L001_R1_001.fastq.gz
~/qiime/ishikawa/raw_data/39320-04_S1_L001_R2_001.fastq.gz
```

---

## 🧾 STEP 3｜manifestテンプレートの作成

このステップでは、FASTQファイルの対応表（manifestファイル）を作成します。

「📋」
```bash
bash ~/qiime/tools/make_manifest.sh
```
自動で $master/manifest/manifest.tsv が生成されます。  
生成後、manifest.tsv を LibreOffice Calc で開きます。

【開き方】
1. LibreOffice Calc を起動
2. 「ファイル」→「開く」から manifest.tsv を選択
3. 文字コード：UTF-8
4. 区切り文字：tab（☑ tab のみチェック）
5. OK を押して開く

**一列目の「sample-id」 列を自分のサンプル名（例：NC1～5, HF1～5, RBR1～5）に編集します。**

**例（Excel表示）**
| sample-id | forward-absolute-filepath                                 | reverse-absolute-filepath                                 |
| --------- | --------------------------------------------------------- | --------------------------------------------------------- |
| 例：NC1     | /home/ishikawa/qiime/raw_data/NC1_S1_L001_R1_001.fastq.gz | /home/ishikawa/qiime/raw_data/NC1_S1_L001_R2_001.fastq.gz |
| 例：HF1     | /home/ishikawa/qiime/raw_data/HF1_S2_L001_R1_001.fastq.gz | /home/ishikawa/qiime/raw_data/HF1_S2_L001_R2_001.fastq.gz |


以上のようにsample_idの編集を行った後、

**「ファイル」→「保存」→「Use テキスト CSV Format」**

うまく名前の編集・保存ができているかを以下で確認します。

「📋」
```bash
head manifest/manifest.tsv
```

出力が以下のようになっていれば問題ないです。
```bash
sample-id       forward-absolute-filepath       reverse-absolute-filepath
NC1     /home/haneishi/qiime/ishikawa/raw_data/39320-04_S1_L001_R1_001.fastq.gz /home/haneishi/qiime/ishikawa/raw_data/39320-04_S1_L001_R2_001.fastq.gz
NC2     /home/haneishi/qiime/ishikawa/raw_data/39320-05_S2_L001_R1_001.fastq.gz /home/haneishi/qiime/ishikawa/raw_data/39320-05_S2_L001_R2_001.fastq.gz
NC3     /home/haneishi/qiime/ishikawa/raw_data/39320-06_S3_L001_R1_001.fastq.gz /home/haneishi/qiime/ishikawa/raw_data/39320-06_S3_L001_R2_001.fastq.gz
NC4     /home/haneishi/qiime/ishikawa/raw_data/39320-07_S4_L001_R1_001.fastq.gz /home/haneishi/qiime/ishikawa/raw_data/39320-07_S4_L001_R2_001.fastq.gz
```

画面を閉じ、次の操作に移ってください。

「📋」
```bash
sed -i 's/\r$//' "$master/manifest/manifest.tsv"
sed -i '1s/^\xEF\xBB\xBF//' "$master/manifest/manifest.tsv"
```

---

## 🧬 STEP 4｜metadataの作成

「📋」
```bash
bash ~/qiime/tools/make_metadata.sh
```
自動で $master/metadata/metadata.tsv　が生成されます。 
metadata.tsv を LibreOffice Calc で開きます。

【開き方】
1. LibreOffice Calc を起動
2. 「ファイル」→「開く」から metadata.tsv を選択
3. 文字コード：UTF-8
4. 区切り文字：タブ（☑ タブ のみチェック）
5. OK を押して開く


**group 列（例：NC, HF, BR, RBR）は各班で手動入力してください。**

**例（Excel表示）**
| sample-id | group       |
| --------- | ----------- |
| #q2:types | categorical |
| NC1       | NC          |
| NC2       | NC          |
| HF1       | HF          |
| HF2       | HF          |

以上のようにgroupの編集を行った後、**💾保存**をしてください。

次に、後処理として以下を実行することで、ご認識を防ぎます。

「📋」
```bash
sed -i 's/\r$//' "$master/metadata/metadata.tsv"
sed -i '1s/^\xEF\xBB\xBF//' "$master/metadata/metadata.tsv"
```

---

## 🧫 STEP 5｜FASTQ → QIIME2（.qza）へのインポート

「📋」
```bash
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path $master/manifest/manifest.tsv \
  --output-path $master/results_qiime/results_dada2/demux.qza \
  --input-format PairedEndFastqManifestPhred33V2
```

✅ 成功メッセージ
Imported … as PairedEndFastqManifestPhred33V2 to …/demux.qza
が出ればOK。
生成後、results_qiime/results_dada2に**qzaファイル**が生成されているか確認してください。

---

## 📊 STEP 6｜シーケンス品質確認（Phredスコア）

「📋」
```bash
qiime demux summarize \
  --i-data "$master/results_qiime/results_dada2/demux.qza" \
  --o-visualization "$master/results_qiime/results_dada2/demux.qzv"
```
**生成された qzvファイルを 👉 https://view.qiime2.org にドラッグ＆ドロップして確認します。**

生成されたグラフを確認し、**トリミング長**を決定します。

この**トリミング長**をもとに、次にうつコードである↓の□□□の数字を各自変更します。

  `export TRUNC_F=□□□
   export TRUNC_R=□□□`

---

### 💡 なぜこの作業が必要なのか？

シーケンサーで得られたリード（配列）は、末端に近づくほど**エラーが増える**という特徴があります。
このエラー部分を残したまま解析を進めると、

＞DADA2 が誤って「ノイズを本物の配列」と判断したり、

＞リード同士の重なり（マージ）が失敗したり、

＞最終的なASV数や分類結果が不正確になる
などの問題が発生します。

そこで、　**品質が急激に低下する部分を切り落とす（トリミング）**　ことで、
データの信頼性を保ち、以降の解析（DADA2・分類・多様性解析など）を正確に行うことができます。

---

### 👀 Phredスコアグラフの見方

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

---

### ✂️ トリミング長の決め方（実践ステップ）

**1.Forward（R1）・Reverse（R2）の両方を確認**

各サンプルのグラフを開き、右端に向かってスコアが急激に下がる箇所を探します。

目安として、**Phredスコアの基準を30とし、グラフの黒部分が30を超え始める箇所**を探します。

**2.品質低下が始まる少し手前をカット位置に設定**

たとえば、R1のスコアが270bp付近で下がり始めたら、

**`TRUNC_F= 270 `**

とします。

同様に、R2が220bpあたりで下がるなら、

**`TRUNC_R=220`**

とします。

**3.短すぎるトリミングは避ける**

ForwardとReverseの重なりが150bp以上残るようにします。
そうしないと、マージ（重ね合わせ）ができずにエラーが発生します。

```
マージ＝（Forward:Lf）＋（Reverse:Lr）－253
```
今回は

270＋220－253＝**237**

で、十分マージを残すことができています。

このトリミング長をもとに次のSTEP7での数字を変更してください

---

## 🧮 STEP 7｜DADA2によるノイズ除去とASV化
DADA2処理は少し時間がかかったり、途中で落ちたりする可能性があるので注意してください。

最短で30分位を見積もっておくといいと思います。

「📋」**□□□を決定したトリミング長に決定してください。**
```bash
tmux new -s dada2 "
bash -lc '
  export master=\"$master\"
  export TRUNC_F=□□□
  export TRUNC_R=□□□
  bash ~/qiime/tools/run_dada2.sh
'" \; attach
```
実行後は自動的に進捗画面に移行し

**リアルタイムで進捗（Filtering / Learning Error Rates など）** が表示されます。👀

また、解析途中でも、一分ごとに[PROGRESS]が表示されるため、基本的には放置で大丈夫です。

この解析には**10分以上**時間がかかることが予想されます。

5分以上[PROGRESS]が表示されなかった場合は、処理が停止またはエラーで落ちた可能性ありです。

その場合、[ERROR] 異常終了と出てくると思うので、再度上のコードを実施してみてください。

正常に終了すると[DONE] 正常終了と表示されるはずです。

完了後、自動で以下3つの可視化ファイルが、results_dada2に生成されます。

 `table.qzv`                   `rep-seqs.qzv`             `denoising-stats.qzv` 

各qzvファイルを👉 https://view.qiime2.org にドラッグ＆ドロップして確認します。

確認方法は以下を参考にしてください。

✅ DADA2出力の簡易チェック（品質確認）

| ファイル名                   | どこを見るか（英語画面上）                              | 確認内容        | 判断の目安                                                                                |
| ----------------------- | ------------------------------------------ | ----------- | ------------------------------------------------------------------------------------ |
| **table.qzv**           | 「**Table summary → Frequency per sample**」 | サンプルごとのリード数 | 🔹 **Minimum frequency** が **3000以上ならOK**<br>🔹 極端に少ないサンプル（例：1000未満）があれば除外検討         |
| **rep-seqs.qzv**        | 「**Overview → Sequence Lengths**」          | 代表配列の長さ分布   | 🔹 多くが **250〜300 bp前後** ならOK（V4領域の場合）<br>🔹 短すぎる／長すぎる配列が多い場合 → トリミング設定を見直す           |
| **denoising-stats.qzv** | 「**Overview → Interactive Sample Detail**」 | 各サンプルでの除去率  | 🔹 **non-chimeric（最終列）** が数千以上ならOK<br>🔹 途中で極端に減っている（inputに対し非キメラが10％未満）サンプル → 品質要確認 |

そして、**table.qzv** の**Maximum・Minimum frequency**は**STEP10で↓の□□□を調整する際に用います。**

　`--p-sampling-depth □□□`

例：Maximum frequency＝8343　　Minimum frequency＝3349

のように覚えておいてください。



## 🧬 STEP 8｜分類（SILVA分類器）

目的：DADA2で得た代表配列を既知データベース（SILVA）と照合し、菌種を分類します。

「📋」
```bash
qiime feature-classifier classify-sklearn \
  --i-classifier ~/qiime/databases/silva-138.1-nr99-v4-classifier.qza \
  --i-reads "$master/results_qiime/results_dada2/rep-seqs.qza" \
  --o-classification "$master/results_qiime/results_taxonomy/taxonomy.qza" \
  --p-reads-per-batch 50 \
  --p-n-jobs 1
```
✅ 出力：

・taxonomy.qza（分類結果）

分類結果を可視化します👇

「📋」
```bash
qiime metadata tabulate \
  --m-input-file "$master/results_qiime/results_taxonomy/taxonomy.qza" \
  --o-visualization "$master/results_qiime/results_taxonomy/taxonomy.qzv"
```
results_qiime\results_taxonomyに**taxonomy.qzv**が生成されます。

→ taxonomy.qzv を 👉 https://view.qiime2.org にドラッグして確認。

**分類された菌群**（例：Firmicutes, Bacteroidetes, Lactobacillus など）が見られます。

---

## 🧩 STEP 9｜分類結果の可視化（Taxa Bar Plot）

### 目的：分類結果をグループ別の棒グラフで表示し、菌群の構成を比較します。

「📋」
```bash
qiime taxa barplot \
  --i-table "$master/results_qiime/results_dada2/table.qza" \
  --i-taxonomy "$master/results_qiime/results_taxonomy/taxonomy.qza" \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --o-visualization "$master/results_qiime/results_taxonomy/taxa-bar-plots.qzv"
```

✅ 出力：
・taxa-bar-plots.qzv（分類棒グラフ）

👉 https://view.qiime2.org で開くと

**グループごとに菌構成の割合**（例：Firmicutes/Bacteroidetes比など）を確認できます。

上部のLevelを1～7で変えることで**より細かい菌構成の比較**ができると思います。

| Level   | 英語名     | 日本語名 | 例                |
| ------- | ------- | ---- | ---------------- |
| 1 | Kingdom | 界    | Bacteria         |
| 2 | Phylum  | 門    | Firmicutes       |
| 3 | Class   | 綱    | Clostridia       |
| 4 | Order   | 目    | Lachnospirales   |
| 5 | Family  | 科    | Lachnospiraceae  |
| 6 | Genus   | 属    | Blautia          |
| 7 | Species | 種    | Blautia wexlerae |


また、exportでCSVを出し、Excelで調整することも可能です。

---

## 🧠 STEP 10｜多様性解析（α・β多様性）

### 目的：菌の「豊かさ」や「グループ間の違い」を解析するため、α・β多様性解析を行います。

DADA2で得られたASV配列をもとに、**系統樹を作成 → α・β多様性**を解析します。

### 🪴 系統樹の作成

代表配列（rep-seqs.qza）から、系統解析用のツリーを自動で構築します。

「📋」
```bash
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences "$master/results_qiime/results_dada2/rep-seqs.qza" \
  --o-alignment "$master/results_qiime/results_coremetrics/aligned-rep-seqs.qza" \
  --o-masked-alignment "$master/results_qiime/results_coremetrics/masked-aligned-rep-seqs.qza" \
  --o-tree "$master/results_qiime/results_coremetrics/unrooted-tree.qza" \
  --o-rooted-tree "$master/results_qiime/results_coremetrics/rooted-tree.qza"
```

✅ 出力：

・rooted-tree.qza（系統樹データ）

→**α・β多様性の計算**に使用します。

### 🌿 多様性解析

#### 🧩 概要
多様性解析は以下の2つに分かれます。
| 種類       | 内容               | depthの決め方                                |
| -------- | ---------------- | ---------------------------------------- |
| **α多様性** | 各サンプル内の菌の豊かさ・均一性 | α-rarefactionで飽和を確認して固定depthを設定（例：10000） |
| **β多様性** | グループ間の菌叢構造の違い    | STEP7で確認したMinimum frequencyを使用（例：3300）   |
α多様性解析は

**飽和深度の確認→解析→結果可視化→統計解析**

β多様性解析は

**解析→結果可視化→統計解析**

で進むため、👉 https://view.qiime2.org で見るのは

**可視化後→確認程度　　統計解析後→解析結果**　がいいと思います。

#### α多様性解析
**①飽和深度の確認（α-rarefaction）**

α多様性指標がどのdepthで安定（飽和）するかを確認します。

STEP7で**table.qzv** の**Maximum frequency**で確認した数値を参考に**以下の□□□の数値を決定**します。

例えば、Minimum frequency＝**8343**の場合、
**`--p-max-depth 8343`**

「📋」
```bash
qiime diversity alpha-rarefaction \
  --i-table "$master/results_qiime/results_dada2/table.qza" \
  --i-phylogeny "$master/results_qiime/results_coremetrics/rooted-tree.qza" \
  --p-max-depth 8343 \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --o-visualization "$master/results_qiime/results_coremetrics/alpha_rarefaction.qzv"
```
✅ 出力

`/results_coremetrics/alpha_rarefaction.qzv`

　→👉 https://view.qiime2.org で確認し、

 **ShannonやFaith PDが横ばいになるdepth**を基準に以降の解析depthを決定します。
 
---

**② 多様性指数の算出（固定depth）**

飽和カーブで得たdepthをもとに、α多様性指標を一括算出します。

depthを下に、以下の□□□の変更をお願いします。

例：`--p-sampling-depth 3000`

「📋」
```bash
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny "$master/results_qiime/results_coremetrics/rooted-tree.qza" \
  --i-table "$master/results_qiime/results_dada2/table.qza" \
  --p-sampling-depth □□□ \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --output-dir "$master/results_qiime/results_coremetrics/alpha_pack" && \
cd "$master/results_qiime/results_coremetrics/alpha_pack" && \
rm -f *_distance_matrix.qza *_pcoa_results.qza *_emperor.qzv
```

✅ 出力（主要ファイル）
```
alpha_pack/
├─ shannon_vector.qza
├─ faith_pd_vector.qza
├─ observed_features_vector.qza
└─ evenness_vector.qza
```

**③ 可視化・統計解析（α多様性）**

ここでは、**各グループ間でα多様性に有意差があるか**を検定します。

まず、結果格納用フォルダを作成します。

「📋」
```bash
mkdir -p "$master/results_qiime/results_coremetrics/alpha"
```

**🔹 可視化（index）**

「📋」
```bash
for f in shannon faith_pd observed_features evenness; do
  qiime metadata tabulate \
    --m-input-file "$master/results_qiime/results_coremetrics/alpha_pack/${f}_vector.qza" \
    --o-visualization "$master/results_qiime/results_coremetrics/alpha/${f}_index.qzv"
done
```

✅ 出力例

```bash
alpha/
├─ shannon_index.qzv
├─ faith_pd_index.qzv
├─ observed_features_index.qzv
└─ evenness_index.qzv
```
👉 https://view.qiime2.org で開くと、各サンプルのα多様性値が確認できます。

**🔹 統計解析（group significance）**

各群間で有意差を検定します（Kruskal-Wallis法）。

「📋」
```bash
for f in shannon faith_pd observed_features evenness; do
  qiime diversity alpha-group-significance \
    --i-alpha-diversity "$master/results_qiime/results_coremetrics/alpha_pack/${f}_vector.qza" \
    --m-metadata-file "$master/metadata/metadata.tsv" \
    --o-visualization "$master/results_qiime/results_coremetrics/alpha/${f}_significance.qzv"
done
```

✅ 出力例

```
alpha/
├─ shannon_significance.qzv
├─ faith_pd_significance.qzv
├─ observed_features_significance.qzv
└─ evenness_significance.qzv
```
👉 qvzファイルを https://view.qiime2.org で有意差等を確認してください。

**📈 確認ポイント**

・H値・p-value を確認（p < 0.05 で有意差あり）

・下部の _pairwise comparison_ で群間差を詳細に確認可能

---

#### β多様性解析

**① 群間差の解析（core-metrics）**

STEP7で**table.qzv** の**Minimum frequency**で確認した数値を参考に**以下の□□□の数値を決定**します。

例えば、Minimum frequency＝**3349**の場合、
**`--p-sampling-depth 3349`**

「📋」
```bash
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny "$master/results_qiime/results_coremetrics/rooted-tree.qza" \
  --i-table "$master/results_qiime/results_dada2/table.qza" \
  --p-sampling-depth □□□ \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --output-dir "$master/results_qiime/results_coremetrics/beta"
```

✅ 出力

```bash
beta/
├─ bray_curtis_emperor.qzv
├─ weighted_unifrac_emperor.qzv
├─ unweighted_unifrac_emperor.qzv
├─ *_distance_matrix.qza
└─ *_pcoa_results.qza
```

👉 ~emperor.qvzを https://view.qiime2.org で開くと、

群間の分離が明瞭なほど、菌叢構造に違いがあると判断できます。

---

**② 統計解析（群間差の有意性）**

各群間のβ多様性差を PERMANOVA で検定します。

「📋」
```bash
for f in bray_curtis weighted_unifrac unweighted_unifrac; do
  qiime diversity beta-group-significance \
    --i-distance-matrix "$master/results_qiime/results_coremetrics/beta/${f}_distance_matrix.qza" \
    --m-metadata-file "$master/metadata/metadata.tsv" \
    --m-metadata-column Group \
    --p-method permanova \
    --o-visualization "$master/results_qiime/results_coremetrics/beta/${f}_significance.qzv"
done
```

✅ 出力
```bash
beta/
├─ bray_curtis_significance.qzv
├─ weighted_unifrac_significance.qzv
└─ unweighted_unifrac_significance.qzv
```

👉 qvzファイルを https://view.qiime2.org で有意差等を確認してください。

**📊 確認ポイント**

・pseudo-F値と p-value を確認（p < 0.05 で有意差あり）

・分離傾向が見られたβプロットとの整合性もチェック


**💡 補足（どんな結果が見られる？）**

| 指標                | 意味             | 解釈のポイント           |
| ----------------- | -------------- | ----------------- |
| Shannon index     | 種の多様性（豊かさ＋均一性） | 値が高いほど多様          |
| Faith’s PD        | 系統的な多様性        | 系統的に異なる菌が多いほど高い   |
| Observed features | ASV数           | 実際に検出された種数の近似     |
| Bray-Curtis PCoA  | 群間の違い          | サンプル間の距離や分離傾向を視覚化 |

>💬 ポイント：
>群間の傾向を見る場合は PCoA プロット（β多様性）を、
>群内の多様さを比較する場合は Shannon指数（α多様性）を確認します。

---

## 🧩 STEP11｜差次的菌種解析（ANCOM解析）
### 🎯 目的
ANCOM（Analysis of Composition of Microbiomes）は、
**グループ間で有意に多い菌（特徴菌）を統計的に抽出**する解析手法です。

---

#### 🧪 ① 実行コード
まず、結果格納用フォルダを作成します。

「📋」
```bash
mkdir -p "$master/results_qiime/results_ancom"
```

以下で、**比較を行うlevel（分類階層）**　の指定を行います。

目安を参考に設定してください。

**💡 level（分類階層）の目安**
| level | 階層           | 例                    | 用途の目安               |
| :---: | :----------- | :------------------- | :------------------ |
|   5   | Family（科）    | Lachnospiraceae      | 大まかな群の傾向を確認したいとき    |
| **6** | **Genus（属）** | **Bacteroides**      | **腸内細菌叢研究で一般的（推奨）** |
|   7   | Species（種）   | Bacteroides vulgatus | データの解像度が十分な場合のみ推奨   |

指定は以下のコードの**LEVEL=□**の変更からできます。
例：**`LEVEL=6`**

「📋」
```bash
LEVEL=□
```

「📋」
```bash
qiime taxa collapse \
  --i-table "$master/results_qiime/results_dada2/table.qza" \
  --i-taxonomy "$master/results_qiime/results_taxonomy/taxonomy.qza" \
  --p-level $LEVEL \
  --o-collapsed-table "$master/results_qiime/results_ancom/table_level${LEVEL}.qza" && \
qiime composition add-pseudocount \
  --i-table "$master/results_qiime/results_ancom/table_level${LEVEL}.qza" \
  --o-composition-table "$master/results_qiime/results_ancom/comp_level${LEVEL}.qza" && \
qiime composition ancom \
  --i-table "$master/results_qiime/results_ancom/comp_level${LEVEL}.qza" \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --m-metadata-column Group \
  --o-visualization "$master/results_qiime/results_ancom/ancom_level${LEVEL}.qzv"
```

**解析を行うLevelを変更したい場合は、Level=の指定から行えばできます。**

✅ 出力
```
results_ancom/
├─ table_level6.qza        ← 集計後テーブル
├─ comp_level6.qza         ← 補正済みテーブル
└─ ancom_level6.qzv        ← 結果（可視化用）
```
👉 ancom_level6.qzv を
https://view.qiime2.org にドラッグして確認します。

---

#### 🔍 ② 結果の見方
| 項目                         | 意味               | 解釈のポイント         |
| -------------------------- | ---------------- | --------------- |
| **W値（W-statistic）**        | 他菌との比較で有意差があった回数 | 値が大きいほど群間差が明確   |
| **Reject null hypothesis** | “True”なら群間で有意差あり | 特徴菌として注目        |
| **箱ひげ図・棒グラフ**              | 各群の菌割合を表示        | どちらの群で多いか直感的に確認 |

**📈  解釈例**
| 結果                             | 解釈                          |
| ------------------------------ | --------------------------- |
| *Bacteroides*（W=85, True）      | RBR群で有意に多い属。脂質代謝への関与が推定される。 |
| *Lachnospiraceae*（W=25, False） | 差は検出されず、群間でほぼ同程度。           |

---

## 🧬 STEP 12｜PICRUSt2解析
### 🎯 目的
PICRUSt2解析では、メタゲノム解析で得た配列情報を下に

**16S配列から、腸内細菌が持つ代謝経路やその機能性を予測します。**

---

#### ①PICRUSt2解析の出力

DADA2で得られたASVテーブルと代表配列をもとに、KEGGやMetaCyc経路の機能予測を行います。

以下のコードで最初の出力を行いますが、

この段階も**時間がかかることが予想されます**ので注意してください。


「📋」
```bash
qiime picrust2 full-pipeline \
  --i-table "$master/results_qiime/results_dada2/table.qza" \
  --i-seq "$master/results_qiime/results_dada2/rep-seqs.qza" \
  --p-threads 1 \
  --output-dir "$master/results_picrust2/results_pipeline"
```

✅ 出力：
| 出力ファイル                  | 内容                        |
| ----------------------- | ------------------------- |
| `KO_metagenome.qza`     | KEGG遺伝子機能の予測              |
| `EC_metagenome.qza`     | 酵素（EC番号）ごとの活性予測           |
| `pathway_abundance.qza` | 代謝経路ごとの量（MetaCyc Pathway） |
| `pathway_coverage.qza`  | 経路の完全性（Coverage）          |

---

#### ②機能量の統計解析（群間比較）

予測された代謝経路（pathway_abundance.qza）をもとに、**群間での有意差を検定**します。

ここでは、qiime composition プラグインを用いたANCOM法で、**群間で多い経路**を抽出します。

「📋」
```bash
mkdir -p "$master/results_picrust2/results_visualization/statistics"

qiime composition add-pseudocount \
  --i-table "$master/results_picrust2/results_pipeline/pathway_abundance.qza" \
  --o-composition-table "$master/results_picrust2/results_visualization/statistics/pathway_comp.qza"

qiime composition ancom \
  --i-table "$master/results_picrust2/results_visualization/statistics/pathway_comp.qza" \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --m-metadata-column Group \
  --o-visualization "$master/results_picrust2/results_visualization/statistics/ancom_pathway.qzv"
```
✅ 出力：
```
results_visualization/statistics/
├─ pathway_comp.qza
└─ ancom_pathway.qzv
```

👉 ancom_pathway.qzv を [QIIME2 View](https://view.qiime2.org )

 にドラッグして、群間で有意差のある代謝経路（Reject=Trueの行）を確認します。

 ---

#### ③主成分分析（PCAによる群間構造の可視化）
機能プロファイル全体の傾向を把握するため、**主成分分析（PCA）**　を行います。

「📋」
```bash
OUT="$master/results_picrust2/results_visualization/pca"
METRIC=braycurtis   
mkdir -p "$OUT" && \

qiime feature-table relative-frequency \
  --i-table "$master/results_picrust2/results_pipeline/pathway_abundance.qza" \
  --o-relative-frequency-table "$OUT/pathway_rel.qza" && \
qiime diversity beta \
  --i-table "$OUT/pathway_rel.qza" \
  --p-metric $METRIC \
  --o-distance-matrix "$OUT/pathway_${METRIC}_dm.qza" && \
qiime diversity pcoa \
  --i-distance-matrix "$OUT/pathway_${METRIC}_dm.qza" \
  --o-pcoa "$OUT/pathway_${METRIC}_pcoa.qza" && \
qiime emperor plot \
  --i-pcoa "$OUT/pathway_${METRIC}_pcoa.qza" \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --o-visualization "$OUT/pathway_${METRIC}_emperor.qzv"
```
✅ 出力フォルダ構成：
```
results_visualization/pca/
├─ pathway_rel.qza                # 相対化テーブル
├─ pathway_braycurtis_dm.qza      # 距離行列（Bray-Curtis）
├─ pathway_braycurtis_pcoa.qza    # 主成分分析結果
└─ pathway_braycurtis_emperor.qzv # 3D可視化（QIIME2 Viewで確認）
```
👉 pathway_braycurtis_emperor.qzv を
https://view.qiime2.org
 にドラッグ＆ドロップして確認します。

**🔍 解析結果の見方**
| 主成分      | 内容           | 解釈のポイント                   |
| -------- | ------------ | ------------------------- |
| **PC1**  | 最も大きな変動要因    | 群間で明確な分離が見られれば、代謝傾向の違いを示す |
| **PC2**  | 次に大きな変動要因    | 群内のばらつきや副次的な傾向を反映         |
| **点の距離** | サンプル間の機能的類似度 | 近い点ほど代謝経路構成が似ている          |

**💡 ワンポイント**

・デフォルトでは Bray–Curtis距離を使用（一般的で安定的）

　→ METRIC= を変更すれば、他の距離指標も試せます（例：jaccard、euclidean）

・群間が明確に分かれる場合、その差を生む代謝経路はSTEP12②（ANCOM）で確認可能です。

---

## 📈 STEP 13｜機能の可視化＆エクスポート

### 🎯 目的
PICRUSt2 の結果を、グループ比較しやすい形に整形し、
図（ヒートマップ）と外部ソフト向けTSVを作ります。

---

#### ① 前処理：相対化＆グループ平均テーブルの作成

各サンプルの値を相対化し、metadata.tsv の Group 列で平均（mean-ceiling）を取ってグループ代表にします。

「📋」
```bash
PIPE="$master/results_picrust2/results_pipeline"
OUTV="$master/results_picrust2/results_visualization"
OUTE="$master/results_picrust2/results_export"
META="$master/metadata/metadata.tsv"
mkdir -p "$OUTV/barplot" "$OUTE"

# 1) カウントのままグループ化（サンプル軸）
qiime feature-table group \
  --i-table "$PIPE/pathway_abundance.qza" \
  --m-metadata-file "$META" \
  --m-metadata-column Group \
  --p-mode sum \
  --p-axis sample \
  --o-grouped-table "$OUTV/barplot/pathway_by_group.qza"

# 2) 相対化（各グループで合計=1）
qiime feature-table relative-frequency \
  --i-table "$OUTV/barplot/pathway_by_group.qza" \
  --o-relative-frequency-table "$OUTV/barplot/pathway_by_group_rel.qza"

# 3) サマリーはカウント側に対して実施
qiime feature-table summarize \
  --i-table "$OUTV/barplot/pathway_by_group.qza" \
  --o-visualization "$OUTV/barplot/pathway_by_group_summary.qzv"
```

👉 pathway_by_group_summary.qzv を
https://view.qiime2.org
 にドラッグ＆ドロップして確認します。

ここで各グループの合計リード数や特徴量数を確認します。

 ---

#### ② 可視化：上位パスウェイのヒートマップ

全部を描くと多すぎて見づらいので、「出現量の少ない経路」をあらかじめ除外します。

このとき使うのが --p-min-frequency **（最小リード数）** です。

**🔍 閾値（しきい値）の決め方**

feature-table filter-features の --p-min-frequency は

整数（リード数） で指定します。

この値より少ない特徴（＝パスウェイ）は削除されます。

**📘 基本の考え方**

閾値は、**「全体のリード数 × 割合」** で計算します。

例）

・各グループ（sample）の平均リード数（Mean frequency）が **1,783,000** のとき
「全体の0.1%未満を除外」したいなら：

`1,783,000 × 0.001 ＝ 約 1,800`

👉 --p-min-frequency 1800 と指定すればOKです。

**💡 簡単な目安表**
| 割合（相対）   | リード数の目安（Mean frequency = 1,783,000 の場合） | 効果の目安            |
| -------- | --------------------------------------- | ---------------- |
| 1%       | 約17,800                                 | 主要経路のみ残す（やや厳しめ）  |
| **0.1%** | **約1,800（おすすめ）**                        | ノイズを除きつつ主要経路を保持  |
| 0.05%    | 約900                                    | 緩やかな除外（細かい経路も残す） |
| 0.01%    | 約180                                    | ほぼ全経路を残す（図が多くなる） |


「📋」
```bash
# 低頻度パスウェイを除外（例：全体の0.1%未満 ≒ 約1,800リード以下をカット）
qiime feature-table filter-features \
  --i-table "$OUTV/barplot/pathway_by_group.qza" \
  --p-min-frequency 1800 \
  --o-filtered-table "$OUTV/barplot/pathway_by_group_top.qza"

# 相対化（上位のみ）
qiime feature-table relative-frequency \
  --i-table "$OUTV/barplot/pathway_by_group_top.qza" \
  --o-relative-frequency-table "$OUTV/barplot/pathway_by_group_rel_top.qza"

# ヒートマップ（グループ比較の俯瞰図）
qiime feature-table heatmap \
  --i-table "$OUTV/barplot/pathway_by_group_top.qza" \
  --m-sample-metadata-file "$META" \
  --m-sample-metadata-column Group \
  --o-visualization "$OUTV/barplot/pathway_by_group_heatmap.qzv"
```

👉 pathway_by_group_heatmap.qzv を
https://view.qiime2.org
 にドラッグ＆ドロップして確認します。

**濃いマスが多いグループ＝その経路が豊富なグループ**です。

見づらい場合は、--p-min-frequency の値を調整して、
残すパスウェイ数をコントロールしてください。

>注：Taxa Bar Plot は分類（SILVA）専用なので、パスウェイはヒートマップが扱いやすいです。
>棒グラフが必要なら、このあとTSVへエクスポートしてExcel/GraphPad/Rで描くのが早いです。

---

#### ③ エクスポート：TSVで持ち出し（論文化・学会図用）

グループ平均（相対量）テーブルをTSVに変換します。

「📋」
```bash
qiime tools export \
  --input-path "$OUTV/barplot/pathway_by_group_rel_top.qza" \
  --output-path "$OUTE/export_pathway_by_group_rel_top"

biom convert \
  -i "$OUTE/export_pathway_by_group_rel_top/feature-table.biom" \
  -o "$OUTE/pathway_by_group_rel_top.tsv" \
  --to-tsv
```
👉 pathway_by_group_rel_top.tsv を Excel や GraphPad Prism に読み込み、
上位20～30経路を棒グラフにすれば発表映えします。

---



---

## 📘 参考情報
- QIIME2 Documentation（2024.5）: https://docs.qiime2.org/2024.5/
- SILVA Database (v138.1): https://www.arb-silva.de/
- 使用環境：Ubuntu 24.04（WSL2）, q2-picrust2-amplicon-2024.5
- 作成：SEROL1（QIIME2共通解析マニュアルプロジェクト）
