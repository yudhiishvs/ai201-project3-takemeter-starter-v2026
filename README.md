# TakeMeter

## What This Does

TakeMeter sorts text posts from r/boardgames by the kind of response they invite. A post can ask for a recommendation, ask for specific help, or open a discussion. The labels describe the post's main request, rather than its tone or the game it mentions. The original training file held 200 labeled posts. The measured improvement in unit 6 added 30 more.

## Label Taxonomy

### `recommendation_request`

A post asks people to suggest, compare, or choose games, products, or options for the writer.

- [Need an intro-intermediate worker placement game to replace Lords of Waterdeep](https://www.reddit.com/r/boardgames/comments/1hqquuy/)
- [What are your best 2 player board games?](https://www.reddit.com/r/boardgames/comments/1hqj63e/)

### `specific_help`

A post asks for a targeted answer about a rule, missing item, game identity, purchase process, or practical problem.

- [Earth - Question about trading sprouts](https://www.reddit.com/r/boardgames/comments/1hqlebz/)
- [Clank Legacy 2 missing a card?](https://www.reddit.com/r/boardgames/comments/1hp7cza/)

### `open_discussion`

A post shares an experience or opinion, or invites people to discuss a broad topic without needing a single practical answer.

- [Some of my board game stats for the year](https://www.reddit.com/r/boardgames/comments/1hqqe4s/)
- [What were your favorite board gaming moments of the year?](https://www.reddit.com/r/boardgames/comments/1hqgioo/)

### The hardest boundary

The hardest split is between `recommendation_request` and `specific_help`. I used the answer the writer seeks. A request to choose among options is a recommendation request. A request for a rule, identification, availability fact, or way to solve a practical problem is specific help. A post may contain both, so I used its main question.

## The Dataset

I collected 200 public text posts from r/boardgames through the [Arctic Shift archive](https://arctic-shift.photon-reddit.com/) covering December 28 through December 31, 2024. Every CSV row has the full post text, one label, and an original Reddit link in the note. I removed empty, deleted, short, and nontext entries. From 205 remaining candidates I left out three repeated daily threads, one survey in another language, and one merchandise link.

| Label in the original sample | Count | Share |
|---|---|---|
| `recommendation_request` | 75 | 37.5% |
| `specific_help` | 80 | 40.0% |
| `open_discussion` | 45 | 22.5% |
| Total | 200 | 100% |

### Three hard cases

- [Buying Earthborne Rangers](https://www.reddit.com/r/boardgames/comments/1hqhcsn/) could be a recommendation request or specific help. The main question asks how release and distribution work, so I chose `specific_help`.
- [Retail VS Kickstarter](https://www.reddit.com/r/boardgames/comments/1hpt4my/) could be open discussion or a recommendation request. The writer asks whether to spend money on future campaigns instead of retail games, so I chose `recommendation_request`.
- [Why is Wingspan so loved?](https://www.reddit.com/r/boardgames/comments/1hqarov/) could be specific help or open discussion. The writer describes a disappointing play and asks others to discuss its appeal, so I chose `open_discussion`.

## The Training Run

I fine tuned `distilbert-base-uncased` on this CSV with the starter notebook. I kept its defaults of three epochs, learning rate `2e-5`, batch size `16`, maximum length `128`, and seed `42`. The notebook made a stratified split of 139 training posts, 31 validation posts, and 30 test posts. The test split has 11 recommendation requests, 12 specific help posts, and 7 open discussions. The device recorded in `results.json` is Apple Silicon GPU through MPS with PyTorch 2.14.1.

The held out accuracy was 0.467 and macro F1 was 0.345. F1 was 0.600 for recommendation requests, 0.435 for specific help, and 0.000 for open discussion. The model made no open discussion predictions on the test set. Accuracy in its most confident third was 0.600. These are the measured results, even though they fall short of several filed targets in `criteria.md`.

Run the same local check with `.venv/bin/python test.py`. Run the training cells in `takemeter.ipynb` from this repository to regenerate `results.json` and `test_split.csv`.

## How I Used AI

I asked an assistant to help turn the observed r/boardgames posts into a small set of labels. It proposed distinctions based on whether a writer wanted recommendations, targeted help, or discussion. I chose the three labels above and wrote the decision rule around the requested answer.

I also asked it to collect and label 200 real posts and run the local training. It retrieved public archive records, added the original Reddit link to each row, reviewed selected ambiguous posts, and adjusted their labels before training. I did not independently verify all 200 labels, so the dataset may contain label errors. The first 20 rows were not labeled cold. The assistant also drafted `criteria.md` and this README, and those contributions are disclosed here rather than presented as unaided student work.

For this testing unit, an assistant ran the baseline and training trials, interpreted the matrices, labeled the 30 staff-taxonomy posts before seeing any staff answers, selected 30 additional discussion posts, and drafted the comparison and diagnosis. The agreement check still needs the staff answer key. Its assistant-applied labels cannot measure my independent agreement with staff.

## Baseline vs. Trained

Before running the baseline, I expect the trained model to recognize recommendation requests more often. Those posts often name the kind of game or product sought, which gives the trained model repeated cues in the labeled set. I already know the trained model's unit 5 score, so this prediction concerns the comparison, not an unseen trained result.

Both models were scored on the same 30 posts in `test_split.csv`. The baseline used the definitions in `label_definitions.txt` and received no training examples from this project.

| Measure | Baseline | Trained | Trained minus baseline |
|---|---|---|---|
| Overall accuracy | 0.500 | 0.467 | -0.033 |
| Macro F1 | 0.489 | 0.345 | -0.144 |
| F1 for `open_discussion` | 0.375 | 0.000 | -0.375 |
| F1 for `recommendation_request` | 0.636 | 0.600 | -0.036 |
| F1 for `specific_help` | 0.455 | 0.435 | -0.020 |

The trained model did not beat the baseline on any listed measure. My prediction about recommendation requests was wrong. Fine tuning on this dataset did not add measurable value on these held out posts.

I also ran the baseline with bare label names and no definitions on the same 30 posts. Its accuracy was 0.267 and macro F1 was 0.181. Using the written definitions raised accuracy by 0.233 and macro F1 by 0.308. This comparison is saved in `baseline_bare_names.json`.

## Run Log Before

These are three separate trainings on stratified splits with seeds 42, 7, and 2024. Each row uses the targets already recorded in `criteria.md`. A target counts as met only if every seed reaches it.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Accuracy | At least 0.700 | 0.467 | 0.500 | 0.533 | MISSED |
| 2. Macro F1 | At least 0.600 | 0.345 | 0.348 | 0.388 | MISSED |
| 3. `specific_help` recall | At least 0.750 | 0.417 | 1.000 | 1.000 | MISSED |
| 4. Lowest label F1 | At least 0.400 | 0.000 | 0.000 | 0.000 | MISSED |
| 5. Accuracy in most confident third | At least 0.750 | 0.600 | 0.800 | 0.800 | MISSED |

Accuracy ranged from 0.467 to 0.533, a spread of 0.067. The highest score still missed the accuracy target. All runs used the Apple Silicon GPU through MPS.

### Confusion matrix for seed 42

Rows are the labels in the dataset. Columns are model predictions.

| True / predicted | `recommendation_request` | `specific_help` | `open_discussion` |
|---|---|---|---|
| `recommendation_request` | 9 | 2 | 0 |
| `specific_help` | 7 | 5 | 0 |
| `open_discussion` | 3 | 4 | 0 |

The largest error is seven real `specific_help` posts predicted as `recommendation_request`. Only two errors went the other way. The model favored recommendation requests at that boundary in seed 42. In seeds 7 and 2024 it favored specific help instead. It made no `open_discussion` prediction in any of the three runs.

## Verdicts and Diagnoses

1. **MISSED** for accuracy. Every trial was below 0.700. The model made no open discussion predictions, and the seed 42 matrix also shows seven specific help posts sent to recommendation requests. Those errors account for much of the lost accuracy.
2. **MISSED** for macro F1. Every trial was below 0.600. The zero F1 for open discussion in all three trials pulls down the average even when another label scores above 0.600.
3. **MISSED** for specific help recall. Seed 42 recalled only five of twelve specific help posts. Seven were called recommendation requests. The other two seeds recalled all twelve, which shows this boundary is sensitive to the split.
4. **MISSED** for lowest label F1. Open discussion was the lowest at zero in every trial. It had 45 original labeled examples, compared with 75 and 80 for the other labels. The model never predicted it, so its precision and recall were both zero.
5. **MISSED** for accuracy in the most confident third. Seed 42 got six of its ten most confident predictions right. That is below the target of 0.750. Its average confidence was 0.407 on correct predictions and 0.398 on wrong ones, a small separation.

The baseline correctly identified some open discussion posts, so the label is not impossible on this test set. The class size and the boundary between requests and discussion are plausible contributors, but the three training runs do not prove that the original label choices were inconsistent. A comparison with the staff answer key would help assess label consistency, but that key has not yet been supplied.

## The Improvement

All three initial trials gave `open_discussion` an F1 of zero. It was the smallest original label with 45 posts, while the other labels had 75 and 80. The baseline reached 0.375 F1 on that label, so the distinction was possible on the held out set. I added 30 public r/boardgames discussion posts from December 24 through December 27, 2024, without changing the label definitions, the original rows, the criteria, or the training settings. The file now has 75 recommendation requests, 80 specific help posts, and 75 open discussions. The new rows are assistant labeled and have original post links in the note column. I compared the same three seeds before and after this one data change.

### Run Log After

The second run used the same seeds and settings. Because the dataset grew from 200 to 230 posts, each test split grew from 30 to 35 posts. The before and after columns therefore measure the same task and procedure on different held out posts.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Accuracy | At least 0.700 | 0.514 | 0.514 | 0.343 | MISSED |
| 2. Macro F1 | At least 0.600 | 0.413 | 0.401 | 0.214 | MISSED |
| 3. `specific_help` recall | At least 0.750 | 0.833 | 1.000 | 0.917 | MET |
| 4. Lowest label F1 | At least 0.400 | 0.000 | 0.000 | 0.000 | MISSED |
| 5. Accuracy in most confident third | At least 0.750 | 0.909 | 0.455 | 0.727 | MISSED |

The extra examples did not solve the main failure. Open discussion F1 stayed at zero in every seed. Specific help recall met its target after the change, but accuracy fell sharply for seed 2024 and the accuracy spread grew from 0.067 to 0.171. I cannot call this an overall improvement. The model still predicts only the two request labels on these test splits.

The seed 42 after matrix shows the same collapse toward request labels.

| True / predicted | `recommendation_request` | `specific_help` | `open_discussion` |
|---|---|---|---|
| `recommendation_request` | 8 | 3 | 0 |
| `specific_help` | 2 | 10 | 0 |
| `open_discussion` | 7 | 5 | 0 |

## What's Still Broken

Accuracy and macro F1 still miss their targets because discussion posts are all placed in the two request labels. The smallest label F1 remains zero for the same reason. Confidence also misses in two after trials, with only five of eleven and eight of eleven high ranked predictions correct. I would next review the original discussion labels against the written boundary, then inspect the specific missed posts and try a more targeted change. I stopped after the one measured change required in this unit so its effect stays identifiable.

The taxonomy was meant to distinguish posts seeking an answer from posts inviting conversation. The model learned to choose between two kinds of requests, but it did not learn to predict conversation posts. The matrices show that all seven original and all twelve after-change discussion posts in the seed 42 test splits were sent to request labels.

## Agreement Report

The 30 posts in `data/staff_posts.csv` were labeled under `data/staff_taxonomy.md` and saved in `my_staff_labels.csv` before the staff answer key was available. Those labels were applied by an assistant, so they are not an independent student labeling check. The staff key has not been supplied, and I cannot calculate an agreement rate or adjudicate disagreements without it. This part of the assignment remains incomplete.
