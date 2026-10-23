<div align="center">

# 🛡️ Does safety survive the language?

**The same unsafe request, asked in English, then in formal Persian, then in colloquial Persian. Does the model still say no?**

</div>

---

> [!NOTE]
> This is my own experiment. Safety training is done mostly in English, so I wanted to find out what happens to it in my first language. I do not write new harmful prompts. I reuse an existing public benchmark and only translate those items into Persian.

## The idea

I got curious about how well an LLM's safety holds up when someone tries to get around it in Persian, so I decided to design a safety check. Almost all of a model's safety training is in English. My question is simple: if I take a request the model refuses in English and ask the same thing in Persian, does it still refuse, or does the guardrail quietly fall away?

If the refusal holds in all three forms, good. If it drops in Persian, that is a real gap, and I can measure how big it is.

## How it works

```mermaid
flowchart LR
    A["prompts.csv<br/>English · formal FA · informal FA"] --> B["ask a local model<br/>(Ollama)"]
    B --> C["label each reply<br/>refusal or not"]
    C --> D["refusal rate<br/>per language"]
```

1. `prompts.csv` holds the test set: safe control questions the model should answer, plus benchmark items it should refuse, each written in three registers (English, formal Persian, colloquial Persian). See [REGISTERS.md](REGISTERS.md) for how each register is defined.
2. `experiment.ipynb` sends every prompt to a local open model, saves the replies, and labels each one as a refusal or not.
3. It charts the refusal rate for each register on the items that should be refused.

## What it found

Llama 3.1 8B, 12 unsafe items from StrongREJECT, one run:

| Register | Refused | Refusal rate |
| --- | --- | --- |
| English | 11 / 12 | 92% |
| Formal Persian | 3 / 12 | 25% |
| Informal Persian | 3 / 12 | 25% |

The model refuses almost everything in English, but a lot of that protection drops away in Persian. The safe control questions were all answered normally, so this is not the model just refusing or breaking at random. The exact Persian figures are approximate: automatically telling a Persian refusal from a normal answer is unreliable, so the firm result here is the large gap between English and Persian, not the precise Persian percentage.

![refusal rate by language](chart.png)

> [!WARNING]
> This is a probe, not a full benchmark. Small prompt set, one model, and refusals are labelled by an automated keyword pass, which is only approximate in Persian. Translation quality matters. Treat it as a first look, not a verdict.

## Can we raise the refusal rate? Translate to English first

If the model's safety is strong in English and weak in Persian, then instead of
trusting it to refuse a Persian request directly, I can translate the request to
English first, let it decide in English, and only then answer. `mitigation.py`
tests exactly that: translate each Persian prompt to English, re-ask, and see if
the model now refuses.

| Register | Asked in Persian | Translated to English first |
| --- | --- | --- |
| Formal Persian | 25% | 42% |
| Informal Persian | 25% | 58% |

![does translating to English first help](chart_mitigation.png)

Translating first roughly doubled the refusal rate, but it did not reach the 92%
the model manages on the original English. So the fix helps a lot, yet it does not
fully close the gap. That tells me the problem is deeper than just the input
language: translation quality and the model's own limits still let some requests
through. A real fix would need multilingual safety training, not just a translation
step in front.

## Run it yourself

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Install Ollama, pull a model (for example `ollama pull llama3.1:8b`), fill `prompts_work.csv` from a benchmark, then run `experiment.ipynb` top to bottom. Write down the exact model version and the date, because model behaviour changes over time.

## Credit and how AI helped

This project is mine. I asked the question, wrote all the Persian translations across
the formal and colloquial registers myself, decided which harm categories to test,
and chose the translate to English idea as the mitigation to try. I used Claude to
write the experiment code, run the local model, and help organize this write up. The
research direction, the Persian, and the judgment calls are my own.
