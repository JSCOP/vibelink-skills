---
name: vibelink-eli5
description: >-
  Use when someone asks for a topic explained from zero — "explain like I'm 5",
  "ELI5", "explain this simply", "I don't understand any of this", "walk me
  through how X works", "쉽게 설명해줘", "초등학생도 알게", "그림으로 설명", or
  a bare /eli5 command. Also use when a correct technical answer has already
  been given and the person still says they do not follow it. Do not use for a
  precise API answer, a debugging session, or anything where the exact
  specification matters more than the intuition.
---

# ELI5

Explain a topic to someone who knows nothing about it, using a single self-contained
HTML page: big pictures, very few words.

Adapted from the MIT-licensed `eli5` plugin by Thariq Shihipar in
`anthropics/claude-plugins-community`.

## Language

This document is written in English so every agent reads one canonical text.
Reply to the user in the language they are writing in — Korean question, Korean
answer — and keep code, commands, paths, flags, and error strings verbatim in
their original form.

## The rule

**Pictures carry the explanation; words only label them.** If the page still
makes sense with every sentence deleted but the visuals kept, it is working. If
it only makes sense by reading, it is an article, not an explainer.

Build one standalone HTML file: inline CSS, inline SVG, no network requests, no
build step, no framework. It must open correctly from a `file://` path.

## Write it

1. **Find the one idea.** Everything else is detail. If you cannot say what the
   single idea is in one sentence, you are not ready to draw it.
2. **Draw the idea, do not describe it.** Boxes, arrows, containers, before and
   after, a sequence of frames. Inline SVG or styled `div`s — never an image the
   page has to download, and never an emoji standing in for a diagram.
3. **Cap the text.** A short title, a one-line caption per visual, and nothing
   else. No paragraphs. If a caption needs a comma-spliced clause to work, the
   drawing is not carrying its weight yet.
4. **Use an everyday analogy the reader already owns** — mail, water pipes,
   queues at a counter, keys and locks. Name the analogy where it stops being
   true, in one line, so nobody walks away with a wrong model.
5. **Order it as a story.** What goes in, what happens, what comes out. A reader
   should be able to follow it top to bottom without scrolling back.
6. **Then say where the real terms attach.** One closing line mapping the
   picture's labels back to the actual jargon, so the explainer becomes a bridge
   to real material instead of a dead end.

## Deliver it

Write the file, then say what you drew and how to open it. Inside a VibeLink
workspace, save it under the workspace folder and open it in a browser content
tab so the user sees it immediately rather than hunting for a path.

Keep the visual honest: correct beats charming. A cute diagram that implies the
wrong mental model is worse than a plain one that is right.

## Rationalizations

| Thought | Reality |
|---|---|
| "The topic is too technical to draw" | Every abstraction is somebody's diagram already. If you cannot draw it, you do not understand it yet. |
| "I'll write a short article instead" | Short prose is still prose. The deliverable is a page whose meaning survives deleting the sentences. |
| "One more paragraph adds useful nuance" | Nuance is what the reader asked to be spared. Put it in the closing line that names real terms. |
| "Emoji are basically pictures" | They are text. Draw the structure with SVG or layout. |
| "Markdown reply is faster than an HTML file" | The artifact is the deliverable. A chat reply is not one. |
| "Simplifying means I can be loose about the facts" | Simple and wrong is the one unacceptable outcome. Cut scope, never accuracy. |
