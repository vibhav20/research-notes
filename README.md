# research-notes

My running notes on papers I'm reading, annotated on iPad and written in MD.

## Structure
```
papers/<slug>/notes.md   - one folder per paper, notes.md inside
topics/                   - topic wise organisation of papers
templates/                - the note template
```

## Workflow
1. Read + annotate the PDF on iPad. PDFs themselves are
   NOT committed here. Just the notes.
2. Copy `templates/paper-note-template.md` into a new `papers/<slug>/notes.md`
   (slug = lastname-year-firstword, e.g. `vaswani-2017-attention`).
3. Notes written on laptop in VS Code.
4. Commit: `git add papers/<slug> && git commit -m "add notes: <slug>"`, then push.

## Papers
- [x] How to Read a Paper (papers/Keshav-2016-how_read/notes.md)
