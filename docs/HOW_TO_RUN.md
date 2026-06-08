# Инструкция по запуску эксперимента

Здесь — пошаговая инструкция, как самостоятельно запустить ноутбук и воспроизвести результаты исследования.

---

## Вариант 1: Просто посмотреть результаты (без запуска)

**Самый простой способ.** Откройте на GitHub файл:

`notebooks/experiment_5f_6f_executed.ipynb`

GitHub автоматически отрисует ноутбук со всеми результатами: таблицами, графиками, выводами LLM. **Ничего запускать не нужно.**

---

## Вариант 2: Запустить эксперимент в Google Colab (рекомендую)

### Шаг 1. Получите API-ключ Groq (бесплатно, 2 минуты)

Groq предоставляет бесплатный доступ к Llama-3.3-70B через свой API.

1. Зайдите на [console.groq.com](https://console.groq.com)
2. Нажмите **Sign Up** (можно через Google или GitHub)
3. После регистрации → меню слева → **API Keys**
4. Нажмите **Create API Key**
5. Имя любое (например, `diploma`), нажмите **Submit**
6. **Скопируйте ключ** (строка вида `gsk_xxxxxxxxxxxx`) — он показывается только один раз!

### Шаг 2. Откройте ноутбук в Google Colab

1. Зайдите на [colab.research.google.com](https://colab.research.google.com)
2. Войдите через Google-аккаунт
3. **File → Upload notebook**
4. Загрузите файл `notebooks/experiment_5f_6f_combined.ipynb` из этого репозитория

### Шаг 3. Вставьте ключ Groq

Прокрутите ноутбук до **Блока 14** («Подключение к Groq API»). Найдите строку:

```python
GROQ_API_KEY = 'gsk_ВАШ_КЛЮЧ_GROQ'
```

Замените `gsk_ВАШ_КЛЮЧ_GROQ` на свой настоящий ключ.

### Шаг 4. Запустите все ячейки

В меню Colab: **Среда выполнения → Выполнить все** (или Ctrl+F9).

Полный прогон занимает:
- **Часть I (5Ф)** — 25-30 минут (загрузка моделей + индексация + поиск)
- **Часть II (6Ф)** — 30-50 минут (генерация 400 ответов через LLM)

**Итого: 1.5-2 часа** на CPU Colab.

### Шаг 5. Результаты

После завершения работы в файловой панели Colab появятся:

- `table1_metrics.csv` — IR-метрики
- `table2_times.csv` — время работы
- `table3_qa_metrics.csv` — QA-метрики
- `plot1_ndcg.png`, `plot2_recall_k.png`, `plot3_qa_summary.png` — графики
- `generated_answers.json` — все ответы LLM (для воспроизводимости)

---

## Возможные проблемы и решения

### Ошибка при загрузке HotpotQA

```
HfUriError: Invalid HF URI 'hf://datasets/hotpot_qa@...'
```

**Решение:** замените `'hotpot_qa'` на `'hotpotqa/hotpot_qa'` в блоке 4:

```python
hotpot = load_dataset('hotpotqa/hotpot_qa', 'distractor',
                      split='validation', trust_remote_code=True)
```

### Ошибка лимита Groq (429 Too Many Requests)

Groq имеет лимит ~30 запросов в минуту на бесплатном плане. Если упёрлись в лимит:

**Решение:** увеличьте паузу в блоке 16:

```python
time.sleep(0.5)  # было 0.1, увеличьте до 0.5 или 1.0
```

### Out of Memory в Colab

При загрузке Cross-Encoder может не хватить RAM.

**Решение:** перезапустите runtime (Среда выполнения → Перезапустить) и запускайте по блокам, а не «все сразу».

### RAGAS не работает

Это известная проблема совместимости версий. В Часть II используется собственная имплементация QA-метрик (EM, F1, BERTScore), которые не требуют RAGAS. RAGAS-метрики в работе не используются.

---

## Запуск локально (для опытных)

Если хотите запустить на своём компьютере:

```bash
# 1. Клонируйте репозиторий
git clone https://github.com/maria-chuvakina/diploma-rag.git
cd diploma-rag

# 2. Создайте виртуальное окружение
python -m venv venv
source venv/bin/activate  # на Windows: venv\Scripts\activate

# 3. Установите зависимости
pip install -r requirements.txt
pip install jupyter

# 4. Создайте файл .env с вашим ключом Groq
echo "GROQ_API_KEY=gsk_your_key_here" > .env

# 5. Запустите Jupyter
jupyter notebook notebooks/experiment_5f_6f_combined.ipynb
```

**Требования к системе:**
- Python 3.10+
- ~4 ГБ свободной RAM
- ~2 ГБ места на диске (для кэша моделей)
- Стабильное интернет-соединение (для загрузки моделей и API-вызовов)

---

## Технические характеристики эксперимента

| Параметр | Значение |
|---|---|
| Датасет | HotpotQA (dev split, distractor config) |
| Запросов | 100 |
| Документов в корпусе | 1000 |
| Модель эмбеддингов | `sentence-transformers/all-MiniLM-L6-v2` |
| Реранкер | `cross-encoder/ms-marco-MiniLM-L-6-v2` |
| Векторный индекс | FAISS IndexFlatIP |
| LLM-генератор | `llama-3.3-70b-versatile` (Groq) |
| k для контекста LLM | 5 |
| RRF параметр k_RRF | 60 |
| BM25 параметры | k₁=1.5, b=0.75 |
| Температура LLM | 0.1 |
| max_tokens LLM | 150 |

---

## Связь с автором

Если возникнут вопросы по запуску — открывайте Issue в репозитории.
