# Building a course from the user's material

Users can give you their own files (lecture notes, a manual, slides exported as PDF, a spreadsheet of data) and
ask for a course built on them. The user chooses one of two modes, and the choice binds you for the whole course.

## 1. Check the material

- **Rights**: ask the user to confirm the material is theirs or that they may use it (their own notes, their
  company's documents with permission, openly licensed works). Do not build a course out of someone else's
  copyrighted book, paid course or article: the content policy rejects plagiarism.
- **Personal data**: do not carry names, emails, phone numbers or other personal data of real people from the
  material into the course unless the user confirms it is meant to be public.
- **Formats**: PDF, DOCX, XLSX, CSV, TSV, TXT and Markdown are supported. Ask the user to re-save older or other
  formats (`.doc`, `.xls`, `.pptx`, `.odt`, Pages, Keynote) as PDF, DOCX or XLSX. A Google Doc can be downloaded
  as DOCX or PDF, a Google Sheet as XLSX or CSV.

## 2. Read it

Run:

```
npx --yes coursmith@latest extract <file> [<file>…] --out-dir coursmith-sources --json
```

It writes one Markdown file per input in `coursmith-sources/` (PDFs page by page as `## Page N`, workbooks sheet
by sheet as Markdown tables) and prints a summary with `warnings`. Then read those files with your normal file
tools, in parts if they are long. Extraction runs on the user's machine; nothing is uploaded.

- A warning that little text was found means a scanned PDF: ask for a text-based file, or read the pages as
  images if you can.
- Images inside documents are not extracted. If the course needs a figure from the material, ask the user for
  the image file and put it in `assets/`.
- If you can already read the file directly (for example a PDF), you may, but `extract` gives the same result
  on every agent and keeps tables intact.

Keep `coursmith-sources/` outside the course folder, or delete it when you are done: it is not part of the course.

## 3a. Mode "Keep my content" (faithful)

The user's words are the course. Your job is structure and presentation, not writing.

Allowed:
- splitting the text into sections and steps, ordering them, writing section and step **titles**;
- turning the user's own sentences into lists or tables when the material already has that structure;
- building **charts, diagrams and animations from data and processes in the material** (never invented numbers);
- fixing obvious typos and broken formatting left by the conversion (hyphenation, page headers, page numbers).

Not allowed:
- rewording, summarizing, shortening or "improving" the user's sentences;
- adding facts, examples or explanations that are not in the material;
- changing the language: if the course language differs from the material's, ask first, because a translation
  is a rewrite.

The course still needs text you must write: glossary definitions, quick checks, the final quiz and its
explanations, the metadata (title, description, objectives). Write them **only from what the material says**
(quote or closely paraphrase it), and show them to the user for approval before you submit. If a term needs a
definition the material does not give, ask the user rather than inventing one.

If the material has gaps the validator requires you to fill (a section with no glossary-worthy term, fewer than
10 quiz questions' worth of content), tell the user and ask how to proceed.

## 3b. Mode "Improve it" (enhanced)

The material is your main source, but you are the writer:
- rewrite for clarity and for the chosen difficulty, one idea per step;
- correct errors you are sure of, and complete gaps with accurate, well-established information;
- add examples, visuals, glossary entries and quiz questions;
- keep the user's facts, data, terminology and point of view unless they are wrong.

At the end, tell the user in a short list what you changed: corrections, additions, removed parts.

## 4. Review

The review agent checks a course built from material like any other. In "Keep my content" mode it may suggest
improvements to the user's text: they are optional. Show them to the user and let them decide (they can publish
as it is). Parts that break the content policy must still be changed, whatever the mode.
