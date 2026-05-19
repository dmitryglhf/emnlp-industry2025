# EMNLP 2025 Industry Track — глубокий разбор RAG-работ

Отдельный детальный отчёт по статьям с RAG-фокусом. Дополнение к общему `industry2025_map.md`.

---

## 0. Scope: что считаем RAG-работой

«RAG» в индустриальном треке — растяжимое понятие. Я выделил две группы:

- **Primary RAG (7 работ)**: статья помечена как RAG в основной карте; retrieval + LLM-генерация являются *центральным* объектом исследования.
  → industry.23, .33, .72, .93, .135, .173, .174.
- **RAG-adjacent (8 работ)**: retrieval-augmented pipeline присутствует как ключевой компонент или объект бенчмарка, но первичная категория — другая (eval, IE, dialog, fact-checking).
  → industry.3, .34, .58, .127, .147, .167, .187, .102 (граничный случай).

Итого RAG-related работ: **15 из 101 (~15%)** — третья по частоте «инженерная» тема после dialog/CS и agents.

---

## 1. Primary RAG papers — подробно

### industry.23 — Fact-check-guided RAG для медицинских QA

**Что**: RAG с фактчек-ориентированным самообучением для медицины. Двухстадийный pipeline: (1) **FC-RAG** генерация → факт-чек верификация → коррекция, (2) self-training на верифицированных трасах + DPO/SimPO.

**Данные**: 6 публичных медицинских (MedQA, MMLU-M, PubMedQA, BioASQ, MedMCQA + три knowledge-corpora: Wikipedia, PubMed, StatPearls + Medical Textbooks) и **208 проприетарных healthcare-документов**.

**Базовые модели**: Qwen2-72B-Instruct (для RAFE-компонента), LLaMA 3 70B Instruct (для FC-RAG), **LLaMA 3 8B Instruct** для self-training — это и есть «студент».

**Бейзлайны (6)**: CoT, ванильный RAG, Factcheck-GPT, RAFE, SFT, SimPO. Хороший охват направлений — prompting / verification / RL.

**Результат**: factuality с 0.438 → **0.803** (+83%). Industry hook: deployed + compliance (healthcare).

**Урок для submission**: связка «верификатор + self-training» — мощный паттерн в доменах с высокой ценой ошибки. Имеет смысл всегда добавлять второй loop поверх RAG.

---

### industry.33 — RDBMS-as-retriever (DuckDB / SQLite для RAG)

**Что**: тезис «не нужен Elasticsearch / отдельный vector store — достаточно того, что уже стоит у клиента». BM25 nDCG@10 в DuckDB и SQLite матчит Anserini в пределах 0.01.

**Данные**: BEIR (flat variant).

**Бейзлайны**: Anserini, Pyserini.

**Hook**: cost-constraint. Самая «infra»-ориентированная RAG-работа в треке.

**Урок**: чисто инфраструктурный вклад без AI-новизны проходит, если у него есть **измеримая экономия в существующем стеке**. Это редкий жанр, но его принимают.

---

### industry.72 — Адаптивный retrieval + query reformulation для multi-turn CS

**Что**: проблема «когда вообще нужно дергать retrieval» в многоходовом customer support чатботе. Использует **Claude-Sonnet-3 как labeler** для генерации training-меток с объяснениями, потом дистиллирует в маленькие модели (RoBERTa / BART / FLAN-T5).

**Данные**: 10 000 синтетических customer-service диалогов по 20 категориям товаров.

**Бейзлайны**: RoBERTa-base (retrieval triggering), BART-base, FLAN-T5-base (query reformulation).

**Hook**: scale.

**Урок**: для маленьких production-моделей **explanation-guided labels от большой модели > синтетические пары без объяснений**. Это становится стандартным дешёвым приёмом.

---

### industry.93 — Knowledge-graph RAG поверх личной памяти (TOBU app)

**Что**: вместо vector-chunks строится **KG из персональных диалогов и изображений** (через мультимодальную LLM для extraction), retrieval идёт по графу.

**Данные**: 80 реальных memory-retrieval запросов от 20 пользователей. Маленький, но **production-realistic** тест.

**Бейзлайны (3)**: RAGv1 (фиксированные чанки), RAGv2 (per-conversation chunk), RAGv3 (per-summary chunk).

**Результат**: **92.84% F1, 75% user preference** против всех трёх RAG-вариантов.

**Hook**: deployed.

**Урок**: KG-RAG бьёт chunk-based RAG в долгоживущих персональных доменах, где нужны *relations across time*. Маленькая but real user-cohort eval достаточна, если это deployed продукт.

---

### industry.135 — Airbnb AITL flywheel для CS RAG

**Что**: **Agent-In-The-Loop**. Live customer-support агенты постоянно поправляют ответы LLM, эти правки автоматически возвращаются в RAG-pipeline (ORPO fine-tuning). Цикл обновления **с месяцев до недель**.

**Данные**: 5000+ live customer-support interactions от агентов Airbnb.

**Бейзлайны (3)**: ванильный baseline (без FT), Offline ORPO, AITL ORPO.

**Hook**: deployed, **A/B**, latency, scale.

**Результат**: **+14.8% precision** в продакшене.

**Урок**: это один из самых сильных «process-papers» в корпусе — научный вклад тут не в архитектуре, а в **flywheel'е дообучения**. Для submission с реальным деплоем — годный паттерн «как ускорить улучшение модели в проде».

---

### industry.173 — Cross-modal distillation для industrial PDF retrieval

**Что**: учится retriever под промышленные PDF (shipbuilding + electrical) через **knowledge distillation от vision-language teacher** (ColQwen2.5-3B) в плотный dense-retriever (BGE-M3, 560M).

**Данные**: shipbuilding (6,625 train / 2,932 test) + electrical (4,109 train / 3,312 test) PDF.

**Бейзлайны (8)**: BGE-M3, GTE-Qwen2-instruct, Qwen3-Embedding, GME, ColPali, ColQwen2.5, Google text-embedding-005, text-embedding-large-exp-03-07. Очень плотный сет.

**Hook**: latency, cost. **21x lower latency** чем учитель ColQwen2.5-3B при сравнимом качестве.

**Урок**: VLM-teacher → dense-student — новый горячий приём для документ-ориентированных индустрий (где важна и визуальная структура страницы, и латенси).

---

### industry.174 — GRIEVER: graph hybrid retrieval для multi-hop QA без LLM-вызовов

**Что**: multi-hop QA через граф-based гибридный retrieval + triple shortlisting, **без LLM-вызовов в самом ретривере**.

**Данные**: MuSiQue, 2WikiMultiHop, HotpotQA. Чисто публичные.

**Бейзлайны (4)**: BM25, Dense, RRFHybrid, Composed.

**Hook**: latency, cost. **100–500 ms latency** на multi-hop.

**Урок**: альтернатива дорогим agentic RAG'ам типа HippoRAG/GraphRAG. Тренд **«RAG без LLM-петель внутри ретривера»** — отдельный sub-niche.

---

## 2. RAG-adjacent papers

### industry.3 — Multilingual biomedical RAG benchmark
Бенчмарк, а не система. Тестирует 10 фронтир-моделей (GPT-4o/5, Claude 3.7, Gemini 1.5/2.5, DeepSeek V3/R1, Doubao, QwQ, o3-mini) на multilingual biomedical citation curation. **Ключевое наблюдение: reasoning-LLM (R1, o3-mini, QwQ) не улучшают качество цитирований**, CE стабильно ~80%. Это важный негативный сигнал — reasoning ≠ retrieval-faithfulness.

### industry.34 — HERB: enterprise deep-search benchmark
Сравнивает 8 RAG-подходов (Zero-shot, Vector, Hybrid, Raptor, GraphRAG, HippoRAG 2, Proposition-Graph RAG, Agentic ReAct) на синтетическом enterprise-deep-search корпусе. **Лучший agentic RAG берёт только 32.96/100** — ретривал является основным боттлнеком. Используйте как точку входа в литературу RAG-архитектур и как доказательство, что «agentic RAG ≠ silver bullet».

### industry.58 — Clinical nurse dictation extraction
GPT-4.1 + RAG для извлечения структурированных полей из медицинских надиктовок. 5 датасетов (2 публичных + 3 hospital), 8 бейзлайнов (включая Mediphi-3.8B). 93% F1. Hook: compliance + latency. Lightweight Mediphi-3.8B матчит большие закрытые модели на order extraction — ещё один аргумент «маленькая domain-tuned модель ≈ GPT-4».

### industry.127 — Dialpad FAQ extraction из call transcripts
Fine-tuned LLaMA-3.1-8B + BGE-Large embeddings для извлечения и дедупликации FAQ пар из шумных транскриптов звонков. 27,500 аннотированных инстансов на 20 клиентских компаниях. **>90% F1, бьёт GPT-4o-Mini и Gemini** при меньшей стоимости. Hook: deployed + cold-start + scale. Классический «open-source 8B бьёт closed-source mini-моделей в узком домене».

### industry.147 — Alibaba MVP-RAG для e-commerce PAVI
Massive scale: ~4.5M product-attribute pairs (Xianyu) + публичный WDC-PAVI. Multi-view retrieval (attribute-value + product) + fine-tuned Qwen2.5-7B-Instruct. Бейзлайны: BERT-CLS, LLM zero/few-shot, Product-RAG, TACLR. **89.5% F1** на огромном масштабе. Hook: deployed + scale.

### industry.167 — PFO fact-checking с RAV agentic-verifier
5-классовая fact-checking задача на «утечко-чистом» PFO-датасете. Бейзлайны: Zero-shot GPT-3.5, ProgramFC, CofCED, HiSS. **+57.5% macro-F1**. Использует RAV как agentic retrieval-verifier — пограничный случай между RAG и agents.

### industry.187 — GEAR: automotive RAG QA eval
Бенчмарк-папир: автоматически генерирует safety-aware QA из Tesla Model 3 manual (3,202 пары), оценивает GPT-4o / Gemini / DeepSeek как RAG-ассистентов по completeness / risk-awareness / correctness. Никаких prod-чисел — но новый bench в свежей вертикали.

### industry.102 (граничный) — EU legal IE
Использует ретривал + few-shot Claude-3.5-Sonnet (achieves F1 0.949) для EU acquis information-obligation detection. Технически больше IE/classification, но retrieval играет роль в few-shot подаче.

---

## 3. Архитектурные паттерны RAG в треке

### 3.1 Какие ретриверы используют

| Ретривер | В каких работах |
|---|---|
| **BGE-M3 / BGE-Large** | .127, .147, .173 (как student), .130 |
| BM25 (Anserini/Pyserini, или DuckDB/SQLite) | .33, .174 |
| ColPali / ColQwen2.5 | .173 (как teacher) |
| GTE-Qwen / Qwen3-Embedding | .103, .168, .173 (как baseline) |
| RoBERTa / DistilBERT (fine-tuned для retriever triggering) | .72, .108 |
| Custom KG-walker | .93, .174 |
| Hybrid (vector + BM25 + reranking) | .18, .34, .174 |

**Доминирует BGE-семейство.** ColPali/ColQwen — нишевый, но рост через VLM-distillation (.173). KG-based ретриверы — нишевые, но в персональных/multi-hop кейсах побеждают chunk-based.

### 3.2 Какие генераторы используют

| Генератор | Где |
|---|---|
| GPT-4o / GPT-4.1 | .58, .82, .187 (как top-line); рисуют ceiling |
| Claude-3.5/3.7 Sonnet | .72 (labeler), .102, .124 |
| LLaMA-3-8B / 3.1-8B fine-tuned | .23 (self-trained student), .127 |
| Qwen2.5-7B fine-tuned | .147, .180 |
| Mixtral-8x7B | .18 |
| Mistral-7B | .74 |

**Pattern**: closed-source = teacher/labeler/ceiling. Open-source 7-8B = production target. **Никто не fine-tune'ит GPT-4o/Claude.**

### 3.3 Архитектурные оси, по которым отличаются работы

1. **Где живёт ретривер**:
   - В существующей DB (.33 DuckDB/SQLite) — низкий infrastructure overhead.
   - В KG (.93, .174) — для multi-hop / persistent memory.
   - В обычном vector store с reranking — большинство.

2. **Что делает LLM**:
   - Только генерирует — большинство.
   - Дополнительно верифицирует (fact-check loop) — .23, .167.
   - Решает «нужен ли retrieval» (router) — .72.
   - Reformulate query — .72.

3. **Как тренируют ретривер**:
   - Frozen pretrained — .33, .174.
   - Fine-tune на доменных триплетах — .103, .168, .147.
   - Distillation от учителя (VLM) — .173.
   - Self-training через факт-чек — .23.

4. **Как обновляют систему в проде**:
   - Без обновления, batch updates — большинство.
   - **Live agent-in-the-loop flywheel** — .135 (Airbnb). Уникальный паттерн в корпусе.

### 3.4 Какие задачи реально решаются

- **Customer support** (.72, .135): retrieve relevant policies/manuals → generate response. Главный production-кейс.
- **Enterprise/knowledge management** (.33, .34): найди документ по NL-запросу внутри компании.
- **Medical QA** (.23, .58): retrieve guidelines/literature → generate diagnosis/extraction.
- **E-commerce attribute extraction** (.147): retrieve relevant attribute-value seen examples → extract.
- **Personal memory** (.93): retrieve past user conversations/images → answer.
- **Multi-hop reasoning QA** (.174): retrieve evidence chain → answer.
- **Industrial document search** (.173): retrieve technical drawings + text from PDFs.

---

## 4. Эксперименты в RAG-работах

### 4.1 Размер бейзлайн-сетов

В RAG-работах бейзлайнов обычно **больше**, чем в среднем по треку:

| Paper | #Baselines |
|---|---:|
| .173 industrial PDFs | 8 |
| .34 HERB benchmark | 8 |
| .23 medical FC-RAG | 6 |
| .174 GRIEVER | 4 |
| .72 multi-turn CS | 3 |
| .93 personal memory KG | 3 |
| .33 DuckDB | 2 |
| .135 Airbnb AITL | 2 |

Для RAG-papers медиана **3-6 бейзлайнов**, чуть выше общего среднего трека (4-5). Особенно много — в работах, где сравниваются *архитектуры* ретриверов (.173, .34).

### 4.2 Метрики

Доминирующая троица:
- **Precision/Recall/F1** в retrieval — все работы.
- **nDCG@k / MAP / Hit@k** — .33, .103, .174.
- **Latency (ms, p50/p95)** — .135, .173, .174 (явная подача).
- **Faithfulness / factuality** — .23, .167.
- **Human eval** — .72, .93.

Только **.135 Airbnb** даёт **prod-A/B-цифру** (+14.8% precision в проде).

### 4.3 Природа данных

| Тип данных | RAG-работы |
|---|---|
| Только публичные | .33, .34, .174 |
| Только проприетарные | .72, .93, .135, .173 |
| Смешанные | .23, .147, .58 |

Соотношение почти такое же, как в общем треке (~25/40/35). RAG как тематика **не более reproducible**, чем остальные категории.

---

## 5. Industry hooks специфично для RAG

| Hook | RAG-работ с ним |
|---|---|
| Deployed in production | .23, .93, .135, .127, .147 (5/7 primary + 2 adjacent) |
| Latency constraint | .135, .173, .174, .58 |
| Cost constraint | .33, .173, .174 |
| Compliance / regulation | .23, .58 |
| Scale (M+ records/users) | .135, .147, .127 |
| Online A/B | **Только .135 Airbnb** |
| Multilingual | .3 |

**Главный наблюдение**: RAG-работы **сильнее «утоплены» в latency/cost-нарратив**, чем средняя статья трека. Это естественно — стоимость каждого LLM-вызова в RAG-pipeline критична.

Но **A/B с цифрами есть только в одной работе (.135)**. Если у вас есть production RAG с A/B-числами — это редкое преимущество.

---

## 6. Тренды и notable patterns

### 6.1 Что обсуждается часто

1. **Latency-first RAG без LLM-петель внутри ретривера** (.174 GRIEVER, .173 cross-modal KD). Тренд: agentic RAG (HippoRAG, GraphRAG, ReAct-RAG) теряет популярность в industry-треке именно из-за стоимости — стало модно показывать что *можно достичь близкого качества дешевле*.

2. **KG-RAG для долгоживущих доменов** (.93 личная память, .174 multi-hop). Хорошо работает там, где **отношения важнее похожести**.

3. **Verification loop поверх RAG** (.23 FC-RAG self-training, .167 RAV). Это становится стандартом в high-stakes доменах.

4. **Open-source small (≤8B) генератор бьёт closed-source mini на узком домене** (.127 LLaMA-3.1-8B > GPT-4o-mini, .58 Mediphi-3.8B ≈ GPT-4.1 на order extraction).

5. **VLM-teacher distillation в dense-retriever** (.173) — новый паттерн, ожидаю распространения в 2026.

6. **In-existing-database retrieval** (.33 DuckDB) — рост enterprise-friendly решений.

### 6.2 Что redko/исключено

- **Чистый RAG-prompting paper без proprietary data и без A/B** — почти нет.
- **«Лучше chunk size = X»** или ablation chunking parameters — нет таких work'ов в primary 7.
- **Reranking-focused работы** — нет (в общем треке тоже мало).
- **RAG для код-генерации (RAG-as-code-assistant)** — не представлено отдельной работой.
- **RAG безопасность / prompt injection через retrieval** — не представлено в primary, есть только касательно через agent safety (.62).

### 6.3 Отрицательные результаты, которые стоит знать

- **Reasoning-LLM (R1, o3-mini) не помогают RAG-faithfulness** в biomedical (.3). Это сильный негативный результат — не стоит «закатывать» o1-like модель в RAG в надежде на буст.
- **Best agentic RAG достигает только 32.96/100** на enterprise deep-search (.34). Намёк: пределы текущей agentic-RAG методологии достигнуты.

---

## 7. Per-paper таблица RAG-работ

| ID | Подкатегория | Domain | Retriever | Generator | #DS | #BL | Hook | Killer метрика |
|---|---|---|---|---|---:|---:|---|---|
| .23 | Verification-loop RAG | Medical QA | Dense | LLaMA-3-8B (self-train) | 6 | 6 | deploy, compliance | factuality 0.438→0.803 |
| .33 | Infra RAG | Enterprise IT | DuckDB/SQLite BM25 | — | 1 | 2 | cost | matches Anserini nDCG within 0.01 |
| .72 | Adaptive RAG router | E-com CS | RoBERTa+ | Claude (labeler), small students | 1 | 3 | scale | LLM-explanations enable cheap students |
| .93 | KG-RAG | Personal memory | KG-walker | n/a | 1 | 3 | deploy | 92.84% F1, 75% user pref |
| .135 | Flywheel RAG | Airbnb CS | Internal RAG + LoRA | Internal | 1 | 2 | deploy, A/B, latency, scale | +14.8% precision in prod |
| .173 | Cross-modal KD retriever | Industrial PDF | BGE-M3 (student of VLM) | Qwen3-32B | 2 | 8 | latency, cost | 21x lower latency than ColQwen2.5-3B |
| .174 | Graph hybrid retrieval | Multi-hop QA | KG + dense + BM25 | None inside retriever | 3 | 4 | latency, cost | 100-500ms no LLM calls |
| .3 | RAG benchmark | Biomedical multilingual | — (evaluated) | 10 frontier LLMs | 1 | 10 | multilingual | reasoning ≠ faithfulness, CE ~80% |
| .34 | RAG architectures benchmark | Enterprise deep-search | 8 variants | GPT-4o | 1 | 8 | — | best agentic RAG only 32.96/100 |
| .58 | Clinical RAG IE | Clinical | Dense | GPT-4.1, Mediphi-3.8B | 5 | 8 | compliance, latency | Mediphi-3.8B matches GPT-4.1 on order extraction |
| .127 | RAG-derived FAQ | Contact center | BGE-Large | LLaMA-3.1-8B (FT) | 1 | 6 | deploy, cold-start, scale | >90% F1, beats GPT-4o-mini |
| .147 | Multi-view RAG | E-com PAVI | BGE-base + TACLR | Qwen2.5-7B (FT) | 2 | 4 | deploy, scale | 89.5% F1 on 4.5M pairs |
| .167 | Agentic RAG verifier | Fact-checking | RAV | Mistral/LLaMA/Gemma/phi-4 | 5 | 4 | — | +57.5% macro-F1 on clean PFO |
| .187 | RAG eval benchmark | Automotive | — | GPT-4o/Gemini/DeepSeek | 1 | 3 | — | safety-aware QA generation |

---

## 8. Roadmap для собственной RAG-submission

Если ваша работа имеет RAG-фокус, для трека EMNLP industry стоит:

### 8.1 Базовый minimum
1. **Хотя бы 4 retriever-бейзлайна**: BM25 + одна dense (BGE-M3 или GTE) + одна гибридная + хотя бы один SOTA agentic/graph (Raptor, HippoRAG2, ColPali).
2. **Хотя бы 2 LLM-генератора в RAG-pipe**: один closed (GPT-4o или Claude 3.7) как ceiling + ваш fine-tuned open-source (Qwen 7B или Llama 8B).
3. **Retrieval-метрики (nDCG@k, Hit@k) + end-to-end метрики** (F1/correctness/faithfulness).
4. **Latency измерения** (p50/p95) — почти все RAG-работы их показывают, ваша должна тоже.
5. **Ablations**: что даёт каждый компонент (retriever / reranker / reformulator / verifier).

### 8.2 Что усиливает submission

- **Domain-specific proprietary data** (`scale-stated`): 10k+ запросов или 1M+ документов.
- **Если есть deployment** — хотя бы один offline-prod-метрика (acceptance rate, deflection rate).
- **A/B-числа если есть** — выносить в abstract first sentence (как Airbnb .135 / Wayfair .31 в общем треке).
- **Compliance/regulation framing**, если домен это позволяет (healthcare, finance, legal).
- **Latency / cost trade-off** в явном виде (одна таблица «качество vs latency vs $ per query»).

### 8.3 Где есть свободные ниши (на 2025)

По мотивам наблюдений выше — там, где **мало работ или явные пробелы**:

- **RAG для code-bases** (code completion / code search) — не покрыто отдельной industry-работой.
- **RAG x privacy/data minimization** — нет ни одной работы.
- **Cross-lingual RAG** (запрос на одном языке, корпус на другом) — только .3 как бенчмарк, не как метод.
- **RAG eviction / cache invalidation** в долго-живущих продакшен-системах — близка только .135 (Airbnb flywheel).
- **RAG observability / debugging tools** — нет.
- **Verification loop под латенси-бюджет** (.23 делает self-training, но без latency-фокуса) — есть свободное место.
- **RAG для tabular/database query** (NL2SQL ∩ RAG) — близко .122 RoSL, но нет полноценного RAG-фрейминга.

### 8.4 Чего стоит избегать

- Чисто «promptingovuyu» RAG-статью без fine-tuning и без deployment.
- Сравнение только с одним baseline (vector retrieval).
- Бесконечные графики «recall@k vs k» без end-to-end задачи.
- Reasoning-LLM (o1/R1) как silver bullet в RAG — .3 уже показал, что это не работает.
- Очередной *agentic-RAG with reflection*-paper без latency-числа — конкуренция слишком плотная.

---

## 9. Резюме

RAG в EMNLP 2025 industry — **зрелая, но не насыщенная категория**. Основные сдвиги 2025 года:

1. **От agentic-RAG обратно к efficient retrieval** (.174, .173).
2. **KG-RAG возвращается в нишевые задачи** (.93, .174).
3. **Verification loops становятся стандартом** в high-stakes (.23, .167).
4. **Open-source small models побеждают closed-source mini** в узких доменах (.127, .58).
5. **Production-flywheels** (.135 Airbnb AITL) — новый process-oriented научный вклад.

Если вы строите RAG-статью под этот трек: фокусируйтесь на **(a) одной чёткой metric (latency / faithfulness / scale)** + **(b) минимум 4-6 бейзлайнах включая SOTA agentic-RAG** + **(c) явных production-числах**, даже скромных.
