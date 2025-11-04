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
自動で $master/manifest/manifest.csv が生成されます。  
生成後、manifest.csvをexcelで開き

**一列目の「sample-id」 列を自分のサンプル名（例：NC1～5, HF1～5, RBR1～5）に編集します。**

「sample-id」列は forward と reverse がペアになるため

**同じサンプル名を2行続けて入力してください。**

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
 --input-path $master/manifest/manifest.csv \
 --output-path $master/results_qiime/demux.qza \
 --input-format PairedEndFastqManifestPhred33V2
```
✅ 成功メッセージ
Imported ... as PairedEndFastqManifestPhred33 to .../demux.qza
が出ればOK。
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

目安として、**Phredスコアの基準を30とし、グラフの黒部分が30を超え始める箇所**を探します。

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
  export TRUNC_F=□□□
  export TRUNC_R=□□□
  bash ~/qiime/tools/run_dada2.sh
'"
```
完了後、自動で以下3つの可視化ファイルが生成されます

📊 出力ファイル一覧：
| ファイル名                 | 内容         | 次の用途     |
| --------------------- | ---------- | -------- |
| `table.qza / qzv`     | ASV出現数テーブル | 多様性解析の基盤 |
| `rep-seqs.qza / qzv`  | 各ASVの代表配列  | 菌種分類に使用  |
| `denoising-stats.qzv` | 除去率・品質統計   | 品質確認     |


---

## 🧬 STEP 8｜分類（SILVA分類器）

目的：代表配列を既知データベース（例：Silva）と照合し、菌種レベルで同定する

「📋」
```bash
qiime feature-classifier classify-sklearn \
  --i-classifier ~/qiime/databases/silva-138.1-nr99-v4-classifier.qza \
  --i-reads $master/results_qiime/rep-seqs.qza \
  --o-classification $master/results_qiime/taxonomy.qza
```
✅ 出力：

・taxonomy.qza（分類結果）

これを可視化するために次のSTEPへ。

---

## 🧩 STEP 9｜分類結果の可視化（Taxa Bar Plot）

目的：菌群の構成比を棒グラフとして表示

「📋」
```bash
qiime taxa barplot \
  --i-table "$master/results_qiime/table.qza" \
  --i-taxonomy "$master/results_qiime/taxonomy.qza" \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --o-visualization "$master/results_qiime/taxa-bar-plots.qzv"
```

✅ 出力：
・taxa-bar-plots.qzv

→ ブラウザで qiime tools view taxa-bar-plots.qzv

→ グループごとの菌構成（例：Firmicutes / Bacteroidetes 比など）を確認。

---

## 🧠 STEP 10｜多様性解析

### α多様性
目的：1サンプル内の多様性（菌の種類の豊かさ）を評価する

「📋」
```bash
qiime diversity alpha \
  --i-table $master/results_qiime/table.qza \
  --p-metric shannon \
  --o-alpha-diversity $master/results_qiime/alpha_shannon.qza
```
✅ 出力例：
| 指標                | 意味          |
| ----------------- | ----------- |
| Shannon index     | 多様性（種数＋均一性） |
| Faith’s PD        | 系統的多様性      |
| Observed features | ASV数（種数の近似） |


### β多様性
目的：サンプル間の構成差を評価（グループ差を可視化）

「📋」
```bash
qiime emperor plot \
  --i-pcoa "$master/results_qiime/core-metrics-results/bray_curtis_pcoa_results.qza" \
  --m-metadata-file "$master/metadata/metadata.tsv" \
  --o-visualization "$master/results_qiime/bray-curtis-emperor.qzv"
```
✅ 出力：

・bray-curtis-emperor.qzv

→ PCoAプロットとしてグループ分離を確認。

---

## 🧬 STEP 11｜PICRUSt2解析
目的：16S配列から代謝経路（KEGG Pathway）を予測

「📋」
```bash
qiime picrust2 full-pipeline \
  --i-table "$master/results_qiime/table.qza" \
  --i-seq "$master/results_qiime/rep-seqs.qza" \
  --output-dir "$master/results_picrust2" \
  --p-threads 0

```
✅ 出力：

・EC_metagenome.qza（酵素活性推定）

・pathway_abundance.qza（代謝経路推定）

・KO_metagenome.qza（遺伝子機能推定）

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
