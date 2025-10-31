---
layout: page
title: "16S rRNA 解析 共通プロトコル"
---

<div class="section-title">QIIME2 / PICRUSt2（ver. 2024.05）</div>

## <span class="section-heading">STEP 0｜解析環境の起動</span>

```bash
conda activate q2-picrust2-amplicon-2024.5
export master=ishikawa   # ← 班名を入力
cd ~/qiime/$master
