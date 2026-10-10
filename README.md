# AI Clinical Scribe: Telehealth Consult to Structured Note

A small end-to-end pipeline that listens to a recorded telehealth visit and turns it into a structured clinical note a clinician can review.

It takes a women's health consultation audio file, transcribes it with OpenAI Whisper, and uses a LangChain prompt chain on GPT-3.5 to pull out the key clinical details, suggest differential diagnoses, and draft a follow-up plan.

> **Note:** The consultation used here is fully synthetic. The script was written with ChatGPT and converted to audio with an AI voice tool. No real patient data was used. This is a learning project and is not meant for clinical use.

---

## How it works

```
Audio file (.mp3)
      │
      ▼
1. Transcription     Whisper (base model) converts speech to text
      │
      ▼
2. Save transcript   Transcript written to transcript.txt
      │
      ▼
3. Summarization     LangChain prompt chain + GPT-3.5 Turbo
      │
      ▼
Structured clinical note
```

1. **Transcribe the visit.** Whisper's `base` model converts the 4 minute consult into text.
2. **Save the transcript.** The raw text is written to `transcript.txt` so it can be checked against the final note.
3. **Generate the note.** A prompt template built for women's reproductive, mental, and physical health guides GPT-3.5 to extract:
   - Date of consultation
   - Patient name and age
   - Chief complaint and history of present illness
   - Menstrual history
   - Mental and physical symptoms
   - Current medications
4. **Draft clinical next steps** for the clinician to review:
   - Three differential diagnoses with reasoning, and the most likely one
   - Relevant lab tests
   - Lifestyle recommendations
   - A checklist of data to collect at the follow-up visit

---

## Keeping the output grounded

LLMs can invent details that were never said. Since a clinical note has to be accurate, the prompt was designed to reduce that risk:

- **Extract only from the transcript.** The model is told to use only what was said in the conversation.
- **Say when something is missing.** If a detail wasn't mentioned, the model returns "Not provided" instead of guessing. In the sample run, it correctly flagged the missing consult date and medications.
- **Low temperature (0.1).** Keeps responses consistent and less creative.
- **Draft, not decision.** Diagnoses and recommendations are framed as suggestions for a clinician to confirm, not final answers.

---

## Sample output

```
Chief Complaint: Mood changes related to menstrual cycle
Menstrual History: Regular 28 to 30 day cycles, symptoms for the past three months
Physical Complaints: Bloating, breast tenderness, headaches, fatigue
Medications: Not provided

Differential Diagnosis:
1. Premenstrual Dysphoric Disorder (PMDD)
2. Premenstrual Syndrome (PMS)
3. Major Depressive Disorder

Probable Diagnosis: PMDD
Suggested Tests: Hormone levels, thyroid function, CBC, Vitamin D
```

---

## Tech stack

| Purpose | Tool |
|---|---|
| Speech to text | OpenAI Whisper (`base`) |
| LLM | GPT-3.5 Turbo |
| Prompt orchestration | LangChain (`PromptTemplate`, LCEL chain) |
| Audio processing | ffmpeg |
| Environment | Python, Google Colab |

---

## How to run it

1. Open the notebook in Google Colab.
2. Install the dependencies:
   ```bash
   pip install git+https://github.com/openai/whisper.git
   pip install langchain langchain-openai
   sudo apt install ffmpeg
   ```
3. Add your OpenAI API key as an environment variable. Never paste it directly into the notebook before sharing.
   ```python
   import os
   os.environ["OPENAI_API_KEY"] = "your-key-here"
   ```
4. Upload your own audio file and update the file name in the notebook.
5. Run all cells.

---

## What I learned

- Whisper handled clear, conversational speech well, but transcript quality sets the ceiling for everything downstream.
- Prompt design matters as much as the model. Telling the model exactly what to extract, and what to do when information is missing, made the output far more reliable.
- In healthcare, an AI output is only useful if a clinician can trust and verify it. Keeping the transcript alongside the note makes that check easy.

## Next steps

- Compare generated notes against a hand-written reference note to measure accuracy
- Link each line of the note back to the part of the transcript it came from
- Test with longer visits, multiple speakers, and background noise
- Output the note in a standard format such as SOAP
