# Sentiment analysis (NLP)

Репозиторий с решением задачи предсказания тона сообщений на английском языке.  

---

## 📌 Intro

Проект выполнен на основе датасета с Kaggle, содержащего интернет-сообщения с разметкой по эмоциональной окраске (positive / negative / neutral). Задача заключалась в том, чтобы по тексту сообщения определить его характер. В решении объединяются BERT модель, классические табличные регрессоры (CatBoost, XGBoost, LogisticRegression) и Naive Bayes, которые объединенны с помощью метамоделей (stacking). 

---

## 🔧 Data preprocessing

- Очистка HTML-тегов, лишних пробелов и не-буквенных символов.  
- Объединение двух текстовых полей (information, text) в единый столбец full_text.  
- Лемматизация и удаление стоп-слов с помощью spaCy для подготовки признаков к TF-IDF → TruncatedSVD.

---

## 🤖 Models

1. Bert + линейные слои  
   - Была использована модель cardiffnlp/twitter-roberta-base-sentiment-latest, после который применялся линейный слой с CrossEntropyLoss. Данная модель обучалась с помощью Trainer (HuggingFace Transformers) в течение 3 эпох с малой скоростью обучения (≈3e-5), чтобы избежать переобучения при ограниченном количестве данных и сохранить знания предобученного трансформера.

2.  Naive Bayes.
   - Модель наивного байеса ComplementNB в качестве входных данных использовались лемматизированные тексты, преобразованные с помощью TF-IDF.
   
3. Табличные модели (CatBoostClassifier, XGBClassifier, LogisticRegression).
   - Каждая из этих моделей использует обработанные признаки из исходных данных, полученные с помощью преобразования full_text → TF-IDF → TruncatedSVD. Для CatBoostClassifier и XGBClassifier гиперпараметры подбирались с помощью hyperopt. 

4.1 LogisticRegression в качестве метамодели  
   - Финальная модель LogisticRegression обучена на предсказаниях CatBoostClassifier, XGBClassifier, LogisticRegression, ComplementNB, Bertclf.
     
4.2 LogisticRegression в качестве метамодели (Стакинг без Bertclf)
   - Здесь для построения финальной модели использовался подход стэкинга с базовыми моделями CatBoostClassifier, XGBClassifier, ComplementNB и LogisticRegression, а в качестве метамодели использовалась LogisticRegression. Стакинг проводился по схеме out-of-fold, чтобы избежать утечки данных между базовыми моделями и метамоделью.
   
---

## ✅ Validation
- Данные делились на три части: train / validation / test.  
- Модель обучалась на train, качество оценивалось на validation.  
- Test использовался для сабмитов на Kaggle.
   
---

## 📈 Результаты

## 📊 Результаты моделей

| Модель                                   |  F1_macro  |
|------------------------------------------|:-------:|
| CatBoostClassifier                       | 0.9613 |
| XGBClassifier                            | 0.9701 |
| LogisticRegression                       | 0.6531 |
| Complement Naive Bayes                   | 0.8171 |
| BERT                                     | 0.9757 |
| **Meta model над всеми моделями**        | **0.9859** |
| Stacking (без BERT)                      |  0.9770 |
 

---
