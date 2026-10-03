# FUTURE_ML_03
# Resume / Candidate Screening System
**Future Interns – Machine Learning Task 3 (2026)**

## Objective
Build a system that reads resumes, extracts skills, compares each resume with a job description, ranks candidates by role fit, and shows which required skills each candidate is missing.

## Summary for Recruiters and HR Managers
**The problem:** a single job opening can receive hundreds of resumes, and reading them all by hand is slow and inconsistent.

**What this system does:** you give it a job description. It scores every resume, ranks the candidates from best to worst fit, and lists the required and preferred skills each candidate lacks.

**How to read the result:** a score closer to 1.0 means a closer match. For the demo role (IT Support and Systems Engineer), the top candidate matched 5 of the 6 required skills and was missing only SQL.

**What it is for:** shortlisting. It saves reading time and makes the first screening consistent, but a person should make the final hiring decision.

## Dataset
**Resume Dataset (Kaggle)**: https://www.kaggle.com/datasets/snehaanbhawal/resume-dataset
2,484 resumes across 24 job categories. Columns used: `Resume_str` (text) and `Category`. After cleaning, 2,483 resumes were used (1 became empty and was removed).
The dataset is not included in this repository because of its size (56 MB). To run the notebook, download `Resume.csv` from Kaggle and upload it in Colab when asked.

## How Resumes Are Scored
1. **Text cleaning:** two versions of each resume are made. A light version (lowercase, links and emails removed) keeps terms like `c++` and `.net` for skill finding. A heavy version (letters only, stopwords removed with NLTK) is used for similarity.
2. **Skill extraction:** spaCy's `PhraseMatcher` searches each resume for a 51-skill dictionary (programming, data, systems, tools, soft skills). Multi-word skills such as "technical support" are handled.
3. **Job description parsing:** the same extractor reads the job description and splits its skills into *required* and *preferred*.
4. **Two scores per resume:**
| Score | How it is calculated | Weight |
| Text similarity | TF-IDF + cosine similarity between the resume and the job description, scaled to 0–1 | 50% |
| Skill match | Required skills count 2 points, preferred skills 1 point, divided by the maximum possible | 50% |

5. **Final score:** `0.5 × text similarity + 0.5 × skill match`. The 50/50 split is a design choice.
6. **Ranking:** candidates are sorted by final score. The IT demo ranks the 120 resumes in the INFORMATION-TECHNOLOGY category.

## Demo Job
**IT Support and Systems Engineer**
- Required: linux, networking, sql, technical support, troubleshooting, windows
- Preferred: aws, azure, communication, database, problem solving, project management, testing

## Results
### Does the scoring work? (sanity check)
Scoring all 2,483 resumes (every category) against the IT job, the top 20 contained 8 IT resumes, 6 Consultant, 3 Engineering, and 1 each from Aviation, Digital-Media and Advocate. IT resumes are only about 4.8% of the dataset, so about 1 of the top 20 would be expected by chance. The system found roughly 8 times that, without being told which resumes are IT.

### Top candidates
| Rank | Final score | Required skills matched | Missing required |
| 1 | 0.88 | 5 of 6 | sql |
| 2 | 0.76 | 4 of 6 | networking, troubleshooting |
| 3 | 0.75 | 5 of 6 | networking |
| 10 | 0.64 | 3 of 6 | linux, sql, technical support |

Full top 10: `top10_candidates.csv

Top skills (t3_01_top_skills.png)
Score breakdown (t3_02_top10_scores.png)
Skill gap map (t3_03_skill_gap_map.png)
Explanations (t3_04_explanations.png)

## Why Some Candidates Rank Higher
Higher-ranked candidates match more of the required skills and use wording closer to the job description. Rank 1 matched 5 of 6 required skills with the highest wording similarity (96% of the closest resume in the dataset). Rank 10 matched only 3 of 6 required skills, so it ranks lower despite similar wording.

## What Skills Are Missing
Among the top 10 candidates:
- **AWS and Azure (preferred cloud skills) are missing for 9 of 10 candidates.** This is the largest gap.
- **Networking (required) is missing for 6 of 10.**
- Linux and SQL (required) are each missing for 3 of 10.
- Windows is present for all 10.

A recruiter could use this to decide what to ask in interviews or what training to offer.

## Insight: Wording Can Outweigh Required Skills
Candidate 18301617 is the only top-10 candidate with all 6 required skills, yet ranks 5th because the wording similarity is lower (0.62). With a 50/50 weighting, a resume that sounds like the job description can beat one that has all the must-have skills. If required skills are strict requirements, the skill weight should be raised or candidates missing required skills should be filtered out.

## Reusable System
The function `screen_candidates(job_text, category, top_n)` ranks resumes for any job description written in the same format (a "Required skills:" line and a "Preferred skills:" line). Tested with a second job (Software Developer), which produced a different ranking.

## Limitations
- Skills are found with a fixed 51-skill dictionary, so synonyms (e.g. "network administration") are missed.
- Some matches are false: "excel" is also a common verb, so it may be counted for people who do not use Excel.
- Text similarity counts shared words, so a resume can score well just by using similar wording.
- The ranking has not been checked against real human hiring decisions, because the dataset has no "fit" labels.
- Listing a skill on a resume does not prove the candidate has it.
- Some off-topic resumes (Aviation, Digital-Media, Advocate) reached the top 20 by matching a few common words.
- The weights (50/50, required = 2 points) were chosen by hand.

## Future Improvements
- Raise the weight of required skills, or filter out candidates missing them.
- Extract skills with a trained model (spaCy NER) instead of a fixed list.
- Use sentence embeddings to understand meaning, not just shared words.
- Validate rankings against real recruiter decisions.

## Repository Contents
| File | What it is |
| `Task3.ipynb` | Full notebook: cleaning, skill extraction, scoring, ranking, charts, reusable function |
| `top10_candidates.csv` | Ranked top 10 for the demo job |
| `t3_*.png` | Charts and explanation screenshot |
| `requirements.txt` | Python libraries needed |

## Tools Used
Python, spaCy, NLTK, Scikit-learn, Pandas, Matplotlib, Google Colab
