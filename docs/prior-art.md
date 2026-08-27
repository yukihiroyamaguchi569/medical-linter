# 先行研究・類似プロダクト調査

**調査日**: 2026-08-25
**問い**: 「電子カルテの医師記録に対する Linter」というコンセプトは既出か。既出なら、ChartLint に残された空白はどこか。

> **出典の扱いについて**
> 本文書に記載した事実は、すべてウェブ上で実在を確認したものです。論文本文まで直接読んで確認した箇所は「**本文確認済み**」と明記しています。それ以外は検索結果・要約レベルの確認です。確認できなかった主張は「未確認」と書いてあります。

---

## サマリー

**発想としては既出であり、しかも直近で最も活発な領域のひとつ。** ただし ChartLint の立て付けには、まだ埋まっていない空白が3つ残っている。

| 観点 | 状況 |
|---|---|
| 記録の不整合を LLM で検出する **研究** | **既出**。2026年7月の論文がほぼ同一コンセプト |
| 記録の誤りを検出・修正する **ベンチマーク** | **既出**。MEDIQA-CORR 2024（ただし問題設定は別物） |
| 医師にリアルタイムで記録の不備を指摘する **商用製品** | **既出**。CDI / CAPD 業界。ただし動機は診療報酬 |
| 「linter」という **フレーミング** | **ほぼ空白**。GitHub 検索でヒット 0 件 |
| 経過記録（progress note）を対象とすること | **空白**。先行研究は退院サマリーのみ |
| 医師が指摘を **受け入れるか** の評価 | **空白**。検出研究には UI の記載が皆無 |

---

## 1. 学術研究

### 1.1 最も近い先行研究 — Lu et al. (2026)

**Toward Automated Detection of Documentation Inconsistencies in Electronic Health Records**
arXiv:2607.22954 / 2026年7月24日
著者: Jian Lu, Panyu Chen, Miriam Treggiari, Robert Blessing, Danyang Zhuo, Chunhua Weng, William W. Stead, Anru R. Zhang
https://arxiv.org/abs/2607.22954

ChartLint とほぼ同一のコンセプト。**本文確認済み。**

- **手法**: Gemini 2.5 Pro（候補抽出）+ Gemini 2.5 Flash（検証）の二段パイプライン
- **データ**: MIMIC-IV-Note の退院サマリー 3,000件
- **結果**: 3,460件の不整合候補を検出。**入院の 69.7% が何らかの不整合を含む**
- **検出領域**: demographics / allergies / procedures / diagnoses / laboratory / medications / care-planning の7領域
- **位置づけ**: formative study。「診断支援ではなく記録の整合性検出の方法論確立」を目標と明記

検出された不整合の実例（論文より）:

1. アレルギーリストにスピロノラクトンがあるのに、退院処方に同薬 100mg が含まれている
2. 「経骨間中足部切断後の退院指示」セクションがあるが、手術リストにも入院経過にも当該手術がない
3. 同じ退院時処方セクション内で、アミオダロンが「1日1回200mg」と「1日2回200mg」の両方で記載されている

> **ChartLint との関係**: 例1は ChartLint の CL-021（ペニシリンアレルギー ⇔ ABPC/SBT）と同一のルール。**ルール設計の妥当性が実データで裏付けられている**と読める。

**本論文に無いもの（本文確認済み）**:

- 対象は「**We focused exclusively on discharge summaries**」— 退院サマリーのみ。経過記録は含まない
- **UI・臨床ワークフローへの統合・医師への提示方法・受け入れ率・アラート疲れの記載は一切ない**
- 正式な Limitations セクションなし。Future work は「臨床医による注釈コーパスの構築」「アルゴリズム選択の最適化」で、いずれも精度の話に閉じている

### 1.2 MEDIQA-CORR 2024 Shared Task

**Overview of the MEDIQA-CORR 2024 Shared Task on Medical Error Detection and Correction**
6th Clinical NLP Workshop (ACL) / 2024年6月21日・メキシコシティ
主催: Asma Ben Abacha, Wen-wai Yim（Microsoft Health AI）/ Yujuan Fu, Zhaoyi Sun, Fei Xia, Meliha Yetisgen（University of Washington）
https://aclanthology.org/2024.clinicalnlp-1.57/

**本文確認済み（PDF を直接読解）。**

臨床テキストの誤りを検出・修正する共通タスク。112チームが登録し、17チームが提出。

**3つのサブタスク** — ChartLint の「検出 → 該当行を指す → 修正案」と構造が一致:

| | 内容 |
|---|---|
| A | 誤りの有無を判定（1: 誤りあり / 0: なし） |
| B | 誤りを含む文の ID を返す（誤りがなければ -1） |
| C | 該当文の修正版を生成 |

**データセット**: 3,848件。誤りは**人為的に注入**されている。

- **MS コレクション**: MedQA（医師国家試験風 QA）の症例文を変形し手作業で誤りを注入。訓練 2,189 / 検証 574
- **UW コレクション**: ワシントン大学医療センターの実際の匿名化カルテ。検証 160。データ使用契約が必要
- テストセット: MS 597 + UW 328
- 誤りの種類: diagnosis / causal organism / management / treatment / pharmacotherapy

**結果**:

| チーム | 誤り有無 (A) | 誤り文特定 (B) | 修正生成 (C, Aggregate) |
|---|---|---|---|
| WangLab（トロント大） | 0.8649 | 0.8357 | 0.7891 |
| PromptMind（Google） | 0.6216 | 0.6086 | 0.7866 |
| HSE NLP | 0.5222 | 0.5200 | 0.7806 |
| **GPT-4 ベースライン** | 0.6562 | 0.5503 | 0.5754 |

論文の総括: 「誤り有無の判定で 70% を超えたのは 2 チームのみ、誤り文の特定で 65% を超えたのは 1 チームのみ」。**多くのチームが GPT-4 の素の性能を超えていない。** なお 1 位の WangLab には「MS テストデータ使用の可能性」（MedQA を用いた test data leakage の疑い）が論文中に明記されている。

> **ChartLint との関係 — 実は別問題を解いている**
> MEDIQA-CORR の誤りは「Histoplasma capsulatum ではなく Aspergillus fumigatus が正しい」という類の、**医学知識がなければ判定できない誤り**。暗黙のうちに「AI が医師より正しく診断できるか」を測っている。
> ChartLint が見るのは「room air ⇔ 酸素 2L 継続」のような、**突き合わせだけで分かる矛盾**。どちらが医学的に正しいかは判断しない。この線引きが技術的難易度と責任範囲の両方を下げている。

**引用しておく価値のある数字**（本論文の導入部、Bell et al., JAMA Netw Open 2020 より）:

> 自分のカルテを読んだ患者の **5人に1人が誤りを見つけたと報告**し、そのうち **40% が「深刻な誤り」だと感じた**

### 1.3 記録品質の評価尺度 — PDQI-9

**Physician Documentation Quality Instrument (PDQI-9)** は実在する。9項目（completeness, correctness, appropriateness, organization, clarity, conciseness, comprehensiveness, usefulness, information synthesis）で診療記録の品質を評価する尺度。

ただし**万能ではない**。救急外来で適用した研究では評価者間一致がほぼゼロで、「PDQI-9 は EMR（スクライブ）記録の品質評価に有用でない」と結論されている。
https://pubmed.ncbi.nlm.nih.gov/28956888/

引用する場合は、この否定的知見も併せて扱う必要がある。

### 1.4 未確認の主張

着想元の ChatGPT との会話に「600件の救急外来記録に意図的なエラーを入れて複数の LLM に監査させ、accuracy 93–94%、precision 98% 以上」という記述があったが、**本調査では該当研究を確認できていない**。引用する前に裏取りが必要。

---

## 2. 商用製品 — CDI / CAPD

**CDI（Clinical Documentation Improvement / Integrity）という業界が確立している。**

| 製品・企業 | URL | 概要 |
|---|---|---|
| Solventum CDI Engage One（旧 3M） | https://www.solventum.com/en-us/home/health-information-technology/solutions/cdi-engage-one/ | CDI 専門職の physician query を支援 |
| Nuance / Microsoft Dragon Medical Advisor | https://www.businesswire.com/news/home/20150928005731/en/ | **2015年から** CAPD 製品を提供 |
| AGS Health Computer-Assisted CDI | https://www.agshealth.com/ai-platform/computer-assisted-cdi/ | AI による CDI |
| Hiteks | https://hiteks.com/ | **Epic 内で、医師が記録を書いている最中に**クエリを提示 |
| Iodine Software / SmarterDx / EvidenceCare / Ambience | — | 同領域の新興〜中堅 |

**CAPD（Computer-Assisted Physician Documentation）** は、医師が EHR 上でカルテを書いている最中にリアルタイムで不備を指摘する仕組み。構造としては ChartLint そのもの。

### 決定的な違い — 動機

**既存 CDI / CAPD の主目的は診療報酬（コーディング・DRG 最適化）である。**

検出するのは「心不全と書いてあるが、急性か慢性か、収縮性か拡張性かを明記してほしい」といった *physician query* であり、臨床的な記録品質は副産物。ベンダーが掲げる ROI も査定額の改善である。

**「治療方針を提案せず、診療報酬も見ず、記録の内部矛盾だけを指摘する」立て付けの製品は、本調査の範囲では見つからなかった。**

---

## 3. OSS / 「linter」というフレーミング

GitHub API で直接検索した結果（2026-08-25 時点）:

| 検索語 | ヒット数 |
|---|---|
| `clinical note linter` | **0** |
| `clinical documentation linter` | **0** |
| `medical record linter` | **0** |
| `EHR linter` | 1（`ncreighton/...optometry-eye-care-linting-a`、★1。眼科の ICD-10 検証で、目的は保険請求の否認削減） |

ウェブ全体でも「ESLint のメタファーを診療記録に持ち込む」議論は見つからなかった。

> 概念は "documentation inconsistency detection" や "CDI" として確立しているのに、**開発者の語彙で語られていない**。ChartLint という名前とフレーミングが伝わりやすい理由はここにある。

---

## 4. 日本

日本では**診療記録監査が人手で行われている**段階。書籍として『【電子カルテ版】診療記録監査の手引き』（医学通信社）が販売され、22の診療記録について様式・記載内容・管理方法を点検する「標準・監査点検表」が用いられている。
https://www.igakutushin.co.jp/products/detail/1218

自動化の余地がそのまま残っている。日本語の経過記録を対象とした不整合検出研究は、本調査の範囲では見つからなかった。

---

## 5. UI と受容 — 別の文献群に大量にある

検出研究に UI の記載がない一方で、**CDS（臨床意思決定支援）のアラート研究には確立した知見と測定手法がある**。数字はかなり厳しい:

| 知見 | 出典 |
|---|---|
| アラートのオーバーライド率は **49〜96%** の幅で報告 | https://pmc.ncbi.nlm.nih.gov/articles/PMC9579928/ |
| Brigham and Women's では薬剤アラートの **73.3% が無視**され、うち 40% は不適切な無視 | 同上 |
| 1回の診療でアラートが1つ増えるごとに、**受け入れ確率が 30% 低下** | https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5387195/ |
| 患者固有の文脈に合わせると、受け入れが **20% 未満 → 60% 超**に上昇 | https://smw.ch/index.php/smw/article/view/3357 |
| アラート疲れの測定手法に関する系統的レビュー | https://pubmed.ncbi.nlm.nih.gov/42148822/ |

**「検出できること」と「医師が受け入れること」の間には巨大な溝があり、後者には測定手法まで存在する。にもかかわらず、記録の不整合検出という新しい対象に対して、この2つの文献をつないだ研究が見当たらない。**

---

## 6. ChartLint に残された空白

問題も技術も既出であり、「誰も思いついていない新領域」ではない。空いているのは次の3点。

### 6.1 動機の純度

保険請求のためではなく記録品質のため、という製品が見当たらない。「治療方針は提案しない」という線引きは、責任範囲を限定すると同時に**技術的難易度も下げている**（知識判断ではなく突き合わせに問題を還元している）。

### 6.2 対象文書とタイミング — 退院サマリー vs 経過記録

先行研究（Lu et al.）は退院サマリーのみを対象とする。経過記録を対象にすることには本質的な差がある:

- **介入可能性**: 退院サマリーで矛盾を見つけても患者は既に退院している。見つかるのは「記録の不備」であって、まだ変えられる診療ではない。経過記録の「低K血症に Plan がない」は、**その日の診療の欠落**である
- **因果の向き**: 退院サマリーに紛れ込む古い情報は、経過記録のコピペで日々増殖したものが流れ込んだ結果。上流で止める方が合理的
- **メタファーとの整合**: ESLint はコードを書いている最中に走るから意味がある。退院サマリーの解析は、静的解析というより事後監査に近い

### 6.3 医師の受容の測定

Lu et al. が「LLM は記録の矛盾を見つけられる」を示した直後の今、次に価値があるのは「**で、それを医師はどう受け取るのか**」。承認率、無視率、ルール別の信頼度 — CDS アラート研究の測定手法をそのまま持ち込める。ChartLint のデモは、承認・無視のログが取れる構造になっており、この実験装置の形をしている。

### 研究デザインの方向

> Lu らが退院サマリーで示した検出能力を、**経過記録にリアルタイム適用**し、**医師の承認率で評価する**

前者だけなら「対象文書を変えただけ」だが、後者を含めることで先行研究に正面から接続しつつ、誰も埋めていない穴を埋める形になる。

---

## 出典一覧

**学術**
- [Toward Automated Detection of Documentation Inconsistencies in EHRs (arXiv:2607.22954)](https://arxiv.org/abs/2607.22954) — 本文確認済み
- [Overview of the MEDIQA-CORR 2024 Shared Task (ACL Anthology)](https://aclanthology.org/2024.clinicalnlp-1.57/) — PDF 本文確認済み
- [MEDIQA-CORR 2024 公式サイト](https://sites.google.com/view/mediqa2024/mediqa-corr)
- [MEDIQA-CORR 評価スクリプト (GitHub)](https://github.com/abachaa/MEDIQA-CORR-2024/tree/main/evaluation)
- [PDQI-9 は救急外来では有用でない (PubMed)](https://pubmed.ncbi.nlm.nih.gov/28956888/)

**商用**
- [Solventum CDI Engage One](https://www.solventum.com/en-us/home/health-information-technology/solutions/cdi-engage-one/)
- [Nuance Dragon Medical Advisor (CAPD, 2015)](https://www.businesswire.com/news/home/20150928005731/en/Nuance-Introduces-Dragon-Medical-Advisor-Computer-Assisted-Physician-Documentation-CAPD-for-ICD-10)
- [AGS Health Computer-Assisted CDI](https://www.agshealth.com/ai-platform/computer-assisted-cdi/)
- [Hiteks](https://hiteks.com/)

**UI・受容**
- [Appropriateness of Alerts and Physicians' Responses](https://pmc.ncbi.nlm.nih.gov/articles/PMC9579928/)
- [Effects of workload and repeated alerts on alert fatigue](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5387195/)
- [Tackling alert fatigue with a semi-automated CDS](https://smw.ch/index.php/smw/article/view/3357)
- [Alert fatigue measurement in CDS: a systematic review](https://pubmed.ncbi.nlm.nih.gov/42148822/)

**日本**
- [【電子カルテ版】診療記録監査の手引き（医学通信社）](https://www.igakutushin.co.jp/products/detail/1218)
