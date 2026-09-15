# NLP and LLMs — Fall 2026

Course materials for **Natural Language Processing and Large Language Models**
(CS40008.01) at Fudan University. Everything students need is in this
repository or on the course website: <https://baojian.github.io/llm-26-fall/>.

<table>
<tr>
<td width="50%" valign="top">

<h3>Getting started</h3>
<ol>
<li>Install <a href="https://git-scm.com/downloads">Git</a> and <a href="https://docs.astral.sh/uv/getting-started/installation/">uv</a>.</li>
<li>Clone the course: <code>git clone https://github.com/baojian/llm-26-fall.git</code>, then <code>cd llm-26-fall</code>.</li>
<li>Start the local course server: <code>uv run python scripts/slides.py serve</code> and open <a href="http://127.0.0.1:8000">http://127.0.0.1:8000</a>. Python, Jupyter, and Reveal.js are prepared automatically.</li>
<li>Open slides and exercises from that page. Notebooks open in JupyterLab as personal copies under <code>workspace/</code>.</li>
<li>Before each class: stop the server (<kbd>Ctrl</kbd>+<kbd>C</kbd>), run <code>git pull</code>, start it again.</li>
</ol>
<p>Weekly activities (surveys, tasks) are submitted as pull requests from the GitHub website. See the <a href="docs/participation-workflow.md">participation workflow</a>.</p>

</td>
<td width="50%" valign="top">

<h3>Course information</h3>
<table>
<tr><td><b>Course code</b></td><td>CS40008.01</td></tr>
<tr><td><b>Semester</b></td><td>Fall 2026 (2026–2027, first semester)</td></tr>
<tr><td><b>Meets</b></td><td>Wednesdays, periods 6–8 (13:30–16:10), weeks 1–16</td></tr>
<tr><td><b>First / last class</b></td><td>September 9 / December 23, 2026</td></tr>
<tr><td><b>Make-up class</b></td><td>Saturday, October 10 (National Day falls on October 7)</td></tr>
<tr><td><b>Location</b></td><td>Handan Campus, HGX103</td></tr>
<tr><td><b>Language</b></td><td>Chinese lectures, English materials</td></tr>
<tr><td><b>Assessment</b></td><td>Quizzes 10%, assignments 45%, individual project 45%</td></tr>
<tr><td><b>Instructor</b></td><td>Baojian Zhou</td></tr>
</table>
<p>Dates, periods, and holidays: <a href="docs/schedule.md">docs/schedule.md</a>. Assessment details: <a href="https://baojian.github.io/llm-26-fall/">course website</a>.</p>

</td>
</tr>
</table>

## What is in this repository

| Folder | Contents |
| --- | --- |
| [`slides/`](slides/README.md) | Reveal.js lecture decks with companion notebooks. [Lecture 01](slides/lecture-01/index.html) (tokenization) and its [exercise notebook](slides/lecture-01/lecture-01-exercise.ipynb) are published; a [tokenization sample deck](slides/example/index.html) shows the format. |
| [`docs/`](docs/README.md) | Reading pages and reference notes (see the list below). |
| [`papers/`](papers/README.md) | PDFs and citations for the course readings. |
| [`surveys/`](surveys/lecture-01/README.md) | Weekly surveys: one response file per student, tallied into a chart. |
| [`tasks/`](tasks/README.md) | Small self-checking exercises: one submission file per student, checked automatically. The [progress board](tasks/PROGRESS.md) shows everyone's merged work. |
| [`workspace/`](workspace/README.md) | Your own notes and experiments; git-ignored, so `git pull` never conflicts with them. |
| `scripts/` | Course tooling: the local server, notebook launcher, survey tally, progress board, and the pretraining-dataset downloader. |

## Lecture 01: Tokenization

- [Slides](slides/lecture-01/index.html) and [exercise notebook](slides/lecture-01/lecture-01-exercise.ipynb); the course page links to both on the local server.
- [Preprocessing reader](docs/lecture-01-pre-tokenization.html) (bilingual): corpus preparation, tokenizer training, and encoding behavior, with inspected examples from Kimi K3, GLM-5.2, and DeepSeek-V4. Sources in [English](docs/lecture-01-pre-tokenization.md) and [Chinese](docs/lecture-01-pre-tokenization.zh.md).
- [Pretraining corpora catalog](docs/lecture-01-pretraining-datasets.md): the public datasets open LLMs are trained on and the samples the course mirrors with `scripts/download_pretraining_datasets.py`.
- [Tokenizer reading list](docs/tokenizer-reading-list.md), also as an [interactive page](https://baojian.github.io/llm-26-fall/docs/tokenizer-papers.html) filterable by question, topic, venue, and year.
- [Regular expressions in LLM training](docs/regex-in-llm-training.md): where regexes do real work in data pipelines and pre-tokenization.
- [Course pretraining plan](docs/pretraining-plan.md): the model the class trains this semester (1B parameters, about 35B tokens) and the proxy ladder used to test decisions first.

## Weekly participation

**Week 1 survey** ([issue #6](https://github.com/baojian/llm-26-fall/issues/6)): which LLM apps do you use most? Submit one file through a PR following the [survey guide](surveys/lecture-01/README.md). Results so far are in the guide and on the course page; 61 responses have been merged as of September 15, 2026.

**Tasks** start with Lecture 02. Each task folder contains the instructions, the checker, and a `submissions/` directory; run the checker locally, then open a PR with your single file. Merged work appears on the [progress board](tasks/PROGRESS.md).

Rules for both: use your lowercase GitHub username as the filename, change only your own file, title the PR as the guide says, and reference the week's issue with `Related to #N` (not `Fixes`). Anyone can propose improvements to course material through issues and PRs; changes are merged through review.

## Your workspace

Put notes, experiments, and exercise solutions in [`workspace/`](workspace/README.md). Everything there except its README is ignored by git, so your files stay out of pulls and pull requests. To modify a course file, copy it into `workspace/` and edit the copy: `uv run python workspace/<file>.py`.

## For contributors and the teaching team

Course content is prepared through pull requests. `slides/README.md` and `slides/AGENTS.md` define the deck layout and the required checks; `docs/README.md` explains how the bilingual reading pages are built; `CLAUDE.md` summarizes the repository conventions for coding agents. Solutions, rubrics, and grading scripts are kept in a private companion repository and never appear here.
