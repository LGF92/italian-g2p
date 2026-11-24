# Italian Grapheme-to-Phoneme: How Far Can Rules Go?

A rule-based converter that turns Italian spelling into IPA, evaluated against about 89,000 Wiktionary pronunciations, with learned word endings and a linguistic error analysis.

## The question

Italian is often called a "phonetic" language: you say it the way you write it. Speech technology (text-to-speech and speech recognition) relies on that through grapheme-to-phoneme (G2P) conversion. This project asks how far hand-written linguistic rules really get, and what kind of errors remain when spelling stops being enough.

## Data

- **WikiPron**, Italian broad transcriptions (`ita_latn_broad.tsv`), scraped from Wiktionary: about 89,000 words with IPA.
- The file has no stress marks. Tie bars on affricates (t͡s, d͡ʒ) were removed before scoring, so only real pronunciation differences count.
- Wiktionary gives standard dictionary pronunciations, not regional or casual speech. All scores are against that standard.

## Method

1. **Rules.** An ordered list of regex rules (sci, gli, gn, ch, ci, gi, qu and so on), where the first matching rule wins. Lookaheads handle context, such as "c before e or i".
2. **Post-processing.** Glides (piede: j, uomo: w), the inherently long consonants ʃ ɲ ʎ ts dz between vowels (pesce: ʃ ʃ), Wiktionary's notation for doubled affricates (cc: t tʃ), and s-voicing between vowels and before voiced consonants.
3. **Whole-word fixes.** -oso keeps a voiceless s, stressed -ìa endings stay in hiatus (fiaschetteria), -ual/-uar keep a full u, -izz- becomes d dz, falling diphthongs (cercai), and -ese keeps a voiceless s.
4. **Learned endings.** For each ending of the rule output (2 to 6 phonemes), the most frequent Wiktionary version is learned from 80% of the words. It is kept only when seen at least 5 times and right at least 60% of the time. Everything is scored on the unseen 20%.

Two scores are reported: **exact match**, and **ignoring open/closed vowels** (e = ɛ, o = ɔ), which isolates the vowel-quality problem.

## Results

| Step | Exact match | Ignoring vowels |
| --- | --- | --- |
| First draft of rules | 39.4% | 51% |
| + glides j, w | 45.7% | 60.6% |
| + long consonants, affricate notation, s-voicing | 63.4% | 84.8% |
| + whole-word fixes | 65.4% | 87.2% |
| **Test set: rules only** | **65.0%** |  |
| **Test set: rules + learned endings** | **76.4%** |  |

The learned endings add over 11 points on words the system had never seen. Removing the s-voicing rule made things worse (63.8% vs 65.4%), so it stays, even though many words are exceptions.

## Error analysis

On the test set, 4,229 of 17,922 words are still wrong.

| Error type | Share |
| --- | --- |
| Vowel quality only (e/ɛ, o/ɔ) | 48.3% |
| Consonant or other swap | 45.4% |
| Different length (glide/hiatus, doubling, foreign words) | 6.4% |

The most common single-sound swaps (ours → Wiktionary) are e → ɛ (1,240), o → ɔ (846), j → i (642), z → s (362), s → z (318) and ts → dz (284).

What this shows:

- **Open vowels are under-predicted.** The system defaults to closed e and o, but Wiktionary has ɛ and ɔ in the stressed syllable.
- **s/z and ts/dz are lexical.** Errors go both ways in roughly equal numbers, so no spelling rule can fix them.
- **Hiatus depends on stress.** A stressed i (fuorvìo, fiaschetterìa) is a full vowel, not a glide.

In short, nearly every remaining error depends on **stress or word knowledge, which Italian spelling does not show.**

## Stress: a first look

The data has no stress marks, but in standard Italian ɛ and ɔ only occur in the stressed syllable. Using the 4,538 test words with exactly one open vowel as a proxy:

- 54.8% are stressed on the second-to-last syllable, 34.2% on the third-to-last, and 3.2% on the last.
- The textbook default ("stress the second-to-last syllable") is right only 56.7% of the time.
- About 5% fall four or more syllables from the end, which is impossible for main stress. These are likely compounds with a secondary stress, so the proxy is noisy.

## Limitations

- Scored against one standard source. Regional pronunciations (for example, a voiced s in -oso in parts of Tuscany and the north) count as errors.
- Exact match is strict: a single wrong sound makes the whole word wrong.
- Foreign words and names (Bush, Guernsey) follow no Italian rule.

## Next steps

- **Learn stress from stress-marked data.** Kaikki.org's Wiktionary extract keeps the stress marks. Predicting stress first, then allowing ɛ and ɔ only on the stressed vowel, targets about half of the remaining errors.
- An exception lexicon for s/z and ts/dz.
- Compare with a neural G2P model, and combine the two: a curated lexicon, a neural model for new words, and linguist-written rules for systematic fixes.
- Report phoneme error rate as well as exact match.

## Reproducing

Open `italian-g2p.ipynb` in Google Colab and run all cells. It runs on CPU only, needs no API keys, and downloads the WikiPron file directly from GitHub.

## License

Code under the MIT license. Pronunciation data comes from Wiktionary via WikiPron and is subject to Wiktionary's license (CC BY-SA).
