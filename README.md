# TakeMeter

## What This Does

TakeMeter sorts text posts from r/boardgames by the kind of response they invite. A post can ask for a recommendation, ask for specific help, or open a discussion. The labels describe the post's main request, rather than its tone or the game it mentions. This repository contains 200 labeled posts and one local DistilBERT training run.

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

The assistant drafted the labels at my request. It reviewed the title list and selected full posts at the boundaries, then corrected several labels. I did not label the first 20 without help. The note column marks every row as assistant labeled, so none is presented as a cold human label. This misses that part of the assignment.

| Label | Count | Share |
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
