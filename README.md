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

同じくデスクトップ上にあるqiimeフォルダを開き、
中に以下のファイルが有ることを確認してください。

```
~/qiime/
├── tools/      ←解析用スクリプトフォルダ（触らない）
├── template/   ←各班が使用する雛形フォルダ（コピペして使用する）
├── database/ 　←解析用機器（触らない）
```

以上の/templateフォルダをフォルダをコピーして**自分の班名**に変更します。
```
~/qiime/
├── tools/      ←解析用スクリプトフォルダ（触らない）
├── template/   ←各班が使用する雛形フォルダ（消さないように注意）
├── database/ 　←解析用機器（触らない）
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
├── database/
```

```
~/qiime/自分の判明/
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

**🧩 3.解析を行う班の設定**

「📋」**□を消して自分の班名に変更**
```bash
export master="/home/seeei/qiime/フォルダ名"
```

「📋」
```bash
cd $master
```

これ以降、すべてのコマンドで　**$master**　が自動的に班名に置き換わります。  

---

**📁 4.フォルダ構成の確認**

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
    
  を 先ほど作成した$master/raw_data/にコピーします。

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
生成後、manifest.tsvをexcelで開き

**一列目の「sample-id」 列を自分のサンプル名（例：NC1～5, HF1～5, RBR1～5）に編集します。**

**例（Excel表示）**
| sample-id | forward-absolute-filepath                                 | reverse-absolute-filepath                                 |
| --------- | --------------------------------------------------------- | --------------------------------------------------------- |
| 例：NC1     | /home/ishikawa/qiime/raw_data/NC1_S1_L001_R1_001.fastq.gz | /home/ishikawa/qiime/raw_data/NC1_S1_L001_R2_001.fastq.gz |
| 例：HF1     | /home/ishikawa/qiime/raw_data/HF1_S2_L001_R1_001.fastq.gz | /home/ishikawa/qiime/raw_data/HF1_S2_L001_R2_001.fastq.gz |


以上のようにsample_idの編集を行った後、**💾上書き保存**をしてください。

次に、後処理として以下を実行することで、ご認識を防ぎます。

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
生成後、metadata.tsvをexcelで開き、
**group 列（例：NC, HF, BR, RBR）は各班で手動入力してください。**

**例（Excel表示）**
| sample-id | group       |
| --------- | ----------- |
| #q2:types | categorical |
| NC1       | NC          |
| NC2       | NC          |
| HF1       | HF          |
| HF2       | HF          |

以上のようにgroupの編集を行った後、**💾上書き保存**をしてください。

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

目安として、**Phredスコアの基準を30とし、グラフの黒部分が30を超え始める箇所**を探します。

**2.品質低下が始まる少し手前をカット位置に設定**

たとえば、R1のスコアが270bp付近で下がり始めたら、

TRUNC_F=**270**

とします。

同様に、R2が220bpあたりで下がるなら、

TRUNC_R=**220**

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
tmux new -s dada2 -d "
bash -lc '
  export master=\"$master\"
  export TRUNC_F=□□□
  export TRUNC_R=□□□
  bash ~/qiime/tools/run_dada2.sh
'"
```
👀 進行状況の確認

・ログで追う：

「📋」
```bash
tail -f "$master"/results_qiime/dada2_*.log
```

ログで追うと、次のような順序で表示が進みます👇

```bash
[INFO] 開始: 2025-11-04 17:01:45 env=q2-picrust2-amplicon-2024.5
[INFO] master=/home/seeei/qiime/test  TRUNC_F=265 TRUNC_R=217  TRIM_F=0 TRIM_R=0
R version 4.3.3 (2024-02-29)
Loading required package: Rcpp
DADA2: 1.30.0 / Rcpp: 1.0.13.1 / RcppParallel: 5.1.9
2) Filtering
3) Learning Error Rates
4) Denoising
5) Merging paired reads
6) Constructing sequence table
7) Removing chimeras
8) Writing output files
[INFO] 可視化ファイル生成中...
[DONE] 終了: 2025-11-04 23:47:18
```

完了後、自動で以下3つの可視化ファイルが生成されます

📊 出力ファイル一覧：
| ファイル名                                                       | 内容                 | 次の用途        |
| ----------------------------------------------------------- | ------------------ | ----------- |
| `results_dada2/table.qza` / `table.qzv`                     | ASV 出現数テーブル        | 多様性解析の基盤    |
| `results_dada2/rep-seqs.qza` / `rep-seqs.qzv`               | 各 ASV の代表配列        | タキソノミー付与に使用 |
| `results_dada2/denoising-stats.qza` / `denoising-stats.qzv` | デノイズ統計（除去率・ペア合致など） | 品質確認        |

---

各qzvファイルを👉 https://view.qiime2.org にドラッグ＆ドロップして確認します。

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
→ taxonomy.qzv を 👉 https://view.qiime2.orgにドラッグして確認。

**分類された菌群**（例：Firmicutes, Bacteroidetes, Lactobacillus など）が見られます。

---

## 🧩 STEP 9｜分類結果の可視化（Taxa Bar Plot）

目的：分類結果をグループ別の棒グラフで表示し、菌群の構成を比較します。

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

👉 https://view.qiime2.orgで開くと

**グループごとに菌構成の割合**（例：Firmicutes/Bacteroidetes比など）を確認できます。

---

## 🧠 STEP 10｜多様性解析（α・β多様性）

菌の「豊かさ」や「グループ間の違い」を解析するため、**α・β多様性解析**を行います。

DADA2で得られたASV配列をもとに、系統樹を作成して多様性解析を一括実行します。

### 🪴 ① 系統樹の作成

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

・α・β多様性の計算に使用します。

### 🌿 ② 多様性解析

「📋」
```bash
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny "$master/results_qiime/results_coremetrics/rooted-tree.qza" \
  --i-table "$master/results_qiime/results_dada2/table.qza" \
  --p-sampling-depth 10000 \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --output-dir "$master/results_qiime/results_coremetrics/core-metrics-results"
```
✅ 出力：

・core-metrics-results/ フォルダ内に、以下がまとめて生成されます👇

| 出力ファイル                       | 内容                 | 確認方法                |
| ---------------------------- | ------------------ | ------------------- |
| shannon_vector.qza           | Shannon指数（多様性）     | view.qiime2.orgで可視化 |
| faith_pd_vector.qza          | 系統的多様性（Faith’s PD） | 同上                  |
| bray_curtis_emperor.qzv      | β多様性（PCoAプロット）     | グループ分離の確認           |
| evenness_vector.qza          | 種の均一性              | 参考指標                |
| observed_features_vector.qza | ASV数               | 種の豊かさの目安            |

### 📊 ③ 結果の確認

結果は以下のコマンドで一覧できます。

「📋」
```bash
ls "$master/results_qiime/core-metrics-results"
```

出力例：
```bash
bray_curtis_emperor.qzv
shannon_vector.qza
faith_pd_vector.qza
observed_features_vector.qza
evenness_vector.qza
```
👉 生成された .qzv ファイルを
https://view.qiime2.org
 にドラッグして可視化しましょう。

**💡 補足（どんな結果が見られる？）**

| 指標                | 意味             | 解釈のポイント           |
| ----------------- | -------------- | ----------------- |
| Shannon index     | 種の多様性（豊かさ＋均一性） | 値が高いほど多様          |
| Faith’s PD        | 系統的な多様性        | 系統的に異なる菌が多いほど高い   |
| Observed features | ASV数           | 実際に検出された種数の近似     |
| Bray-Curtis PCoA  | 群間の違い          | サンプル間の距離や分離傾向を視覚化 |


---

## 🧬 STEP 11｜PICRUSt2解析
目的：16S配列から、腸内細菌が持つ代謝経路を予測します。

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

## 📈 STEP 12｜相対値変換とエクスポート（KEGG/EC/Pathway）

STEP11（PICRUSt2解析）で生成された各ファイル用いて

**相対値（割合データ）に変換し、表形式で出力する作業**です。

「📋」（KEGG KO）
```bash
qiime feature-table relative-frequency \
  --i-table "$master/results_picrust2/results_pipeline/KO_metagenome.qza" \
  --o-relative-frequency-table "$master/results_picrust2/results_visualization/KO_metagenome_rel.qza"

qiime tools export \
  --input-path "$master/results_picrust2/results_visualization/KO_metagenome_rel.qza" \
  --output-path "$master/results_picrust2/results_export/export_KO_rel"

biom convert \
  -i "$master/results_picrust2/results_export/export_KO_rel/feature-table.biom" \
  -o "$master/results_picrust2/results_export/KO_metagenome_rel.tsv" \
  --to-tsv
```


「📋」（EC）
```bash
qiime feature-table relative-frequency \
  --i-table "$master/results_picrust2/results_pipeline/EC_metagenome.qza" \
  --o-relative-frequency-table "$master/results_picrust2/results_visualization/EC_metagenome_rel.qza"

qiime tools export \
  --input-path "$master/results_picrust2/results_visualization/EC_metagenome_rel.qza" \
  --output-path "$master/results_picrust2/results_export/export_EC_rel"

biom convert \
  -i "$master/results_picrust2/results_export/export_EC_rel/feature-table.biom" \
  -o "$master/results_picrust2/results_export/EC_metagenome_rel.tsv" \
  --to-tsv
```


「📋」（Pathway）
```bash
qiime feature-table relative-frequency \
  --i-table "$master/results_picrust2/results_pipeline/pathway_abundance.qza" \
  --o-relative-frequency-table "$master/results_picrust2/results_visualization/pathway_abundance_rel.qza"

qiime tools export \
  --input-path "$master/results_picrust2/results_visualization/pathway_abundance_rel.qza" \
  --output-path "$master/results_picrust2/results_export/export_pathway_rel"

biom convert \
  -i "$master/results_picrust2/results_export/export_pathway_rel/feature-table.biom" \
  -o "$master/results_picrust2/results_export/pathway_abundance_rel.tsv" \
  --to-tsv
```

## 📈 STEP 13｜グラフ化と解析（KEGG / EC / Pathway

**💡 目的**

STEP12で作成した .tsv ファイルは、
各サンプルごとの「機能（KEGG・EC・Pathway）の割合」が整理されたデータです。

ここではそのデータを使って、
**グループごとの差・特徴・相関** をわかりやすく可視化・解析していきます。

**🧾 使うファイル**
| フォルダ                 | ファイル名                       | 内容          |
| -------------------- | --------------------------- | ----------- |
| `export_KO_rel`      | `KO_metagenome_rel.tsv`     | KEGG遺伝子の相対量 |
| `export_EC_rel`      | `EC_metagenome_rel.tsv`     | 酵素番号の相対量    |
| `export_pathway_rel` | `pathway_abundance_rel.tsv` | 代謝経路の相対量    |

これらをExcelなどで開くと、
行＝機能名、列＝サンプル名 になっています。

**🧠 解析でできること（例）**
| 解析内容                            | 使用ソフト                          | 目的・例               |
| ------------------------------- | ------------------------------ | ------------------ |
| **① グループごとの平均・差分比較**            | Excel / R / GraphPad           | 例：RBR群とHF群で多い経路の比較 |
| **② 経路名の付け直し（MetaCyc ID → 名前）** | Excel / KEGG Mapper            | 結果を読みやすくするための翻訳    |
| **③ 主成分分析（PCA）やクラスタリング**        | R（prcomp, ggplot2）             | 各群の傾向を図で確認         |
| **④ 相関解析（スピアマン / ピアソン）**        | R / Excel                      | 例：胆汁酸量や血中脂質との関連を見る |
| **⑤ 棒グラフ・ヒートマップ作成**             | Excel / GraphPad / R（pheatmap） | 発表・論文用の見やすい図を作成    |

**📊 グループ平均を出してみる（Excel）**

1.pathway_abundance_rel.tsv を開く

2.グループごと（例：NC群・HF群・RBR群）のサンプル列を選択

3.平均値関数 =AVERAGE(範囲) で平均を算出

4.差分を計算して「増加」「減少」を判断

→ 値の大きい経路が「活発な代謝経路」を示します。

**RStudioによる可視化（初めてのRの使い方つき）**

**💡目的**

STEP12で作成した 相対値変換データ（例：pathway_abundance_rel.tsv） を、

RStudioを使って図としてわかりやすく可視化します。

ここでは例として「ヒートマップ」を作成します。

**🪟 1. RStudioの起動**

Ubuntuターミナルで以下を入力します。

「📋」
```bash
rstudio
```
GUI（グラフィカル画面）が起動したらOKです。

**📂 2. 作業フォルダを設定**

RStudioが起動したら、左下の Console に次のコードを入力します。

これにより、解析結果のあるフォルダを作業場所に指定します。

「📋」
```bash
setwd("/home/seeei/qiime/test/results_picrust2/export_pathway_rel/")
```

**🎨 3. ヒートマップの作成**

RStudioの Console に以下を入力します。

「📋」
```bash
library(pheatmap)
data <- read.table("pathway_abundance_rel.tsv", sep="\t", header=TRUE, row.names=1)
pheatmap(data, scale="row", clustering_distance_rows="correlation")
```

**💾 5. 図の保存**

ヒートマップを画像として保存する場合は：

Plots ペイン右上の「Export」→「Save as Image」
→ PNG / PDF / TIFF などの形式を選択可能です。



---

## 📘 参考情報
- QIIME2 Documentation（2024.5）: https://docs.qiime2.org/2024.5/
- SILVA Database (v138.1): https://www.arb-silva.de/
- 使用環境：Ubuntu 24.04（WSL2）, q2-picrust2-amplicon-2024.5
- 作成：SEROL1（QIIME2共通解析マニュアルプロジェクト）
