# EMNLP 2025 Industry Track — карта принятых статей

Подробный аналитический отчёт по 101 принятой статье индустриального трека EMNLP 2025, на основе summary-версий из `industry2025/md/`. Цель документа — дать автору будущей submission системное представление о том, **что принимают в этот трек, в какой форме, с каким объёмом экспериментов и под каким индустриальным «крючком»**.

---

## 0. Executive summary

- Изучено **101 принятая статья** (с summary-файлами). Всего в директории `industry2025/md/` лежит 117 поддиректорий — 16 без summary не учитывались.
- **Доминирующая тройка категорий**: evaluation/benchmark (≈17 работ), dialog/customer-support (12), agents/tool-use (9) — суммарно ~38% трека.
- **«Типичная принятая статья»** — это end-to-end пайплайн с 1–2 датасетами (минимум один проприетарный), 3–5 бейзлайнов, обязательными ablations, опционально — human eval, и offline-числами на проприетарном тесте + словесным упоминанием продакшен-деплоя. Чисто алгоритмических single-method papers практически нет — это формат main track.
- **94% статей содержат числовое сравнение хотя бы с одним бейзлайном**. Чисто case-study («у нас есть система, вот как работает») — 6%, и каждая такая работа имеет либо громкий deployment-факт (Ryt Bank, TikTok, национальная telehealth-платформа), либо новую метрику.
- **Настоящие production-метрики** (числовые CTR/CVR/QPS/$/A/B-uplift) есть только в **~11 статьях**. Это редкий, но мощный сигнал для рецензента — если он есть, его обязательно выносят в abstract.
- **75% статей используют проприетарные данные** (полностью или в смеси с публичными). Чистая reproducibility — нехарактерный паттерн.
- **Open-source LLM доминируют как fine-tune target** (Qwen 7-14B, Llama 8B). Closed-source (GPT-4o, Claude, Gemini) почти всегда выступают в роли strong baseline, а не объекта обучения.

---

## 1. Корпус и методология

**Источник**: `/home/glhf/dev/emnlp-prep/industry2025/md/2025.emnlp-industry.<N>/2025.emnlp-industry.<N>.summary.md`.
**Объём**: 101 summary-файл единого шаблона (TL;DR / Problem & Motivation / Approach / Experimental Setup / Results / Industry Impact / Limitations).
**Метод**: 5 параллельных Sonnet-агентов прочитали по ~20 summary, каждый вернул структурированную карточку на работу с фиксированными полями (категория, домен, framing, датасеты, бейзлайны, base-модели, industry hooks, наличие ablations/human-eval/A-B, comparative vs case-only, takeaway). Затем агрегирование вручную.

**Ограничения метода:**
- Summary написаны автоматически и местами содержат «Not reported» — это часто означает не «отсутствует в статье», а «не извлеклось»; реальные доли deploy/A-B/human-eval могут быть на 5-10% выше указанных здесь.
- Кросс-агентная гетерогенность тегов: один и тот же hook мог быть размечен слегка по-разному; для агрегатов это сглажено, но точные числа имеют погрешность ±2-3 работы.
- Категоризация многомерна (e-com + dialog + RAG), и одна статья отнесена к **одной основной** категории; вторичные принадлежности отражены в кросс-разрезах.

---

## 2. Тематические категории

### 2.1 Сводная таблица

| Категория | N | Доля |
|---|---:|---:|
| Evaluation / benchmark | 17 | 17% |
| Dialog / customer-support | 12 | 12% |
| Agents / tool-use | 9 | 9% |
| Efficiency / distillation / quantization | 8 | 8% |
| Domain-LLM fine-tuning | 8 | 8% |
| RAG | 7 | 7% |
| Structured extraction / IE | 6 | 6% |
| E-commerce (catalog/listing/ads) | 5 | 5% |
| Moderation / safety | 5 | 5% |
| Data-generation / synthetic | 5 | 5% |
| Multimodal (vision/audio gen) | 4 | 4% |
| Speech / voice | 4 | 4% |
| Search / recsys (ranking) | 3 | 3% |
| Summarization | 2 | 2% |
| Translation / i18n | 2 | 2% |
| Code (NL→SQL ускорение) | 1 | 1% |
| Other (KG-to-text, fact-checking, data-quality) | 3 | 3% |

### 2.2 Расширенный разбор по категориям

**Evaluation / benchmark (17)** — самая крупная категория. Делится на три подгруппы:
- *Новые бенчмарки под вертикаль*: industry.3 (multilingual biomedical RAG), .19 (multimodal e-com agent), .34 (HERB enterprise deep-search), .69 (anatomy True/False), .83 (TelAgentBench Korean telecom), .130 (oil&gas IR), .132 (Text2Cypher 11 KGs), .152 (cyberbullying BullyBench), .167 (PFO clean fact-checking), .187 (GEAR automotive RAG QA), .188 (FQ-Eval follow-up questions).
- *Новые метрики / диагностики*: .10 (RCI patch-vs-full), .14 (PCRI multimodal global vs local), .39 (LLM-free SFAL для SAE), .47 (SBM auxiliary features для reward models), .60 (Combo-Eval cost-efficient LLM judge), .82 (INSTAJUDGE few-shot LLM-as-judge).
- *Большие comparative-studies*: .35 (17 RLHF алгоритмов на 30k TPU-часах).

**Dialog / customer-support (12)** — second largest. Финансовые/банковские чат-боты (.45, .51), e-com support (.21, .74, .108, .121, .127, .150), healthcare/telemed (.101, .125, .126, .133), telecom (.16). Заметная подкатегория — **clinical/telehealth** (5 работ), что отражает рост LLM в healthcare 2025.

**Agents / tool-use (9)** — почти все мульти-агентные системы с reflection/планированием: .15 (CitySim социальная симуляция), .41 (policy-compliance guards), .55 (finance portfolio), .100 (BPA/BPMN), .113 (CLIMB labor market), .115 (TikTok ARIA compliance), .116 (WeChat RLTR), .123 (declarative SQL+API), .144 (AMAS dynamic graph). Часто комбинируется с RL/DPO.

**Efficiency (8)** — три подвида:
- *On-device / mobile*: .52 (iPhone first-order FT), .118 (mobile encoders LoRA), .164 (Recover-LoRA для квантованных SLM).
- *Distillation / pruning*: .71 (HOLA edge), .89 (online mutual KD GPT-2), .119 (LinkedIn 100B→Qwen-1.5B).
- *Prompt-level efficiency*: .5 (LoRA rank allocation), .20 (ProCut prompt pruning).

**Domain-LLM fine-tuning (8)** — таргет-домены: SNS/контент-модерация (.180 RedOne), telecom (.168 T-VEC), finance multilingual (.111), payment networks (.61), healthcare billing (.84), education K-12 (.68), personalization/Copilot (.56), industrial search German (.103).

**RAG (7)** — все mid-to-late pipeline-работы: medical (.23), enterprise RDBMS-в-роли-индекса (.33), multi-turn customer service (.72), personal memory KG (.93), Airbnb agent feedback (.135), industrial PDFs multimodal KD (.173), multi-hop sub-second (.174).

**Structured extraction / IE (6)** — clinical (.58), schema mapping (.75), JSON-patch (.88), EU legal (.102), NL2SQL (.122), e-com PAVI (.147).

**E-commerce specifically (5)**: Amazon catalog (.18, .63), India logistics (.70), ads compliance (.79), product defect classification (.96). Если добавить пересекающиеся ranking/IE/dialog для e-com — суммарно вертикаль e-com покрывает ≥12 работ (~12% трека).

**Moderation / safety (5)** — ad moderation (.1 RAVEN++), agent red-teaming (.62), clinical intent safety (.124), depression detection (.151), cyberbullying (.152).

**Data-generation / synthetic (5)** — marketing dialogs (.26), math reasoning data (.43), education problems (.97), document KIE (.105), medical EMR (.181).

**Speech (4)**, **Multimodal (4)**, **Search/recsys (3)**, **Summarization (2)**, **Translation (2)**, **Code (1)** — нишевые, но представлены.

### 2.3 Что не представлено

В принятых работах **почти нет**:
- pure pre-training scaling-laws статей (это main track),
- чисто теоретического анализа LLM (interpretability представлен только .39 как метрика),
- low-resource линвистики ради самой себя (только если есть индустриальный продукт типа Romanian telemed .133),
- однодоменных reasoning/math работ без data-pipeline (math появляется только как target для data-gen .43 или для benchmark .35).

---

## 3. Бенчмарки и датасеты

### 3.1 Самые частые публичные бенчмарки

| Бенчмарк | N статей | Контекст использования |
|---|---:|---|
| MMLU / MMLU-Pro / CMMLU | 7 | Generic knowledge baseline |
| GSM8K, MATH, MATH500 | 6 | Reasoning baseline |
| HumanEval, MBPP, BBH | 4 | Code / reasoning baseline |
| BIRD, Spider | 5 | Text-to-SQL (.60, .90, .122, .123, .132) |
| HotpotQA, 2WikiMultiHop, MuSiQue | 3 | Multi-hop QA (.174 и др.) |
| BEIR, MS MARCO | 3 | Retrieval (.33, .103) |
| Multimodal VL cluster: AI2D, BLINK, MMStar, ScienceQA, AMBER, MME, POPE, HallusionBench, ChartQA, GQA, TextVQA, VizWiz, RealWorldQA | 3-4 каждый | Используются пакетом в multimodal-eval статьях .10, .14 |
| WMT-22/23/24 | 3 | MT baseline |
| MedQA, PubMedQA, MedMCQA, BioASQ | 2-3 | Medical QA |
| AIME 2024/2025 | 2 | Math RL |
| AgentHarm, tau-bench | 2 | Agent safety / tool-use |

### 3.2 Распределение по природе данных

| Тип | N | Доля |
|---|---:|---:|
| Только публичные | ~25 | 25% |
| Только проприетарные | ~40 | 40% |
| Смешанные | ~36 | 35% |

**Вывод**: только-публичные используются почти исключительно в бенчмарк-статьях и в efficiency/algorithm-focused работах. Все deployed-системы тестируются на проприетарных данных, что снижает межстатейную сопоставимость, но именно это и валидно для industry-трека.

### 3.3 Размер проприетарных датасетов

Типовые числа из summary:
- **Миллионы записей**: .61 (1B txns), .18 (30M+20M+20M catalog), .68 (3.89M interactions), .180 (100B токенов), .21 (55k+9k dialogs), .49 (~1M turns), .180 (SNS), .147 (~4.5M PAVI).
- **Десятки-сотни тысяч**: .13 (~495k+257k), .26, .56 (Copilot 8000), .70 (1M addresses).
- **Сотни-тысячи (small but annotated)**: .55 (100 synthetic), .69 (1,077 anatomy QA), .58 (149 transcripts), .84 (52+65 encounters), .124 (1,600 queries), .130 (5,066 pairs), .133 (100/871), .181 (617 consultations).

**Pattern**: для clinical/legal/regulatory data выпускают мало — но с очень тщательной экспертной разметкой; для e-com/recsys/моделирования платформ — миллионы примеров, часто слабо размеченных.

---

## 4. Постановка задачи (task framing)

| Framing | N | Доля |
|---|---:|---:|
| End-to-end pipeline | 33 | 33% |
| Evaluation framework | 17 | 17% |
| Agentic system | 12 | 12% |
| Classification | 11 | 11% |
| Generation | 11 | 11% |
| Retrieval | 8 | 8% |
| Ranking | 3 | 3% |
| Other (schema-map, KG-build, data-quality, model-recovery) | ~6 | 6% |

**Pipeline-доминирование** — ключевой паттерн. Треть работ — это система из 3-5 компонентов (например: ASR → классификатор интента → retrieval → LLM-rewriter → постпроцессинг). Это означает, что **single-method paper** (один алгоритм, одна модель, один датасет) — почти всегда отклоняется или направляется в main track. Для industry submission стоит планировать **систему целиком**, даже если технический вклад сосредоточен в одном модуле.

**Agentic system (12)** — почти все 2025 работы здесь содержат либо multi-agent reflection, либо tool-use + RL. Простой ReAct без новизны проходит редко.

---

## 5. Привязка к индустрии (Industry hooks)

### 5.1 Распределение типов hook'ов (статья может иметь несколько)

| Hook | N | Доля |
|---|---:|---:|
| Deployed in production | 38 | 38% |
| Scale stated (M/B users/queries/records) | 30 | 30% |
| Latency constraint | 25 | 25% |
| Cost constraint | 25 | 25% |
| Compliance / regulation | 15 | 15% |
| Multilingual production | 12 | 12% |
| Online A/B упомянут | 12 | 12% |
| Cold-start | 3 | 3% |
| None stated | ~15 | 15% |

### 5.2 Иерархия силы hook'ов

От **слабого к сильному**, как это воспринимается рецензентами industry-трека:

1. **Vertical mention only** («for healthcare» / «for e-commerce»). Без proprietary data — самый слабый hook. Чтобы пройти, нужна сильная техническая новизна и публичные бенчмарки.
2. **Proprietary dataset + offline evaluation**. Базовый industry baseline.
3. **Compliance / latency / cost — числа**. «Sub-100ms», «$15k saved», «regulator-approved» — конкретика весомее обещаний.
4. **Deployed-словами**. «Currently in production at X» — добавляет доверия даже без A/B.
5. **Scale-stated**. «150M MAU», «1B transactions», «50K queries/day» — рецензент видит масштаб.
6. **Online A/B с числами** (+x% CTR, +y% CVR). Самый сильный сигнал.

### 5.3 Жёсткий разрез: статьи с реальными prod-числами

Только **11 статей** (~11%) показывают конкретные production KPI с числами:

| Paper | Метрика deployment'а |
|---|---|
| .13 (Alibaba recruitment) | +17.29% CTCVR в A/B |
| .26 (marketing dialogs) | +22.4% conversion online |
| .31 (Wayfair reviews) | +0.5% CVR, +0.3% ATC на 493k продуктах за 3 недели |
| .51 (WeBank chatbot) | 84% acceptance, +18% user satisfaction |
| .97 (CBIT math problems) | 6,732 learners served, -17.8% error rate vs experts |
| .126 (CLARITY telemed) | 55K+ dialogs, 3x faster triage, 96% specialist routing recall |
| .133 (Dr.Copilot Romania) | +83.89% positive patient reviews с 41 доктором |
| .135 (Airbnb AITL RAG) | +14.8% precision, цикл обновления месяц→неделя |
| .140 (Airbnb summarizer) | -3-9% average handle time |
| .150 (TeleBot telemarketing) | На уровне top-25% human sales |
| .180 (RedOne SNS) | -11% harmful content, +15% search CTR |

**Вывод**: prod-числа — золото. Если они у вас есть, всё остальное может быть проще; если их нет, заменяйте их **(a)** масштабом данных, **(b)** compliance/regulatory гарантиями, **(c)** хорошим offline-сравнением на проприетарном тесте + словесным deployed-маркером.

### 5.4 Compliance/regulation как отдельный «крючок»

15% работ привязывают релевантность к compliance. Сильные домены:
- **Finance/banking**: .45 (Ryt Bank, regulator-approved), .51 (WeBank), .55 (portfolio), .61 (payment network), .115 (TikTok Pay AML).
- **Healthcare**: .23, .58, .69, .84 (CPT E/M coding), .124, .126, .181.
- **Legal**: .102 (EU acquis obligation detection).
- **Cybersecurity**: .75 (Microsoft Defender schema).

Это валидный путь для submission даже без громких prod-чисел, если у вас есть формальные гарантии правил.

---

## 6. Эксперименты: объём и формат

### 6.1 Количество датасетов на статью

| #Datasets | Доля |
|---|---:|
| 1 | 38% |
| 2 | 25% |
| 3 | 15% |
| 4+ | 22% |

Большое число датасетов (10-21) почти всегда означает либо бенчмарк-папир (.14, .132), либо efficiency/CPT-папир, прогоняющий генерик-suite (.43, .118, .180).

### 6.2 Количество бейзлайнов

| #Baselines | Доля |
|---|---:|
| 0 | 6% |
| 1-2 | 22% |
| 3-5 | 38% |
| 6-9 | 22% |
| 10+ | 12% |

**Медиана: 4-5 бейзлайнов.** Это де-факто целевое число. Меньше 3 — почти всегда требует компенсации (case-study + deployment, либо очень тщательные ablations).

### 6.3 Форматы дополнительных экспериментов

- **Ablations**: 67% (де-факто обязательны)
- **Human evaluation**: 28% (плюс для качественных задач — генерация, dialog, eval-метрики)
- **Online A/B упомянут**: 12%
- **Case-study only**: 6%

### 6.4 Critical cut — Comparative vs Case-only

| Тип | N | Доля |
|---|---:|---:|
| **Comparative** (числовое сравнение ≥1 baseline) | ~95 | 94% |
| **Case-only** | 6 | 6% |

**Case-only исключения**: .10 (новая метрика без модельного baseline), .45 (Ryt Bank — regulator-approved deployment без публичных чисел), .123 (нет числовых результатов), .125 (нет финальных чисел), .126 (deployment-paper, числа есть, но не сравнение с моделями), .133 (deployment-paper Romania).

**Вывод для submission**: если вы пишете case-study — у вас должен быть один из этих громких якорей. Без них приоритет — сделать comparative.

---

## 7. Модели-основания

### 7.1 Топ моделей по встречаемости

| Модель/семейство | N | Типичная роль |
|---|---:|---|
| GPT-4o (+ 4.1, mini, nano, o3-mini, o4-mini) | ~30 | Strong baseline, judge, data generator |
| Qwen2.5 / Qwen3 (0.5B–235B, + VL) | ~25 | Fine-tune target, embeddings (BGE родственник), agents |
| Llama 3 / 3.1 / 3.2 / 3.3 (1B–70B) | ~22 | Fine-tune target, baseline |
| Claude (3.5/3.7 Sonnet, Haiku, Opus 4) | ~15 | Strong baseline, judge, data labeler |
| Gemini (1.5/2.0/2.5 Flash/Pro) | ~12 | Baseline, judge |
| DeepSeek (V3, R1, R1-Distill) | ~12 | Strong reasoning baseline, distillation source |
| Mistral / Mixtral (7B, 8x7B, 8x22B, Large) | ~12 | Fine-tune target |
| BERT-family (mBERT, RoBERTa, DeBERTa, etc.) | ~15 | Router/classifier/embedding в pipeline |
| Embedding models (BGE-M3, E5, GTE, mE5, MPNet) | ~12 | Retrieval |
| Phi-3.5 / Phi-4 | 6 | Lightweight FT target |
| Gemma 2/3 (2B–27B + MedGemma) | 6 | Lightweight FT target |
| Whisper | 3 | ASR |
| Proprietary in-house FMs | 6 | ILMU (Ryt Bank), LinkedIn 100B MoE, GigaChat, Sarashina, Apple Siri TN, internal Amazon Nova |

### 7.2 Pattern: «закрытые — как бейзлайны, открытые — как объекты»

В корпусе **нет ни одной статьи, где fine-tune'или бы GPT-4o / Claude / Gemini** (фокус на promptingе или их API). При этом:
- Qwen 7-14B — самый частый fine-tune target;
- Llama 8B — близкий второй;
- Phi/Gemma 2-4B — для on-device;
- DeepSeek-R1-Distill — частый старт для RL-работ.

Если вы делаете finetune, типовое portfolio — Qwen2.5-7B + Llama-3.1-8B как два сравнительных тюна.

---

## 8. Cross-cutting наблюдения

### 8.1 Что общего у «модальной» принятой статьи

1. **Pipeline > single algorithm**: 3-5 компонентов (ASR/retriever/LLM/postproc/judge).
2. **Хотя бы один proprietary dataset** для главного evaluation.
3. **3-5 бейзлайнов** с числовым сравнением.
4. **Ablations** обязательны.
5. **Industry hook**: либо deployment-marker, либо latency/cost-числа, либо compliance, либо scale.
6. **Open-source LLM как fine-tune target**, GPT-4o/Claude как baseline.

### 8.2 Тренды 2025 года (специфичные для трека)

- **LLM-as-judge как самостоятельный вклад**: .60, .82, .188. Калибровка judge'ей под домен — растущая ниша.
- **Test-time training/RL без размеченных данных**: .75, .115 (ARIA). Это новый legitimate angle.
- **Synthetic data pipelines становятся отдельным научным жанром**: .26, .43, .97, .105, .181. Раньше синтетика была средством — теперь это объект исследования.
- **Cross-modal distillation в retrieval**: .173 (VLM teacher → dense retriever). Многообещающий low-cost паттерн.
- **DSPy-based prompt optimization как метод выбора**: .125, .133.
- **Cost transparency**: явные доллары в abstract и conclusions (.16 — $70 eval, .20 — $15k savings, .121 — $34k→$18k daily).

### 8.3 Что бы я **не делал** в submission, исходя из карты

- Не делать чисто «promptingovuyu» статью на public benchmark без deployment-сигнала — она проигрывает либо main track'у в новизне, либо industry-track'у в реализме.
- Не строить case-study без громкого якоря.
- Не делать full fine-tune закрытой модели — это не воспроизводимо и не репрезентативно.
- Не отказываться от ablations, даже если результат и так очевиден — рецензенты их ищут.
- Не оставлять «Industry Impact: Not reported» — даже короткая секция с qualitative-описанием pipeline'а в проде усиливает.

### 8.4 Что бы я **сделал**, чтобы попасть в распределение

- Подобрать вертикаль с слабым покрытием в треке: cybersec (.75 — единственная), legal beyond EU acquis, agriculture, manufacturing (только .103 industrial search).
- Выпустить новый бенчмарк/датасет — это даёт acceptance c минимальной техникой (см. .130 — drilling reports, .132 — Cypher, .187 — automotive RAG, .188 — follow-up Qs).
- Если есть deployed-система — сфокусироваться на A/B-числах и масштабе, **даже если** технический вклад скромный (.31 Wayfair, .140 Airbnb).
- Multilingual / non-English данные — недорогой способ усилить industry-релевантность.
- Pipeline ≥3 компонентов с одним нестандартным элементом (RAG + token-tree, или RAG + multi-agent reflection).

---

## 9. Приложение: per-paper мини-карточки

Сокращённый формат: `industry.N | Category | Domain | Framing | #DS | #BL | Hook (краткий) | Comp/Case`.

| ID | Category | Domain | Framing | #DS | #BL | Hook | Type |
|---|---|---|---|---:|---:|---|---|
| .1 | moderation/safety | digital ads | pipeline | 2 | 3 | deploy, A/B, compliance | comp |
| .3 | eval/benchmark | biomedical | eval-fw | 1 | 10 | multilingual | comp |
| .5 | efficiency | healthcare CDS | pipeline | 1 | 11 | none | comp |
| .6 | speech | TTS preproc | pipeline | 2 | 1 | deploy, multilingual, latency | comp |
| .7 | speech | audio AI | agentic | 4 | 2 | latency, cost | comp |
| .9 | translation | enterprise multilingual | pipeline | 5 | 6 | multilingual | comp |
| .10 | eval/benchmark | multimodal | eval-fw | 13 | 0 | none | case |
| .13 | search/recsys | HR/recruitment | ranking | 1 | 5 | deploy, A/B, scale, cost | comp |
| .14 | eval/benchmark | multimodal | eval-fw | 15 | 19 | none | comp |
| .15 | agents | urban simulation | agentic | 3 | 5 | scale | comp |
| .16 | dialog/CS | telecom retail | eval-fw | 1 | 13 | deploy, cost | comp |
| .18 | e-commerce | Amazon catalog | retrieval | 1 | 1 | deploy, scale, multilingual | comp |
| .19 | eval/benchmark | e-com agent | agentic | 1 | 7 | none | comp |
| .20 | efficiency | LLM serving | pipeline | 7 | 6 | deploy, cost, latency, scale | comp |
| .21 | dialog/CS | food delivery (Meituan) | classification | 1 | 9 | deploy, scale | comp |
| .23 | RAG | medical QA | pipeline | 6 | 6 | deploy, compliance | comp |
| .26 | data-gen | marketing | pipeline | 4 | 4 | deploy, A/B | comp |
| .31 | summarization | Wayfair | generation | 1 | 1 | deploy, A/B, scale | comp |
| .33 | RAG | enterprise IT | retrieval | 1 | 2 | cost | comp |
| .34 | eval/benchmark | enterprise KM | eval-fw | 1 | 8 | none | comp |
| .35 | eval/benchmark | RLHF | eval-fw | 2 | 17 | cost, scale | comp |
| .37 | multimodal | T2I | pipeline | 1 | 5 | none | comp |
| .39 | eval/benchmark | LLM interp | eval-fw | 2 | 1 | cost, scale, compliance | comp |
| .41 | agents | enterprise compliance | agentic | 1 | 5 | compliance, deploy | comp |
| .43 | data-gen | math data pipelines | eval-fw | 21 | 1 | cost, scale | comp |
| .45 | dialog/CS | banking | agentic | 1 | 0 | deploy, compliance, latency, multilingual | case |
| .47 | eval/benchmark | reward models | eval-fw | 3 | 7 | none | comp |
| .49 | speech | call centers | pipeline | 4 | 4 | deploy, scale, latency | comp |
| .51 | dialog/CS | banking (WeBank) | generation | 1 | 5 | deploy, compliance, multilingual | comp |
| .52 | efficiency | on-device | pipeline | 3 | 2 | latency, cost | comp |
| .55 | agents | finance | agentic | 1 | 3 | none | comp |
| .56 | domain-FT | LLM personalization | pipeline | 2 | 3 | deploy, scale | comp |
| .58 | structured-extr | clinical | pipeline | 5 | 8 | compliance, latency | comp |
| .60 | eval/benchmark | text-to-SQL judge | eval-fw | 1 | 8 | cost | comp |
| .61 | domain-FT | payment networks | pipeline | 1 | 1 | deploy, latency, scale, cold-start | comp |
| .62 | moderation/safety | agentic AI | eval-fw | 2 | 1 | compliance, scale | comp |
| .63 | e-commerce | Amazon catalog | pipeline | 1 | 3 | scale, cost, multilingual | comp |
| .66 | multimodal | video grounding | pipeline | 9 | 5 | none | comp |
| .68 | domain-FT | K-12 e-learning | pipeline | 1 | 8 | scale, cost | comp |
| .69 | eval/benchmark | clinical | eval-fw | 1 | 3 | compliance | comp |
| .70 | e-commerce | India logistics | pipeline | 3 | 7 | deploy, scale, cost, latency | comp |
| .71 | efficiency | edge inference | pipeline | 2 | 9 | latency, cost | comp |
| .72 | RAG | CS chatbot | pipeline | 1 | 3 | scale | comp |
| .74 | dialog/CS | e-com CS | classification | 1 | 5 | scale, cost | comp |
| .75 | structured-extr | cybersecurity | other | 1 | 2 | compliance | comp |
| .79 | e-commerce | ad compliance | generation | 2 | 7 | latency, cost, deploy | comp |
| .80 | translation | ads JP | generation | 3 | 2 | none | comp |
| .82 | eval/benchmark | LLM-as-judge | eval-fw | 4 | 3 | cost | comp |
| .83 | eval/benchmark | telecom Korean | eval-fw | 1 | 0 | multilingual, compliance, scale | comp |
| .84 | domain-FT | medical billing | classification | 1 | 2 | deploy, compliance | comp |
| .88 | structured-extr | media JSON | generation | 1 | 2 | cost, latency | comp |
| .89 | efficiency | continual NLP | generation | 8 | 3 | none | comp |
| .90 | code | text-to-SQL | generation | 2 | 4 | latency, compliance | comp |
| .92 | search/recsys | e-com CTR | ranking | 3 | 5 | scale, latency | comp |
| .93 | RAG | personal memory | retrieval | 1 | 3 | deploy | comp |
| .96 | e-commerce | product quality | classification | 1 | 2 | deploy, cost, scale | comp |
| .97 | data-gen | math education | generation | 1 | 1 | deploy, scale, prod-metrics | comp |
| .100 | agents | BPA/BPMN | agentic | 1 | 3 | none | comp |
| .101 | dialog/CS | medical | generation | 1 | 1 | none | comp |
| .102 | structured-extr | EU legal | classification | 1 | 13 | compliance | comp |
| .103 | domain-FT | manufacturing (DE) | retrieval | 2 | 10 | cost, multilingual, scale | comp |
| .105 | data-gen | document KIE | pipeline | 2 | 3 | deploy, cost | comp |
| .108 | dialog/CS | e-com/medical CS | pipeline | 3 | 2 | deploy, latency, multilingual | comp |
| .110 | speech | Chinese ASR | pipeline | 4 | 2 | none | comp |
| .111 | domain-FT | finance multilingual | classification | 1 | 2 | multilingual | comp |
| .113 | agents | labor market | pipeline | 3 | 3 | scale, multilingual | comp |
| .115 | agents | TikTok Pay AML | agentic | 1 | 8 | deploy, scale, compliance | comp |
| .116 | agents | WeChat | agentic | 2 | 4 | latency, scale | comp |
| .118 | efficiency | on-device | pipeline | 21 | 1 | latency, cost | comp |
| .119 | efficiency | LinkedIn rec | pipeline | 3 | 6 | deploy, latency, cost, scale | comp |
| .121 | dialog/CS | contact center | pipeline | 1 | 7 | cost, scale, latency | comp |
| .122 | structured-extr | NL2SQL | pipeline | 2 | 4 | cost, compliance | comp |
| .123 | agents | enterprise analytics | agentic | 2 | 2 | none | comp* |
| .124 | moderation/safety | clinical Korean | classification | 2 | 3 | latency, compliance, multilingual | comp |
| .125 | dialog/CS | telemedicine | agentic | 2 | 0 | deploy | case |
| .126 | dialog/CS | telemedicine RU | agentic | 1 | 0 | deploy, scale, latency, compliance | case |
| .127 | dialog/CS | contact center (Dialpad) | pipeline | 1 | 6 | deploy, cold-start, scale | comp |
| .130 | eval/benchmark | oil & gas | eval-fw | 1 | 3 | cost, compliance | comp |
| .132 | eval/benchmark | graph DBs | eval-fw | 12 | 10 | none | comp |
| .133 | dialog/CS | telemedicine RO | agentic | 1 | 0 | deploy, multilingual, scale, prod | case |
| .135 | RAG | Airbnb CS | pipeline | 1 | 2 | deploy, A/B, latency, scale | comp |
| .140 | summarization | Airbnb CS | pipeline | 3 | 3 | deploy, latency, scale | comp |
| .141 | search/recsys | e-com | ranking | 4 | 7 | cold-start, cost | comp |
| .142 | other:KG-to-text | general | generation | 5 | 2 | latency, cost | comp |
| .144 | agents | general | agentic | 5 | 8 | none | comp |
| .147 | structured-extr | e-com (Alibaba) | pipeline | 2 | 4 | deploy, scale | comp |
| .150 | dialog/CS | telemarketing | generation | 1 | 3 | deploy, A/B, scale | comp |
| .151 | moderation/safety | mental health | classification | 2 | 12 | none | comp |
| .152 | eval/benchmark | cyberbullying | eval-fw | 4 | 3 | none | comp |
| .157 | multimodal | automotive | other | 1 | 2 | none | comp |
| .164 | efficiency | edge SLM | other | 7 | 2 | cost, latency | comp |
| .167 | other:fact-checking | media | classification | 5 | 4 | none | comp |
| .168 | domain-FT | telecom | retrieval | 2 | 9 | deploy, scale, multilingual | comp |
| .173 | RAG | industrial PDFs | retrieval | 2 | 8 | latency, cost | comp |
| .174 | RAG | multi-hop QA | retrieval | 3 | 4 | latency, cost | comp |
| .180 | domain-FT | SNS moderation | pipeline | 15+ | 5 | deploy, A/B, scale, multilingual | comp |
| .181 | data-gen | EMR | generation | 1 | 3 | compliance, scale | comp |
| .183 | other:data-quality | tabular | pipeline | 10 | 8 | none | comp |
| .185 | multimodal | short-drama ads | pipeline | 1 | 2 | scale | comp |
| .187 | eval/benchmark | automotive | eval-fw | 1 | 3 | none | comp |
| .188 | eval/benchmark | chat-LLM services | eval-fw | 1 | 12 | none | comp |

*comp\* означает, что бейзлайны заявлены, но числа не приведены в summary; технически compativ-формат, но слабый.

---

## 10. Roadmap для собственной submission

На основе карты, последовательность принятия решений:

1. **Выбрать вертикаль с дефицитом покрытия** (cybersec, manufacturing, agriculture, low-resource MT, niche legal) или подкрепить distinguished hook'ом deployment'а.
2. **Сформулировать как pipeline**, не как single method. Минимум 3 компонента.
3. **Подобрать proprietary dataset** (≥1) + 1-2 публичных для контекста.
4. **Зафиксировать 4-6 бейзлайнов**, включая обязательно GPT-4o / Claude как top-line и Qwen-7B/Llama-8B как fair compute-matched.
5. **Запланировать ablations с самого начала** — не как послесловие.
6. **Зарезервировать слот в paper'е под cost/latency-числа** (даже одну таблицу).
7. **Если есть deployment** — выносить A/B-числа в abstract first sentence.
8. **Compliance/safety/multilingual** — добавлять как secondary hook, даже если основной вклад технический.
9. **Если делаете benchmark** — фокусируйтесь на узкой вертикали + human validation; широкие задачи уже забиты.
10. **Не пишите case-study** без громкого якоря.

---

*Сгенерировано на основе автоматических summary 101 принятой статьи. Все числовые агрегаты имеют погрешность ±2-3 работы из-за неоднородности машинных аннотаций.*
