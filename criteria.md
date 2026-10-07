# TakeMeter acceptance criteria

These targets were recorded before the first training run. They apply to the held out test set for each seed in the later three seed evaluation.

## 1. Overall accuracy

Accuracy reaches at least 0.70 for each seed.

The posts are drawn from one community, and many help requests announce themselves plainly. A result below 0.70 would leave too many routine posts mislabeled.

## 2. Macro F1

Macro F1 reaches at least 0.60 for each seed.

Help requests are common in this sample. Macro F1 prevents that large group from hiding weak performance on discussion prompts and experience reports.

## 3. Help request recall

Recall for `help_request` reaches at least 0.75 for each seed.

People asking for rules, recommendations, or game identification need a useful answer. Missing one in four such posts would make the sorting less useful.

## 4. Smallest label F1

Each label has an F1 of at least 0.40 for each seed.

The test split holds about 30 posts. A less common label may have fewer than ten test examples, so its score can move sharply when the split changes.

## 5. Confidence

Among test predictions with confidence of at least 0.70, accuracy reaches at least 0.75 for each seed, provided at least five predictions meet that threshold.

High confidence should mean more than a strong looking number. The minimum count keeps a single lucky prediction from satisfying the target.

## Authorship note

These criteria were drafted with an assistant at the repository owner's request. They were committed before training, but they were not written unaided by the student.
